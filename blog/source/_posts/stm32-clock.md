---
layout  : post
title   : STM32(Cortex-M4) 时钟配置
data    : 2017/9/18
tags    : STM32
comment : false
reward  : false
brief   :

---
### 前言
```
        学习、总结，交流，进步！
```
测试环境为Keil 5.20 && (STM32F405ZET6 || STM32F407ZET6)。
外部晶振频率不同，就需要配置时钟以此获知各外设的时钟频率，比如常用的定时器。

原创，转载请注明出处。

---
### 时钟树介绍
介绍时钟配置之前，安利一个ST的针对于STM32提供的一个软件，STM32CubeMX，常用来进行硬件选型，外设资源分配以及可以进行配置生成基于HAL库的配置程序等功能。
当然了，大多数都应该知道这个软件，就不赘述了。其中还有一个功能是提供时钟树的配置，配置后可以直观的看到各个部分/外设的工作时钟，个人觉得挺好的，如下图所示：

<!-- more -->

![1_sysClock](/picture/img/stm32-clock/1_sysClock.png)

这里简单介绍上图。
红色外框：从这里选中HSE(外部时钟)，并配置外部时钟的频率。(我这里外部晶振使用的是8MHz，内部时钟部分自行参考)
黑色外框：M N P参数，常说的倍频部分。
黄色外框：通过设置倍频系数，得到的系统最大频率。(系统最大频率，请参考计数文档)
绿色外框：AHB 分频得到HCLK频率。
蓝色外框：下面两个框分频后得到APB1和APB2的时钟。
最后面是到各个外设的工作频率。

---
### 时钟配置
时钟配置其实最主要的就是设置倍频和分频的系数，很简单。这里就这几个系数在什么位置说明一下，请打开system_stm32f4xx.c文件。
这里使用的是F4的库，STM32F4xx_DSP_StdPeriph_Lib_V1.8.0，芯片f405/f407，所以在Keil中定义了STM32F40_41xxx(根据此宏确定该系数的唯一位置)，下面程序是截取system_stm32f4xx.c文件，需要修改的位置在程序中有说明，请仔细查看这一部分。
首先时倍频的系数设置：
```
    /************************* PLL Parameters *************************************/
    #if defined(STM32F40_41xxx) || defined(STM32F427_437xx) || defined(STM32F429_439xx) || defined(STM32F401xx) || defined(STM32F469_479xx)
     /* PLL_VCO = (HSE_VALUE or HSI_VALUE / PLL_M) * PLL_N */
     #define PLL_M      8//25  /******** 修改PLL_M系数，源文件中默认为25，因为ST默认使用25MHz晶振。修改为8分频是根据前面时钟树的配置进行修改 ********/
    #elif defined(STM32F412xG) || defined (STM32F446xx)
     #define PLL_M      8
    #elif defined (STM32F410xx) || defined (STM32F411xE)
     #if defined(USE_HSE_BYPASS)
      #define PLL_M      8
     #else /* !USE_HSE_BYPASS */
      #define PLL_M      16
     #endif /* USE_HSE_BYPASS */
    #else
    #endif /* STM32F40_41xxx || STM32F427_437xx || STM32F429_439xx || STM32F401xx || STM32F469_479xx */

    /* USB OTG FS, SDIO and RNG Clock =  PLL_VCO / PLLQ */
    #define PLL_Q      7

    #if defined(STM32F446xx)
    /* PLL division factor for I2S, SAI, SYSTEM and SPDIF: Clock =  PLL_VCO / PLLR */
    #define PLL_R      7
    #elif defined(STM32F412xG)
    #define PLL_R      2
    #else
    #endif /* STM32F446xx */

    #if defined(STM32F427_437xx) || defined(STM32F429_439xx) || defined(STM32F446xx) || defined(STM32F469_479xx)
    #define PLL_N      360
    /* SYSCLK = PLL_VCO / PLL_P */
    #define PLL_P      2
    #endif /* STM32F427_437x || STM32F429_439xx || STM32F446xx || STM32F469_479xx */

    #if defined (STM32F40_41xxx)
    #define PLL_N      336      /******** 修改PLL_N，根据前面时钟树的配置进行修改 ********/
    /* SYSCLK = PLL_VCO / PLL_P */
    #define PLL_P      2        /******** 修改PLL_P，根据前面时钟树的配置进行修改 ********/
    #endif /* STM32F40_41xxx */

    #if defined(STM32F401xx)
    #define PLL_N      336
    /* SYSCLK = PLL_VCO / PLL_P */
    #define PLL_P      4
    #endif /* STM32F401xx */

    #if defined(STM32F410xx) || defined(STM32F411xE) || defined(STM32F412xG)
    #define PLL_N      400
    /* SYSCLK = PLL_VCO / PLL_P */
    #define PLL_P      4
    #endif /* STM32F410xx || STM32F411xE */

    /******************************************************************************/
```

下面程序截取自system_stm32f4xx.c文件中设置时钟的函数，时钟初始化设置。
分频部分的系数设置：
```
    static void SetSysClock(void)
    {
    #if defined(STM32F40_41xxx) || defined(STM32F427_437xx) || defined(STM32F429_439xx) || defined(STM32F401xx) || defined(STM32F412xG) || defined(STM32F446xx)|| defined(STM32F469_479xx)
    /******************************************************************************/
    /*            PLL (clocked by HSE) used as System clock source                */
    /******************************************************************************/
      __IO uint32_t StartUpCounter = 0, HSEStatus = 0;

      /* Enable HSE */
      RCC->CR |= ((uint32_t)RCC_CR_HSEON);

      /* Wait till HSE is ready and if Time out is reached exit */
      do
      {
        HSEStatus = RCC->CR & RCC_CR_HSERDY;
        StartUpCounter++;
      } while((HSEStatus == 0) && (StartUpCounter != HSE_STARTUP_TIMEOUT));

      if ((RCC->CR & RCC_CR_HSERDY) != RESET)
      {
        HSEStatus = (uint32_t)0x01;
      }
      else
      {
        HSEStatus = (uint32_t)0x00;
      }

      if (HSEStatus == (uint32_t)0x01)
      {
        /* Select regulator voltage output Scale 1 mode */
        RCC->APB1ENR |= RCC_APB1ENR_PWREN;
        PWR->CR |= PWR_CR_VOS;

        /* HCLK = SYSCLK / 1*/
        RCC->CFGR |= RCC_CFGR_HPRE_DIV1;        /******** AHB 分频，通过对sysclk分频后得到HCLK时钟频率，此处1分频 ********/

    #if defined(STM32F40_41xxx) || defined(STM32F427_437xx) || defined(STM32F429_439xx) ||  defined(STM32F412xG) || defined(STM32F446xx) || defined(STM32F469_479xx)
        /* PCLK2 = HCLK / 2*/
        RCC->CFGR |= RCC_CFGR_PPRE2_DIV8;//RCC_CFGR_PPRE2_DIV2;     /******** APB2 分频，通过对前面HCLK分频后得到，PCLK2时钟频率，默认2分频，根据前面时钟树配置为8分频 ********/

        /* PCLK1 = HCLK / 4*/
        RCC->CFGR |= RCC_CFGR_PPRE1_DIV4;//RCC_CFGR_PPRE1_DIV4;     /******** APB1 分频，通过对前面HCLK分频后得到，PCLK1时钟频率，默认4分频，根据前面时钟树配置为4分频不变 ********/
    #endif /* STM32F40_41xxx || STM32F427_437x || STM32F429_439xx  || STM32F412xG || STM32F446xx || STM32F469_479xx */
```

其实通过这样简单几步设置，时钟配置就完成了，各个部分的时钟如前面图片所示。接下来我们通过库函数来获取HCLK，PCLK1和PCLK2的时钟频率。
在函数库文件stm32f4xx_rcc.c中提供函数RCC_GetClocksFreq来获取SYSCLK，HCLK，PCLK1和PCLK2的时钟频率：
```
    RCC_ClocksTypeDef rcc_clock;   /* 结构体，成员就是4个时钟频率 */

    RCC_GetClocksFreq(&rcc_clock);
```

通过串口将结构体成员的值打印出来，就可以很直观的知道各个外设的时钟频率了。
其实在函数库文件stm32f4xx_rcc.c中还有设置AHB，APB1和APB2分频系数的函数，这里不做介绍，具体请参考此文件。

### 结束语
时钟配置的过程很简单，我觉得更重要的是熟悉STM32的时钟结构，各个外设使用的是哪个部分的时钟，这样才能在使用的时候少犯错，尤其是定时器，时钟错了，定时就错了，尴尬。
F1的系统时钟初始化这部分，跟F4差别挺大的，可能不适用，大致参考一下。


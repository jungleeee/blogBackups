---
layout  : post
title   : STM32(Cortex-M4) 定时器-DMA WS2812-GRB-LED
data    : 2017/9/15
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
采用来控制WS2812-GRB全彩LED，主要是因为其控制时序要求较高，频率快。如果使用中断的方式，处理时序容易出错。

原创，转载请注明出处。

---
### 关于WS28212说明
WS2812实际可以认为有红、蓝、绿三个LED组成，通过控制其中的灯是否点亮以及点亮时的光照强度的不同组合，实现更多色彩的显示(三原色原理)。
因此控制WS2812色彩显示的时候需要控制“三个LED”，总共包含24bit的控制数据，红、蓝、绿各8bit；由于8bit数据表示范围为0-255，所以光照强度可以分成256各等级。
24bit数据格式如下图所示：

<!-- more -->

![ws2812_data_format](/picture/img/stm32-timer-dma/1_ws2812_data.png)


前面提到0-255表示该原色的亮度，因此就需要改变8bit数据每一位的0/1状态，其0码和1码的格式如下图所示：

![ws2812_0_1_code](/picture/img/stm32-timer-dma/2_ws2812_01code.png)

从图中可以看到0码/1码是在固定周期中高低电平时间不同的组合。整个周期时长1.25us左右，允许误差600ns。虽然STM32指令周期很短了，但是使用中断方式的话，明显不是很好。

---
### Timer-DMA
请注意以下几点：
1、关于STM32定时器色的DMA功能，需要注意的时TIM9-TMI14(100pin封装)是没有DMA功能的，所以在做硬件设计时需要注意。
2、请注意定时器所挂载的时钟频率，保证定时器配置后输出频率正确：TIM2、3、4、5、6、7、12、13、14使用APB1时钟；TIM1、8、9使用APB2时钟。
3、关于如何配置定时器输出PWM波，此处不介绍。

接下来开始程序搭建。我选择了TIM5-Channel2输出控制时序，TIM5的时钟挂载在APB1上面，所以打开該时钟，以及不要要忘记打开IO的映射功能。
```
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM5, ENABLE);    /* open tim5 clock */
    /* 其它此处不列出 */
    GPIO_PinAFConfig(GOIOA,GPIO_PinSource1,GPIO_AF_TIM5);   /* GPIOA-pin1 AF tim5-channel2 */
```

打开定时器时钟后，需要配置PWM的输出频率。上面介绍到WS2812每一位数据的周期是1.25us，也就是说需要设置PWM输出周期为1.25us。
在我的STM32时钟配置中，APB1的时钟输出为84MHz，所以有以下配置：
```
    TIM_TimeBaseInitStructure.TIM_Prescaler = 0;            /* 1/84 = 0.0119047619047619us */
    TIM_TimeBaseInitStructure.TIM_Period = 105;             /* 0.0119047619047619 * 105 = 1.254us //71 34 */
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
```

这里设置了定时器不分频，及其计数频率为(84 / (0+1))MHz，一次计数时间为0.01190476...us，所以继续配置计数重装载值为105，实现1.254us的PWM输出周期。
那么可以继续得到，71和34两个数值分别表示高低电平的时长。PWM输出方式配置为小于捕获/比较寄存器的值时，输出高电平：
```
    TIM_OCInitStructure.TIM_Pulse = 0;
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;   /* 小于TIM_Pluse的值时输出高电平 */
```

输出连续的0/1的状态，也就是在定时器当前周期计数溢出开始下一个周期计数时，修改比较寄存器的值，改变输出高电平的时长，达到输出0/1的目的。
也就是说需要在定时器计数溢出更新时触发DMA事件，通过DMA将新的比较寄存器的值写入到寄存器中。DMA配置如下：
```
    DMA_InitStructure.DMA_Channel = DMA_Channel_6;
    DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&TIM5->CCR2;	        /* Peripheral address */
    DMA_InitStructure.DMA_Memory0BaseAddr = NULL;                               /* Memory address */
    DMA_InitStructure.DMA_DIR = DMA_DIR_MemoryToPeripheral;				        /* DMA dir Memory To Peripheral */
    DMA_InitStructure.DMA_BufferSize = 1;	                                    /* length */
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;		    /* Peripheral address not add */
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;			            /* Memory address add */
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Word;     /* Peripheral data length */
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Word;             /* Memory data length */
    DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;				                /* normal mode */
    DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;                       /* DMA Priority */
    DMA_Init(DMA1_Stream6, &DMA_InitStructure);
```

DMA数据长度初始化设置没有关系，在数据发送时设置即可。这里设置了外设数据字长为一个字(32bit)，需要注意一下。定时器2和定时器5的捕获/比较寄存器是32bit的，而其他是16bit的，如下图所示，更多请参考技术文档，希望大家注意。

![5_timer](/picture/img/stm32-timer-dma/5_timer.png)

![6_timer](/picture/img/stm32-timer-dma/6_timer.png)

关于DMA数据流和通道选择，请参考下图，更多信息请参考STM32技术文档。因为需要触发DMA的是溢出更新事件，所以选择的是TIM5_UP。

![3_timer_dma](/picture/img/stm32-timer-dma/3_timer_dma.png)

接下来选择DMA的触发源，打开定时器和DMA功能：
```
    TIM_SelectCCDMA(TIM5, ENABLE);                                          /* 配置为发生更新事件时产生DMA请求，详情请参考TIMx-CR2 寄存器 */
    TIM_DMACmd(TIM5, TIM_DMA_Update, ENABLE);                               /* 使能TIMx的更新DMA请求，详情请参考TIMx-DIER 寄存器 */
    TIM_UpdateRequestConfig(TIM5, TIM_UpdateSource_Regular);                /* 使能URS位，只在定时器上溢/下溢时产生中断/DMA请求，详情请参考TIMx-CR1 寄存器 */
    TIM_UpdateDisableConfig(TIM5, DISABLE);                                 /* 详情请参考TIMx-CR1 寄存器 */

    DMA_Cmd(DMA1_Stream6, ENABLE);
    DMA_ITConfig(DMA1_Stream6, DMA_IT_TC, DISABLE);
    DMA_ClearITPendingBit(DMA1_Stream6, DMA_IT_TCIF6);

    TIM_Cmd(TIM5, ENABLE);
    TIM_ITConfig(TIM5, TIM_IT_Update, DISABLE);
    TIM_ClearFlag(TIM5, TIM_FLAG_Update);
```

定时器的配置大致这样，有其他问题请参考其他。

---
### ws2812 控制
这里就简单说明以下。通过前面分析，知道了通过写入捕获比较寄存器的值(34/71)，实现0/1的位输出，所以定义：
```
                            |g7|g6|g5|g4|g3|g2|g1|g0|r7|r6|r5|r4|r3|r2|r1|r0|b7|b6|b5|b4|b3|b2|b1|b0|
    unsigned char green[] = {71,71,71,71,71,71,71,71,34,34,34,34,34,34,34,34,34,34,34,34,34,34,34,34,0}; /* 11111111 00000000 00000000 */
    unsigned char red[]   = {34,34,34,34,34,34,34,34,71,71,71,71,71,71,71,71,34,34,34,34,34,34,34,34,0};
    unsigned char blue[]  = {34,34,34,34,34,34,34,34,34,34,34,34,34,34,34,34,71,71,71,71,71,71,71,71,0};
    unsigned char white[] = {71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,71,0};
    unsigned char reset[50] = {0};  /* 复位需要长达50us的低电平时间 */
```

在实际控制中，只需要配置好需要的颜色，将数组的起始地址设置为DMA的内存地址后启动DMA即可。如果上一次操作完成后关闭了定时器，请记得打开。
有一点需要注意以下，上面每个数组实际上都定义了25个成员，最后一个都为0。
是因为DMA传输完成后，捕获/比较寄存器的值是最后一次传输的值，34/71，如果没有关闭定时器，那么定时器就会一直输出0码/1码(对于WS2812)，也就是WS2812的状态会变成全部熄灭(全0)/全亮(全1)，导致出错，所以DMA最后一次发送0到捕获/比较寄存器中，使得PWM输出一直为低。
WS2812技术文档有说到输出低电平50us会复位，但在实际使用中发现，前一刻的状态并不会被改变，也就是说全输出为低电平时能够保持之前状态，经测试可行。
上面提到了两种方式：1、传输完成后，在DMA传输完成中断函数中关闭定时器，下一次传输前打开定时器。！！！！！此方式未测试！！！！！！
                    2、DMA传输最后一次数据到捕获/比较寄存器的值设置为0，通过输出低电平保持前一刻的状态。实测可行。


参考程序：https://github.com/jungleeee/Cortex-M/tree/master/Cortex-M4/timer-dma

### 结束语
工作当中有这个LED的功能需求，在开发过程中还是遇到了一些问题，认认真真的看了TIMx-DMA着一块，还是有一些需要注意的东西。
比如一开始选择DMA通道的时候，因为PWM输出时TIM5的通道2，就选择了DMA通道TIM5_CH2，实际应该是TIM5_UP，导致出错。
还有一点是DMA发送完成后，传输完成位，会置位，如果配置了传输完成中断，可以在中断中清除此位，没有配置中断的话，必须在下一次发送前清除此位，否则下一次传输会失败。
![4_dma](/picture/img/stm32-timer-dma/4_dma.png)
记录，分享！

---
layout  : post
title   : Lwip 2.0.3 移植(FreeRTOS + CLI)
data    : 2018/5/24
tags    :
        - STM32
        - FreeRTOS
        - CLI
        - LWIP
comment : false
reward  : false
brief   :

---
### 前言
```
        学习、总计、交流、进步！
```

测试环境为Keil 5.20 && STM32F407ZET6 && FreeRtos 10.0.0 && Lwip 2.0.3 && DP83848；晶振频率为8Mhz。
请根据实际情况修改后进行测试。

原创，转载请注明出处。

---
### 介绍
主要介绍Lwip 2.0.3 的移植过程。

作者对Lwip的认识一点也不深刻，只能简单的记录移植过程中主要的修改位置。方便其他需要使用此协议栈的小伙伴。

在移植开始之前需要下载Lwip最新源码包，以及STM32官方提供的基于STM32F407 && FreeRTOS && lwip 1.4.1 && DP83848 的例程源码。
获取方式如下：
1、Lwip 直接百度，下载需要的版本。
2、百度STSW—STM32070，下载此源码包。

<!-- more -->

---
### FIRST
将上述两个源码包解压。
Lwip目录下主要包括`doc、src、test`三个文件夹，以及一些其他文件（不重要）。所有源文件都在`src`文件夹下。
可以直接将lwip整个文件夹直接放到工程目录下。

然后是ST官方的移植例程。
解压后在目录:`STM32F4x7_ETH_LwIP_V1.1.1\Libraries`下，请把`STM32F4x7_ETH_Driver`文件夹移到工程相关目录下。
然后在`STM32F4x7_ETH_Driver\inc`目录下存在`stm32f4x7_eth.h`和`stm32f4x7_eth_conf_template.h`两个文件，打开`stm32f4x7_eth.h`可以看到有包含头文件`#include "stm32f4x7_eth_conf.h"`，此处我选择直接将另一个.h文件直接重命名为`stm32f4x7_eth_conf.h`。
此时目录下存在`stm32f4x7_eth.h`和`stm32f4x7_eth_conf.h`两个文件。
对`stm32f4x7_eth_conf.h`文件进行如下修改:

```
    line 60: #define CUSTOM_DRIVER_BUFFERS_CONFIG   //取消注释，允许使用自定义的网络接收/发送的数据缓冲区大小。

    //此处使用系统较为精确的延时，函数原型如下，请在适当位置进行定义：
    //void Delay(uint32_t ms)
    //{
    //  vTaskDelay(ms);//由于使用的FreeRTOS系统，使用系统延时。
    //}
    line 51: extern void Delay(uint32_t ms);/* Header file where the Delay function prototype is exported */

    //在允许使用自定义收发缓存大小后，请根据实际网络收发数据量的大小定义此处大小。
    line 66: #define ETH_RXBUFNB        4                    /* 4 Rx buffers of size ETH_RX_BUF_SIZE */
    line 67: #define ETH_TXBUFNB        4                    /* 4 Tx buffers of size ETH_TX_BUF_SIZE */

```

---
### SECOND
在工程目录下的lwip文件夹下创建`arch`文件夹，用来存放其他文件。
ST官方例程目录`STM32F4x7_ETH_LwIP_V1.1.1\Utilities\Third_Party\lwip-1.4.1\port`下，是针对于f4x7系列写的相关程序（具体作用还未详细了解）。
接下来把部分源文件复制到新建的`arch`文件夹中，因为使用了FreeRTOS系统，所以需要注意选择`FreeRTOS`目录下的文件：
1、把`STM32F4x7_ETH_LwIP_V1.1.1\Utilities\Third_Party\lwip-1.4.1\port\STM32F4x7\arch`目录下`cc.h cpu.h perf.h`三个文件。
2、把`STM32F4x7_ETH_LwIP_V1.1.1\Utilities\Third_Party\lwip-1.4.1\port\STM32F4x7\FreeRTOS`目录下全部四个文件。
3、把`STM32F4x7_ETH_LwIP_V1.1.1\Project\FreeRTOS\httpserver_socketi\inc`目录下`lwipopts.h`一个文件。
总共8个文件复制到新建`arch`目录下。

还需要将目录`STM32F4x7_ETH_LwIP_V1.1.1\Project\FreeRTOS\httpserver_socket\src`下`stm32f4x7_eth_bsp.c`
以及目录`STM32F4x7_ETH_LwIP_V1.1.1\Project\FreeRTOS\httpserver_socket\inc`下`stm32f4x7_eth_bsp.h`
复制到工程相关目录下。作如下修改：

```
    文件不要随意改变，不然行数对不上。
    1、`stm32f4x7_eth_bsp.h`:
        line 38-48 注释掉。
        line 74-78 注释掉。
        添加宏定义：#define RMII_MODE //use MII，因为我这里硬件使用的时MII接口，根据硬件接口情况定义使用MII或者RMII。
    2、`stm32f4x7_eth_bsp.c`:
        line 32 注释掉：#include "main.h"
        lin3 34 #include "netconf.h"修改为#include "ethernetif.h"
        line 81-101 注释掉。这里定义和创建网线链接检测的认为，我硬件上无此设计，所以注释掉，不使用。
        line 132-133 源文件是注释掉的，取消注释。
        line 294-556 全部注释掉。这里配置网线链接的硬件IO。
```

---
### THIRD
按下图将文件相应文件添加到工程中(这里需要注意：源文件`ethernetif.c`、`sys_arch.c` 在前面创建的文件夹`arch`下)：
![1_lwip](/picture/img/lwip/1_lwip.png)

此时如果进行编译的话，会出现较多警告和错误，下面进行一一处理（情况不一定完全一样，这里解决我出现的现象）：

```
    1、 error:  #256: invalid redeclaration of type name "u32_t" (declared at line 41 of "..\..\_Code\LWIP\arch/cc.h")
    这里在`arch.h`和`cc.h`两个文件中出现了重复定义。
    请将`arch.h` - `line 113` - `#define LWIP_NO_STDINT_H 0` - 定义为1：`#define LWIP_NO_STDINT_H 1`
    2、 error directive: "LWIP_COMPAT_MUTEX cannot prevent priority inversion. It is recommended to implement priority-aware mutexes. (Define LWIP_COMPAT_MUTEX_ALLOWED to disable this error.)"
    请在`lwipopts.h`文件中添加：`#define LWIP_COMPAT_MUTEX_ALLOWED       0`
    3、 error:  #5: cannot open source input file "lwip/timers.h": No such file or directory
    请注释掉`ethernetif.c`中`#include "lwip/timers.h"`，同时注释掉`#include "main.h"`，这是例程中有相关的定义，这里重新定义，所以注释掉。
    注释掉后，就会出现信号量声明未定义的错误，需要添加相应头文件：
    `#include "FreeRTOS.h"
     #include "task.h"
     #include "semphr.h"`
    然后会出现MAC地址未定义的错误，这里直接将宏定义修改为数字就可以解决了。
    在MAX地址初始化下面位置，设置`netif->flags = NETIF_FLAG_BROADCAST | NETIF_FLAG_ETHARP;`的位置请添加`NETIF_FLAG_LINK_UP`，为`netif->flags = NETIF_FLAG_BROADCAST | NETIF_FLAG_ETHARP | NETIF_FLAG_LINK_UP;`，否则网络无法连接。
    4、 warning:  #223-D: function "sys_timeout" declared implicitly
    直接注释，不使用。
    5、 warning:  #177-D: function "arp_timer" was declared but never referenced
    直接注释，不使用。

```

---
### FOURTH
理论上应该没有什么编译警告和错误了。
然后因为ST官方例程中使用的网络的DMA进行收发控制，所以还需构造网络中断函数，实际上在例程的`stm32f4xx_it.c`文件中有。为了不进行信号量的外部声明，我把中断函数直接放到了`ethernetif.c`中。

```
    void ETH_IRQHandler(void)
    {
        portBASE_TYPE xHigherPriorityTaskWoken = pdFALSE;

        // Frame received
        if ( ETH_GetDMAFlagStatus(ETH_DMA_FLAG_R) == SET)
        {
            // Give the semaphore to wakeup LwIP task
            xSemaphoreGiveFromISR( s_xSemaphore, &xHigherPriorityTaskWoken );
        }

        // Clear the interrupt flags.
        // Clear the Eth DMA Rx IT pending bits
        ETH_DMAClearITPendingBit(ETH_DMA_IT_R);
        ETH_DMAClearITPendingBit(ETH_DMA_IT_NIS);

        // Switch tasks if necessary.
        if( xHigherPriorityTaskWoken != pdFALSE )
        {
            portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );

        }
    }
```

---
### FIFTH
到此整个工程文件移植完成，接下来构造进行初始化函数。如下：
```
/* Inlcude ---------------------------------------------------------------------*/
#include "stdio.h"
#include "stdint.h"
#include "string.h"

#include "tcpip.h"
#include "ethernetif.h"
#include "stm32f4x7_eth_bsp.h"

#include <lwip/sockets.h>
#include <lwip/err.h>
#include <lwip/sys.h>

/** @addtogroup Cli_Project
  * @{
  */
/* Private typedef -------------------------------------------------------------*/
/* Private constants define ----------------------------------------------------*/
/* Static IP ADDRESS */
#define IP_ADDR0        192
#define IP_ADDR1        168
#define IP_ADDR2        11
#define IP_ADDR3        241

/* NETMASK */
#define NETMASK_ADDR0   255
#define NETMASK_ADDR1   255
#define NETMASK_ADDR2   255
#define NETMASK_ADDR3   0

/* Gateway Address */
#define GW_ADDR0        192
#define GW_ADDR1        168
#define GW_ADDR2        11
#define GW_ADDR3        1

/* Private macro define --------------------------------------------------------*/
/* Private variables -----------------------------------------------------------*/
struct netif xnetif;

/* Private function declaration ------------------------------------------------*/
/* Private functions -----------------------------------------------------------*/
/**
  * @brief  Initializes the lwIP stack
  * @param  None
  * @retval None
  */
void network_lwip_init(void)
{
    struct ip4_addr ipaddr;
    struct ip4_addr netmask;
    struct ip4_addr gw;

#ifndef USE_DHCP
//    uint8_t iptab[4] = {0};
//    uint8_t iptxt[20];
#endif

    /* Create tcp_ip stack thread */
    tcpip_init( NULL, NULL );

    /* IP address setting */
#ifdef USE_DHCP
    ipaddr.addr = 0;
    netmask.addr = 0;
    gw.addr = 0;
#else
    IP4_ADDR(&ipaddr, IP_ADDR0, IP_ADDR1, IP_ADDR2, IP_ADDR3);
    IP4_ADDR(&netmask, NETMASK_ADDR0, NETMASK_ADDR1 , NETMASK_ADDR2, NETMASK_ADDR3);
    IP4_ADDR(&gw, GW_ADDR0, GW_ADDR1, GW_ADDR2, GW_ADDR3);
#endif

    /* - netif_add(struct netif *netif, struct ip_addr *ipaddr,
    struct ip_addr *netmask, struct ip_addr *gw,
    void *state, err_t (* init)(struct netif *netif),
    err_t (* input)(struct pbuf *p, struct netif *netif))

    Adds your network interface to the netif_list. Allocate a struct
    netif and pass a pointer to this structure as the first argument.
    Give pointers to cleared ip_addr structures when using DHCP,
    or fill them with sane numbers otherwise. The state pointer may be NULL.

    The init function pointer must point to a initialization function for
    your ethernet netif interface. The following code illustrates it's use.*/

    netif_add(&xnetif, &ipaddr, &netmask, &gw, NULL, ethernetif_init, tcpip_input);

    /*  Registers the default network interface.*/
    netif_set_default(&xnetif);

    /* When the netif is fully configured this function must be called.*/
    netif_set_up(&xnetif);
}

/**
  * @brief  :
  * @param  : None
  * @retval : None
  */
void network_init(void)
{
    /* configure ethernet (GPIOs, clocks, MAC, DMA) */
    ETH_BSP_Config();//这个函数在stm32f4x7_eth_bsp.c 中定义

    network_lwip_init();//这个函数也就是上文定义的函数，源码在ST官方工程中的main.c文件中，main函数中的`LwIP_Init()`函数便是，我这里只使用了前面一部分。具体查看原函数。
}
```
把`network_init()`添加到初始化中去。进行调测。
可以通过ping IP 地址简单验证网络是否连通。然后在建立TCP SERVER，可以用SecureCRT连接，验证数据收发。

---
### NOTICE
    1、我在调试过程中，出现了初始化一切正常，但ping IP 地址始终ping不通。后来调试发现初始化过程中有bug：
    在`stm32f4x7_eth.c`文件`ETH_Init`函数中，以下代码
```
    do
    {
        timeout++;
    } while (!(ETH_ReadPHYRegister(PHYAddress, PHY_BSR) & PHY_Linked_Status) && (timeout < PHY_READ_TO));
```
    函数 ETH_ReadPHYRegister(PHYAddress, PHY_BSR)前几次获取的值为0x0000ffff，导致此处初始化通过，实际该值并不正确（可以通过打印该值看现象）。
    所以我把此处修改为(总共有三处，只修改可第一处位置)：
```
    int get_PHY_Linked_Status = 0;
    do
    {
        get_PHY_Linked_Status = ETH_ReadPHYRegister(PHYAddress, PHY_BSR);
        //printf("******* 0x%08x. \r\n", get_PHY_Linked_Status);
        get_PHY_Linked_Status = ((get_PHY_Linked_Status == 0x0000ffff) ? 0 : get_PHY_Linked_Status);
        timeout++;
    } while (!(get_PHY_Linked_Status & PHY_Linked_Status) && (timeout < PHY_READ_TO));
```
    但此处还存在一个bug，就是如果网线未插，这一直会停在此处，所以请务必注意，进行优化处理。

[源码链接](https://github.com/jungleeee/FreeRTOS-CLI)

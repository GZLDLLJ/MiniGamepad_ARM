# 序



由于作者水平有限，文档和视频中难免有出错和讲得不好的地方，欢迎各位读者和观众善意地提出意见和建议，谢谢！



| **实例**             | **描述**                             |
| -------------------- | ------------------------------------ |
| Eg1_Joystick         | 实现一个Joystick摇杆设备             |
| Eg2_WS2812B          | 点亮WS2812B灯珠并实现七彩渐变        |
| Eg3_MultiTimer       | 移植MultiTimer软件定时器模块         |
| Eg4_Mouse            | 实现模拟鼠标功能                     |
| Eg5_KeyBoard         | 实现模拟键盘功能                     |
| Eg6_DoubleJoystick   | 实现一个USB双摇杆                    |
| Eg7_CompositeGMK     | 实现Joystick、MOUSE、Keyboard的组合  |
| Eg8_Gamepad          | 实现游戏手柄Gamepad的功能            |
| Eg9_AbsoluteMouse    | 实现绝对值鼠标的功能                 |
| Eg10_Xinput          | 实现Xbox手柄功能，Xinput（出厂默认） |
| Eg11_Xinput01        | 外接摇杆电位器实现Xbox手柄功能       |
| Eg12_MultiAxisButton | 实现8轴32键摇杆                      |

# 第一部分、硬件概述

## 1.1 实物概图

 图1.1Gamepad实物概图
![image](https://img2022.cnblogs.com/blog/1966993/202210/1966993-20221015162528917-648827262.jpg)

如图1.1所示Gamepad评估板配置了8个6*6轻触按键，一个摇杆（Joystick），搭载一颗WS2812B灯珠，并将UART1串口，编程接口（SWD），外接Joystick接口，microUSB接口引出;  


## 1.2 Gamepad原理图
Gamepad原理图如图1.2所示，如看不清可打开Doc目录下的PDF文档查阅  
		图1.2 Gamepad原理图  
![image](https://img2022.cnblogs.com/blog/1966993/202211/1966993-20221112115509438-565811249.png)




# 第二部分、软件工具
## 2.1 软件概述
   在 /Software 目录下是常用的工具软件：
   1. Dt2_4：配置USB设备Report描述符的工具；
   2. USBHID调试助手/呀呀USB： USB调试工具，相当于串口调试助手功能；
   3. BUSHound：总线调试工具；
   4. USBlyzer：一款专业的USB协议分析软件
   5. MDK:常用编译器；
   6. STM32CubeMX：代码生成工具；
# 第三部分、实战训练
## 3.1 实例Eg1_Joystick
目标是实现 Joystick:枚举成XY轴的平面坐标和8个按键的USB HID。  

### 3.1.1硬件设计   

图1.3 Joystick原理图

   ![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214224348752-1607121797.png)

其中VRX1与VRY1是摇杆的电位器输出的电压信号（ADC检测)；SW1则是按键，右侧H1是外接的Joystick口备用;  



图1.4 KEY原理图  


![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214224447009-112297073.png)


如图1.4是KEY原理图，我们只要配置8个GPIO作为输入去检测按键信号;  

### 3.1.2 软件设计
USB设备开发需要具备一定的USB设备开发知识；关于Usb的学习，这里推荐两个学习视频和一个学习网站：

1. USB技术应用与开发：

   <https://www.bilibili.com/video/BV1sy4y1n7d9/?spm_id_from=333.33.header_right.fav_list.click&vd_source=2bbde87de845d5220b1d8ba075c12fb0>

2. CherryUSB设备协议栈教程：

   <https://www.bilibili.com/video/BV1Ef4y1t73d/?spm_id_from=333.33.header_right.fav_list.click&vd_source=2bbde87de845d5220b1d8ba075c12fb0>

3. USB中文网：

   <https://www.usbzh.com/>

我们主要做USB HID开发，一般我们需要了解一些标准请求，还有HID类的请求；其中标准请求主要是主机获取设备描述符、配置描述符、接口描述符、端点描述符、字符串描述符的过程，如果是HID，还有HID描述符的过程 ，以及报表描述符的过程；

一般的，我们配套的视频都有讲解USB设备枚举过程在代码中的实现，这里主要是基于STM32 HAL库的；

首先是初始化代码，我们通过STM32cubeMX软件去生成代码，具体配置请打开GamePad.ioc查阅，配套视频也有关键部分的讲解，这里不再赘述;  我们的工程使用的是Keil-MDK编译器，生成的工程目录如图1.5  

![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214224618747-792020857.png)
图1.5 工程目录
其中

> - __Application/MDK-ARM__ 存放的是启动代码；
> - __Application/User/Core：__ main函数，中断Handler，MSP相关代码；
> - __Application/User/USB_DEVICE/App：__* USB设备应用代码；
> - __Application/User/USB_DEVICE/Target：__ USB设备配置代码；
> - __Drivers/STM32F1xx_HAL_Driver：__ HAL库驱动代码
> - __Drivers/CMSIS：__ CMSIS相关代码	
> - __Middlewares/USB_Device_Library/__ USB设备库代码，对应cubemx Middleware；
> - __Customer：__ 这是我们自定义的代码；
> - __Doc：__ 存放说明文本文档；

工程目录这里只做一次介绍，后面的样例目录大同小异。

以下USB部分内容请大家务必多看几遍代码， 我们打开工程中的main函数可以看到

```c
int main(void)
{
  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();
  /* Configure the system clock */
  SystemClock_Config();
  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_DMA_Init();
  MX_ADC1_Init();
  MX_USB_DEVICE_Init();
  MX_USART1_UART_Init();  
  Gp_ADC_Start_DMA();//启动ADC DMA并开启中断
  while (1)
  {
	Gp_SendReport();	
  }
}
```

其中只有Gp_ADC_Start_DMA和Gp_SendReport是我们自定义的代码；其他均是STM32CUBEMX生成；由于篇幅的原因，我们主要介绍MX_USB_DEVICE_Init；

```c
void MX_USB_DEVICE_Init(void)
{
  /* Init Device Library, add supported class and start the library. */
  if (USBD_Init(&hUsbDeviceFS, &FS_Desc, DEVICE_FS) != USBD_OK)
  {
    Error_Handler();
  }
  if (USBD_RegisterClass(&hUsbDeviceFS, &USBD_CUSTOM_HID) != USBD_OK)
  {
    Error_Handler();
  }
  if (USBD_CUSTOM_HID_RegisterInterface(&hUsbDeviceFS, &USBD_CustomHID_fops_FS) != USBD_OK)
  {
    Error_Handler();
  }
  if (USBD_Start(&hUsbDeviceFS) != USBD_OK)
  {
    Error_Handler();
  }
}
```

其中USBD_Init函数中的FS_Desc数据结构里面是获取UBS设备描述符USBD_FS_DeviceDescriptor，语言ID字符串描述符USBD_FS_LangIDStrDescriptor，厂商字符串描述符USBD_FS_ManufacturerStrDescriptor，产品字符串描述符USBD_FS_ProductStrDescriptor，序列号字符串描述符USBD_FS_SerialStrDescriptor，配置字符串描述符USBD_FS_ConfigStrDescriptor，接口描述符USBD_FS_InterfaceStrDescriptor的函数指针；

上述描述符中，UBS设备描述符是必需的，其他都是字符串描述符，可选的；我们不妨打开usb描述符获取函数与usb描述符报表

```c
/** USB standard device descriptor. */
__ALIGN_BEGIN uint8_t USBD_FS_DeviceDesc[USB_LEN_DEV_DESC] __ALIGN_END =
{
  0x12,                       /*bLength */
  USB_DESC_TYPE_DEVICE,       /*bDescriptorType*/
  0x00,                       /*bcdUSB */
  0x02,
  0x00,                       /*bDeviceClass*/
  0x00,                       /*bDeviceSubClass*/
  0x00,                       /*bDeviceProtocol*/
  USB_MAX_EP0_SIZE,           /*bMaxPacketSize*/
  LOBYTE(USBD_VID),           /*idVendor*/
  HIBYTE(USBD_VID),           /*idVendor*/
  LOBYTE(USBD_PID_FS),        /*idProduct*/
  HIBYTE(USBD_PID_FS),        /*idProduct*/
  0x00,                       /*bcdDevice rel. 2.00*/
  0x02,
  USBD_IDX_MFC_STR,           /*Index of manufacturer  string*/
  USBD_IDX_PRODUCT_STR,       /*Index of product string*/
  USBD_IDX_SERIAL_STR,        /*Index of serial number string*/
  USBD_MAX_NUM_CONFIGURATION  /*bNumConfigurations*/
};

uint8_t * USBD_FS_DeviceDescriptor(USBD_SpeedTypeDef speed, uint16_t *length)
{
  UNUSED(speed);
  *length = sizeof(USBD_FS_DeviceDesc);
  return USBD_FS_DeviceDesc;
}
```

关于USB设备描述符的介绍，注释已经非常清楚；不了解的地方可以学习一下前面的推荐两个学习视频和一个学习网站；

设备描述符找到了，我们还需要的配置描述符，接口描述符，HID描述符（如果是HID设备），端点描述符；

我们继续回到MX_USB_DEVICE_Init，可以看到USBD_RegisterClass中的USBD_CUSTOM_HID，其中USBD_CUSTOM_HID_Setup是处理USB主机的一些请求过程，包括标准请求等；而USBD_CUSTOM_HID_GetHSCfgDesc（高速）和USBD_CUSTOM_HID_GetFSCfgDesc（全速）都是配置描述符；

```c

USBD_ClassTypeDef  USBD_CUSTOM_HID =
{
  USBD_CUSTOM_HID_Init,
  USBD_CUSTOM_HID_DeInit,
  USBD_CUSTOM_HID_Setup,
  NULL, /*EP0_TxSent*/
  USBD_CUSTOM_HID_EP0_RxReady, /*EP0_RxReady*/ /* STATUS STAGE IN */
  USBD_CUSTOM_HID_DataIn, /*DataIn*/
  USBD_CUSTOM_HID_DataOut,
  NULL, /*SOF */
  NULL,
  NULL,
  USBD_CUSTOM_HID_GetHSCfgDesc,
  USBD_CUSTOM_HID_GetFSCfgDesc,
  USBD_CUSTOM_HID_GetOtherSpeedCfgDesc,
  USBD_CUSTOM_HID_GetDeviceQualifierDesc,
};
```

由于我们的MCU是支持全速的，所以这里应该是USBD_CUSTOM_HID_GetFSCfgDesc，

```
/* USB CUSTOM_HID device FS Configuration Descriptor */
__ALIGN_BEGIN static uint8_t USBD_CUSTOM_HID_CfgFSDesc[USB_CUSTOM_HID_CONFIG_DESC_SIZ] __ALIGN_END =
{
  0x09, /* bLength: Configuration Descriptor size */
  USB_DESC_TYPE_CONFIGURATION, /* bDescriptorType: Configuration */
  USB_CUSTOM_HID_CONFIG_DESC_SIZ,
  /* wTotalLength: Bytes returned */
  0x00,
  0x01,         /*bNumInterfaces: 1 interface*/
  0x01,         /*bConfigurationValue: Configuration value*/
  0x00,         /*iConfiguration: Index of string descriptor describing
  the configuration*/
  0xC0,         /*bmAttributes: bus powered */
  0x32,         /*MaxPower 100 mA: this current is used for detecting Vbus*/

  /************** Descriptor of CUSTOM HID interface ****************/
  /* 09 */
  0x09,         /*bLength: Interface Descriptor size*/
  USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
  0x00,         /*bInterfaceNumber: Number of Interface*/
  0x00,         /*bAlternateSetting: Alternate setting*/
  0x02,         /*bNumEndpoints*/
  0x03,         /*bInterfaceClass: CUSTOM_HID*/
  0x00,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
  0x00,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
  0,            /*iInterface: Index of string descriptor*/
  /******************** Descriptor of CUSTOM_HID *************************/
  /* 18 */
  0x09,         /*bLength: CUSTOM_HID Descriptor size*/
  CUSTOM_HID_DESCRIPTOR_TYPE, /*bDescriptorType: CUSTOM_HID*/
  0x11,         /*bCUSTOM_HIDUSTOM_HID: CUSTOM_HID Class Spec release number*/
  0x01,
  0x00,         /*bCountryCode: Hardware target country*/
  0x01,         /*bNumDescriptors: Number of CUSTOM_HID class descriptors to follow*/
  0x22,         /*bDescriptorType*/
  USBD_CUSTOM_HID_REPORT_DESC_SIZE,/*wItemLength: Total length of Report descriptor*/
  0x00,
  /******************** Descriptor of Custom HID endpoints ********************/
  /* 27 */
  0x07,          /*bLength: Endpoint Descriptor size*/
  USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

  CUSTOM_HID_EPIN_ADDR,     /*bEndpointAddress: Endpoint Address (IN)*/
  0x03,          /*bmAttributes: Interrupt endpoint*/
  CUSTOM_HID_EPIN_SIZE, /*wMaxPacketSize: 2 Byte max */
  0x00,
  CUSTOM_HID_FS_BINTERVAL,          /*bInterval: Polling Interval */
  /* 34 */

  0x07,          /* bLength: Endpoint Descriptor size */
  USB_DESC_TYPE_ENDPOINT, /* bDescriptorType: */
  CUSTOM_HID_EPOUT_ADDR,  /*bEndpointAddress: Endpoint Address (OUT)*/
  0x03, /* bmAttributes: Interrupt endpoint */
  CUSTOM_HID_EPOUT_SIZE,  /* wMaxPacketSize: 2 Bytes max  */
  0x00,
  CUSTOM_HID_FS_BINTERVAL,  /* bInterval: Polling Interval */
  /* 41 */
};
static uint8_t  *USBD_CUSTOM_HID_GetFSCfgDesc(uint16_t *length)
{
  *length = sizeof(USBD_CUSTOM_HID_CfgFSDesc);
  return USBD_CUSTOM_HID_CfgFSDesc;
}
```

细心的同学可以发现，其实这个函数是获取了配置描述符，接口描述符，HID描述符，端点描述符；不过一般的，主机一般都是先请求配置描述符，然后通过配置描述符就知道了整个描述符集合的大小USB_CUSTOM_HID_CONFIG_DESC_SIZ；

现在配置描述符，接口描述符，HID描述符，端点描述符都有了，因为我们是HID设备，故而还需要报表描述符；还是回到MX_USB_DEVICE_Init，在USBD_CUSTOM_HID_RegisterInterface这个函数中的USBD_CustomHID_fops_FS结构体中的第一个成员，真是众里寻他千百度,蓦然回首,那报表描述符却在灯火阑珊处。

```c
/** Usb HID report descriptor. */
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
    0x09, 0x04,                    // USAGE (Joystick)
    0xa1, 0x01,                    // COLLECTION (Application)
    0xa1, 0x02,                    //     COLLECTION (Logical)
    0x09, 0x30,                    //     USAGE (X)
    0x09, 0x31,                    //     USAGE (Y)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
    0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
    0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
    0x75, 0x08,                    //     REPORT_SIZE (8)
    0x95, 0x02,                    //     REPORT_COUNT (2)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0x05, 0x09,                    //     USAGE_PAGE (Button)
    0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
    0x29, 0x08,                    //     USAGE_MAXIMUM (Button 8)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
    0x95, 0x08,                    //     REPORT_COUNT (8)
    0x75, 0x01,                    //     REPORT_SIZE (1)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0xc0,                          //     END_COLLECTION
    0xC0    					   //END_COLLECTION	             */
};
USBD_CUSTOM_HID_ItfTypeDef USBD_CustomHID_fops_FS =
{
  CUSTOM_HID_ReportDesc_FS,
  CUSTOM_HID_Init_FS,
  CUSTOM_HID_DeInit_FS,
  CUSTOM_HID_OutEvent_FS
};
```

另外需要注意USBD_CUSTOM_HID_REPORT_DESC_SIZE,  这个宏是报告描述符实际数组大小，大小不对会导致枚举失败;  

"\#define USBD_CUSTOM_HID_REPORT_DESC_SIZE     46"

以上报表描述符通过生成工具Dt2_4配置生成报告描述符，可以看出X,Y轴定义成无符号8位数，XY的描述占用2个字；按键一共有8个，每个大小描述是bit，8个bit即1个byte；因此XY坐标+8个按键=3个byte；我们需要上报3个byte的数据给主机（HOST）。 

最后是应用程序的编写，在main函数中调用Gp_ADC_Start_DMA和Gp_SendReport，其中Gp_ADC_Start_DMA是启动ADC DMA 中断完成采样的，而Gp_SendReport则是解析ADC采样后的数据并上报；

我们是通过Gp_ADC_Start_DMA调用HAL_ADC_Start_DMA启动ADC DMA模式采样，需要传入hadc1句柄，AD_DATA数据缓存，AD_DATA_SIZE缓存大小，其中偶数索引是ADC通道0的采样数据，如AD_DATA[0],AD_DATA[2]，反之则是通道2的数据；HAL_ADC_Start_IT是启动ADC1的全局中断，HAL_ADC_ConvCpltCallback是采样完成的中断回调函数，在stm32f1xx_it.c的ADC1_2_IRQHandler里面可遍历找到。

```c
//启动ADC DMA
void Gp_ADC_Start_DMA(void)
{
    HAL_ADC_Start_DMA(&hadc1, (uint32_t*)&AD_DATA, AD_DATA_SIZE);
    HAL_ADC_Start_IT(&hadc1);
}
/**	ADC ISR
*		Handles the values from ADC after the conversion finished
*/
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc) {

    if( adcValueReady == 0 ) {
        for(i=0; i<AD_DATA_SIZE;)
        {
            AdXSum += AD_DATA[i];
            i++;
            AdYSum += AD_DATA[i];
            i++;
        }
        adcValueReady = 1;
    }

}
```

再来看我们的数据处理上报函数

```c
//按键扫描
u8 key_scan(void)
{
    key=0;
    if((UPKEY)==0)
    {
        key|=BIT0;
    }else{
		key&=(~BIT0);
	}
    if((LFKEY)==0)
    {
        key|=BIT1;
    }else{
		key&=(~BIT1);
	}
    if((RGKEY)==0)
    {
        key|=BIT2;
    }else{
		key&=(~BIT2);
	}
    if((DNKEY)==0)
    {
        key|=BIT3;
    }else{
		key&=(~BIT3);
	}
    if((TBKEY)==0)
    {
        key|=BIT4;
    }else{
		key&=(~BIT4);
	}
    if((BKKEY)==0)
    {
        key|=BIT5;
    }else{
		key&=(~BIT5);
	}
    if((MDKEY)==0)
    {
        key|=BIT6;
    }else{
		key&=(~BIT6);
	}
    if((STKEY)==0)
    {
        key|=BIT7;
    }else{
		key&=(~BIT7);
	}
    return key;

}


/*	Re-maps a number from one range to another
*
*/
int32_t map(int32_t x, int32_t in_min, int32_t in_max, int32_t out_min, int32_t out_max){
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}


//处理并上报数据
void Gp_SendReport(void)
{
	u8 X=0,Y=0,Ktemp=0;
	if( adcValueReady == 1 )
	{
		Xtemp=AdXSum/10;
		AdXSum=0;
		Ytemp=AdYSum/10;
		AdYSum=0;
		if(Xtemp>Xmax)
			Xtemp=Xmax;
		if(Xtemp<Xmin)
			Xtemp=Xmin;

		if(Ytemp>Ymax)
			Ytemp=Ymax;
		if(Ytemp<Ymin)
			Ytemp=Ymin;		
		adcValueReady=0;
	}

    X=(uint8_t)map( Xtemp, Xmin, Xmax, 0, UINT8_MAX );
    Y=(uint8_t)map( Ytemp, Ymin, Ymax, 0, UINT8_MAX );
	Ktemp=key_scan();
	Joystick_Report[0]=Y;
	Joystick_Report[1]=X;
	Joystick_Report[2]=Ktemp;
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Joystick_Report, 3);	
					
	HAL_Delay(5);
	
}
```

key_scan是扫描按键，map是转换数据的范围，因为我们的ADC采样是12bit，取值0~4095，需要转成8bit，0~255；

最后通过USBD_CUSTOM_HID_SendReport上报数据给主机。

### 3.1.3 下载验证
我们把固件程序下载进去可以，打开“设备与打印机”可以看到USB设备枚举成了一个Gamepad，如下图。  
![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214231513890-445089246.png)  

图1.5 Gamepad设备  
右键打开游戏控制器后，点击属性得到下图所示界面  
![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214231614778-314243803.png)
图1.6 游戏控制器  
我们可以摇Joystick和按按键可以发现上图游戏控制器界面也跟着响应。



### 3.1.4入门视频

本期的入门视频如下：

https://www.bilibili.com/video/BV1X84y1v7u9/?vd_source=2bbde87de845d5220b1d8ba075c12fb0



## 3.2 实例Eg2_WS2812B
我们想要实现Rainbow彩虹七彩渐变的效果，我们当前版本评估板式板载了一颗WS2812B灯珠，如需学习WS2812的控制原理以及实现酷炫的效果，可以看我们关于WS2812控制专题的视频合集：

https://www.bilibili.com/video/BV1oS4y1b72F/?vd_source=2bbde87de845d5220b1d8ba075c12fb0

### 3.2.1硬件设计
 如图1.7和1.8所示MCU通过1根数据线控制WS2812B
 ![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214232506349-1712207719.png)
图1.7 WS2812B单颗灯珠原理图  
![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214232618275-889665108.png)
图1.8 MCU连接WS2812控制引脚 



### 3.2.2 软件设计

WS2812B要求的时序
![image](https://img2022.cnblogs.com/blog/1966993/202211/1966993-20221110171925970-1084375874.png)

   我们将DIN配置为PWM+DMA的方式去驱动WS2812B，初始化配置如图1.9所示  
   ![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214233059962-2123538503.png)  
   图1.9 DIN引脚配置  
   MCU主频为72M，Counter period=89,即计数周期=1/72*89约等于1.23us  ;根据高中物理知识周期等于频率的倒数T=1/f可得1/72Mhz=0.0138888888888889us,

```
#define WS_HIGH 45
#define WS_LOW  22
```

   我们的代码中配置0码和1码如上，计算过程如下：

- 1码计算：T1H=45*0.0138888888888889us=0.6250000000000005us满足T1H的580nS~1uS的需求，1.23us-T1H约等于0.605us满足T1L；
- 0码计算：同理可得T0H=0.306us，T0L=0.924us

我们本节的代码完全是参考以下博文移植过来，这里大家自行学习。

- WS2812驱动请参考该博文链接学习：

https://www.twblogs.net/a/5d5f26e7bd9eee5327fdec7c/?lang=zh-cn

### 3.2.3 下载验证
我们把固件程序下载进去可以，可以看到板载的5050灯珠，进行七彩变化； 建议看视频效果。
![image](https://img2022.cnblogs.com/blog/1966993/202211/1966993-20221110171953828-1565635387.png)




## 3.3 实例Eg3_MultiTimer

目标是移植MultiTimer实现多任务。MultiTimer是一款开源定时器扩展模块，以下是开源链接

https://github.com/0x1abin/MultiTimer

### 3.3.1简介

MultiTimer 是一个软件定时器扩展模块，可无限扩展你所需的定时器任务，取代传统的标志位判断方式， 更优雅更便捷地管理程序的时间触发时序。

### 3.3.2使用方法

1. 配置系统时间基准接口，安装定时器驱动；

```c
uint64_t PlatformTicksGetFunc(void)
{
    /* Platform implementation */
}

MultiTimerInstall(PlatformTicksGetFunc);
```

2. 实例化一个定时器对象；

```c
MultiTimer timer1;
```

3. 设置定时时间，超时回调处理函数， 用户上下指针，启动定时器；

```c
int MultiTimerStart(&timer1, uint64_t timing, MultiTimerCallback_t callback, void* userData);
```

4. 在主循环调用定时器后台处理函数

```c
int main(int argc, char *argv[])
{
    ...
    while (1) {
        ...
        MultiTimerYield();
    }
}
```

### 3.3.3功能限制

1.定时器的时钟频率直接影响定时器的精确度，尽可能采用1ms/5ms/10ms这几个精度较高的tick;

2.定时器的回调函数内不应执行耗时操作，否则可能因占用过长的时间，导致其他定时器无法正常超时；

3.由于定时器的回调函数是在 MultiTimerYield 内执行的，需要注意栈空间的使用不能过大，否则可能会导致栈溢出。

### 3.3.4Examples

见example目录下的测试代码，main.c为普通平台测试demo，test_linux.c为linux平台的测试demo。

```c
#include <stdio.h>
#include <sys/time.h>
#include <time.h>
#include "MultiTimer.h"

MultiTimer timer1;
MultiTimer timer2;
MultiTimer timer3;

uint64_t PlatformTicksGetFunc(void)
{
    struct timespec current_time;
    clock_gettime(CLOCK_MONOTONIC, &current_time);
    return (uint64_t)((current_time.tv_sec * 1000) + (current_time.tv_nsec / 1000000));
}

void exampleTimer1Callback(MultiTimer* timer, void *userData)
{
    printf("exampleTimer1Callback-> %s.\r\n", (char*)userData);
    MultiTimerStart(timer, 1000, exampleTimer1Callback, userData);
}

void exampleTimer2Callback(MultiTimer* timer, void *userData)
{
    printf("exampleTimer2Callback-> %s.\r\n", (char*)userData);
}

void exampleTimer3Callback(MultiTimer* timer, void *userData)
{
    printf("exampleTimer3Callback-> %s.\r\n", (char*)userData);
    MultiTimerStart(timer, 4567, exampleTimer3Callback, userData);
}

int main(int argc, char *argv[])
{
    MultiTimerInstall(PlatformTicksGetFunc);

    MultiTimerStart(&timer1, 1000, exampleTimer1Callback, "1000ms CYCLE timer");
    MultiTimerStart(&timer2, 5000, exampleTimer2Callback, "5000ms ONCE timer");
    MultiTimerStart(&timer3, 3456, exampleTimer3Callback, "3456ms delay start, 4567ms CYCLE timer");

    while (1) {
        MultiTimerYield();
    }
}
```

### 3.3.4软件设计

直接复制MultiTimer.c和MultiTimer.h到我们的Customer目录下，包含头文件MultiTimer.h到main.c中，根据3.3.2使用方法可知需要配置系统时间基准接口，安装定时器驱动，在我们的STM32 HAL库中，使用Systick提供时基，如下：

```c
uint64_t PlatformTicksGetFunc(void)
{
    return (uint64_t)HAL_GetTick();
}
```

接着我们实例化两个个定时器对象timer1和timer2，设置定时时间，超时回调处理函数， 用户上下指针，启动定时器；用以定时执行Joystick和WS2812B两个任务处理。

```c
MultiTimer timer1;
MultiTimer timer2;
void JoystickTimer1Callback(MultiTimer* timer, void *userData)
{
	Gp_SendReport();
    MultiTimerStart(timer, 5, JoystickTimer1Callback, userData);
}
void WS2812BTimer2Callback(MultiTimer* timer, void *userData)
{
	WS281x_Rainbow(0);
    MultiTimerStart(timer, 100, WS2812BTimer2Callback, userData);
}
int main(void)
{
  HAL_Init(); 
  SystemClock_Config();
  MultiTimerInstall(PlatformTicksGetFunc);
  MX_GPIO_Init();
  MX_DMA_Init();
  MX_ADC1_Init();
  MX_USB_DEVICE_Init();
  MX_USART1_UART_Init();
  MX_TIM3_Init();
  Gp_ADC_Start_DMA();
  MultiTimerStart(&timer1, 5, JoystickTimer1Callback, NULL);
  MultiTimerStart(&timer2, 10, WS2812BTimer2Callback, NULL);		
  while (1)
  {	
	 MultiTimerYield();
  }
}
```

### 3.3.5 下载验证

本节的实验现象是前面两节的结合，不过2812渐变速度被调慢了。

## 3.4 实例Eg4_Mouse
目标是模拟鼠标功能，具备XY坐标和左右中键以及滚轮上下的鼠标功能。
### 3.4.1硬件设计
如图2.0所示，我们将使用VRX1和VRY1作为鼠标相对坐标，SW1作为鼠标中键  
![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214233602931-400523260.png)
       图2.0 Joystick原理图
![image](https://img2020.cnblogs.com/blog/1966993/202112/1966993-20211214233636774-1038002781.png)
如上图2.1，UP，DN，LF，RG分别代表鼠标滚轮向上，向下，鼠标左键，右键；  

### 3.4.2 软件设计
我们将usbd_custom_hid_if.c中CUSTOM_HID_ReportDesc_FS修改为鼠标的报告描述符;

鼠标的报告描述符：

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
            0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
            0x09, 0x02,                    // USAGE (Mouse)
            0xa1, 0x01,                    // COLLECTION (Application)
            0x09, 0x01,                    //   USAGE (Pointer)
            0xa1, 0x00,                    //   COLLECTION (Physical)
            0x05, 0x09,                    //     USAGE_PAGE (Button)
            0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
            0x29, 0x03,                    //     USAGE_MAXIMUM (Button 3)
            0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
            0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
            0x95, 0x03,                    //     REPORT_COUNT (3)
            0x75, 0x01,                    //     REPORT_SIZE (1)
            0x81, 0x02,                    //     INPUT (Data,Var,Abs)
            0x95, 0x01,                    //     REPORT_COUNT (1)
            0x75, 0x05,                    //     REPORT_SIZE (5)
            0x81, 0x03,                    //     INPUT (Cnst,Var,Abs)
            0x05, 0x01,                    //     USAGE_PAGE (Generic Desktop)
            0x09, 0x30,                    //     USAGE (X)
            0x09, 0x31,                    //     USAGE (Y)
            0x09, 0x38,                    //     USAGE (Wheel)
            0x15, 0x81,                    //     LOGICAL_MINIMUM (-127)
            0x25, 0x7f,                    //     LOGICAL_MAXIMUM (127)
            0x75, 0x08,                    //     REPORT_SIZE (8)
            0x95, 0x03,                    //     REPORT_COUNT (3)
            0x81, 0x06,                    //     INPUT (Data,Var,Rel)
            0xc0,                          //   END_COLLECTION
 			0xC0    					   // END_COLLECTION	            
};
```

然后按照报告描述符上面定义的数据，修改Gp_SendReport中的Joystick_Report数组；详见代码

```c
void Gp_SendReport(void)
{
	u8 X=0,Y=0;
	memset(Joystick_Report,0,4);
	for(u8 i=0; i<AD_DATA_SIZE;)
	{
		AdXSum += AD_DATA[i];
		i++;
		AdYSum += AD_DATA[i];
		i++;
	}
	Xtemp=AdXSum/10;
	AdXSum=0;
	Ytemp=AdYSum/10;
	AdYSum=0;
	if(Xtemp>Xmax)
		Xtemp=Xmax;
	if(Xtemp<Xmin)
		Xtemp=Xmin;

	if(Ytemp>Ymax)
		Ytemp=Ymax;
	if(Ytemp<Ymin)
		Ytemp=Ymin;	
    X=(uint8_t)map( Xtemp, Xmin, Xmax, 0, UINT8_MAX );
    Y=(uint8_t)map( Ytemp, Ymin, Ymax, 0, UINT8_MAX );	
    if(X>(X_BASE+20))
    {
        Joystick_Report[2]=((X-X_BASE)>>DIV)+1;
    }
    if(X<(X_BASE-20))
    {
        Joystick_Report[2]=(u8)-(((X_BASE-X)>>DIV)+1);
    }
    if(Y>(Y_BASE+20))
    {
        Joystick_Report[1]=((Y-Y_BASE)>>DIV)+1;
    }
    if(Y<(Y_BASE-20))
    {
        Joystick_Report[1]=(u8)-(((Y_BASE-Y)>>DIV)+1);
    }	
	Key_Handle(Joystick_Report);
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Joystick_Report, 4);	
}
```

Joystick_Report[1]和Joystick_Report[2]分别是相对坐标XY的值，Joystick_Report[0]是按键；if(X>(X_BASE+20))是摇杆右移了，

(X-X_BASE)是左移了多少，((X-X_BASE)>>DIV)是分频一下减少一下鼠标移动速度，而+1目的是防止位移被分频掉了而没有反应。

其他情况分别是其他方向的位移的解析。

另外就是Key_Handle，分别是左右中键的解析，以及上下键解析成滚轮。

```c
void Key_Handle(uint8_t* kv)
{
	static uint16_t c_tick=0;
    if((LFKEY)==0)
    {
        kv[0]|=BIT0;
    }else{
		kv[0]&=~BIT0;
	}
    if((RGKEY)==0)
     {
        kv[0]|=BIT1;
    }else{
		kv[0]&=~BIT1;
	}
    if((SW1)!=1)
    {
        kv[0]|=BIT2;
    }else{
		kv[0]&=~BIT2;
	}
    if((UPKEY)==0)
    {
        if(c_tick++>5)
        {
            kv[3]=1;
            c_tick=0;
        }
    }
    if((DNKEY)==0)
    {
        if(c_tick++>5)
        {
            kv[3]=(u8)-1;
            c_tick=0;
        }
    }
}
```



### 3.4.3 下载验证

我们把固件程序下载进去，摇动摇杆，桌面鼠标指针对应移动，按对应按键，鼠标按键作出相对应的反应；



## 3.5 实例Eg5_KeyBoard

目标是模拟USB Keyboard功能，主要实现shift、1~8键功能。

### 3.5.1硬件设计

我们将使用SW1作为左shift键，SW2、SW4、SW5、SW3、SW6、SW9、SW8、SW7分别作为1~8按键。 

### 3.5.2 软件设计

我们将usbd_custom_hid_if.c中CUSTOM_HID_ReportDesc_FS修改为USB Keyboard的报告描述符;

USB Keyboard的报告描述符：

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
    0x05, 0x01,        // Usage Page (Generic Desktop Ctrls)
    0x09, 0x06,        // Usage (Keyboard)
    0xA1, 0x01,        // Collection (Application)
    0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
    0x19, 0xE0,        //   Usage Minimum (0xE0)
    0x29, 0xE7,        //   Usage Maximum (0xE7)
    0x15, 0x00,        //   Logical Minimum (0)
    0x25, 0x01,        //   Logical Maximum (1)
    0x75, 0x01,        //   Report Size (1)
    0x95, 0x08,        //   Report Count (8)
    0x81, 0x02,        //   Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
    0x75, 0x08,        //   Report Size (8)
    0x95, 0x01,        //   Report Count (1)
    0x81, 0x01,        //   Input (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
    0x95, 0x05,        //   Report Count (5)
    0x75, 0x01,        //   Report Size (1)
    0x05, 0x08,        //   Usage Page (LEDs)
    0x19, 0x01,        //   Usage Minimum (Num Lock)
    0x29, 0x03,        //   Usage Maximum (Scroll Lock)
    0x29, 0x03,        //   Usage Maximum (Scroll Lock)
    0x91, 0x02,        //   Output (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
    0x95, 0x01,        //   Report Count (1)
    0x75, 0x03,        //   Report Size (3)
    0x75, 0x03,        //   Report Size (3)
    0x91, 0x01,        //   Output (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
    0x95, 0x06,        //   Report Count (6)
    0x75, 0x08,        //   Report Size (8)
    0x15, 0x00,        //   Logical Minimum (0)
    0x25, 0x65,        //   Logical Maximum (101)
    0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
    0x19, 0x00,        //   Usage Minimum (0x00)
    0x29, 0x65,        //   Usage Maximum (0x65)
    0x81, 0x00,        //   Input (Data,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)
    0xC0,              // End Collection	            
};
```

然后按照报告描述符，我们通过捕获鼠标数据可以得到如下协议

键盘发送给PC的数据每次8个字节
BYTE1 BYTE2 BYTE3 BYTE4 BYTE5 BYTE6 BYTE7 BYTE8
定义分别是：
BYTE1 --
    |--bit0:  Left Control是否按下，按下为1 
    |--bit1:  Left Shift 是否按下，按下为1 
    |--bit2:  Left Alt  是否按下，按下为1 
    |--bit3:  Left GUI  是否按下，按下为1 
    |--bit4:  Right Control是否按下，按下为1 
    |--bit5:  Right Shift 是否按下，按下为1 
    |--bit6:  Right Alt  是否按下，按下为1 
    |--bit7:  Right GUI  是否按下，按下为1 
BYTE2 -- 保留字节
BYTE3--BYTE8 -- 这六个为普通按键

根据我们之前定好的SW1作为左shift按键，所以这里BYTE1的bit1位1即为按下，其他是1~8键则依次填充BYTE3~BYTE8；

```c
void Key_Board_Handle(void)
{
    uint8_t Buf[8]={0,0,0,0,0,0,0,0};
    static uint8_t lastBuf[8]={0,0,0,0,0,0,0,0};
    uint8_t i=2;
    if((SW1)!=0)
    {
        Buf[0]|=0x02;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }else{
		Buf[0]&=~0x02;		
	}
    if((UPKEY)==0)//1!
    {
        Buf[i]=CODE1;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((LFKEY)==0)//2@
    {
        Buf[i]=CODE2;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((RGKEY)==0)//Rt
    {
        Buf[i]=CODE3;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((DNKEY)==0)//1!
    {
        Buf[i]=CODE4;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((TBKEY)==0)//2@
    {
        Buf[i]=CODE5;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((BKKEY)==0)//左CTRL
    {
        Buf[i]=CODE6;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if((MDKEY)==0)//左ALT
    {
        Buf[i]=CODE7;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }

    if((STKEY)==0)//I
    {
        Buf[i]=CODE8;
        if(++i==8)//切换到下个位置。
        {
            i=2;
        }
    }
    if(memcmp(Buf,lastBuf,8)!=0)
    {
       USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Buf, 8);
    }
    memcpy(lastBuf,Buf,8);
}
```




### 3.5.3 下载验证

我们把固件程序下载进去，pc端的设备与打印机面板显示枚举成功的键盘设备；按下SW1即是shift键按下，SW2、SW3、SW4、SW5、SW9、SW8、SW7、SW6按下也对应是主键盘上的1~8键按下，按住“shift+1”也可以打印“！”，其他组合键也同理可得。



















## 3.6 实例Eg6_DoubleJoystick

目标是实现一个USB带两个joystick摇杆；功能完全与实例Eg1_GamePad一致；

### 3.6.1硬件设计

参考原理图； 

### 3.6.2 软件设计

首先要修改的是报表描述符：

```c
/** Usb HID report descriptor. */
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
  /* USER CODE BEGIN 0 */
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
    0x09, 0x04,                    // USAGE (Joystick)
    0xa1, 0x01,                    // COLLECTION (Application)
	0x85, 0x01,                    //   REPORT_ID (1)
    0xa1, 0x02,                    //     COLLECTION (Logical)
    0x09, 0x30,                    //     USAGE (X)
    0x09, 0x31,                    //     USAGE (Y)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
    0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
    0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
    0x75, 0x08,                    //     REPORT_SIZE (8)
    0x95, 0x02,                    //     REPORT_COUNT (2)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0x05, 0x09,                    //     USAGE_PAGE (Button)
    0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
    0x29, 0x08,                    //     USAGE_MAXIMUM (Button 8)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
    0x95, 0x08,                    //     REPORT_COUNT (8)
    0x75, 0x01,                    //     REPORT_SIZE (1)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0xc0, 0xc0,                    //               END_COLLECTION

    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
    0x09, 0x04,                    // USAGE (Joystick)
    0xa1, 0x01,                    // COLLECTION (Application)
	0x85, 0x02,                    //   REPORT_ID (2)
    0xa1, 0x02,                    //     COLLECTION (Logical)
    0x09, 0x30,                    //     USAGE (X)
    0x09, 0x31,                    //     USAGE (Y)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
    0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
    0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
    0x75, 0x08,                    //     REPORT_SIZE (8)
    0x95, 0x02,                    //     REPORT_COUNT (2)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0x05, 0x09,                    //     USAGE_PAGE (Button)
    0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
    0x29, 0x08,                    //     USAGE_MAXIMUM (Button 8)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
    0x95, 0x08,                    //     REPORT_COUNT (8)
    0x75, 0x01,                    //     REPORT_SIZE (1)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0xc0,                      //               END_COLLECTION
  /* USER CODE END 0 */
  0xC0    /*     END_COLLECTION	             */
};
```

与前面几个实例不同的是，这里增加了Report ID，就是说每个USB接口都支持多个Report ID;每个Report ID都支持不同的报表描述符，如某些复合的USB键鼠一体设备，就是通过USB Report ID区分的键盘与鼠标的；

数据解析：XY_Handle是解析X，Y坐标的，key_scan是对8颗按键进行扫描，Joystick_Report[0]就是Report ID，占用1Byte，也就是如果带宽允许，最大支持255个报表；

```c
void GamepadHandle(void)
{
	XY_Handle();
	Joystick_Report[0]=1;//Report 1;
	Joystick_Report[1]=Y;
	Joystick_Report[2]=X;
	key_scan(&Joystick_Report[3]);	
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Joystick_Report, JOYBUFSIZE);
	HAL_Delay(8);
	Joystick_Report[0]=2;//Report 1;
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Joystick_Report, JOYBUFSIZE);
	HAL_Delay(8);	
}
```

### 3.6.3 下载验证

我们把固件程序下载进去，可以看到游戏控制器界面有两个控制器，调开属性界面两个都可以控制；
![image](https://img2022.cnblogs.com/blog/1966993/202210/1966993-20221018221056179-1624529137.png)

我们可以打开Bus Hound，抓取报文，端点0传输的是枚举过程，然后我们看Device：66.1的4个字节的，报文01 7e 7d 00是Report id为1的报文， 报文02 7e 7d 00是Report id为2的报文，因为我们上报的是相同的数据，就ID不同，故而控制的是两个joystick设备；
![image](https://img2022.cnblogs.com/blog/1966993/202210/1966993-20221018221123968-1460373926.png)





## 3.7 实例Eg7_ComDev_JMK

本节目标是实现Joystick、Mouse和Keyboard的组合，即把实例Eg1_Joystick、实例Eg4_Mouse与实例Eg5_KeyBoard组合成一个设备，通过按键SW2按下依次切换成Joystick、Mouse和Keyboard设备，而2812灯珠则通过红绿蓝指示切换三种不同的设备。

### 3.7.1硬件设计

参考原理图； 

### 3.7.2 软件设计

1. 准备Joystick、Mouse和Keyboard三个报表，这三个报表分别是实例Eg1_Joystick、实例Eg4_Mouse与实例Eg5_KeyBoard的报表，

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
	
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
    0x09, 0x04,                    // USAGE (Joystick)
    0xa1, 0x01,                    // COLLECTION (Application)
    0xa1, 0x02,                    //     COLLECTION (Logical)
    0x09, 0x30,                    //     USAGE (X)
    0x09, 0x31,                    //     USAGE (Y)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
    0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
    0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
    0x75, 0x08,                    //     REPORT_SIZE (8)
    0x95, 0x02,                    //     REPORT_COUNT (2)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0x05, 0x09,                    //     USAGE_PAGE (Button)
    0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
    0x29, 0x08,                    //     USAGE_MAXIMUM (Button 8)
    0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
    0x95, 0x08,                    //     REPORT_COUNT (8)
    0x75, 0x01,                    //     REPORT_SIZE (1)
    0x81, 0x02,                    //     INPUT (Data,Var,Abs)
    0xc0,                          //     END_COLLECTION
	0xc0   						   // END_COLLECTION	
	

};

/* USER CODE BEGIN PRIVATE_VARIABLES */
__ALIGN_BEGIN static uint8_t Mouse_ReportDesc_FS[USBD_MOUSE_REPORT_DESC_SIZE] __ALIGN_END =
{
	0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
	0x09, 0x02,                    // USAGE (Mouse)
	0xa1, 0x01,                    // COLLECTION (Application)
	0x09, 0x01,                    //   USAGE (Pointer)
	0xa1, 0x00,                    //   COLLECTION (Physical)
	0x05, 0x09,                    //     USAGE_PAGE (Button)
	0x19, 0x01,                    //     USAGE_MINIMUM (Button 1)
	0x29, 0x03,                    //     USAGE_MAXIMUM (Button 3)
	0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
	0x25, 0x01,                    //     LOGICAL_MAXIMUM (1)
	0x95, 0x03,                    //     REPORT_COUNT (3)
	0x75, 0x01,                    //     REPORT_SIZE (1)
	0x81, 0x02,                    //     INPUT (Data,Var,Abs)
	0x95, 0x01,                    //     REPORT_COUNT (1)
	0x75, 0x05,                    //     REPORT_SIZE (5)
	0x81, 0x03,                    //     INPUT (Cnst,Var,Abs)
	0x05, 0x01,                    //     USAGE_PAGE (Generic Desktop)
	0x09, 0x30,                    //     USAGE (X)
	0x09, 0x31,                    //     USAGE (Y)
	0x09, 0x38,                    //     USAGE (Wheel)
	0x15, 0x81,                    //     LOGICAL_MINIMUM (-127)
	0x25, 0x7f,                    //     LOGICAL_MAXIMUM (127)
	0x75, 0x08,                    //     REPORT_SIZE (8)
	0x95, 0x03,                    //     REPORT_COUNT (3)
	0x81, 0x06,                    //     INPUT (Data,Var,Rel)
	0xc0,                          //   END_COLLECTION
	0xc0    					   // END_COLLECTION
};


__ALIGN_BEGIN static uint8_t KEY_ReportDesc_FS[USBD_KEY_REPORT_DESC_SIZE] __ALIGN_END =
{
  /* USER CODE BEGIN 0 */
  0x05, 0x01, // USAGE_PAGE (Generic Desktop)
  0x09, 0x06, // USAGE (Keyboard)
  0xa1, 0x01, // COLLECTION (Application)
  0x05, 0x07, // USAGE_PAGE (Keyboard)
  0x19, 0xe0, // USAGE_MINIMUM (Keyboard LeftControl)
  0x29, 0xe7, // USAGE_MAXIMUM (Keyboard Right GUI)
  0x15, 0x00, // LOGICAL_MINIMUM (0)
  0x25, 0x01, // LOGICAL_MAXIMUM (1)
  0x75, 0x01, // REPORT_SIZE (1)
  0x95, 0x08, // REPORT_COUNT (8)
  0x81, 0x02, // INPUT (Data,Var,Abs)
  0x95, 0x01, // REPORT_COUNT (1)
  0x75, 0x08, // REPORT_SIZE (8)
  0x81, 0x03, // INPUT (Cnst,Var,Abs)
  0x95, 0x05, // REPORT_COUNT (5)
  0x75, 0x01, // REPORT_SIZE (1)
  0x05, 0x08, // USAGE_PAGE (LEDs)
  0x19, 0x01, // USAGE_MINIMUM (Num Lock)
  0x29, 0x05, // USAGE_MAXIMUM (Kana)
  0x91, 0x02, // OUTPUT (Data,Var,Abs)
  0x95, 0x01, // REPORT_COUNT (1)
  0x75, 0x03, // REPORT_SIZE (3)
  0x91, 0x03, // OUTPUT (Cnst,Var,Abs)
  0x95, 0x06, // REPORT_COUNT (6)
  0x75, 0x08, // REPORT_SIZE (8)
  0x15, 0x00, // LOGICAL_MINIMUM (0)
  0x25, 0xFF, // LOGICAL_MAXIMUM (255)
  0x05, 0x07, // USAGE_PAGE (Keyboard)
  0x19, 0x00, // USAGE_MINIMUM (Reserved (no event indicated))
  0x29, 0x65, // USAGE_MAXIMUM (Keyboard Application)
  0x81, 0x00, // INPUT (Data,Ary,Abs)
  /* USER CODE END 0 */
  0xC0    /*     END_COLLECTION	             */

};
```
2. 修改USBD_CUSTOM_HID_ItfTypeDef结构体及其实例USBD_CustomHID_fops_FS，可以看到其实这个结构体的前三个成员其实就是指向了以上三个报表描述符。

```c
typedef struct _USBD_CUSTOM_HID_Itf
{
	uint8_t                  *jReport;
	uint8_t                  *mReport;
    uint8_t                  *kReport;
    int8_t (* Init)(void);
    int8_t (* DeInit)(void);
    int8_t (* OutEvent)(uint8_t event_idx, uint8_t state);

} USBD_CUSTOM_HID_ItfTypeDef;

USBD_CUSTOM_HID_ItfTypeDef USBD_CustomHID_fops_FS =
{
  CUSTOM_HID_ReportDesc_FS,
  Mouse_ReportDesc_FS,
  KEY_ReportDesc_FS,	
  CUSTOM_HID_Init_FS,
  CUSTOM_HID_DeInit_FS,
  CUSTOM_HID_OutEvent_FS
};
```

3.  修改配置描述符集合。增加接口，端点等。

```c
__ALIGN_BEGIN static uint8_t USBD_CUSTOM_HID_CfgFSDesc[USB_CUSTOM_HID_CONFIG_DESC_SIZ] __ALIGN_END =
{
    0x09, /* bLength: Configuration Descriptor size */
    USB_DESC_TYPE_CONFIGURATION, /* bDescriptorType: Configuration */
    USB_CUSTOM_HID_CONFIG_DESC_SIZ,
    /* wTotalLength: Bytes returned */
    0x00,
    0x03,         /*bNumInterfaces: 1 interface*/
    0x01,         /*bConfigurationValue: Configuration value*/
    0x00,         /*iConfiguration: Index of string descriptor describing the configuration*/
    0x80,         /*bmAttributes: bus powered */
    0x32,         /*MaxPower 100 mA: this current is used for detecting Vbus*/

    //----------- Descriptor of Joystick interface ---------------------//
    /* 09 */
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x00,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x01,         /*bNumEndpoints*/
    0x03,         /*bInterfaceClass: CUSTOM_HID*/
    0x00,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x00,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0,            /*iInterface: Index of string descriptor*/
    //-----------HID Descriptor of Joystick  ---------------------//
    /* 18 */
    0x09,         /*bLength: CUSTOM_HID Descriptor size*/
    CUSTOM_HID_DESCRIPTOR_TYPE, /*bDescriptorType: CUSTOM_HID*/
    0x11,         /*bCUSTOM_HIDUSTOM_HID: CUSTOM_HID Class Spec release number*/
    0x01,
    0x00,         /*bCountryCode: Hardware target country*/
    0x01,         /*bNumDescriptors: Number of CUSTOM_HID class descriptors to follow*/
    0x22,         /*bDescriptorType*/
    USBD_CUSTOM_HID_REPORT_DESC_SIZE,/*wItemLength: Total length of Report descriptor*/
    0x00,
    //----------- Descriptor of Joystick endpoints ---------------------//
    /* 27 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    CUSTOM_HID_EPIN_ADDR,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    CUSTOM_HID_EPIN_SIZE, /*wMaxPacketSize: 2 Byte max */
    0x00,
    CUSTOM_HID_FS_BINTERVAL,          /*bInterval: Polling Interval */
    /* 34 */

    //-----------  Descriptor of Mouse interface    ---------------------//
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x01,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x01,         /*bNumEndpoints*/
    0x03,         /*bInterfaceClass: CUSTOM_HID*/
    0x01,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x02,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0,            /*iInterface: Index of string descriptor*/
    //-----------   Descriptor of Mouse CUSTOM_HID    ---------------------//
    /* 43 */
    0x09,         /*bLength: CUSTOM_HID Descriptor size*/
    CUSTOM_HID_DESCRIPTOR_TYPE, /*bDescriptorType: CUSTOM_HID*/
    0x11,         /*bCUSTOM_HIDUSTOM_HID: CUSTOM_HID Class Spec release number*/
    0x01,
    0x00,         /*bCountryCode: Hardware target country*/
    0x01,         /*bNumDescriptors: Number of CUSTOM_HID class descriptors to follow*/
    0x22,         /*bDescriptorType*/
    USBD_MOUSE_REPORT_DESC_SIZE,/*wItemLength: Total length of Report descriptor*/
    0x00,
    //-----------    Descriptor of Mouse endpoints    ---------------------//
    /* 52 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    MOUSE_HID_EPIN_ADDR,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    MOUSE_HID_EPIN_SIZE, /*wMaxPacketSize: 2 Byte max */
    0x00,
    CUSTOM_HID_FS_BINTERVAL,          /*bInterval: Polling Interval */
    /* 59 */
    //-----------  Descriptor of KeyBoard interface    ---------------------//
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x02,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x01,         /*bNumEndpoints*/
    0x03,         /*bInterfaceClass: CUSTOM_HID*/
    0x00,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x00,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0,            /*iInterface: Index of string descriptor*/
    //-----------   Descriptor of KeyBoard CUSTOM_HID    ---------------------//
    /* 68 */
    0x09,         /*bLength: CUSTOM_HID Descriptor size*/
    CUSTOM_HID_DESCRIPTOR_TYPE, /*bDescriptorType: CUSTOM_HID*/
    0x11,         /*bCUSTOM_HIDUSTOM_HID: CUSTOM_HID Class Spec release number*/
    0x01,
    0x00,         /*bCountryCode: Hardware target country*/
    0x01,         /*bNumDescriptors: Number of CUSTOM_HID class descriptors to follow*/
    0x22,         /*bDescriptorType*/
    USBD_KEY_REPORT_DESC_SIZE,/*wItemLength: Total length of Report descriptor*/
    0x00,
    //-----------    Descriptor of KeyBoard endpoints    ---------------------//
    /* 77 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    KEY_HID_EPIN_ADDR,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    KEY_HID_EPIN_SIZE, /*wMaxPacketSize: 2 Byte max */
    0x00,
    CUSTOM_HID_FS_BINTERVAL,          /*bInterval: Polling Interval */
    /* 84 */
};
```

   

4. 在usbd_conf.h修改最大接口数量为3（这一步非常关键）

```
#define USBD_MAX_NUM_INTERFACES     3
```



5. 根据接口索引修改获取描述符的请求:
   定位到USBD_CUSTOM_HID->USBD_CUSTOM_HID_Setup在USB_REQ_GET_DESCRIPTOR请求中通过req->wIndex增加报告描述符的请求；

```c
static uint8_t  USBD_CUSTOM_HID_Setup(USBD_HandleTypeDef *pdev,
                                      USBD_SetupReqTypedef *req)
{
  USBD_CUSTOM_HID_HandleTypeDef *hhid = (USBD_CUSTOM_HID_HandleTypeDef *)pdev->pClassData;
  uint16_t len = 0U;
  uint8_t  *pbuf = NULL;
  uint16_t status_info = 0U;
  uint8_t ret = USBD_OK;
  switch (req->bmRequest & USB_REQ_TYPE_MASK)
  { 
    case USB_REQ_TYPE_STANDARD:
      switch (req->bRequest)
      {
        case USB_REQ_GET_DESCRIPTOR:
          if (req->wValue >> 8 == CUSTOM_HID_REPORT_DESC)
          {			  
			  if (req->wIndex == 0)
			  {
				len = MIN(USBD_CUSTOM_HID_REPORT_DESC_SIZE, req->wLength);
				pbuf = ((USBD_CUSTOM_HID_ItfTypeDef *)pdev->pUserData)->jReport;				  
			  }else if (req->wIndex == 1){//mouse
				len = MIN(USBD_MOUSE_REPORT_DESC_SIZE, req->wLength);
				pbuf = ((USBD_CUSTOM_HID_ItfTypeDef *)pdev->pUserData)->mReport;	
			  
			  }else if (req->wIndex == 2){//
				len = MIN(USBD_KEY_REPORT_DESC_SIZE, req->wLength);
				pbuf = ((USBD_CUSTOM_HID_ItfTypeDef *)pdev->pUserData)->kReport;				  
			  }				  			  		  
          }
          else
          {
            if (req->wValue >> 8 == CUSTOM_HID_DESCRIPTOR_TYPE)
            {								
              pbuf = USBD_CUSTOM_HID_Desc;
              len = MIN(USB_CUSTOM_HID_DESC_SIZ, req->wLength);
            }
          }
          USBD_CtlSendData(pdev, pbuf, len);
          break;
  }
  return ret;
}
```

6. 为端点增加PAM：
   定位到MX_USB_DEVICE_Init->USBD_Init->USBD_LL_Init；在USBD_LL_Init函数中找到HAL_PCDEx_PMAConfig，通过该接口为EP（端点）配置PMA（Packet Buffer Memory Area ，即USB硬件缓冲区））

```c
  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , 0x00 , PCD_SNG_BUF, 0x40);
  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , 0x80 , PCD_SNG_BUF, 0x80);

  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , CUSTOM_HID_EPIN_ADDR , PCD_SNG_BUF, 0xC0);
  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , CUSTOM_HID_EPOUT_ADDR , PCD_SNG_BUF, 0x100);
  
  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , MOUSE_HID_EPIN_ADDR , PCD_SNG_BUF, 0x140);
  HAL_PCDEx_PMAConfig((PCD_HandleTypeDef*)pdev->pData , KEY_HID_EPIN_ADDR , PCD_SNG_BUF, 0x180); 
```

7. 打开和配置端点:
   定位到USBD_CUSTOM_HID->USBD_CUSTOM_HID_Init通过USBD_LL_OpenEP函数中和pdev->ep_in[EP_ADDR & 0xFU].is_used = 1U;打开并使能端点，同时将USBD_LL_PrepareReceive中第四个参数改为大（如果用到接收端点）；

```c
static uint8_t  USBD_CUSTOM_HID_Init(USBD_HandleTypeDef *pdev,uint8_t cfgidx)
{
  uint8_t ret = 0U;
  USBD_CUSTOM_HID_HandleTypeDef     *hhid;
  /* Open EP IN */
  USBD_LL_OpenEP(pdev, CUSTOM_HID_EPIN_ADDR, USBD_EP_TYPE_INTR,CUSTOM_HID_EPIN_SIZE);
  pdev->ep_in[CUSTOM_HID_EPIN_ADDR & 0xFU].is_used = 1U;
  /* Open EP OUT */
  USBD_LL_OpenEP(pdev, CUSTOM_HID_EPOUT_ADDR, USBD_EP_TYPE_INTR,CUSTOM_HID_EPOUT_SIZE);
  pdev->ep_out[CUSTOM_HID_EPOUT_ADDR & 0xFU].is_used = 1U;				 
  /* Open MOUSE EP IN */
  USBD_LL_OpenEP(pdev, MOUSE_HID_EPIN_ADDR, USBD_EP_TYPE_INTR,MOUSE_HID_EPIN_SIZE);
  pdev->ep_in[MOUSE_HID_EPIN_ADDR & 0xFU].is_used = 1U;	
  /* Open KEY EP IN */
  USBD_LL_OpenEP(pdev, KEY_HID_EPIN_ADDR, USBD_EP_TYPE_INTR,KEY_HID_EPIN_SIZE);
  pdev->ep_in[KEY_HID_EPIN_ADDR & 0xFU].is_used = 1U;	
  pdev->pClassData = USBD_malloc(sizeof(USBD_CUSTOM_HID_HandleTypeDef));
  if (pdev->pClassData == NULL)
  {
    ret = 1U;
  }
  else
  {
    hhid = (USBD_CUSTOM_HID_HandleTypeDef *) pdev->pClassData;

    hhid->state = CUSTOM_HID_IDLE;
    ((USBD_CUSTOM_HID_ItfTypeDef *)pdev->pUserData)->Init();

    /* Prepare Out endpoint to receive 1st packet */
    USBD_LL_PrepareReceive(pdev, CUSTOM_HID_EPOUT_ADDR, hhid->Report_buf,
                           USBD_CUSTOMHID_OUTREPORT_BUF_SIZE);
  }
  return ret;
}
```

8. 设置USB设备发生复位则实现端点关闭，释放内存。

```c
static uint8_t  USBD_CUSTOM_HID_DeInit(USBD_HandleTypeDef *pdev,
                                       uint8_t cfgidx)
{
  /* Close CUSTOM_HID EP IN */
  USBD_LL_CloseEP(pdev, CUSTOM_HID_EPIN_ADDR);
  pdev->ep_in[CUSTOM_HID_EPIN_ADDR & 0xFU].is_used = 0U;

  /* Close CUSTOM_HID EP OUT */
  USBD_LL_CloseEP(pdev, CUSTOM_HID_EPOUT_ADDR);
  pdev->ep_out[CUSTOM_HID_EPOUT_ADDR & 0xFU].is_used = 0U;

  /* Close MOUSE_HID EP IN */
  USBD_LL_CloseEP(pdev, MOUSE_HID_EPIN_ADDR);
  pdev->ep_in[MOUSE_HID_EPIN_ADDR & 0xFU].is_used = 0U;	
	
  /* Close JOYSTICK_HID EP IN */
  USBD_LL_CloseEP(pdev, KEY_HID_EPIN_ADDR);
  pdev->ep_in[KEY_HID_EPIN_ADDR & 0xFU].is_used = 0U;		

  /* FRee allocated memory */
  if (pdev->pClassData != NULL)
  {
    ((USBD_CUSTOM_HID_ItfTypeDef *)pdev->pUserData)->DeInit();
    USBD_free(pdev->pClassData);
    pdev->pClassData = NULL;
  }
  return USBD_OK;
}
```

至此，插上USB已经可以枚举成三个组合的HID了。

```c
MultiTimer timer1;
MultiTimer timer2;
MultiTimer timer3;
void SystemClock_Config(void);
typedef enum
{
	DEVICE_JOYSTICK=0,
	DEVICE_MOUSE,
	DEVICE_KEYBOARD
}DEVICE_TypeDef;
DEVICE_TypeDef myHid=DEVICE_JOYSTICK;
void RportTimer1Callback(MultiTimer* timer, void *userData)
{
	if(myHid==DEVICE_JOYSTICK)
	{
		Joystick_Handle();
	}else if(myHid==DEVICE_MOUSE){
		Mouse_Handle();		
	}else if(myHid==DEVICE_KEYBOARD){
		Key_Board_Handle();
	}    
    MultiTimerStart(timer, 10, RportTimer1Callback, userData);
}
void WS2812BTimer2Callback(MultiTimer* timer, void *userData)
{
	if(myHid==DEVICE_JOYSTICK)
	{
		WS_WriteAll_RGB(0xFF,0x00,0x00);
	}else if(myHid==DEVICE_MOUSE){
		WS_WriteAll_RGB(0x00,0xFF,0x00);		
	}else if(myHid==DEVICE_KEYBOARD){
		WS_WriteAll_RGB(0x00,0x00,0xFF);
	}	
    MultiTimerStart(timer, 100, WS2812BTimer2Callback, userData);
}
void DeviceSwitchTimer3Callback(MultiTimer* timer, void *userData)
{
	static u16 swtick=0;
	if((UPKEY)==0)
	{
		if(swtick++>300)
		{
			swtick=0;
			if(++myHid>DEVICE_KEYBOARD)
			{
				myHid=DEVICE_JOYSTICK;
			}		
		}
	}else{
		swtick=0;
	}   
    MultiTimerStart(timer, 10, DeviceSwitchTimer3Callback, userData);
}
int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MultiTimerInstall(PlatformTicksGetFunc);
    MX_GPIO_Init();
    MX_DMA_Init();
    MX_ADC1_Init();
    MX_USB_DEVICE_Init();
    MX_USART1_UART_Init();
    MX_TIM3_Init();
    Gp_ADC_Start_DMA();
    MultiTimerStart(&timer1, 5, RportTimer1Callback, NULL);
    MultiTimerStart(&timer2, 10, WS2812BTimer2Callback, NULL);
	MultiTimerStart(&timer3, 10, DeviceSwitchTimer3Callback, NULL);
    while (1)
    {
        MultiTimerYield();
    }
}

```



### 3.7.3 下载验证

我们把固件程序下载进去，摇动摇杆，按住SW2大于4s，可依次切换成鼠标模式成摇杆、鼠标、键盘模式，2812显示对应的红绿蓝色。





## 3.8 实例Eg8_Gamepad

本节目标是实现Gamepad，Gamepad和Joystick是有很大区别的，我们先看一下Joystick，如下图Joystick是带有XY轴并带有若干个按键的HID。


![image](https://img2023.cnblogs.com/blog/1966993/202212/1966993-20221228223710523-725267649.png)


而Gamepad除了XY轴和按键，还有Z轴，Rx和Ry（旋转），并带有视觉头盔。

![image](https://img2023.cnblogs.com/blog/1966993/202212/1966993-20221228223640742-1235345320.png)


本节的就是实现如上图这样一个Gamepad。

### 3.8.1硬件设计

参考原理图； 

### 3.8.2 软件设计

1. 准备Gamepad报表，根据报表定义，buf[0]和buf[1]分别是XY轴，buf[2]和buf[3]分别是Rx和Ry，而buf[4]代表着Z轴，而按钮是有10个所以占用了1~10bit；而视觉头盔是HatSwitch，4bit；另有2bit是无定义的，而报表还有2BYTE是保留字节。这样算下来一共定义了9个byte。

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
        0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)
        0x09, 0x05,                    // USAGE (Game Pad)
        0xa1, 0x01,                    // COLLECTION (Application)
        0xa1, 0x00,                    //   COLLECTION (Physical)
        0x09, 0x30,                    //     USAGE (X)
        0x09, 0x31,                    //     USAGE (Y)
        0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
        0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
        0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
        0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
        0x95, 0x02,                    //     REPORT_COUNT (2)
        0x75, 0x08,                    //     REPORT_SIZE (8)
        0x81, 0x02,                    //     INPUT (Data,Var,Abs)
        0xc0,                          //     END_COLLECTION
        0xa1, 0x00,                    //   COLLECTION (Physical)
        0x09, 0x33,                    //     USAGE (Rx)
        0x09, 0x34,                    //     USAGE (Ry)
        0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
        0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
        0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
        0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
        0x95, 0x02,                    //     REPORT_COUNT (2)
        0x75, 0x08,                    //     REPORT_SIZE (8)
        0x81, 0x02,                    //     INPUT (Data,Var,Abs)
        0xc0,                          // END_COLLECTION
        0xa1, 0x00,                    //   COLLECTION (Physical)
        0x09, 0x32,                    //     USAGE (Z)
        0x15, 0x00,                    //     LOGICAL_MINIMUM (0)
        0x26, 0xff, 0x00,              //     LOGICAL_MAXIMUM (255)
        0x35, 0x00,                    //     PHYSICAL_MINIMUM (0)
        0x46, 0xff, 0x00,              //     PHYSICAL_MAXIMUM (255)
        0x95, 0x01,                    //     REPORT_COUNT (1)
        0x75, 0x08,                    //     REPORT_SIZE (8)
        0x81, 0x02,                    //     INPUT (Data,Var,Abs)
        0xc0,                          // END_COLLECTION
        0x05, 0x09,                    //   USAGE_PAGE (Button)
        0x19, 0x01,                    //   USAGE_MINIMUM (Button 1)
        0x29, 0x0a,                    //   USAGE_MAXIMUM (Button 10)
        0x95, 0x0a,                    //   REPORT_COUNT (10)
        0x75, 0x01,                    //   REPORT_SIZE (1)
        0x81, 0x02,                    //   INPUT (Data,Var,Abs)
        0x05, 0x01,                    //   USAGE_PAGE (Generic Desktop)
        0x09, 0x39,                    //   USAGE (Hat switch)
        0x15, 0x01,                    //   LOGICAL_MINIMUM (1)
        0x25, 0x08,                    //   LOGICAL_MAXIMUM (8)
        0x35, 0x00,                    //   PHYSICAL_MINIMUM (0)
        0x46, 0x3b, 0x10,              //   PHYSICAL_MAXIMUM (4155)
        0x66, 0x0e, 0x00,              //   UNIT (None)
        0x75, 0x04,                    //   REPORT_SIZE (4)
        0x95, 0x01,                    //   REPORT_COUNT (1)
        0x81, 0x42,                    //   INPUT (Data,Var,Abs,Null)
        0x75, 0x02,                    //   REPORT_SIZE (2)
        0x95, 0x01,                    //   REPORT_COUNT (1)
        0x81, 0x03,                    //   INPUT (Cnst,Var,Abs)
        0x75, 0x08,                    //   REPORT_SIZE (8)
        0x95, 0x02,                    //   REPORT_COUNT (2)
        0x81, 0x03,                    //   INPUT (Cnst,Var,Abs)
        0xC0    					   // END_COLLECTION 
};
```

2. 我们在Eg5_KeyBoard这个实例基础上修改一下报表描述符即可，然后，我们需要对硬件摇杆和按键进行映射，主要映射XY轴，以及视觉头盔，并且映射button1~4。如下：

```c
void Gamepad_Handle(void)
{
    uint8_t Buf[9]={128,128,128,128,128,0,0,0,0};
    static uint8_t lastBuf[9]={0,0,0,0,0,0,0,0};

	//X-Y
    for(u8 i=0; i<AD_DATA_SIZE;)
	{
		AdXSum += AD_DATA[i];
		i++;
		AdYSum += AD_DATA[i];
		i++;
	}
	Xtemp=AdXSum/10;
	AdXSum=0;
	Ytemp=AdYSum/10;
	AdYSum=0;
	if(Xtemp>Xmax)
		Xtemp=Xmax;
	if(Xtemp<Xmin)
		Xtemp=Xmin;

	if(Ytemp>Ymax)
		Ytemp=Ymax;
	if(Ytemp<Ymin)
		Ytemp=Ymin;		


    Buf[1]=(uint8_t)map( Xtemp, Xmin, Xmax, 0, UINT8_MAX );
    Buf[0]=(uint8_t)map( Ytemp, Ymin, Ymax, 0, UINT8_MAX );
 
    //Hat
    if((UPKEY==0)&&(DNKEY==0)&&(LFKEY==0)&&(RGKEY!=0)) //0001
    {
        Buf[6]|=HATSW7;
    }else if((UPKEY==0)&&(DNKEY==0)&&(LFKEY!=0)&&(RGKEY==0))//0010
    {
        Buf[6]|=HATSW3;
    }else if((UPKEY==0)&&(DNKEY!=0)&&(LFKEY==0)&&(RGKEY==0))//0100
    {
        Buf[6]|=HATSW1;
    }else if((UPKEY==0)&&(DNKEY!=0)&&(LFKEY==0)&&(RGKEY!=0))//0101
    {
        Buf[6]|=HATSW8;
    }else if((UPKEY==0)&&(DNKEY!=0)&&(LFKEY!=0)&&(RGKEY==0))//0110
    {
        Buf[6]|=HATSW2;
    }else if((UPKEY==0)&&(DNKEY!=0)&&(LFKEY!=0)&&(RGKEY!=0))//0111
    {
        Buf[6]|=HATSW1;
    }else if((UPKEY!=0)&&(DNKEY==0)&&(LFKEY==0)&&(RGKEY==0))//1000
    {
        Buf[6]|=HATSW5;
    }else if((UPKEY!=0)&&(DNKEY==0)&&(LFKEY==0)&&(RGKEY!=0))//1001
    {
        Buf[6]|=HATSW6;
    }else if((UPKEY!=0)&&(DNKEY==0)&&(LFKEY!=0)&&(RGKEY==0))//1010
    {
        Buf[6]|=HATSW4;
    }else if((UPKEY!=0)&&(DNKEY==0)&&(LFKEY!=0)&&(RGKEY!=0))//1011
    {
        Buf[6]|=HATSW5;
    }else if((UPKEY!=0)&&(DNKEY!=0)&&(LFKEY==0)&&(RGKEY!=0))//1101
    {
        Buf[6]|=HATSW7;
    }else if((UPKEY!=0)&&(DNKEY!=0)&&(LFKEY!=0)&&(RGKEY==0))//1110
    {
        Buf[6]|=HATSW3;
    }else{
        Buf[6]&=(~0x3C);
    }	
	//Button
	 if((STKEY)==0)
    {
        Buf[5]|=BIT0;
    }else{
		Buf[5]&=(~BIT0);
	}
    if((MDKEY)==0)
    {
        Buf[5]|=BIT1;
    }else{
		Buf[5]&=(~BIT1);
	}
    if((BKKEY)==0)
    {
        Buf[5]|=BIT2;
    }else{
		Buf[5]&=(~BIT2);
	}
    if((TBKEY)==0)
    {
        Buf[5]|=BIT3;
    }else{
		Buf[5]&=(~BIT3);
	}

    if(memcmp(Buf,lastBuf,9)!=0)
    {
       USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Buf, 9);
    }

    memcpy(lastBuf,Buf,9);

}
```

### 3.8.3 下载验证

我们把固件程序下载进去，界面如下图，摇动摇杆，XY轴响应，按下左边4个按键可控制视觉头盔。板子中间四颗按键实现1~4按钮按下。

![image](https://img2023.cnblogs.com/blog/1966993/202212/1966993-20221228223601164-293653265.png)



## 3.9 实例Eg9_AbsoluteMouse

本节目标是实现绝对值鼠标，绝对值鼠标和相对鼠标是有很大区别的，下面就让我们一起探讨如何实现一个绝对值鼠标.

### 3.9.1硬件设计

参考原理图； 

### 3.9.2 软件设计

1. 准备绝对值鼠标报表，

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
		0x05, 0x01,		  // USAGE_PAGE (Generic Desktop)
		0x09, 0x02,		  // USAGE (Mouse)
		0xa1, 0x01,		  // COLLECTION (Application)
		0x09, 0x01,		  //   USAGE (Pointer)
		0xa1, 0x00,		  //   COLLECTION (Physical)
		0x05, 0x09,		  //     USAGE_PAGE (Button)
		0x19, 0x01,		  //     USAGE_MINIMUM (Button 1)
		0x29, 0x03,		  //     USAGE_MAXIMUM (Button 3)
		0x95, 0x03,		  //     REPORT_COUNT (3)
		0x75, 0x01,		  //     REPORT_SIZE (1)
		0x15, 0x00,		  //     LOGICAL_MINIMUM (0)
		0x25, 0x01,		  //     LOGICAL_MAXIMUM (1)
		0x81, 0x02,		  //     INPUT (Data,Var,Abs)
		0x95, 0x01,		  //     REPORT_COUNT (1)
		0x75, 0x05,		  //     REPORT_SIZE (5)
		0x81, 0x01,		  //     INPUT (Cnst,Ary,Abs)
		0x05, 0x01,		  //     USAGE_PAGE (Generic Desktop)
		0x09, 0x38,		  //     USAGE (Wheel)
		0x15, 0x81,		  //     LOGICAL_MINIMUM (-127)
		0x25, 0x7f,		  //     LOGICAL_MAXIMUM (127)
		0x95, 0x01,		  //     REPORT_COUNT (1)
		0x75, 0x08,		  //     REPORT_SIZE (8)
		0x81, 0x06,		  //     INPUT (Cnst,Ary,Abs)
		0x16, 0x00,0x00, 	//     LOGICAL_MINIMUM (0)
		0x26, 0x00,0x01,  //     LOGICAL_MAXIMUM (256)
		0x36, 0x00,0x00,	//     PHYSICAL_MINIMUM (0)
		0x46, 0x00,0x01, 	//     PHYSICAL_MAXIMUM (256)
		0x66, 0x00,0x00, //     UNIT (None)
		0x09, 0x30,		  //     USAGE (X)
		0x09, 0x31,		  //     USAGE (Y)
		0x75, 0x10,		  //     REPORT_SIZE (16)
		0x95, 0x02,		  //     REPORT_COUNT (2)
		0x81, 0x62,		  //     INPUT (Data,Var,Abs,NPrf,Null)
		0xc0,			  //     END_COLLECTION
		0xc0			  // END_COLLECTION
};
```

2. 根据报表定义，可以得到如下定义，buttons的bit0、bit1、bit2分别对应左键、右键、中键，而wheel代表着鼠标滚轮，最后x与y是定义XY轴的。

```c
typedef struct mouseHID_t
{
    uint8_t buttons;
    int8_t wheel;
    uint16_t x;
    uint16_t y;
} mouseHID_T;
```

3. 最后解析数据。

```c
void AbsoluteMouse_Handle(void)
{
    static uint16_t c_tick=0;
    memset(&MouseBuf, 0,sizeof(MouseBuf));
    //X-Y
    for(u8 i=0; i<AD_DATA_SIZE;)
    {
        AdXSum += AD_DATA[i];
        i++;
        AdYSum += AD_DATA[i];
        i++;
    }
    Xtemp=AdXSum/10;
    AdXSum=0;
    Ytemp=AdYSum/10;
    AdYSum=0;
    if(Xtemp>Xmax)
        Xtemp=Xmax;
    if(Xtemp<Xmin)
        Xtemp=Xmin;

    if(Ytemp>Ymax)
        Ytemp=Ymax;
    if(Ytemp<Ymin)
        Ytemp=Ymin;
    MouseBuf.y=(uint8_t)map( Xtemp, Xmin, Xmax, 0, 255 );
    MouseBuf.x=(uint8_t)map( Ytemp, Ymin, Ymax, 0, 255 );

    //Button
    if(LFKEY==0)
    {
        MouseBuf.buttons|=BIT0;
    } else {
        MouseBuf.buttons&=(~BIT0);
    }
    if(RGKEY==0)
    {
        MouseBuf.buttons|=BIT1;
    } else {
        MouseBuf.buttons&=(~BIT1);
    }
    if(SW1==0)
    {
        MouseBuf.buttons&=(~BIT2);
    } else {
        MouseBuf.buttons|=BIT2;
    }

    if((UPKEY)==0)
    {
        if(c_tick++>5)
        {
            MouseBuf.wheel=1;
            c_tick=0;
        }
    }
    if((DNKEY)==0)
    {
        if(c_tick++>5)
        {
            MouseBuf.wheel=-1;
            c_tick=0;
        }
    }
    if(lastMouse_Buffer.x!=MouseBuf.x||lastMouse_Buffer.y!=MouseBuf.y||
            lastMouse_Buffer.buttons!=MouseBuf.buttons||MouseBuf.wheel!=0)
    {
        USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&MouseBuf, sizeof(MouseBuf));
    }
    lastMouse_Buffer.x=MouseBuf.x;
    lastMouse_Buffer.y=MouseBuf.y;
    lastMouse_Buffer.buttons=MouseBuf.buttons;
    lastMouse_Buffer.wheel=MouseBuf.wheel;


}
```



### 3.9.3 下载验证

我们把固件程序下载进去，鼠标居中，按键滚轮基本同相对鼠标，而XY轴则是整个屏幕是作为XY轴的。

https://www.bilibili.com/video/BV1uR4y1D7Uy/?vd_source=2bbde87de845d5220b1d8ba075c12fb0





## 3.10 实例Eg10_Xinput

本节目标是实现实现Xbox 360 Controller for Windows.

### 3.10.1硬件设计

参考原理图； 

### 3.10.2 软件设计

1. **修改设备描述符**

```c

#define USBD_VID     0x045E
#define USBD_PID_FS     0x028E

__ALIGN_BEGIN uint8_t USBD_FS_DeviceDesc[USB_LEN_DEV_DESC] __ALIGN_END =

{
  0x12,                       /*bLength */
  USB_DESC_TYPE_DEVICE,       /*bDescriptorType*/
  0x00,                       /*bcdUSB */
  0x02,
  0x00,                       /*bDeviceClass*/
  0x00,                       /*bDeviceSubClass*/
  0x00,                       /*bDeviceProtocol*/
  USB_MAX_EP0_SIZE,           /*bMaxPacketSize*/
  LOBYTE(USBD_VID),           /*idVendor*/
  HIBYTE(USBD_VID),           /*idVendor*/
  LOBYTE(USBD_PID_FS),        /*idProduct*/
  HIBYTE(USBD_PID_FS),        /*idProduct*/
  0x00,                       /*bcdDevice rel. 2.00*/
  0x02,
  USBD_IDX_MFC_STR,           /*Index of manufacturer  string*/
  USBD_IDX_PRODUCT_STR,       /*Index of product string*/
  USBD_IDX_SERIAL_STR,        /*Index of serial number string*/
  USBD_MAX_NUM_CONFIGURATION  /*bNumConfigurations*/
};
```

**2.修改配置/接口/HID/端点/厂商描述符：**

```c
__ALIGN_BEGIN static uint8_t USBD_CUSTOM_HID_CfgFSDesc[USB_CUSTOM_HID_CONFIG_DESC_SIZ] __ALIGN_END =
{
    /************** Configuration Descriptor 1 Bus Powered, 500 mA ****************/
    0x09, /* bLength: Configuration Descriptor size */
    USB_DESC_TYPE_CONFIGURATION, /* bDescriptorType: Configuration */
    USB_CUSTOM_HID_CONFIG_DESC_SIZ,
    /* wTotalLength: Bytes returned */
    0x00,
    0x04,         /*bNumInterfaces: 1 interface*/
    0x01,         /*bConfigurationValue: Configuration value*/
    0x00,         /*iConfiguration: Index of string descriptor describing
  the configuration*/
    0xA0,         /*bmAttributes: Bus Powered, Remote Wakeup*/
    0xFA,         /*MaxPower 500 mA: this current is used for detecting Vbus*/

    /**************Interface Descriptor 0/0 Vendor-Specific, 2 Endpoints****************/
    /* 09 */
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x00,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x02,         /*bNumEndpoints*/
    0xFF,         /*Vendor-Specific*/
    0x5D,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x01,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0x00,            /*iInterface: Index of string descriptor*/
    /**************Unrecognized Class-Specific Descriptor***********************/
    /* 18 */
    0x11,         /*bLength: CUSTOM_HID Descriptor size*/
    0x21, /*bDescriptorType: CUSTOM_HID*/
    0x10,0x01,0x01,0x25,0x81,0x14,0x03,0x03,
    0x03,0x04,0x13,0x02,0x08,0x03,0x03,
    /**************Endpoint Descriptor 81 1 In, Interrupt, 4 ms******************/
    /* 27 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    CUSTOM_HID_EPIN_ADDR,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    CUSTOM_HID_EPIN_SIZE, /*wMaxPacketSize: 2 Byte max */
    0x00,
    CUSTOM_HID_FS_BINTERVAL,          /*bInterval: Polling Interval */
    /**************Endpoint Descriptor 02 2 Out, Interrupt, 8 ms******************/
    /* 34 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    0x02,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    0x20, /*wMaxPacketSize: 2 Byte max */
    0x00,
    0x08,          /*bInterval: Polling Interval */

    /**************Interface Descriptor 1/0 Vendor-Specific, 2 Endpoints****************/
    /* 09 */
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x01,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x02,         /*bNumEndpoints*/
    0xFF,         /*Vendor-Specific*/
    0x5D,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x01,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0x00,            /*iInterface: Index of string descriptor*/
    /**************Unrecognized Class-Specific Descriptor***********************/
    /* 18 */
    0x1B,         /*bLength: CUSTOM_HID Descriptor size*/
    0x21, /*bDescriptorType: CUSTOM_HID*/
    0x00,0x01,0x01,0x01,0x83,0x40,0x01,0x04,
    0x20,0x16,0x85,0x00,0x00,0x00,0x00,0x00,
    0x00,0x16,0x05,0x00,0x00,0x00,0x00,0x00,
    0x00,
    /**************Endpoint Descriptor 81 1 In, Interrupt, 4 ms******************/
    /* 27 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    0x83,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    0x20, /*wMaxPacketSize: 2 Byte max */
    0x00,
    0x02,          /*bInterval: Polling Interval */
    /**************Endpoint Descriptor 02 2 Out, Interrupt, 8 ms******************/
    /* 34 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    0x04,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    0x20, /*wMaxPacketSize: 2 Byte max */
    0x00,
    0x04,          /*bInterval: Polling Interval */

    /**************Interface Descriptor 2/0 Vendor-Specific, 2 Endpoints****************/
    /* 09 */
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x02,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x01,         /*bNumEndpoints*/
    0xFF,         /*Vendor-Specific*/
    0x5D,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x02,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0x00,            /*iInterface: Index of string descriptor*/
    /**************Unrecognized Class-Specific Descriptor***********************/
    /* 18 */
    0x09,         /*bLength: CUSTOM_HID Descriptor size*/
    0x21, /*bDescriptorType: CUSTOM_HID*/
    0x00,0x01,0x01,0x22,0x86,0x07,0x00,
    /**************Endpoint Descriptor 81 1 In, Interrupt, 4 ms******************/
    /* 27 */
    0x07,          /*bLength: Endpoint Descriptor size*/
    USB_DESC_TYPE_ENDPOINT, /*bDescriptorType:*/

    0x86,     /*bEndpointAddress: Endpoint Address (IN)*/
    0x03,          /*bmAttributes: Interrupt endpoint*/
    0x20, /*wMaxPacketSize: 2 Byte max */
    0x00,
    0x10,          /*bInterval: Polling Interval */

    /**************nterface Descriptor 3/0 Vendor-Specific, 0 Endpoints****************/
    /* 09 */
    0x09,         /*bLength: Interface Descriptor size*/
    USB_DESC_TYPE_INTERFACE,/*bDescriptorType: Interface descriptor type*/
    0x03,         /*bInterfaceNumber: Number of Interface*/
    0x00,         /*bAlternateSetting: Alternate setting*/
    0x00,         /*bNumEndpoints*/
    0xFF,         /*Vendor-Specific*/
    0xFD,         /*bInterfaceSubClass : 1=BOOT, 0=no boot*/
    0x13,         /*nInterfaceProtocol : 0=none, 1=keyboard, 2=mouse*/
    0x04,            /*iInterface: Index of string descriptor*/
    /**************Unrecognized Class-Specific Descriptor***********************/
    /* 18 */
    0x06,         /*bLength: CUSTOM_HID Descriptor size*/
    0x41, /*bDescriptorType: CUSTOM_HID*/
    0x00,0x01,0x01,0x03
};

```

**3. 添加厂商请求的处理。**

```c
static uint8_t  USBD_CUSTOM_HID_Setup(USBD_HandleTypeDef *pdev,
                                      USBD_SetupReqTypedef *req)
{
  USBD_CUSTOM_HID_HandleTypeDef *hhid = (USBD_CUSTOM_HID_HandleTypeDef *)pdev->pClassData;
  uint16_t len = 0U;
  uint8_t  *pbuf = NULL;
  uint16_t status_info = 0U;
  uint8_t ret = USBD_OK;
  switch (req->bmRequest & USB_REQ_TYPE_MASK)
  {
    case USB_REQ_TYPE_VENDOR:
        switch (req->bRequest)
        {
        case 0x01:
        {
            /* code */
            if(req->wLength == 0x14)
            {
                len  = MIN(0x14, req->wLength);
                pbuf = vendor_request_content;
            }
            else if(req->wLength == 0x08)
            {
                len  = MIN(0x8, req->wLength);
                pbuf = vendor_re2;
            }
            else if(req->wLength == 0x04)
            {
                len  = MIN(0x04, req->wLength);
                pbuf = vendor_re2;
            }
            break;
        }
        default:
            break;
        }
        USBD_CtlSendData(pdev, pbuf, len);
        break;
    default:
      USBD_CtlError(pdev, req);
      ret = USBD_FAIL;
      break;
  }
  return ret;
}
```

**4. 解析摇杆电位器和按键数据。**

```c
void Xinput_Handle(void)
{
	int16_t X=0,Y=0;
    //X-Y
    for(u8 i=0; i<AD_DATA_SIZE;)
    {
        AdXSum += AD_DATA[i];
        i++;
        AdYSum += AD_DATA[i];
        i++;
    }
    Xtemp=AdXSum/10;
    AdXSum=0;
    Ytemp=AdYSum/10;
    AdYSum=0;
    if(Xtemp>Xmax)
        Xtemp=Xmax;
    if(Xtemp<Xmin)
        Xtemp=Xmin;
    if(Ytemp>Ymax)
        Ytemp=Ymax;
    if(Ytemp<Ymin)
        Ytemp=Ymin;
    X=(int16_t)map( Xtemp, Xmin, Xmax, INT16_MAX, INT16_MIN );
    Y=(int16_t)map( Ytemp, Ymin, Ymax, INT16_MIN, INT16_MAX );
	//Button
    if((UPKEY)==0)//Y
    {
        TXData[BUTTON_PACKET_2] |= Y_MASK_ON;
    }else{
		TXData[BUTTON_PACKET_2] &= Y_MASK_OFF;
	}
    if((DNKEY)==0)//A
    {
        TXData[BUTTON_PACKET_2] |= A_MASK_ON;
    }else{
		TXData[BUTTON_PACKET_2] &= A_MASK_OFF;		
	}
    if(LFKEY==0)
    {
        TXData[BUTTON_PACKET_2] |= X_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_2] &= X_MASK_OFF;
    }
    if(RGKEY==0)
    {
        TXData[BUTTON_PACKET_2] |= B_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_2] &= B_MASK_OFF;
    }
	
    if(BKKEY==0)//BUTTON_BACK
    {
        TXData[BUTTON_PACKET_1] |= BACK_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_1] &= BACK_MASK_OFF;
    }
    if(MDKEY==0)//BUTTON_LB
    {
        TXData[BUTTON_PACKET_2] |= LB_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_2] &= LB_MASK_OFF;
    }
    if(STKEY==0)//BUTTON_START
    {
        TXData[BUTTON_PACKET_1] |= START_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_1] &= START_MASK_OFF;
    }	
    if(TBKEY==0)//BUTTON_RB
    {
        TXData[BUTTON_PACKET_2] |= RB_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_2] &= RB_MASK_OFF;
    }	
    if(SW1!=0)
    {
        TXData[BUTTON_PACKET_2] |= LOGO_MASK_ON;
    } else {
        TXData[BUTTON_PACKET_2] &= LOGO_MASK_OFF;
    }
	TXData[LEFT_STICK_X_PACKET_LSB] = LOBYTE(Y);		// (CONFERIR)
	TXData[LEFT_STICK_X_PACKET_MSB] = HIBYTE(Y);
	//Left Stick Y Axis
	TXData[LEFT_STICK_Y_PACKET_LSB] = LOBYTE(X);
	TXData[LEFT_STICK_Y_PACKET_MSB] = HIBYTE(X);
	//Clear DPAD
	TXData[BUTTON_PACKET_1] &= DPAD_MASK_OFF;
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&TXData, 20);
}
```



### 3.10.3 下载验证

我们把固件程序下载进去，Xbox 360 Controller for Windows出现在电脑上。





## 3.11 实例Eg11_Xinput01

本节目标还是实现Xbox 360 Controller for Windows.

### 3.10.1硬件设计

参考原理图； H2外拓摇杆

### 3.10.2 软件设计

本节在上一节的基础上修改了ADC和按键部分，具体请自行查看代码和视频

### 3.10.3 下载验证

我们把固件程序下载进去，Xbox 360 Controller for Windows出现在电脑上。





## 3.12 实例Eg12_MultiAxisButton

本节目标是实现多轴多按键摇杆

### 3.12.1硬件设计

参考原理图； 

### 3.12.2 软件设计

1. 准备摇杆报表，

```c
__ALIGN_BEGIN static uint8_t CUSTOM_HID_ReportDesc_FS[USBD_CUSTOM_HID_REPORT_DESC_SIZE] __ALIGN_END =
{
  /* USER CODE BEGIN 0 */
        0x05, 0x01,                                  // Usage Page (Generic Desktop)
        0x09, 0x04,                                  // Usage (Joystick)
        0xA1, 0x01,                                  // Collection (Application)
        0x05, 0x01,                                  // Usage Page (Generic Desktop)
        0x09, 0x01,                                  // Usage (Pointer)
        0xA1, 0x02,                                  // Collection (Logical)
        0x09, 0x30,                                  // Usage (X)
        0x09, 0x31,                                  // Usage (Y)
        0x09, 0x32,                                  // Usage (Z)
        0x09, 0x33,                                  // Usage (Rx)
        0x09, 0x34,                                  // Usage (Ry)
        0x09, 0x35,                                  // Usage (Rz)
        0x09, 0x36,                                  // Usage (Slider)
        0x09, 0x37,                                  // Usage (Dial)
        0x15, 0x00,                                  // Logical Minimum (0)
        0x26, 0xFF, 0x03,                            // Logical Maximum (1023)
        0x75, 0x10,                                  // Report Size (16)
        0x95, 0x08,                                  // Report Count (8)
        0x81, 0x02,                                  // Input (Data,Variable,Absolute)
        0x05, 0x09,                                  // Usage Page (Button)
        0x19, 0x01,                                  // Usage Minimum (Button 1)
        0x29, 0x20,                                  // Usage Minimum (Button 32)
        0x15, 0x00,                                  // Logical Minimum (0)
        0x25, 0x01,                                  // Logical Maximum (1)
        0x75, 0x01,                                  // Report Size (1)
        0x95, 0x20,                                  // Report Count (32)
        0x81, 0x02,                                  // Input (Data,Variable,Absolute)
        0xC0,                                        // End Collection
        0xA1, 0x02,                                  // Collection (Logical)
        0x95, 0x07,                                  // Report Count (7)
        0x75, 0x08,                                  // Report Size (8)
        0x09, 0x01,                                  // Usage (Button 1)
        0x91, 0x02,                                  // Output (Data,Variable,Absolute)
        0xC0,                                        // End Collection    

  /* USER CODE END 0 */
  0xC0    /*     END_COLLECTION	             */
};
```

2. 最后解析数据。

```c
//处理并上报数据
void Gp_SendReport(void)
{
	u8 Ktemp=0;
	u16 X=0,Y=0;
	for(u8 i=0; i<AD_DATA_SIZE;)
	{
		AdXSum += AD_DATA[i];
		i++;
		AdYSum += AD_DATA[i];
		i++;
	}
	Xtemp=AdXSum/10;
	AdXSum=0;
	Ytemp=AdYSum/10;
	AdYSum=0;
	if(Xtemp>Xmax)
		Xtemp=Xmax;
	if(Xtemp<Xmin)
		Xtemp=Xmin;

	if(Ytemp>Ymax)
		Ytemp=Ymax;
	if(Ytemp<Ymin)
		Ytemp=Ymin;		


    X=(uint16_t)map( Xtemp, Xmin, Xmax, 0, 1023 );
    Y=(uint16_t)map( Ytemp, Ymin, Ymax, 0, 1023 );
	Ktemp=key_scan();
	Joystick_Report[0]=Y;
	Joystick_Report[1]=Y>>8;
	Joystick_Report[2]=X;
	Joystick_Report[3]=X>>8;
	Joystick_Report[4]=Y;
	Joystick_Report[5]=Y>>8;
	Joystick_Report[6]=X;
	Joystick_Report[7]=X>>8;
	Joystick_Report[8]=Y;
	Joystick_Report[9]=Y>>8;
	Joystick_Report[10]=X;
	Joystick_Report[11]=X>>8;	
	Joystick_Report[12]=Y;
	Joystick_Report[13]=Y>>8;
	Joystick_Report[14]=X;
	Joystick_Report[15]=X>>8;
	Joystick_Report[16]=Ktemp;
	Joystick_Report[17]=Ktemp;
	Joystick_Report[18]=Ktemp;
	Joystick_Report[19]=Ktemp;
	USBD_CUSTOM_HID_SendReport(&hUsbDeviceFS,(u8*)&Joystick_Report, 20);	
}
```



### 3.12.3 下载验证

我们把固件程序下载进去出现多轴多按键。

# stm32 使用hal+spi驱动中景园屏幕
@[toc]
***
## 1.开发介绍：
- 硬件是中景园1.47英寸 172*320 屏幕
- stm32h750
- 是用stm32cubemx+hal库
- 开发环境 vscode+eide
- 编译器 ARMCC6

 ***

**屏幕引脚定义：**
| 引脚     |含义 |
| ----------- | ----------- |
| VDD      | 接到3V3      |
| GND   | 接地      |
| SCL   | 时钟线-接SCLK     |
| SDA   | 数据线-接MOSI      |
| CS  | SPI使能引脚，接普通IO口，SPI数据传输期间拉低      |
| RES  | st7789芯片复位线-接普通IO口，低电平复位，推荐上电拉低50ms再拉高      |
| D.C | DATA or CMD 根据引脚电平判断当前传输的是命令还是数据      |
| L.A | 背光模组 + ，常量接3v3，控制背光可接普通IO     |
| L.K | 背光模组 - ，接地      |
***
## 2.cubemx配置
**stm32H7时钟配置**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7fa890417f9742b1bdb2133f1ca89bae.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ3h0X2t0,size_20,color_FFFFFF,t_70,g_se,x_16)

**实测`30MBITS`能够正常显示，60MBITS就不行了**
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e86bd6f7f0348f881b83b240b1f7d18.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ3h0X2t0,size_20,color_FFFFFF,t_70,g_se,x_16)
**定义相关IO输出：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d9df8e4910241c0bf2d711965bce69d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAZ3h0X2t0,size_20,color_FFFFFF,t_70,g_se,x_16)
## 3.注释事项
源码基础中景园例程修改，主要修改内容可以往下看
***
原例程使用软件spi
我在保留原有软件spi的基础上加入硬件spi，使用宏定义HW_SPI进行修改
```c
// SW_SPI or HW_SPI or HW_SPI_DMA  （HW_SPI_DMA暂时还用不了，相关代码还没写）
#define HW_SPI
```
***
对相关GPIO进行了封装重写

```c
//==========================================
//GPIO-重写
//==========================================
#define LCD_RES_GPIO        GPIOC
#define LCD_RES_GPIO_PIN    GPIO_PIN_4
#define LCD_DC_GPIO         GPIOC
#define LCD_DC_GPIO_PIN     GPIO_PIN_5
#define LCD_CS_GPIO         GPIOB
#define LCD_CS_GPIO_PIN     GPIO_PIN_0
#define LCD_BLK_GPIO        GPIOB
#define LCD_BLK_GPIO_PIN    GPIO_PIN_1
```
***
用户还需要修改初始化部分代码，主要是不同屏幕初始化不同，和硬件有关，具体要看厂家初始化代码
初始化函数为

```c
void LCD_Init(void);//LCD初始化
```
移植时不要忘了修改
***
移植时还需要修改显示方式和宽高

```c
#define USE_HORIZONTAL 0  //设置横屏或者竖屏显示 0或1为竖屏 2或3为横屏

#if USE_HORIZONTAL==0||USE_HORIZONTAL==1
#define LCD_W 172
#define LCD_H 320
#else
#define LCD_W 320
#define LCD_H 172
#endif
```


***
## 4.源代码
源代码由于合并fonts到一个文件有点长
[https://github.com/gxt-kt/st7789-drv](https://github.com/gxt-kt/st7789-drv)
***
### 5.使用示例
```c
LCD_Init();
LCD_Fill(0,0,LCD_W,LCD_H,BLUE);
Draw_Circle(50,50,5,WHITE);
LCD_ShowString(10,10,(const uint8_t *)"TEST",YELLOW,GREEN,32,0);
```

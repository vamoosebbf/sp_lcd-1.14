<img src="img/icon_sipeed2.png" style="zoom: 80%;" />

# SP_LCD-1.14 Module User Guide

[中文](README_CN.md)

## Directory Structure
| Directory | Description                                   |
| :-------: | :-------------------------------------------- |
|    doc    | Reference documentation                       |
|    img    | Images                                        |
|  script   | Maixpy script example                         |
|    src    | C program example based on the standalone sdk |

## Introduce

<img src="img/sp_lcd.png" style="padding-right:100px;" align="right" width="" height="500" />

SP_LCD has a 1.14 'inch LCD, 4-line SPI interface control, 8P FPC(0.5mm interval) interface TFT LCD, 180° Angle of view, using SP_MOD connection.

*See [Module Specification](doc/SP-LCD1.14规格书V1.0.pdf) for more information.*

## Pin figure

<img src="img/back.png" width="300" />

## Mode of connection

<img src="img/connection.png" height="250">

|   MCU:FUN(IO)   | SP_EINK |
| :-------------: | :-----: |
|  GPIOHS7(IO_7)  |   RST   |
| GPIOHS15(IO_15) |   D/C   |
| SPI:SS0(IO_20)  |   CS    |
| SPI:SCK(IO_21)  |   SCK   |
| SPI:MOSI(IO_8)  |   SI    |
|  GPIOHS6(IO_6)  |   BL    |
|    2.5~4.8V     |  3.3V   |
|       GND       |   GND   |

## MCU configuration

### IO port configuration

Configure IO port corresponding to MCU as SPI function pin.

* C

  ```c
  fpioa_set_function(SPI_IPS_LCD_CS_PIN_NUM, FUNC_SPI1_SS0);   // SPI_IPS_LCD_CS_PIN_NUM: 20;
  fpioa_set_function(SPI_IPS_LCD_SCK_PIN_NUM, FUNC_SPI1_SCLK); // SPI_IPS_LCD_SCK_PIN_NUM: 21;
  fpioa_set_function(SPI_IPS_LCD_MOSI_PIN_NUM, FUNC_SPI1_D0);  // SPI_IPS_LCD_MOSI_PIN_NUM: 8;
  fpioa_set_function(SPI_IPS_LCD_DC_PIN_NUM, FUNC_GPIOHS0 + SPI_IPS_LCD_DC_GPIO_NUM);   // SPI_IPS_LCD_DC_PIN_NUM: 15; SPI_IPS_LCD_DC_GPIO_NUM: 15;
  fpioa_set_function(SPI_IPS_LCD_RST_PIN_NUM, FUNC_GPIOHS0 + SPI_IPS_LCD_RST_GPIO_NUM); // SPI_IPS_LCD_RST_PIN_NUM: 7; SPI_IPS_LCD_RST_GPIO_NUM: 7;
  fpioa_set_function(SPI_IPS_LCD_BL_PIN_NUM, FUNC_GPIOHS0 + SPI_IPS_LCD_BL_GPIO_NUM);   // SPI_IPS_LCD_BL_PIN_NUM: 6; SPI_IPS_LCD_BL_GPIO_NUM: 6;
  
  // set gpiohs work mode to output mode
  gpiohs_set_drive_mode(SPI_IPS_LCD_DC_GPIO_NUM, GPIO_DM_OUTPUT);
  gpiohs_set_drive_mode(SPI_IPS_LCD_RST_GPIO_NUM, GPIO_DM_OUTPUT);
  gpiohs_set_drive_mode(SPI_IPS_LCD_BL_GPIO_NUM, GPIO_DM_OUTPUT);
  ```
  
* MaixPy

  ```python
    # 20: SPI_LCD_CS_PIN_NUM;
    fm.register(SPI_LCD_CS_PIN_NUM, fm.fpioa.GPIOHS20, force=True)
    # 15: SPI_LCD_DC_PIN_NUM;
    fm.register(SPI_LCD_DC_PIN_NUM, fm.fpioa.GPIOHS15, force=True)
    # 6: SPI_LCD_BUSY_PIN_NUM;
    fm.register(SPI_LCD_BUSY_PIN_NUM, fm.fpioa.GPIOHS6, force=True)
    # 7: SPI_LCD_RST_PIN_NUM;
    fm.register(SPI_LCD_RST_PIN_NUM, fm.fpioa.GPIOHS7, force=True)

    # set gpiohs work mode to output mode
    cs = GPIO(GPIO.GPIOHS20, GPIO.OUT)
    dc = GPIO(GPIO.GPIOHS15, GPIO.OUT)
    busy = GPIO(GPIO.GPIOHS6, GPIO.OUT)
    rst = GPIO(GPIO.GPIOHS7, GPIO.OUT)
  ```
  
  Some of the pins are also configured during SPI initialization

### SPI initialization

* C

  ```c
  spi_init(1, SPI_WORK_MODE_0, SPI_FF_STANDARD, DATALENGTH, 0);
  ```

* MaixPy

  ```python
    # 21: SPI_LCD_SCK_PIN_NUM; 8: SPI_LCD_MOSI_PIN_NUM;
    spi1 = SPI(SPI_LCD_NUM, mode=SPI.MODE_MASTER, baudrate=SPI_LCD_FREQ_KHZ * 1000,
               polarity=0, phase=0, bits=8, firstbit=SPI.MSB, sck=SPI_LCD_SCK_PIN_NUM, mosi=SPI_LCD_MOSI_PIN_NUM)
  ```

## Usage

* Process

  1. Initialization
  2. Create an image an fill it
  3. Send the image data

* C

  ```c
    ips_lcd_init(); // init
    LCD_ShowPicture(0, 0, LCD_W, LCD_H, gImage_nanke); // display
  ```
  
* MaixPy

  It is mainly used to configure the SPI it needs. The width and height of the screen (240/135), IPS_MODE is used to set the direction of the screen, 0/1 is horizontal, and 2/3 is vertical.

  ```python
    # init
    ips = SpiIps(spi1, cs, dc, rst, busy, IPS_WIDTH, IPS_HEIGHT, IPS_MODE)

    # create an 'image' and fill it
    img = image.Image()
    img.draw_rectangle(80, 80, 30, 30)

    # display
    ips.display(img)
  ```

## Runtime environments

| Language | Boards   | SDK/Firmware version           |
| :------: | :------- | :----------------------------- |
|    C     | MaixCube | kendryte-standalone-sdk v0.5.6 |
|  MaixPy  | MaixCube | maixpy v0.5.1                  |

## Result

* C

  <img src="img/sp_lcd1.14_c.png" height="250" />

* MaixPy

  <img src="img/sp_lcd1.14_py.png" alt="sp_lcd1.14_py" height="250" />

## Transplant
  
The following parameters need to be modified.

* C

```c
  // board_config.h
  #define SPI_INDEX           1
  #define SPI_SCLK_RATE       600*1000
  #define SPI_CHIP_SELECT_NSS 0 // SPI_CHIP_SELECT_0

  #define SPI_IPS_LCD_CS_PIN_NUM      20
  #define SPI_IPS_LCD_SCK_PIN_NUM     21
  #define SPI_IPS_LCD_MOSI_PIN_NUM    8

  #define SPI_IPS_LCD_DC_PIN_NUM     15
  #define SPI_IPS_LCD_BL_PIN_NUM      6
  #define SPI_IPS_LCD_RST_PIN_NUM     7

  #define SPI_IPS_LCD_DC_GPIO_NUM     15
  #define SPI_IPS_LCD_BL_GPIO_NUM     6
  #define SPI_IPS_LCD_RST_GPIO_NUM    7

  #define USE_HORIZONTAL 3    // 0/1 is horizontal, and 2/3 is vertical.
```

* Maixpy

```python
  ################### config ###################
  SPI_LCD_NUM = SPI.SPI1
  SPI_LCD_DC_PIN_NUM = const(15)
  SPI_LCD_BUSY_PIN_NUM = const(6)
  SPI_LCD_RST_PIN_NUM = const(7)
  SPI_LCD_CS_PIN_NUM = const(20)
  SPI_LCD_SCK_PIN_NUM = const(21)
  SPI_LCD_MOSI_PIN_NUM = const(8)
  SPI_LCD_FREQ_KHZ = const(600)
  ##############################################
```

## LICENSE

See [LICENSE](LICENSE.md) file.

## Other information

| Version |   Editor   |   Date    |
| :-----: | :--------: | :-------: |
|  v1.0   | vamoosebbf | 2020.12.2 |

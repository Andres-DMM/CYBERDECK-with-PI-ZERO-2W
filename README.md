# ST7789 Raspberry Pi Console Mirror
- Turn an ST7789 ($320 \times 240$) SPI display into a live, color-inverted terminal monitor for Raspberry Pi OS (64-bit).
- This setup mirrors /dev/fb0 live onto the display using fbcat, automatically logs in on boot, and features enlarged console text for high readability.

## 1. Hardware WiringConnect your ST7789 display breakout to the Raspberry Pi Zero 2 W GPIO header:
- ST7789 Pin Label  Function               Raspberry Pi Pin    Physical Pin
- VCC3.3V          Power3V3                Power               Pin1 
- GND               Ground                  Ground              Pin 6
- SCL/SCK           SPI ClockSPI0            SCLK (GPIO 11)     Pin 23
- SDA/MOSISPI       Master OutSPI0           MOSI (GPIO 10)     Pin 19
- RES/RST           Reset                    3.3V Power         Pin 17
- DC                Data/Command             GPIO 25            Pin 22
- CS                Chip SelectSPI0          CE0 (GPIO 8)       Pin 24
- BLK/LED           Backlight                ControlGPIO 18     Pin 12

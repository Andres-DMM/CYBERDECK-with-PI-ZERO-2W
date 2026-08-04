# ST7789 Raspberry Pi Console Mirror
- Turn an ST7789 ($320 \times 240$) SPI display into a live, color-inverted terminal monitor for Raspberry Pi OS (64-bit).
- This setup mirrors /dev/fb0 live onto the display using fbcat, automatically logs in on boot, and features enlarged console text for high readability.

## 1. Hardware WiringConnect your ST7789 display breakout to the Raspberry Pi Zero 2 W GPIO header:
| ST7789 Pin Label | Function           | Raspberry Pi Pin      | Physical Pin |
|------------------|--------------------|-----------------------|--------------|
| VCC (3.3V)       | Power              | 3.3V                  | Pin 1        |
| GND              | Ground             | Ground                | Pin 6        |
| SCL / SCK        | SPI Clock          | GPIO 11 (SPI0 SCLK)   | Pin 23       |
| SDA / MOSI       | SPI Master Out     | GPIO 10 (SPI0 MOSI)   | Pin 19       |
| RES / RST        | Reset              | 3.3V Power            | Pin 17       |
| DC               | Data/Command       | GPIO 25               | Pin 22       |
| CS               | SPI Chip Select    | GPIO 8 (SPI0 CE0)     | Pin 24       |
| BLK / LED        | Backlight Control  | GPIO 18               | Pin 12       |

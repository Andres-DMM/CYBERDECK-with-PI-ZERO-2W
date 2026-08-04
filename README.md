# ST7789 Raspberry Pi Console Mirror
- Turn an ST7789 ($320 \times 240$) SPI display into a live, color-inverted terminal monitor for Raspberry Pi OS (64-bit).
- This setup mirrors /dev/fb0 live onto the display using fbcat, automatically logs in on boot, and features enlarged console text for high readability.

## Hardware Connections

| ST7789 Pin | Function | Raspberry Pi GPIO | Physical Pin |
|------------|----------|-------------------|--------------|
| VCC (3.3V) | Power | 3.3V | Pin 1 |
| GND | Ground | GND | Pin 6 |
| SCL / SCK | SPI Clock | GPIO 11 (SPI0 SCLK) | Pin 23 |
| SDA / MOSI | SPI Data | GPIO 10 (SPI0 MOSI) | Pin 19 |
| RES / RST | Reset | 3.3V | Pin 17 |
| DC | Data / Command | GPIO 25 | Pin 22 |
| CS | Chip Select | GPIO 8 (SPI0 CE0) | Pin 24 |
| BLK / LED | Backlight | GPIO 18 | Pin 12 |

---

# Software Setup

## 1. Enable SPI & Console Auto Login

Open Raspberry Pi configuration:

```bash
sudo raspi-config
```

### Enable SPI

Navigate to:

```
Interface Options
    └── SPI
            └── Yes
```

### Enable Console Auto Login

Navigate to:

```
System Options
    └── Boot / Auto Login
            └── Console Autologin
```

Select **Finish** and choose **No** when asked to reboot.

---

## 2. Force Framebuffer Output Without HDMI

Edit:

```bash
sudo nano /boot/firmware/cmdline.txt
```

At the end of the **single line**, add:

```text
video=HDMI-A-1:640x480@60e
```

Example:

```text
... rootwait video=HDMI-A-1:640x480@60e
```

Save with:

- Ctrl + O
- Enter
- Ctrl + X

---

## 3. Increase Console Font Size

Edit:

```bash
sudo nano /etc/default/console-setup
```

Modify these lines:

```text
FONTFACE="TerminusBold"
FONTSIZE="12x24"
```

Save and exit.

---

## 4. Install Dependencies

Update packages:

```bash
sudo apt update
```

Install required packages:

```bash
sudo apt install -y \
python3-pip \
python3-pil \
python3-spidev \
python3-rpi.gpio \
fbcat
```

Install the ST7789 Python library:

```bash
sudo pip3 install st7789 --break-system-packages
```

---

# Create the Display Mirror Script

Create the script:

```bash
nano /home/andypi/fbgrab_monitor.py
```

Paste the following code:

```python
import time
import subprocess
from io import BytesIO
from PIL import Image, ImageDraw, ImageOps
import st7789

# Initialize ST7789 display (320x240)
disp = st7789.ST7789(
    port=0,
    cs=0,
    dc=25,
    backlight=18,
    width=320,
    height=240,
    rotation=0,
    spi_speed_hz=60000000
)

disp.begin()

# Splash screen
splash = Image.new("RGB", (320, 240), (0, 0, 0))
draw = ImageDraw.Draw(splash)

draw.rectangle(
    (0, 0, 319, 239),
    outline=(0, 255, 0),
    fill=(0, 0, 100)
)

draw.text(
    (20, 100),
    "ST7789 DISPLAY ONLINE\nLoading Console...",
    fill=(255, 255, 255)
)

splash = ImageOps.invert(splash)

disp.display(splash)

time.sleep(2)

# Continuous framebuffer capture
while True:

    try:

        raw_ppm = subprocess.check_output(
            ["fbcat"],
            stderr=subprocess.DEVNULL
        )

        with Image.open(BytesIO(raw_ppm)) as img:

            img = img.convert("RGB").resize((320, 240))

            # Invert colors
            img = ImageOps.invert(img)

            disp.display(img)

    except Exception:
        pass

    # ~12 FPS
    time.sleep(0.08)
```

Save the file.

---

# Configure Auto Start

Create a systemd service:

```bash
sudo nano /etc/systemd/system/st7789-monitor.service
```

Paste:

```ini
[Unit]
Description=ST7789 Framebuffer Console Mirror
After=multi-user.target systemd-user-sessions.service

[Service]
Type=simple
User=root
WorkingDirectory=/home/andypi
ExecStart=/usr/bin/python3 /home/andypi/fbgrab_monitor.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Save and exit.

---

## Enable the Service

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable the service:

```bash
sudo systemctl enable st7789-monitor.service
```

Reboot:

```bash
sudo reboot
```

---

# Verification

After rebooting, the following sequence should occur:

1. The display backlight turns on.
2. The **ST7789 DISPLAY ONLINE** splash screen appears for approximately **2 seconds**.
3. The Raspberry Pi terminal begins mirroring to the display automatically.
4. The mirror updates at approximately **12 FPS**.
5. The system works without an HDMI monitor connected.

---

# Features

- SPI-driven 320×240 ST7789 display
- Automatic startup using systemd
- Live Linux framebuffer mirroring
- Console Auto Login support
- Works headless (no HDMI required)
- Automatic display refresh (~12 FPS)
- Inverted colors for improved visibility

---

## Project Structure

```
/home/andypi/
└── fbgrab_monitor.py

/etc/systemd/system/
└── st7789-monitor.service
```

---

## License

This project is provided as-is for educational and hobbyist use.

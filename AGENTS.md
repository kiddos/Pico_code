# AGENTS.md

Instructions and guidelines for AI agents working in the `Pico_code` repository.

---

## 1. Project Overview

`Pico_code` contains driver libraries, GUI utilities, and demonstration firmware for Waveshare's Raspberry Pi Pico (RP2040) display boards and expansion modules.

The repository provides dual implementations:
- **C Implementation (`c/`)**: Built on the official Raspberry Pi Pico C/C++ SDK and CMake. Targets bare-metal RP2040 firmware outputting `.uf2`, `.elf`, `.bin`, and `.hex` binaries.
- **MicroPython Implementation (`Python/`)**: Self-contained scripts using MicroPython's `machine` and `framebuf` modules for quick testing and prototyping.

### Supported Hardware
- **LCD Displays**: 0.96", 1.14", 1.3", 1.44", 1.54", 1.8", 2.0" (ST7735S, ST7789 drivers).
- **OLED Displays**: 1.3" (SH1107), 2.23" (SSD1305) supporting SPI and I2C modes.
- **Sensors & Peripherals**: ICM-20948 9-DOF IMU (accelerometer, gyroscope, magnetometer), onboard buttons, joysticks, and Infrared receiver.

---

## 2. Repository Layout

```
.
├── .clang-format                 # Google-based C/C++ formatting rules (2-space indent)
├── .gitignore                    # Ignores build/ directories
├── AGENTS.md                     # This agent guide
├── README.md                     # Bilingual introductory readme
├── Python/                       # MicroPython sample drivers and demos
│   ├── Pico-LCD-0.96/
│   ├── Pico-LCD-1.14/
│   ├── Pico-LCD-1.3/
│   ├── Pico-LCD-1.44/
│   ├── Pico-LCD-1.8/
│   ├── Pico-LCD-2/
│   ├── Pico-OLED-1.3/            # Includes both I2C and SPI scripts
│   └── Pico-OLED-2.23/           # Includes both I2C and SPI scripts
└── c/                            # Pico C/C++ SDK project
    ├── CMakeLists.txt            # Main CMake project file
    ├── main.c                    # Main entry point (uncomment the target demo)
    ├── pico_sdk_import.cmake     # Raspberry Pi Pico SDK loader
    ├── examples/                 # Board-specific demo routines (e.g. LCD_2in_test.c)
    └── lib/
        ├── Config/               # Hardware Abstraction Layer (GPIO, SPI, I2C, PWM, Delays)
        ├── Fonts/                # ASCII (Font8..Font24) and Chinese bitmap fonts
        ├── GUI/                  # Framebuffer drawing primitives (Paint engine)
        ├── Icm20948/             # ICM-20948 9-DOF sensor driver
        ├── Infrared/             # IR receiver driver
        ├── LCD/                  # Display controller drivers for LCD panels
        └── OLED/                 # Display controller drivers for OLED panels
```

---

## 3. Architecture & Key C Modules

### Hardware Abstraction Layer (`c/lib/Config/DEV_Config.[ch]`)
- Initializes GPIO, SPI (`spi1`), I2C (`i2c1`), and PWM channels.
- Provides standard primitives: `DEV_Digital_Write`, `DEV_Digital_Read`, `DEV_SPI_WriteByte`, `DEV_SPI_Write_nByte`, `DEV_I2C_Write`, `DEV_SET_PWM`, and delays (`DEV_Delay_ms`, `DEV_Delay_us`).
- Handles lifecycle via `DEV_Module_Init()` and `DEV_Module_Exit()`.

### Graphic Engine (`c/lib/GUI/GUI_Paint.[ch]`)
- Manages an in-memory frame buffer (`PAINT Paint`) with coordinate transformation, rotation (0, 90, 180, 270), and mirroring.
- Supported primitives: points, lines, rectangles, circles, strings (ASCII + Chinese), numbers, and bitmaps.
- Colors are defined in RGB565 format (16-bit) or 1-bit monochrome for OLEDs.

### Switching Demos in C
In `c/main.c`, only one demo function should be uncommented at any time (e.g., `LCD_2in_test()`). When adding or testing a new feature for a specific board, update `c/main.c` accordingly before building.

---

## 4. Hardware Pinout Reference

Default wiring for standard Waveshare Pico display boards:

| Signal | RP2040 GPIO Pin | Function |
|---|---|---|
| **MOSI / DIN** | GP11 | SPI1 TX |
| **CLK / SCK** | GP10 | SPI1 SCK |
| **CS** | GP9 | Chip Select (Active Low) |
| **DC** | GP8 | Data / Command Control |
| **RST** | GP12 | Hardware Reset (Active Low) |
| **BL** | GP13 | Backlight Control (PWM capable) |
| **SDA** | GP6 | I2C1 SDA (for I2C OLED / Sensors) |
| **SCL** | GP7 | I2C1 SCL (for I2C OLED / Sensors) |
| **KEY0 / Key A** | GP15 | User input button |
| **KEY1 / Key B** | GP17 | User input button |
| **KEY2 / Key X** | GP2 | User input button |
| **KEY3 / Key Y** | GP3 | User input button |

*Note: For joystick-equipped boards (e.g. 1.3" LCD), directional pins are typically UP=GP2, DOWN=GP18, LEFT=GP16, RIGHT=GP20, CTRL=GP3, A=GP15, B=GP17, X=GP19, Y=GP21.*

---

## 5. Development & Build Commands

### Prerequisites
- `cmake` (>= 3.12)
- `arm-none-eabi-gcc` and `arm-none-eabi-newlib`
- `make`
- `PICO_SDK_PATH` environment variable pointing to the cloned `pico-sdk` (defaults to `/home/kiddos/projects/pico-sdk` on this workstation).

### Building the C Firmware

Run from the repository root:

```bash
# 1. Configure CMake build directory
cmake -B c/build -S c

# 2. Compile the binary
make -C c/build -j$(nproc)
```

The build generates firmware outputs in `c/build/`:
- `main.uf2`: Drag-and-drop binary for Raspberry Pi Pico in BOOTSEL mode.
- `main.elf`: ELF binary for debugger/OpenOCD/gdb.
- `main.bin`, `main.hex`: Raw flash images.

To clean up build artifacts:
```bash
rm -rf c/build
```

### Formatting Code

Follow `.clang-format` (Google style, 2 spaces indentation):
```bash
clang-format -i c/main.c c/lib/**/*.c c/lib/**/*.h c/examples/**/*.c c/examples/**/*.h
```

### MicroPython Workflow
1. Flash official MicroPython UF2 to the Pico.
2. Transfer or run scripts under `Python/` using Thonny or CLI tools:
   ```bash
   mpremote run Python/Pico-LCD-2/Pico-LCD-2.py
   ```

---

## 6. Important Caveats for Agents

1. **Case Sensitivity in Headers**:
   - Header filenames must match case exactly on Linux (e.g., `#include "LCD_2in.h"`, not `#include "LCD_2IN.h"`).
2. **File Encoding & Character Sets**:
   - Several legacy files under `c/examples/` and `c/lib/` contain Chinese comments encoded in `ISO-8859-1` / `GB2312` / `GBK`.
   - Take care when editing these files with automated tools so non-UTF8 strings and comments are not corrupted.
3. **RP2040 Memory Limitations**:
   - RP2040 has 264 KB on-chip SRAM.
   - A full-screen 16-bit RGB565 framebuffer for a 320x240 display requires `320 * 240 * 2 = 153,600 bytes` (> 58% total RAM). Avoid allocating multiple large framebuffers concurrently.
4. **Tool Preferences on this Machine**:
   - Prefer `fd` over `find`.
   - Prefer `rg` over `grep`.
   - Prefer `bat` over `cat`.
   - Prefer `sd` over `sed`.
   - Prefer `xcp` over `cp`.
5. **Testing Verification**:
   - When modifying C code or headers, always run `cmake -B c/build -S c && make -C c/build -j$(nproc)` to verify that all library targets and `main` compile cleanly without warnings or link failures.

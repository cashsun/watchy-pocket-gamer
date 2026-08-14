# Watchy Project Context

## Project Overview
**Watchy Pocket Gamer** is a customizable open-source smartwatch based on the ESP32-S3 microcontroller. It's part of the larger Watchy ecosystem by SQFMI. This project implements the core firmware and provides example watchface implementations that users can customize.

**Repository Reference**: [sqfmi/watchy-docs](https://github.com/sqfmi/watchy-docs)

## Hardware Components

### Microcontroller
- **ESP32-S3** (v3.0): Dual-core processor with WiFi/BLE capabilities
  - **USB**: Built-in CDC/JTAG (no external USB-Serial needed)
  - **Bootload/Reset**: Buttons (not DTR/RTS)
- **ESP32-PICO-D4** (v1.5-v2.0): Earlier variant with external CP2102/CP2104 serial chip

### Real-Time Clock (RTC)
- **v3.0**: External 32KHz Crystal (higher accuracy)
- **v1.5-v2.0**: PCF8563 I2C RTC chip
- **v1.0**: DS3231 I2C RTC chip
- Watchy library: Alarm2 triggers every minute to wake ESP32 from deep sleep
- I2C connected via Wire library

### Display
- **Specs**: E-ink/E-paper display (200x200 pixels, black & white only)
- **v3.0 Model**: GDEY0154D67 (high contrast)
- **v1.5-v2.0 Model**: GDEH0154D67
- **Connector**: AFC07-S24ECC-00 FPC ribbon cable
  - Gold pins must face UP
  - Pull black tabs before inserting cable, push back to secure
- Driver: GxEPD2 Arduino library with SPI communication
- Drawing via Adafruit GFX API: `display.drawRect()`, `display.drawBitmap()`, `display.println()`

### Power / Battery
- Battery: 3.7V LiPo, 200mAh (402030 form factor); regulator ME6211C33M5G-N LDO
- Baseline life: 5-7 days timekeeping only; 2-3 days with WiFi data fetching
- ESP32 wakes every 60s (RTC alarm) to update display, then deep sleeps — `init()` handles this automatically
- Coding rules to preserve battery:
  - Turn off WiFi/BLE radios right after use — don't leave them on
  - Call `display.hibernate()` after display updates (automatic inside standard Watchy class flow; must call manually if bypassing it)
  - Skip BMA423 accelerometer init if watchface doesn't need step counting/gestures
  - RTC alarm interval can be changed beyond 60s for watchfaces that don't need per-minute updates (e.g. word clocks)

### Key Sensors & Modules
- **BMA423**: 6-axis accelerometer (motion detection, step counting)
  - Files: `src/bma.cpp`, `src/bma.h`, `src/bma4.c`, `src/bma4.h`, `src/bma423.c`, `src/bma423.h`
  - Config: `src/bma4_defs.h`
  
- **Display**: E-ink/E-paper display (exact specs in `src/Display.cpp`)
  - Files: `src/Display.cpp`, `src/Display.h`
  - Font assets: `src/DSEG7_Classic_Bold_53.h`

- **RTC (Real-Time Clock)**: Timekeeping module
  - Files: `src/WatchyRTC.cpp`, `src/WatchyRTC.h`
  - 32K variant: `src/Watchy32KRTC.cpp`, `src/Watchy32KRTC.h`

- **Bluetooth Low Energy (BLE)**:
  - Files: `src/BLE.cpp`, `src/BLE.h`
  - Used for companion app communication

## Directory Structure

### `/src/` - Core Library
Main library code compiled into the firmware:
- `Watchy.cpp / Watchy.h` - Main Watchy class and core functionality
- `Display.cpp / Display.h` - Display driver and rendering
- `BLE.cpp / BLE.h` - Bluetooth Low Energy implementation
- `WatchyRTC.cpp / WatchyRTC.h` - Real-time clock driver
- `bma*.{cpp,h,c}` - Accelerometer/BMA sensor drivers
- `config.h` - Configuration defines

### `/examples/WatchFaces/` - Example Implementations
Complete watchface examples with their own custom rendering logic:

| Watchface | Files | Features |
|-----------|-------|----------|
| **7_SEG** | `7_SEG.ino`, `Watchy_7_SEG.cpp/h` | 7-segment display, multiple fonts, retro look |
| **Basic** | `Basic.ino` | Minimal example, good starting point |
| **DOS** | `DOS.ino`, `Watchy_DOS.cpp/h` | IBM BIOS font, terminal aesthetic |
| **MacPaint** | `MacPaint.ino`, `Watchy_MacPaint.cpp/h` | Classic Mac aesthetic |
| **Mario** | `Mario.ino`, `Watchy_Mario.cpp/h` | Game Boy-style graphics |
| **Pokemon** | `Pokemon.ino`, `Watchy_Pokemon.cpp/h` | Pokémon-themed display |
| **StarryHorizon** | `StarryHorizon.ino`, `stars.h` | Animated stars and horizon |
| **Tetris** | `Tetris.ino`, `Watchy_Tetris.cpp/h` | Playable Tetris game |

Each watchface has:
- `settings.h` - Configuration for that watchface
- Custom font files (`.h`) if needed
- Custom asset files (graphics, sprites)

### `/extras/` - Additional Resources
- `WatchFaces/index.json` - Metadata for watchface discovery/catalog

## Build System & Setup

### Arduino IDE Configuration
1. Download latest [Arduino IDE](https://www.arduino.cc/en/software)
2. File > Preferences > Additional Board Manager URLs:
   - `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
3. Tools > Board > Boards Manager > Install **esp32 by Espressif Systems** (NOT Arduino ESP32 Boards)
4. Sketch > Include Library > Manage Libraries > Install **Watchy** (latest version)
5. Install all dependencies: GxEPD2, WiFiManager, rtc_pcf8563, etc.

### Board Settings for Upload
- **Board**: ESP32 Arduino > ESP32S3 Dev Module
- **Flash Size**: 8MB (64Mb)
- **Partition Scheme**: 8M with spiffs
- Leave everything else as default

### Bootloader Mode (for firmware upload)
1. Plug USB into Watchy
2. Press & hold top 2 buttons (Back & Up) for 4+ seconds
3. Release Back button first, then Up button
4. ESP32S3 device should enumerate a serial port (COM/cu.*)
5. Upload firmware via Arduino IDE

### Reset Watchy
1. Press & hold top 2 buttons (Back & Up) for 4+ seconds
2. Release Up button first, then Back button
3. Wait a few seconds for boot and screen refresh

### Flashing Notes
- Use USB **data cable** (not charge-only)
- Try different USB ports if serial port not found
- After upload, reset device to run new firmware

- **Language**: Arduino C++ (compatible with Arduino IDE, PlatformIO)
- **Build Artifacts**: Compiled firmware for ESP32-S3
- Format script: `src/format.sh` - Code formatting tool

## Key Concepts

### Watchfaces
Custom watchface implementations inherit from or follow the Watchy pattern:
1. Extend main Watchy class or implement display update functions
2. Override `drawWatchFace()` — the one required method, called each wake cycle
3. Handle button inputs for interactions
4. Manage display updates efficiently (e-paper refresh is slow)
5. Each watchface is a complete `.ino` sketch with supporting headers

Drawing API available on the `display` object inside `drawWatchFace()`:
- `display.setFont()` — select typeface
- `display.setCursor(x, y)` — position text
- `display.print()` / `display.println()` — render text
- `display.drawBitmap(x, y, array, width, height, color)` — render images
- `display.drawRect()` — shapes
- Current time is available via a `currentTime` struct (`.Hour`, `.Minute`, etc.)

External tools for watchface dev (see `docs/create-watchface`):
- **Watchy Watchface Designer** — drag-and-drop web tool, live preview, generates code
- **WatchySim** — simulator for testing without flashing hardware
- **image2cpp** — converts images to byte arrays for `drawBitmap`
- **truetype2gfx** — converts TTF fonts to GFX font headers

### Configuration Pattern
Each example uses a `settings.h` file containing:
- Display resolution constants
- Pin mappings
- Font selections
- Behavior tuning

### Hardware Features
- **Low Power**: E-paper display uses minimal power, RTC keeps time during sleep
- **Motion Sensing**: BMA423 enables gesture recognition and step counting
- **Wireless**: BLE for companion app connectivity
- **User Interaction**: Buttons for time setting, mode switching, etc.

## Development Workflow

1. **Creating a new watchface**: Copy an example (e.g., `Basic/`) and modify
2. **Editing display logic**: Update the `.cpp` and `.h` files for your watchface
3. **Configuration**: Adjust `settings.h` for your design
4. **Compilation**: Use Arduino IDE or PlatformIO with ESP32-S3 board support
5. **Testing**: Compile and flash to device via USB

## File Dependencies

### Core Library Dependencies
- `Watchy.h` is the main header to include in watchface sketches
- Display operations require `Display.h`
- Time operations use `WatchyRTC.h`
- Motion features use BMA headers

### Example Watchface Pattern
```cpp
#include "Watchy.h"
#include "settings.h"

class WatchyCustom : public Watchy {
public:
  void drawWatchFace();
};
```

## Important Config Files
- `src/config.h` - Core library configuration
- `library.properties` - PlatformIO/Arduino library metadata
- `library.json` - Library configuration (dependencies, etc.)

## Documentation Resources
- Full docs site: https://watchy.sqfmi.com/docs (source: [sqfmi/watchy-docs](https://github.com/sqfmi/watchy-docs))
  - `/docs/getting-started` — assembly, Arduino setup, WiFi captive-portal config (192.168.4.1)
  - `/docs/libs` — library/API reference (GxEPD2, DS3232RTC, BMA423, WiFiManager, Arduino_JSON)
  - `/docs/create-watchface` — watchface dev guide, design tools
  - `/docs/battery-life` — power optimization
  - `/docs/hardware` — pinout/pin map lives in the library's `config.h`, not the docs page itself; revision comparison table
  - `/docs/faqs` — troubleshooting
  - `/docs/legacy`, `/docs/3D`, `/docs/license`
- Watchface community gallery: https://watchy.sqfmi.com/watchfaces

## Known Gotchas
- **GxEPD2 display ghosting/static**: requires GxEPD2 library **v1.2.16+** (fixes GDEH0154D67 driver bug); also check FPC cable is fully seated with lock engaged
- **"library DS3232RTC claims to run on avr architecture(s)..."** compiler warning is expected/harmless on ESP32 builds
- **esptool failures on macOS Big Sur**: known issue, see [espressif/arduino-esp32#4408](https://github.com/espressif/arduino-esp32/issues/4408)
- Screen removal: never pry glass or use a heat gun (>60°C damages it) — use dental floss technique instead

## Common Tasks

**Adding a new watchface**: Create new folder in `examples/WatchFaces/`, copy structure from `Basic/`, customize
**Modifying display**: Edit `src/Display.cpp` for core rendering or individual watchface `.cpp` files
**Adding sensors**: Extend BLE or BMA drivers in `src/`
**Building**: PlatformIO with ESP32-S3 target or Arduino IDE with appropriate board selection

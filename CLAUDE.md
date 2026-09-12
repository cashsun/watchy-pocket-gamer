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
| **StarField** *(default — ships pre-flashed on the hardware)* | `StarField.ino`, `Watchy_7_SEG.cpp/h`, `Dusk2Dawn.cpp/h`, `moonPhaser.cpp/h`, `icons.h` | HUD-style 7-seg; dusk/dawn solar arc, moon phase, step count, battery, WiFi. Vendored from [Prokuon/watchy-starfield](https://github.com/Prokuon/watchy-starfield). Class name is still `Watchy7SEG`. BACK = dark/light, UP/DOWN = 12/24h. Set `#define LOC lat, lon, tz` in `Watchy_7_SEG.cpp` for correct sun times. |
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
- Use USB **data cable** (not charge-only) — symptom of a charge-only/bad cable: device LED stays off, or port enumerates but chip never responds (see arduino-cli notes below); swapping cables fixes it
- Try different USB ports if serial port not found
- After upload, reset device to run new firmware

### Flashing via arduino-cli (verified working, no Arduino IDE GUI)
1. Install: `brew install arduino-cli`
2. Config + ESP32 core:
   ```
   arduino-cli config init
   arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   arduino-cli core update-index
   arduino-cli core install esp32:esp32
   ```
3. Registry libs: `arduino-cli lib install "Adafruit GFX Library" "Arduino_JSON" "DS3232RTC" "NTPClient"`
4. Git-only libs (clone into `~/Documents/Arduino/libraries/`): `GxEPD2` (ZinggJM/GxEPD2, master is fine, v1.6.9+ has the ghosting fix), `WiFiManager` (tzapu/WiFiManager, master fine). **`Rtc_Pcf8563` (orbitalair/Rtc_Pcf8563) must be pinned to tag `1.0.3`** — master HEAD adds a `WireBase&` constructor overload that doesn't compile against ESP32 core 3.x's `Wire` (`WireBase does not name a type`). `git checkout 1.0.3` after cloning.
5. Symlink this repo into the libraries dir so sketches can `#include <Watchy.h>`: `ln -s /path/to/watchy-pocket-gamer ~/Documents/Arduino/libraries/Watchy`
6. **Use the built-in `esp32:esp32:watchy` board FQBN**, not a generic `ESP32S3 Dev Module`/`esp32:esp32:esp32s3` FQBN — it exists in the esp32 core's `boards.txt` and sets correct flash size/partitions/pins automatically. It only actually matches **v3.0 (ESP32-S3)** hardware; for pre-v3.0 boards (ESP32-PICO-D4) you must also pick the revision (step 7).
7. **Pre-v3.0 boards need an explicit hardware revision**, or `config.h` silently defaults to `ARDUINO_WATCHY_V20` (with only a compile-time `#pragma message` warning, easy to miss in build output). Getting this wrong compiles and flashes fine but **breaks the UP button** (pin 32 on v1.0/v1.5 vs pin 35 on v2.0 — see Known Gotchas). Pass it via FQBN option: `esp32:esp32:watchy:Revision=v20` (or `v10`, `v15`). No way to auto-detect revision from the chip — ask the user or try v2.0 first (most common) and verify the UP button works before assuming it's correct.
8. Compile: `arduino-cli compile --fqbn "esp32:esp32:watchy:Revision=v20" examples/WatchFaces/<Name>`
9. Identify the port: pre-v3.0 boards with a CP2102/CP2104 adapter show up as `/dev/cu.usbserial-*` (macOS), not `/dev/cu.usbmodem-*` — the latter is the native-USB-CDC naming used by true ESP32-S3 (v3.0) boards. Don't assume port type from the CLAUDE.md hardware table alone — check `ls /dev/cu.*` and match against which cable/device is actually live (`ioreg`/`system_profiler` can be sandboxed and return nothing — the port list itself plus asking the user "which cable/LED is on" is more reliable).
10. Upload: `arduino-cli upload -p /dev/cu.usbserial-XXXX --fqbn "esp32:esp32:watchy:Revision=v20,UploadSpeed=115200" examples/WatchFaces/<Name>`. **If upload fails right after "Changing baud rate to 921600... Changed." with "Unable to verify flash chip connection" / "No more data to read from serial port"**, the adapter/cable can't sustain 921600 baud reliably — drop to `UploadSpeed=115200` (slower, ~73s vs a few seconds, but reliable).
11. Boards with a CP2102/CP2104 adapter (pre-v3.0) auto-enter bootloader via DTR/RTS toggling during upload — **no manual button hold needed**, unlike native-USB-CDC v3.0 boards where DTR/RTS doesn't exist and the manual Back+Up button sequence is mandatory.

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
- **UP button dead / doesn't work on watchface or in settings menu**: hardware-revision mismatch in `config.h`. `UP_BTN_PIN` is **32** on v1.0/v1.5 but **35** on v2.0 — if `config.h` isn't told which revision (no `ARDUINO_WATCHY_V10/15/20` define set), it silently defaults to v2.0 behavior via a `#pragma message` warning that's easy to miss in build output. Fix: explicitly build with the matching `Revision=v10|v15|v20` FQBN option (see arduino-cli flashing notes above) and reflash. Back/Down/Menu buttons are wired the same across v1.0-v2.0, so only UP is affected.
- **`Rtc_Pcf8563` library master branch doesn't compile on ESP32 core 3.x**: `error: 'WireBase' does not name a type`. Pin to git tag `1.0.3`, not `master`/HEAD.
- **arduino-cli upload dies right after baud-rate switch to 921600** ("Unable to verify flash chip connection" / "No more data to read from serial port"): the USB-serial adapter/cable can't sustain that speed. Set `UploadSpeed=115200` in the FQBN — slower but reliable.

## Common Tasks

**Adding a new watchface**: Create new folder in `examples/WatchFaces/`, copy structure from `Basic/`, customize
**Modifying display**: Edit `src/Display.cpp` for core rendering or individual watchface `.cpp` files
**Adding sensors**: Extend BLE or BMA drivers in `src/`
**Building**: PlatformIO with ESP32-S3 target or Arduino IDE with appropriate board selection

# Watchy Project Context

## Project Overview
**Watchy Pocket Gamer** is a customizable open-source smartwatch based on the ESP32-S3 microcontroller. It's part of the larger Watchy ecosystem by SQFMI. This project implements the core firmware and provides example watchface implementations that users can customize.

**Repository Reference**: [sqfmi/watchy-docs](https://github.com/sqfmi/watchy-docs)

## Hardware Components

### Microcontroller
- **ESP32-S3**: Dual-core processor with WiFi/BLE capabilities
- Custom PCB board design for power management and peripherals

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

## Build System

- **Language**: Arduino C++ (compatible with PlatformIO/Arduino IDE)
- **Build Artifacts**: Compiled firmware for ESP32-S3
- Format script: `src/format.sh` - Code formatting tool

## Key Concepts

### Watchfaces
Custom watchface implementations inherit from or follow the Watchy pattern:
1. Extend main Watchy class or implement display update functions
2. Override `drawWatchFace()` or similar display methods
3. Handle button inputs for interactions
4. Manage display updates efficiently (e-paper refresh is slow)
5. Each watchface is a complete `.ino` sketch with supporting headers

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
- Full docs: https://github.com/sqfmi/watchy-docs
- Hardware schematics and board design specs available in docs
- API reference and watchface development guide in docs repository

## Common Tasks

**Adding a new watchface**: Create new folder in `examples/WatchFaces/`, copy structure from `Basic/`, customize
**Modifying display**: Edit `src/Display.cpp` for core rendering or individual watchface `.cpp` files
**Adding sensors**: Extend BLE or BMA drivers in `src/`
**Building**: PlatformIO with ESP32-S3 target or Arduino IDE with appropriate board selection

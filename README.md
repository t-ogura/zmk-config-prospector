# Prospector Scanner - ZMK Status Display Device

> ⚠️ **Dev build v2.1a available**: Pre-release with Zephyr 4.1 support and fixes for modifier key display and USB connection indicator bugs. Try it before the official release → [v2.1a tag](https://github.com/t-ogura/zmk-config-prospector/releases/tag/v2.1a)

> 🎉 **NEW: v2.0.0 "Touch & Precision" Released! (November 20, 2025)**
>
> **🎯 TOUCH PANEL SUPPORT**: Optional CST816S touch integration with swipe gestures
>
> **⚡ USB CONNECTION FIX**: Displays "> USB" when keyboard is USB-connected (Issue #5)
>
> **🔒 THREAD-SAFE**: Complete freeze elimination with LVGL mutex protection
>
> **⏱️ TIMEOUT DIMMING**: Configurable brightness reduction when no keyboard activity
>
> 👉 **[See complete v2.0 release notes →](docs/RELEASES/v2.0.0/release_notes.md)**

<p align="center">
  <img src="https://img.shields.io/badge/version-v2.0.0-brightgreen" alt="Version 2.0.0 Touch & Precision">
  <img src="https://img.shields.io/badge/status-production%20ready-brightgreen" alt="Production Ready">
  <img src="https://img.shields.io/badge/ZMK-compatible-blue" alt="ZMK Compatible">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="MIT License">
</p>

---

## 📋 Table of Contents

1. [What is Prospector Scanner?](#what-is-prospector-scanner)
2. [Key Features](#-key-features)
3. [Touch Mode vs Non-Touch Mode](#touch-mode-vs-non-touch-mode)
4. [Quick Start](#quick-start)
5. [Hardware & Wiring](#hardware--wiring)
6. [Configuration Guide](#configuration-guide)
7. [Architecture Overview](#️-architecture-overview)
8. [Protocol Specification](#-protocol-specification)
9. [Display Features](#-display-features)
10. [Keyboard Integration](#keyboard-integration)
11. [Documentation](#documentation)

---

## What is Prospector Scanner?

**Prospector Scanner** is an independent BLE status display device for ZMK keyboards. It monitors your keyboard's status (battery, layer, modifiers, WPM, etc.) in real-time without consuming a BLE connection slot.

### About Prospector

**Prospector** is a community hardware platform originally created by [carrefinho](https://github.com/carrefinho/prospector) as a universal ZMK keyboard dongle (keyboard → dongle → PC connection). This project takes the same hardware platform but uses it in a completely different way:

**Original Prospector (Dongle Mode)**:
- Keyboard connects to Prospector via BLE
- Prospector connects to PC via USB or BLE
- ⚠️ **Limitation**: Keyboard can only connect to dongle (loses multi-device capability)
- ⚠️ **Limitation**: Requires keyboard-specific dongle shield configuration
- ⚠️ **Limitation**: PC can't connect to keyboard directly

**This Project (Scanner Mode)**:
- Keyboard broadcasts status via BLE Advertisement (observer mode)
- Prospector only **listens** - does NOT connect to keyboard
- ✅ **Advantage**: Keyboard maintains full 5-device connectivity
- ✅ **Advantage**: Works with ANY ZMK keyboard (no shield needed)
- ✅ **Advantage**: Keyboard can connect directly to PC/tablet/phone

### Why Scanner Mode?

```
Dongle Mode (Original):              Scanner Mode (This Project):
┌─────────────┐                      ┌─────────────┐
│  Keyboard   │─BLE Connection──┐    │  Keyboard   │  BLE Adv (broadcast)
└─────────────┘                 │    └─────────────┘       ↓
                                ↓           │              ↓
                        ┌──────────────┐   ├── PC     ┌──────────────┐
                        │   Dongle     │   ├── Tablet │   Scanner    │
                        │ (Prospector) │   ├── Phone  │ (Prospector) │
                        └──────────────┘   └── ...    └──────────────┘
                                │                      (just monitors)
                                └─→ PC (only 1 device)
Keyboard loses multi-device!         Keyboard keeps 5 devices!
```

**Key Benefit**: Your keyboard stays fully functional with all devices while Scanner provides visual monitoring.

### Hardware Platform

Both modes use the same hardware:
- **MCU**: Seeeduino XIAO BLE (nRF52840)
- **Display**: Waveshare 1.69" Round LCD with touch panel (ST7789V + CST816S)
- **3D Case**: Open-source design from original Prospector

**Important**: Even "non-touch mode" uses the touch-enabled LCD - we simply don't wire the 4 touch pins. Original Prospector uses the same display but doesn't utilize touch functionality.

### Why Use It?

- ✅ **See Everything**: Battery, layer, modifiers, WPM, signal strength
- ✅ **Multi-Keyboard**: Monitor up to 5 keyboards simultaneously
- ✅ **Zero Connection Cost**: Uses BLE advertisements (observer mode)
- ✅ **Professional UI**: YADS-style widget layout with NerdFont icons
- ✅ **Split Keyboard Support**: Shows both left/right side information

## 🎯 Key Features

### 📱 **YADS-Style Professional UI**
- **Multi-Widget Display**: Connection status, layer indicators, modifier keys, WPM tracking, battery visualization, and signal strength
- **Split Keyboard Support**: Unified display showing both left and right side information with intelligent layout
- **Color-Coded Status**: 5-level battery indicators (Green/Light Green/Yellow/Orange/Red), unique pastel layer colors
- **Real-time Updates**: Instant response to keyboard state changes with sub-second latency
- **WPM Tracking**: Real-time Words Per Minute calculation with intelligent decay during idle periods
- **NerdFont Icons**: Professional modifier key indicators (󰘴 Ctrl, 󰘵 Shift, 󰘶 Alt,  GUI)

### 🔋 **Smart Power Management** (v2.0 Enhanced)
- **Activity-Based Intervals**: 10Hz (100ms) when typing, 0.03Hz (30s) when idle for maximum battery efficiency
- **Automatic Transitions**: Seamless switching between active/idle states with configurable timeouts
- **WPM-Aware Updates**: Higher frequency during active typing sessions with decay algorithm
- **Scanner Battery Support**: Real-time battery monitoring for scanner device with charging indicator
- **Timeout Dimming**: Automatic brightness reduction to 5% after configurable inactivity (v2.0)
- **USB/Battery Profiles**: Different brightness and power settings for USB vs battery operation

### 🎮 **Universal Compatibility**
- **Any ZMK Keyboard**: Works with split, unibody, or any ZMK-compatible device
- **Non-Intrusive**: Keyboards maintain full 5-device connectivity
- **Multi-Keyboard**: Monitor up to 5 keyboards simultaneously with channel isolation
- **No Pairing Required**: Uses BLE advertisements (observer mode) - no connection slot consumed

### 🎯 **Touch Panel Support** (v2.0 New)
- **Optional CST816S Touch**: Enable with `CONFIG_PROSPECTOR_TOUCH_ENABLED=y`
- **4-Direction Swipe Gestures**: Navigate settings screens intuitively
- **Display Settings Screen**: Adjust brightness, auto-brightness on/off via touch slider
- **System Settings Screen**: Future expansion for advanced configuration
- **Thread-Safe LVGL**: Complete freeze elimination with mutex protection
- **Dynamic Memory**: Widgets created on-demand, destroyed when not needed

### 🌞 **Adaptive Brightness** (v1.1+)
- **APDS9960 Integration**: Ambient light sensor with 4-pin mode (no interrupt required)
- **Smooth Transitions**: Configurable fade duration and steps (default: 800ms, 12 steps)
- **Non-linear Response**: Intelligent mapping for comfortable viewing in all lighting conditions
- **Auto/Manual Toggle**: Switch between sensor control and manual adjustment (touch mode)
- **Activity-Based Dimming**: Automatic reduction to 5% when keyboards go idle (v2.0)

### What's New in v2.0?

- 🎯 **Touch Panel Support**: Optional CST816S touch for interactive settings
- 🔌 **USB Display Fix**: Shows "> USB" when keyboard is USB-connected
- 🔒 **Thread-Safe**: Eliminates random freezes during gestures
- ⏱️ **Timeout Dimming**: Auto-dim to 5% on keyboard inactivity
- 📊 **Signal Updates**: Stable 1Hz update for reception strength
- 🌞 **4-Pin APDS9960**: Ambient light sensor without interrupt pin

---

## Touch Mode vs Non-Touch Mode

v2.0 supports **two build configurations**: Touch Mode and Non-Touch Mode.

### Quick Comparison

| Feature | Non-Touch Mode | Touch Mode |
|---------|---------------|------------|
| **Display** | Waveshare 1.69" Touch LCD | Same display |
| **Wiring** | 6 display pins + power (touch pins **not connected**) | **+4 touch pins** (TP_SDA/SCL/INT/RST) |
| **Settings** | Kconfig only (rebuild to change) | Interactive on-device adjustment |
| **Gestures** | Not supported | 4-direction swipe gestures |
| **Firmware Size** | ~900KB | ~920KB (+20KB) |
| **Configuration** | Edit `prospector_scanner.conf` | Edit `prospector_scanner.conf` + enable touch |

**Note**: Both modes use the **same Waveshare 1.69" Touch LCD** hardware. Non-touch mode simply leaves the 4 touch pins unconnected (same as original Prospector).

### Which Mode Should I Choose?

#### Choose **Non-Touch Mode** if:
- ✅ You want simpler wiring (6 display pins only, no touch pins)
- ✅ You don't need on-device settings adjustment
- ✅ You want maximum firmware simplicity
- ✅ You're following original Prospector hardware setup

#### Choose **Touch Mode** if:
- ✅ You want to wire the 4 touch pins for interactive control
- ✅ You want to adjust settings without rebuilding firmware
- ✅ You want swipe gestures for future features
- ✅ You're comfortable with +4 pin wiring

### Touch Mode Details

👉 **[Complete Touch Mode Guide →](docs/TOUCH_MODE.md)**

Touch mode requires:
- Same Waveshare 1.69" Round LCD (with CST816S touch controller)
- **4 additional connections**: TP_SDA, TP_SCL, TP_INT, TP_RST
- Configuration change: Set `CONFIG_PROSPECTOR_TOUCH_ENABLED=y` in `prospector_scanner.conf`

**This guide focuses on Non-Touch Mode** (standard Prospector wiring). For touch-specific setup, see the touch mode guide.

---

## Quick Start

### Prerequisites

- **Hardware**: Seeeduino XIAO BLE + Waveshare 1.69" Round LCD
- **Keyboard**: ZMK keyboard with status advertisement enabled (see [Keyboard Integration](#keyboard-integration))

### Step 1: Get Firmware

#### Option A: GitHub Actions (Recommended)

1. **Fork this repository**
   - Go to https://github.com/t-ogura/zmk-config-prospector
   - Click "Fork" (top-right)

2. **Enable GitHub Actions**
   - In your fork, go to "Actions" tab
   - Click "I understand my workflows, enable them"

3. **Trigger Build**
   - Go to "Actions" tab
   - Select "Build" workflow
   - Click "Run workflow" → "Run workflow"
   - Wait ~5-10 minutes

4. **Download Firmware**
   - Scroll to "Artifacts" section
   - Download "firmware" zip
   - Extract `prospector_scanner-seeeduino_xiao_ble-zmk.uf2`
   - **Default**: Non-touch mode (6 display pins wired)
   - **For touch mode**: See [Touch Mode Guide](docs/TOUCH_MODE.md) to enable touch in your fork

#### Option B: Local Build

```bash
# Clone and setup
git clone https://github.com/YOUR_USERNAME/zmk-config-prospector.git
cd zmk-config-prospector
python3 -m venv .venv
source .venv/bin/activate
pip install west

# Initialize workspace
west init -l config
west update

# Build (non-touch mode)
west build -b seeeduino_xiao_ble -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"

# Output: build/zephyr/zmk.uf2
```

### Step 2: Wire Hardware

See [Hardware & Wiring](#hardware--wiring) for detailed pinout.

**Minimum connections** (6 display pins + power):
```
LCD_DIN  → Pin 10 (MOSI)
LCD_CLK  → Pin 8  (SCK)
LCD_CS   → Pin 9  (CS)
LCD_DC   → Pin 7  (Data/Command)
LCD_RST  → Pin 3  (Reset)
LCD_BL   → Pin 6  (Backlight PWM)
VCC      → 3.3V
GND      → GND
```

**Optional**: APDS9960 sensor (4-pin, no interrupt needed)
```
SDA → D4 (P0.04)
SCL → D5 (P0.05)
VCC → 3.3V
GND → GND
```

### Step 3: Flash Firmware

1. **Enter bootloader**:
   - Connect XIAO BLE via USB
   - Press RESET button **twice quickly** (within 0.5 seconds)
   - `XIAO-SENSE` drive appears

2. **Flash firmware**:
   - Drag `.uf2` file to `XIAO-SENSE` drive
   - Drive disconnects automatically
   - Scanner reboots

### Step 4: Configure Keyboard

Add to your keyboard's `.conf` file:

```conf
# Enable status advertisement
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"

# Power optimization (10Hz active, 0.03Hz idle)
CONFIG_ZMK_STATUS_ADV_ACTIVITY_BASED=y
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=100
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000

# Split keyboard battery monitoring (if applicable)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
```

Rebuild and flash your keyboard firmware.

### Step 5: Test

1. Power on scanner (should show "Waiting for keyboards...")
2. Power on keyboard
3. Scanner should detect keyboard within 1-2 seconds
4. Check display shows: device name, layer, battery, etc.

**Success!** Your scanner is working.

---

## Hardware & Wiring

### Required Components

| Component | Specification | Link |
|-----------|--------------|------|
| **MCU** | Seeeduino XIAO BLE (nRF52840) | [Seeed Studio](https://www.seeedstudio.com/Seeed-XIAO-BLE-nRF52840-p-5201.html) |
| **Display** | Waveshare 1.69" Round LCD (ST7789V) | [Waveshare](https://www.waveshare.com/1.69inch-lcd-module.htm) |

### Optional Components

| Component | Purpose | Link |
|-----------|---------|------|
| **APDS9960** | Ambient light sensor (auto-brightness) | [Adafruit](https://www.adafruit.com/product/3595) |
| **LiPo Battery** | Portable operation (400-600mAh) | Generic 3.7V LiPo |

### Pin Mapping

#### Display Connections (Required)

| Display Pin | XIAO BLE Pin | nRF52840 GPIO | Function |
|------------|--------------|---------------|----------|
| LCD_DIN | Pin 10 | P1.15 | SPI MOSI (data out) |
| LCD_CLK | Pin 8 | P1.13 | SPI clock |
| LCD_CS | Pin 9 | P1.14 | Chip select |
| LCD_DC | Pin 7 | P1.12 | Data/Command select |
| LCD_RST | Pin 3 | P0.03 | Display reset |
| LCD_BL | Pin 6 | P1.11 | Backlight PWM |
| VCC | 3.3V | - | Power (3.3V) |
| GND | GND | - | Ground |

#### APDS9960 Sensor (Optional)

| Sensor Pin | XIAO BLE Pin | nRF52840 GPIO | Function |
|-----------|--------------|---------------|----------|
| VCC | 3.3V | - | Power (3.3V) |
| GND | GND | - | Ground |
| SDA | D4 | P0.04 | I2C data |
| SCL | D5 | P0.05 | I2C clock |

**Note**: v2.0 supports 4-pin connection - no interrupt pin needed (polling mode).

#### Battery (Optional)

| Battery Wire | XIAO BLE Pad | Description |
|-------------|--------------|-------------|
| + (Red) | BAT+ | Positive terminal |
| - (Black) | BAT- | Ground |

### Wiring Diagram

```
Waveshare 1.69" LCD             Seeeduino XIAO BLE
┌─────────────────┐             ┌──────────────┐
│                 │             │              │
│  Display Pins   │             │  3.3V ───────┼─── VCC
│  ├─ LCD_DIN ────┼─────────────┤  Pin 10      │
│  ├─ LCD_CLK ────┼─────────────┤  Pin 8       │
│  ├─ LCD_CS ─────┼─────────────┤  Pin 9       │
│  ├─ LCD_DC ─────┼─────────────┤  Pin 7       │
│  ├─ LCD_RST ────┼─────────────┤  Pin 3       │
│  ├─ LCD_BL ─────┼─────────────┤  Pin 6       │
│  ├─ VCC ────────┼─────────────┤  3.3V        │
│  └─ GND ────────┼─────────────┤  GND         │
│                 │             │              │
└─────────────────┘             └──────────────┘

Optional: APDS9960 Sensor
┌─────────────┐
│  APDS9960   │
│  VCC ───────┼─────────────────┤  3.3V        │
│  GND ───────┼─────────────────┤  GND         │
│  SDA ───────┼─────────────────┤  D4          │
│  SCL ───────┼─────────────────┤  D5          │
└─────────────┘                 └──────────────┘

Optional: LiPo Battery
             ┌─ BAT+ (Red wire)
     Battery ┤
             └─ BAT- (Black wire)
```

### Wiring Tips

1. **Test Display First**: Wire display pins and verify basic operation before adding sensor
2. **Keep Wires Short**: I2C works best with wires < 10cm
3. **Built-in Pull-ups**: XIAO BLE has I2C pull-ups on D4/D5 - no external resistors needed
4. **Polarity Check**: Double-check VCC/GND before powering on
5. **Clean Solder**: Poor solder joints cause intermittent display issues

---

## Configuration Guide

Configuration file: `config/prospector_scanner.conf`

### Essential Settings

#### Scanner Mode (Required)

```conf
# Enable scanner mode
CONFIG_PROSPECTOR_MODE_SCANNER=y

# Multi-keyboard support
CONFIG_PROSPECTOR_MULTI_KEYBOARD=y
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5     # Monitor up to 5 keyboards
```

#### Display & LVGL

```conf
# Display subsystem
CONFIG_ZMK_DISPLAY=y
CONFIG_DISPLAY=y
CONFIG_LVGL=y

# Required LVGL widgets
CONFIG_LV_USE_BTN=y
CONFIG_LV_USE_SLIDER=y
CONFIG_LV_USE_SWITCH=y

# Enable fonts (REQUIRED for widgets to compile)
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y        # Default font
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y             # Pixel fonts
CONFIG_LV_FONT_UNSCII_16=y

# Set default font
CONFIG_LV_FONT_DEFAULT_MONTSERRAT_20=y
```

**Why fonts matter**: If fonts are not enabled, build will fail with `'lv_font_montserrat_XX' undeclared` errors. Enable ALL fonts listed above.

### Layer Display Configuration

```conf
# Number of layer indicators shown on screen
CONFIG_PROSPECTOR_MAX_LAYERS=7        # Range: 4-10

# Visual effect:
# - 4 layers: Wide spacing, large indicators
# - 7 layers: Medium spacing (default)
# - 10 layers: Tight spacing, maximum capacity
```

**Match your keyboard**: Set this to match your keyboard's layer count for best appearance. If your keyboard has 5 layers, set to 5 for optimal spacing.

### Brightness Control

#### Fixed Brightness (Simple)

```conf
# Disable ambient light sensor
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n

# Set fixed brightness (0-100%)
CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=85
```

Best for: Users without APDS9960 sensor, or who prefer manual control.

#### Automatic Brightness (Advanced)

```conf
# Enable ambient light sensor (requires APDS9960 hardware)
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y

# Brightness range
CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS=20       # Minimum (dark rooms)
CONFIG_PROSPECTOR_ALS_MAX_BRIGHTNESS_USB=100  # Maximum (USB power)

# Smooth fade transitions
CONFIG_PROSPECTOR_BRIGHTNESS_FADE_DURATION_MS=800  # 800ms fade
CONFIG_PROSPECTOR_BRIGHTNESS_FADE_STEPS=12         # 12 steps
```

**What happens**: Scanner reads ambient light every few seconds and smoothly adjusts brightness (20-100%) with 800ms fade. No jarring changes.

**Hardware requirement**: APDS9960 sensor wired to D4/D5 (4-pin connection, no interrupt pin needed).

### Timeout Brightness (v2.0 New)

```conf
# Auto-dim when no keyboard activity
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=480000      # 8 minutes (0=disabled)
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_BRIGHTNESS=5   # Dim to 5%
```

**What it does**:
1. If no keyboard data received for 8 minutes → dim to 5%
2. When keyboard sends data again → restore previous brightness
3. Works with both USB and battery power

**Use case**: Battery-powered scanner - extends battery life when keyboards are turned off.

**Disable**: Set `CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=0` to disable timeout.

### Scanner Battery Support

```conf
# Enable scanner's own battery monitoring
CONFIG_PROSPECTOR_BATTERY_SUPPORT=y   # Requires LiPo connected to XIAO BLE
CONFIG_ZMK_BATTERY_REPORTING=y        # ZMK battery subsystem
```

**What you see**: Battery icon (🔋) in top-right corner with percentage and charging indicator.

**Hardware requirement**: LiPo battery connected to XIAO BLE's BAT+/BAT- pads.

**No battery?** Set `CONFIG_PROSPECTOR_BATTERY_SUPPORT=n` - no battery widget shown.

### USB Logging (Development)

```conf
# Enable USB serial logging
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y

# Reduce BT noise
CONFIG_BT_LOG_LEVEL_WRN=y
CONFIG_LOG_DEFAULT_LEVEL=4
```

**How to use**:
1. Enable these settings
2. Rebuild firmware
3. Connect via USB
4. Open serial monitor (e.g., `screen /dev/ttyACM0 115200`)
5. See debug logs for troubleshooting

**Production**: Set `CONFIG_LOG=n` to disable logging and reduce firmware size.

### Complete Configuration Example

```conf
# ===== SCANNER MODE =====
CONFIG_PROSPECTOR_MODE_SCANNER=y
CONFIG_PROSPECTOR_MULTI_KEYBOARD=y
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5

# ===== DISPLAY =====
CONFIG_ZMK_DISPLAY=y
CONFIG_DISPLAY=y
CONFIG_LVGL=y
CONFIG_PROSPECTOR_MAX_LAYERS=7

# ===== BRIGHTNESS =====
# Option 1: Fixed brightness (simple)
CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n
CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=85

# Option 2: Auto-brightness (requires APDS9960)
# CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y
# CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS=20
# CONFIG_PROSPECTOR_ALS_MAX_BRIGHTNESS_USB=100
# CONFIG_PROSPECTOR_BRIGHTNESS_FADE_DURATION_MS=800

# ===== TIMEOUT =====
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS=480000  # 8 min (0=disabled)
CONFIG_PROSPECTOR_SCANNER_TIMEOUT_BRIGHTNESS=5

# ===== BATTERY =====
CONFIG_PROSPECTOR_BATTERY_SUPPORT=n  # Enable if LiPo connected

# ===== FONTS (REQUIRED) =====
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y
CONFIG_LV_FONT_UNSCII_16=y
CONFIG_LV_FONT_DEFAULT_MONTSERRAT_20=y

# ===== LOGGING (optional) =====
# CONFIG_LOG=y
# CONFIG_ZMK_LOG_LEVEL_DBG=y
```

Copy this template and customize for your needs.

---

## 🏗️ Architecture Overview

### System Design

**Scanner Mode Design** (Independent Monitoring):
- Keyboard → Multiple Devices (up to 5 via normal BLE connections)
- Keyboard → Scanner (BLE Advertisement broadcast only - no connection)
- Scanner operates independently without consuming connection slots

### System Components

```
┌─────────────┐    BLE Adv     ┌──────────────┐
│   Keyboard  │ ──────────────→│   Scanner    │
│             │    26-byte     │   Display    │
│ (10Hz/0.03Hz)│   Protocol    │  (USB/Battery)│
└─────────────┘                └──────────────┘
       │
       ├── Device 1 (PC)
       ├── Device 2 (Tablet)
       ├── Device 3 (Phone)
       ├── Device 4 (...)
       └── Device 5 (...)
```

**Key Points**:
- Keyboard broadcasts status via BLE Advertisement (observer mode)
- Scanner only **listens** - does NOT connect to keyboard
- Keyboard maintains full 5-device connectivity
- Scanner can be powered via USB or battery (optional)

### Communication Flow

```
Keyboard                           Scanner
────────                          ────────
[Keypress detected]
    │
    ├─→ Update internal state
    │   (layer, modifiers, WPM)
    │
    ├─→ Package into 26-byte
    │   advertisement payload
    │
    └─→ Broadcast BLE Adv ─────→  [BLE Observer Mode]
        (100ms interval when           │
         typing, 30s when idle)        ├─→ Parse advertisement
                                       │   (battery, layer, etc.)
                                       │
                                       ├─→ Update LVGL widgets
                                       │   (battery bars, layer
                                       │    indicators, WPM, etc.)
                                       │
                                       └─→ Display to screen
                                           (YADS-style UI)
```

---

## 📊 Protocol Specification

### BLE Advertisement Format (26 bytes)

The keyboard broadcasts its status using a custom BLE Advertisement payload. Scanner receives this in observer mode (no connection needed).

| Offset | Field | Size | Description | Example |
|--------|-------|------|-------------|---------|
| 0-1 | Manufacturer ID | 2 bytes | `0xFF 0xFF` (Custom/Local use) | `FF FF` |
| 2-3 | Service UUID | 2 bytes | `0xAB 0xCD` (Prospector Protocol ID) | `AB CD` |
| 4 | Protocol Version | 1 byte | Protocol version (current: `0x01`) | `01` |
| 5 | Battery Level | 1 byte | Main battery 0-100% (Central for split) | `5A` (90%) |
| 6 | Active Layer | 1 byte | Current layer 0-15 | `02` (Layer 2) |
| 7 | Profile Slot | 1 byte | BLE profile 0-4 | `01` (Profile 1) |
| 8 | Connection Count | 1 byte | Number of connected BLE devices 0-5 | `03` (3 devices) |
| 9 | Status Flags | 1 byte | USB/BLE/Charging/Caps Lock bits | `05` (USB+Caps) |
| 10 | Device Role | 1 byte | `0`=Standalone, `1`=Central, `2`=Peripheral | `01` (Central) |
| 11 | Device Index | 1 byte | Split keyboard index (0=left, 1=right) | `00` |
| 12-14 | Peripheral Batteries | 3 bytes | Left/Right/Aux battery levels 0-100% | `52 00 00` (82%, none, none) |
| 15-18 | Layer Name | 4 bytes | ASCII layer identifier (optional) | `4C30...` ("L0") |
| 19-22 | Keyboard ID | 4 bytes | Hash of keyboard name for multi-device | `12345678` |
| 23 | Modifier Flags | 1 byte | L/R Ctrl,Shift,Alt,GUI states | `05` (LCtrl+LShift) |
| 24 | WPM Value | 1 byte | Words per minute 0-255 | `3C` (60 WPM) |
| 25 | Reserved | 1 byte | Future expansion | `00` |

### Status Flags (Offset 9)

```
Bit 7 6 5 4 3 2 1 0
    │ │ │ │ │ │ │ └─ USB Connected (1=yes, 0=no)
    │ │ │ │ │ │ └─── BLE Active (1=yes, 0=no)
    │ │ │ │ │ └───── Charging (1=yes, 0=no)
    │ │ │ │ └─────── Caps Lock (1=on, 0=off)
    │ │ │ └───────── Reserved
    │ │ └─────────── Reserved
    │ └───────────── Reserved
    └─────────────── Reserved
```

### Modifier Flags (Offset 23)

```
Bit 7 6 5 4 3 2 1 0
    │ │ │ │ │ │ │ └─ Left Ctrl
    │ │ │ │ │ │ └─── Left Shift
    │ │ │ │ │ └───── Left Alt
    │ │ │ │ └─────── Left GUI (Win/Cmd)
    │ │ │ └───────── Right Ctrl
    │ │ └─────────── Right Shift
    │ └───────────── Right Alt
    └─────────────── Right GUI
```

### Broadcast Intervals

The keyboard adjusts broadcast frequency based on activity to save battery:

- **Active Mode** (typing detected): 100ms interval (10 Hz)
  - Triggered by any keypress
  - Provides real-time WPM and layer updates

- **Idle Mode** (no typing): 30000ms interval (0.03 Hz)
  - Activated after 10 seconds of no keypresses
  - Battery-friendly for long idle periods
  - Still broadcasts status for monitoring

---

## 🎨 Display Features

### Widget Layout (v2.0)

```
┌─────────────────────────────┐
│ MyKeyboard            🔋 85%│ ← Device name (left), Scanner battery (right, v1.1+)
│ > USB                 [P0] │ ← Connection status (left: > USB or BLE icon, right: profile)
│                             │
│ WPM                    RX   │ ← WPM (left), Signal info (right)
│ 045             -45dBm 1.0Hz│   (RSSI + reception rate)
│                             │
│          Layer              │ ← Layer title
│    0  1  2  3  4  5  6      │ ← Pastel colored layer indicators (0-15 supported)
│                             │
│        󰘴  󰘶  󰘵              │ ← Modifier key indicators (NerdFont icons)
│                             │   󰘴=Ctrl, 󰘵=Shift, 󰘶=Alt, =GUI
│                             │
│    ████████████ 85%         │ ← Keyboard battery (main/central)
│    ████████████ 78%         │ ← Split keyboard peripheral battery (if applicable)
└─────────────────────────────┘
```

### Visual Elements

#### Battery Color Coding (5-Level System)

Battery bars use color-coded indicators for quick status recognition:

| Range | Color | Hex Code | Visual |
|-------|-------|----------|--------|
| **80-100%** | Green | `#00CC66` | 🟢 Full/Healthy |
| **60-79%** | Light Green | `#66CC00` | 🟢 Good |
| **40-59%** | Yellow | `#FFCC00` | 🟡 Medium |
| **20-39%** | Orange | `#FF9900` | 🟠 Low |
| **0-19%** | Red | `#FF3333` | 🔴 Critical |

#### Layer Display

- **Pastel Colors**: 7-15 unique pastel colors for different layers
- **Configurable Count**: Adjust max layers via `CONFIG_PROSPECTOR_MAX_LAYERS` (default: 7)
- **Dynamic Centering**: Automatically adjusts spacing based on layer count
  - 4 layers = wide spacing
  - 10 layers = tight fit
  - Always perfectly centered
- **Active Indicator**: Current layer highlighted with brighter color

#### Connection Status

- **USB Mode**: Shows "> USB" text (v2.0 fix for Issue #5)
- **BLE Mode**: Shows Bluetooth icon
- **Profile**: Displays active BLE profile `[P0]` - `[P4]`

#### WPM Display (Words Per Minute)

- **Real-time Calculation**: Updates during typing with decay algorithm
- **3-Digit Format**: `000` - `255` WPM
- **Idle Decay**: Gradually decreases when typing stops
- **Window**: Configurable (10s/30s/60s) via `CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS`

#### Signal Status (Bottom Right)

- **RSSI**: Signal strength in dBm (e.g., `-45dBm` = excellent, `-80dBm` = weak)
- **Reception Rate**: Update frequency in Hz (e.g., `10Hz` = active typing, `0.03Hz` = idle)

#### Scanner Battery (Top Right, v1.1+)

- **Display**: Shows scanner's own battery level `🔋 85%`
- **Charging Indicator**: Icon changes when USB charging
- **Optional**: Only shown when `CONFIG_PROSPECTOR_BATTERY_SUPPORT=y`

### Display Brightness (v2.0)

#### Auto-Brightness (APDS9960 Sensor)

- **Sensor**: APDS9960 ambient light sensor (4-pin mode, no interrupt)
- **Non-linear Response**: Maps sensor values to comfortable brightness levels
- **Smooth Transitions**: Configurable fade (default: 800ms, 12 steps)
- **USB/Battery Profiles**: Different max brightness for each power source
- **Activity-Based Dimming**: Auto-dim to 5% after timeout (configurable)

#### Manual Brightness (Touch Mode)

- **Swipe Gesture**: Swipe DOWN to access Display Settings
- **Slider Control**: Adjust brightness 0-100% via touch slider
- **Persistent**: Settings saved to flash memory
- **Auto-Brightness Toggle**: Enable/disable sensor control on-device

#### Timeout Dimming (v2.0)

- **Automatic**: Dims to 5% when no keyboard activity detected
- **Configurable Timeout**: Set via `CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS` (default: 8 minutes)
- **Wake on Activity**: Returns to normal brightness when keyboard activity resumes

---

## Keyboard Integration

Your ZMK keyboard needs to broadcast status via BLE Advertisement.

### Step 1: Add Module to Keyboard

Edit your keyboard's `config/west.yml`:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: prospector
      url-base: https://github.com/t-ogura

  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3
      import: app/west.yml

    # Add this:
    - name: prospector-zmk-module
      remote: prospector
      revision: v2.0.0  # Use specific version tag
      path: modules/prospector-zmk-module
```

### Step 2: Enable Status Advertisement

Add to your keyboard's `.conf` file:

```conf
# Enable status advertisement
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"  # Shown on scanner

# Activity-based intervals (battery optimization)
CONFIG_ZMK_STATUS_ADV_ACTIVITY_BASED=y
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=100      # 10Hz when typing
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000      # 0.03Hz when idle

# Split keyboard battery monitoring (if applicable)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y

# (Optional) Central side selection for split keyboards
# CONFIG_ZMK_STATUS_ADV_CENTRAL_SIDE="LEFT"  # or "RIGHT" (default)
```

### Step 3: Rebuild Keyboard Firmware

```bash
# In your keyboard config directory
west update
west build -b your_board -- -DSHIELD=your_shield

# Flash to keyboard
# (Copy .uf2 to bootloader drive)
```

### Step 4: Verify

1. Power on keyboard
2. Power on scanner
3. Scanner should detect keyboard within 1-2 seconds
4. Device name should match `CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME`

---

## Documentation

### Guides
- **[Touch Mode Guide](docs/TOUCH_MODE.md)** - Complete touch panel setup and usage
- **[v2.0 Release Notes](docs/RELEASES/v2.0.0/release_notes.md)** - Full changelog and migration guide
- **[Architecture Design](SCANNER_RECONSTRUCTION_DESIGN.md)** - Technical architecture (local dev file)

### Release History
- **[All Releases](docs/RELEASES.md)** - Version history and release notes
- **v2.0.0** (2025-11-20): Touch support, USB display fix, thread safety
- **v1.1.1** (2025-08-29): Universal hardware compatibility, 10-layer support
- **v1.1.0** (2025-08-26): Ambient light sensor, power optimization
- **v1.0.0** (2025-08-01): Initial release with YADS-style UI

### GitHub Resources
- **[Issues](https://github.com/t-ogura/zmk-config-prospector/issues)** - Bug reports and feature requests
- **[Actions](https://github.com/t-ogura/zmk-config-prospector/actions)** - Automated firmware builds
- **[Releases](https://github.com/t-ogura/zmk-config-prospector/releases)** - Pre-built firmware downloads

### Community & Related Projects
- **[ZMK Discord](https://zmk.dev/community/discord/invite)** - General ZMK support
- **[Original Prospector (Dongle Mode)](https://github.com/carrefinho/prospector)** - Hardware platform by carrefinho
- **[Original Prospector Firmware](https://github.com/carrefinho/prospector-zmk-module)** - Dongle mode implementation

---

## Troubleshooting

### Scanner Shows "Waiting for keyboards..."

**Problem**: Scanner not detecting keyboard.

**Solutions**:
1. Check keyboard has `CONFIG_ZMK_STATUS_ADVERTISEMENT=y` enabled
2. Verify keyboard firmware rebuilt and flashed after adding module
3. Check keyboard is powered on and BLE is active
4. Try power cycling both devices

### Build Error: `'lv_font_montserrat_XX' undeclared`

**Problem**: LVGL font not enabled.

**Solution**: Enable ALL fonts in `prospector_scanner.conf`:
```conf
CONFIG_LV_FONT_MONTSERRAT_12=y
CONFIG_LV_FONT_MONTSERRAT_16=y
CONFIG_LV_FONT_MONTSERRAT_18=y
CONFIG_LV_FONT_MONTSERRAT_20=y
CONFIG_LV_FONT_MONTSERRAT_22=y
CONFIG_LV_FONT_MONTSERRAT_24=y
CONFIG_LV_FONT_MONTSERRAT_28=y
CONFIG_LV_FONT_UNSCII_8=y
CONFIG_LV_FONT_UNSCII_16=y
```

### Display Shows Nothing / Blank Screen

**Problem**: Display not initializing.

**Solutions**:
1. Check wiring - especially VCC/GND polarity
2. Verify backlight pin (LCD_BL → Pin 6)
3. Test with settings reset firmware
4. Check XIAO BLE has power (LED indicator)
5. Verify display module (try with non-touch firmware if you have touch display)

### Connection Shows "BLE" When USB Connected

**Problem**: USB connection not detected (should show "> USB").

**This is a known issue in v1.x** - **Fixed in v2.0**.

**Solution**: Upgrade to v2.0 firmware (both scanner and keyboard).

### APDS9960 Sensor Not Working

**Problem**: Brightness doesn't change with room lighting.

**Solutions**:
1. Check 4-pin wiring: VCC/GND/SDA (D4)/SCL (D5)
2. Verify `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=y` enabled
3. Check sensor has clear view (not blocked by case)
4. Try increasing `CONFIG_PROSPECTOR_ALS_MIN_BRIGHTNESS` (sensor might be too dim)

**Note**: v2.0 does NOT require interrupt pin - 4-pin connection works.

### Scanner Freezes During Use

**Problem**: Display stops updating, becomes unresponsive.

**This is a known issue in v1.x** - **Fixed in v2.0** with LVGL mutex protection.

**Solution**: Upgrade to v2.0 firmware.

### Battery Widget Not Showing

**Problem**: No battery icon in top-right corner.

**Solutions**:
1. Check LiPo battery connected to BAT+/BAT- pads
2. Verify `CONFIG_PROSPECTOR_BATTERY_SUPPORT=y` enabled
3. Rebuild firmware after enabling battery support

**No battery?** Set `CONFIG_PROSPECTOR_BATTERY_SUPPORT=n` - this is expected behavior.

---

## Advanced Topics

### Building Custom Keyboard List

Scanner can monitor up to 5 keyboards. Configure in `prospector_scanner.conf`:

```conf
CONFIG_PROSPECTOR_MAX_KEYBOARDS=5  # Increase if needed (max: 5)
```

Display automatically shows keyboards that broadcast status. No pairing needed.

### WPM Calculation Windows

Adjust WPM responsiveness:

```conf
# Ultra-responsive (10s window, 6x multiplier)
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=10

# Balanced (30s window, 2x multiplier) - DEFAULT
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=30

# Stable (60s window, 1x multiplier)
# CONFIG_ZMK_STATUS_ADV_WPM_WINDOW_SECONDS=60
```

Shorter window = more responsive but jumpier. Longer window = more stable but slower to reflect changes.

### Split Keyboard Central Side

For split keyboards, specify which side is central (has BLE profiles):

```conf
# In keyboard's .conf file
CONFIG_ZMK_STATUS_ADV_CENTRAL_SIDE="LEFT"  # or "RIGHT" (default)
```

This tells scanner which side to treat as "main" for connection status display.

### Debug Widget (Development)

Enable debug overlay for development:

```conf
CONFIG_PROSPECTOR_DEBUG_WIDGET=y
```

Shows technical information overlaid on screen. **Disable for production** (`=n`).

---

## License & Attribution

### License

This project is licensed under the **MIT License**. See `LICENSE` file for details.

### Third-Party Components

#### ZMK Firmware
- **License**: MIT License
- **Source**: https://github.com/zmkfirmware/zmk

#### YADS (Yet Another Dongle Screen)
- **License**: MIT License
- **Source**: https://github.com/janpfischer/zmk-dongle-screen
- **Attribution**: UI widget designs and NerdFont modifier symbols

#### NerdFonts
- **License**: MIT License
- **Source**: https://www.nerdfonts.com/
- **Usage**: Modifier key symbols

### Original Prospector

This project builds upon the Prospector hardware platform created by carrefinho:
- **Original Project**: [prospector](https://github.com/carrefinho/prospector) by carrefinho
- **Original Firmware**: [prospector-zmk-module](https://github.com/carrefinho/prospector-zmk-module)
- **Hardware Design**: Seeeduino XIAO BLE + Waveshare 1.69" Round LCD with touch panel
- **3D Case Design**: Open-source STL files for 3D printing
- **License**: MIT License

**Difference**: Original Prospector uses the hardware as a dongle (keyboard connects to it), while this project uses it as an independent status monitor (keyboard stays independent). Both are valid uses of the same excellent hardware platform.

---

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Test thoroughly
5. Commit (`git commit -m 'Add amazing feature'`)
6. Push to branch (`git push origin feature/amazing-feature`)
7. Open a Pull Request

For major changes, please open an issue first to discuss what you would like to change.

---

**Questions?** Open an [issue](https://github.com/t-ogura/zmk-config-prospector/issues) or join [ZMK Discord](https://zmk.dev/community/discord/invite).

**Prospector Scanner v2.0.0** - ZMK Status Display Device

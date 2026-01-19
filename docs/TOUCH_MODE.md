# Prospector Scanner Touch Mode Guide

**Version**: v2.1.0
**Last Updated**: January 2026

## 📋 Table of Contents

1. [What's Different?](#whats-different) ⚠️ **Start Here**
2. [Quick Start](#quick-start)
3. [Screen Guide](#screen-guide)
4. [Gesture Reference](#gesture-reference)
5. [Troubleshooting](#troubleshooting)
6. [FAQ](#faq)

---

## ⚠️ What's Different?

**IMPORTANT**: Touch Mode requires **additional wiring** compared to standard Prospector Scanner.

### Key Differences from Non-Touch Mode

| Feature | Non-Touch Mode | Touch Mode |
|---------|---------------|------------|
| **Wiring** | 6 display pins + VCC/GND | **+4 touch pins required** |
| **Build Configuration** | `CONFIG_PROSPECTOR_TOUCH_ENABLED=n` | `CONFIG_PROSPECTOR_TOUCH_ENABLED=y` |
| Display Settings | Kconfig only | Interactive screen |
| Gestures | Not supported | 4-direction swipes |
| Firmware Size | ~900KB | ~920KB (+20KB) |
| On-Device Adjustment | Not supported | Yes (max layers, etc.) |

### ⚠️ Additional Wiring Required

Touch Mode adds **4 extra connections** to the standard Prospector Scanner wiring:

```
Standard Scanner: 6 pins (LCD_DIN, LCD_CLK, LCD_CS, LCD_DC, LCD_RST, LCD_BL) + Power
Touch Mode: +4 pins (TP_SDA, TP_SCL, TP_INT, TP_RST)
Total: 10 signal pins + VCC/GND
```

**You cannot use touch firmware without these 4 additional connections.**

### What Touch Mode Enables

- ✅ **Swipe Gestures**: Navigate between 5 screens with 4-direction swipes
- ✅ **Display Settings**: Adjust brightness, max layers, slide mode
- ✅ **Keyboard List**: View detected keyboards with channel filter
- ✅ **Quick Actions**: Fast access to common operations
- ✅ **Pong Wars**: Fun visual screensaver

---

## Quick Start

### Step 1: Verify Hardware

**How to Identify Touch Display**:
- Product page mentions "CST816S" or "touch sensor"
- Has **extra 4-pin connector** for touch (TP_RST, TP_INT, TP_SDA, TP_SCL)
- Product name includes "Touch" - e.g., "1.69inch Touch LCD"
- [Waveshare Official Link](https://www.waveshare.com/1.69inch-touch-lcd.htm)

### Step 2: Add Touch Wiring (4 Additional Pins)

**⚠️ These 4 connections are REQUIRED in addition to standard scanner wiring:**

| Touch Pin | XIAO BLE Pin | Description |
|-----------|--------------|-------------|
| TP_SDA | D4 (P0.04) | I2C Data |
| TP_SCL | D5 (P0.05) | I2C Clock |
| TP_INT | D0 (P0.02) | Interrupt |
| TP_RST | D1 (P0.28) | Reset |

**Note**: If you already have standard Prospector Scanner wired (6 display pins), just add these 4 touch pins. See [README - Hardware & Wiring](../README.md#hardware--wiring) for full pinout.

### Step 3: Build Touch Firmware

#### Option A: GitHub Actions (Recommended)

The default workflow builds both touch and non-touch firmware automatically.

1. **Fork repository**: Go to [zmk-config-prospector](https://github.com/t-ogura/zmk-config-prospector) → Click **"Fork"**
2. **Enable Actions**: Go to "Actions" tab → Click "I understand my workflows, enable them"
3. **Trigger build**: Actions tab → "Build" workflow → "Run workflow" → "Run workflow"
4. **Download**: After ~5-10 minutes, download `prospector_scanner_touch` artifact
5. **Extract**: Get the `.uf2` file from the zip

**To customize settings** (optional): Edit `config/prospector_scanner.conf` before triggering build. See [README - Configuration Guide](../README.md#configuration-guide) for options.

### Step 4: Flash Firmware

1. Connect XIAO BLE to computer via USB
2. **Enter bootloader mode**:
   - Press **RESET button twice quickly** (within 0.5 seconds)
   - RGB LED should pulse green/blue
   - `XIAO-SENSE` drive appears
3. **Copy firmware**:
   - Drag `.uf2` file to `XIAO-SENSE` drive
   - Drive will disconnect automatically
   - Scanner boots with touch firmware

#### Option B: Local Build

**Prerequisites**

```bash
# Install dependencies
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install west

# Initialize workspace
west init -l config
west update
```

**Enable Touch Mode**

Edit `config/prospector_scanner.conf`:

```conf
# Enable touch panel support
CONFIG_PROSPECTOR_TOUCH_ENABLED=y

# Increase max layers for touch mode
CONFIG_PROSPECTOR_MAX_LAYERS=10
```

**Build Command**

```bash
# From zmk-config-prospector directory
west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"

# Output: build/zephyr/zmk.uf2
```

**Flash**: Same as Option A - copy `build/zephyr/zmk.uf2` to XIAO-SENSE drive.

### Step 5: Test Touch

1. Scanner should boot and show main screen
2. Try **DOWN swipe** - Display Settings should appear
3. Try **UP swipe** - Return to main screen
4. Try **UP swipe** - Keyboard List should appear
5. Try **DOWN swipe** - Return to main screen
6. Try **RIGHT swipe** - Quick Actions should appear
7. Try **LEFT swipe** - Return to main screen
8. Success! Touch mode is working

**If touch doesn't work**: See [Troubleshooting](#troubleshooting) section.

**For other configuration options** (brightness, timeout, battery, etc.): See [README - Configuration Guide](../README.md#configuration-guide).

---

## Screen Guide

### Main Screen (Default)

The main screen is **identical to non-touch mode**. See [README - Display Features](../README.md#-display-features) for UI element descriptions.

#### Gestures from Main Screen

- **DOWN Swipe**: Open Display Settings
- **UP Swipe**: Open Keyboard List
- **RIGHT Swipe**: Open Quick Actions
- **LEFT Swipe**: Open Pong Wars (bonus!)

---

### Display Settings Screen

Interactive configuration screen - accessed via **DOWN swipe** from main screen.

#### Available Settings

Settings appear in this order from top to bottom:

##### 1. Brightness

**Slider**: Adjust display brightness (0-100%)

**Auto Toggle**: When enabled, brightness automatically adjusts based on ambient light sensor (requires APDS9960 hardware).

##### 2. Scanner Battery

**Toggle**: Show/hide scanner's own battery indicator on main screen.

**Note**: Only meaningful if LiPo battery is connected to XIAO BLE.

##### 3. Max Layers

**Slider**: Controls how many layer indicators are shown on main screen.

**Range**: 4 - 10 layers (limited by `CONFIG_PROSPECTOR_MAX_LAYERS`)

**Slide Toggle**: Switch between two layer display modes:

| Mode | Description |
|------|-------------|
| **List Mode** (Slide OFF) | Shows all layer numbers. Active layer is highlighted. |
| **Slide Mode** (Slide ON) | Dial-style animated display. Active layer slides to center position. |

**Slide Mode Details**:
- Layer change animates based on direction (increase = slide from right, decrease = slide from left)
- When layer exceeds MAX_LAYERS, displays a single large number instead of the layer list
- Smooth transition animation between modes

#### Persistence Behavior

⚠️ **Current Limitation**: Settings do **NOT persist** across power cycles.

- Adjustments apply immediately while powered on
- Settings reset to Kconfig defaults on reboot/power loss
- **Workaround**: Set preferred defaults in Kconfig and rebuild

#### Gestures from Display Settings

- **UP Swipe**: Return to main screen

---

### Keyboard List Screen

Accessed via **UP swipe** from main screen. Shows all detected keyboards.

#### Features

- **Keyboard Cards**: Each detected keyboard shown as a card with name, battery, layer info
- **Channel Filter**: Filter which keyboards to display

#### Channel Filter Operations

| Action | Description |
|--------|-------------|
| **Swipe LEFT** | Decrease channel (e.g., 2 → 1) |
| **Swipe RIGHT** | Increase channel (e.g., 1 → 2) |
| **Tap "Ch" label** | Open channel selection popup (0-9) |

**Channel Values**:
- **0**: Show all keyboards (no filter)
- **1-9**: Only show keyboards broadcasting on that channel

**Use case**: In multi-keyboard environments, filter to show only your keyboards.

**Keyboard side**: Set `CONFIG_PROSPECTOR_CHANNEL=N` in your keyboard's config (see [README](../README.md#channel-feature-v112)).

#### Gestures from Keyboard List

- **DOWN Swipe**: Return to main screen
- **LEFT/RIGHT Swipe**: Change channel filter

---

### Quick Actions Screen

Accessed via **RIGHT swipe** from main screen.

Quick access buttons for common operations.

#### Gestures from Quick Actions

- **LEFT Swipe**: Return to main screen

---

### Pong Wars Screen

Accessed via **LEFT swipe** from main screen.

Visual entertainment with two-color balls competitively painting the screen.

- **RIGHT Swipe**: Return to main screen

---

## Gesture Reference

### Swipe Detection

- **Minimum Distance**: ~30 pixels (~20% of screen width)
- **Timeout**: Must complete within 500ms
- **Directions**: 4-way (Up/Down/Left/Right)

### Gesture Map (v2.1)

| Gesture | Main Screen | Display Settings | Keyboard List | Quick Actions | Pong Wars |
|---------|-------------|------------------|---------------|---------------|-----------|
| **DOWN** | Display Settings | - | Main Screen | - | - |
| **UP** | Keyboard List | Main Screen | - | - | - |
| **LEFT** | Pong Wars | - | Channel -1 | Main Screen | - |
| **RIGHT** | Quick Actions | - | Channel +1 | - | Main Screen |

### Gesture Feedback

- **Visual**: Screen transitions with LVGL animation
- **No Haptic**: Display has no vibration motor
- **No Sound**: Silent operation

---

## Troubleshooting

### Touch Not Working

#### Symptom: No response to touch input

**Check List**:

1. **Verify Touch Hardware**:
   ```bash
   # Check I2C device detection (requires USB logging enabled)
   # Should show device at address 0x15
   ```

2. **Check Wiring**:
   - TP_SDA → D4 (P0.04)
   - TP_SCL → D5 (P0.05)
   - TP_INT → D0 (P0.02)
   - TP_RST → D1 (P0.28)

3. **Verify Configuration**:
   ```conf
   # Must be enabled
   CONFIG_PROSPECTOR_TOUCH_ENABLED=y
   CONFIG_INPUT_CST816S=y
   ```

4. **Check Firmware Build**:
   - Did you enable touch in config? (`CONFIG_PROSPECTOR_TOUCH_ENABLED=y`)
   - GitHub Actions: Did you edit `prospector_scanner.conf` to enable touch?

5. **I2C Bus Conflict**:
   - If APDS9960 also connected, verify both devices can coexist
   - Try disconnecting sensor temporarily to isolate issue

#### Symptom: Touch works but gestures don't register

**Possible Causes**:

1. **Swipe Too Slow**: Must complete within 500ms
2. **Swipe Too Short**: Minimum ~30 pixels distance required
3. **Screen Busy**: Wait for animations to complete before next gesture

### Display Settings Issues

#### Symptom: Cannot adjust slider beyond certain value

**Cause**: Kconfig `CONFIG_PROSPECTOR_MAX_LAYERS` limits maximum.

**Solution**: Edit `prospector_scanner.conf`:
```conf
CONFIG_PROSPECTOR_MAX_LAYERS=10  # Increase to desired maximum
```

Rebuild and reflash firmware.

#### Symptom: Settings reset after reboot

**Expected Behavior**: Settings do not persist across power cycles (current limitation).

**Workaround**: Set desired defaults in Kconfig:
```conf
CONFIG_PROSPECTOR_MAX_LAYERS=7  # Your preferred default
```

Rebuild and reflash firmware with your preferred defaults.

### Build Errors

#### Error: `'lv_font_montserrat_XX' undeclared`

**Cause**: LVGL font not enabled in config.

**Solution**: Use the default `prospector_scanner.conf` which includes all required fonts. If using a custom config, copy the font settings from the default file.

#### Error: `undefined reference to touch_handler_register_lvgl_indev`

**Cause**: Touch code compiled but not properly initialized.

**Solution**: Verify widget creation order in code (developers only - this should not occur in release builds).

### Performance Issues

#### Symptom: Screen freezes during swipes

**Rare Issue**: Should be resolved in v2.0+ with mutex protection.

**If it occurs**:
1. Check firmware version - update to latest v2.1.0
2. Reduce number of active keyboards being scanned (< 5)
3. Disable debug widget if enabled: `CONFIG_PROSPECTOR_DEBUG_WIDGET=n`

**Report**: If freezing persists, please open GitHub issue with:
- Firmware version
- Number of keyboards being scanned
- Gesture that triggers freeze
- USB log output (if available)

#### Symptom: Slow response to gestures

**Normal Behavior**: Slight delay (100-200ms) is expected for:
- LVGL mutex acquisition
- Screen transition animations

**Abnormal Delay** (>500ms):
- Check CPU load - multiple keyboards + sensor + touch may be intensive
- Disable ambient light sensor temporarily: `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n`
- Reduce update frequency: `CONFIG_PROSPECTOR_SCANNER_TIMEOUT_MS` to higher value

---

## Advanced Topics

### Shared I2C Bus

Touch panel (0x15) and APDS9960 (0x39) share I2C bus on D4/D5:

```
I2C Bus (D4/D5)
├── CST816S Touch (0x15)
│   └── Interrupt-driven
└── APDS9960 Sensor (0x39)
    └── Polling mode (no interrupt)
```

**Compatibility**: Both devices tested working simultaneously - no conflicts observed.

### Memory Usage

Touch mode adds ~20KB to firmware size:

| Component | Size |
|-----------|------|
| CST816S Driver | ~8KB |
| Touch Handler | ~3KB |
| Swipe Detection | ~2KB |
| Display Settings Screen | ~7KB |
| **Total Touch Overhead** | **~20KB** |

**Memory Safety**: XIAO BLE has 1MB flash - plenty of room for touch features.

### Power Consumption

Touch panel impact on battery life:

- **Touch Idle**: ~1mA (interrupt mode - waiting for touch)
- **Touch Active**: ~5mA (during gesture detection)
- **Negligible Impact**: Touch adds <2% to total power draw

**Battery Life Estimate** (with 500mAh LiPo):
- Display active: ~10 hours
- With timeout dimming: ~20+ hours
- Similar to non-touch mode

---

## FAQ

### Can I use touch firmware on non-touch display?

**No** - touch firmware requires CST816S hardware. It will build but touch features won't work and I2C errors may occur.

**Solution**: Use non-touch firmware (`prospector_scanner.conf`) for non-touch displays.

### Can I disable touch temporarily?

**Yes** - set in Kconfig:
```conf
CONFIG_PROSPECTOR_TOUCH_ENABLED=n
```

Rebuild firmware. All touch code excluded at compile time.

### Will settings persist across reboots?

**Not currently** - settings reset to Kconfig defaults on power cycle.

**For permanent settings**, edit Kconfig values and rebuild:
```conf
CONFIG_PROSPECTOR_MAX_LAYERS=7  # Your preferred default
```

---

## Additional Resources

### Documentation
- [Main README](../README.md) - Project overview
- [v2.1 Release Notes](RELEASES/v2.1.0/release_notes.md) - Complete changelog
- [v2.0 Release Notes](RELEASES/v2.0.0/release_notes.md) - Previous version changelog

### Hardware Guides
- [Waveshare 1.69" Touch LCD Datasheet](https://www.waveshare.com/wiki/1.69inch_Touch_LCD)
- [CST816S Touch Controller Datasheet](https://files.waveshare.com/upload/6/66/CST816S_Datasheet_EN.pdf)
- [XIAO BLE Pinout](https://wiki.seeedstudio.com/XIAO_BLE/#pin-diagram)

### Community
- [GitHub Issues](https://github.com/t-ogura/zmk-config-prospector/issues) - Bug reports and feature requests
- [ZMK Discord](https://zmk.dev/community/discord/invite) - General ZMK support

---

## Special Thanks

### Touch Implementation Reference

This touch mode implementation was inspired by and references the excellent work by **kot149**:

- **Article**: [Prospector Touch Panel Integration](https://zenn.dev/kot149/articles/prospector-touch) (Japanese)
- **Author**: kot149
- **Contribution**: Pioneering work on CST816S touch integration with Prospector hardware

We are deeply grateful for kot149's detailed documentation and exploration of touch panel possibilities, which provided invaluable insights for developing this touch mode implementation.

---

**Version History**:
- **v2.1.0** (2026-01): Zephyr 4.x compatibility, Keyboard List with channel filter, Layer Slide Mode, Pong Wars
- **v2.0.0** (2025-11-20): Initial touch mode with Display Settings screen

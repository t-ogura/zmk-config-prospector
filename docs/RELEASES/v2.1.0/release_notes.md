# Prospector Scanner v2.1.0 "Zephyr 4.x Ready"

**Release Date**: January 2026
**Status**: ✅ **Production Ready** - Major Feature & Compatibility Release
**Version Type**: Minor version with Zephyr 4.x compatibility, new features, and stability improvements

## 🌟 Release Highlights

v2.1.0 brings **Zephyr 4.x / ZMK main branch compatibility** along with major new features including dynamic battery display, channel filtering, layer slide mode, and Pong Wars screensaver.

### 🏆 **Major New Features**

#### **1. Zephyr 4.x Compatibility** 🔧
- **Board Name Change**: `seeeduino_xiao_ble` → `xiao_ble/nrf52840`
- **LVGL 9 Migration**: Updated for new LVGL API
- **Extended Advertising**: Native `BT_EXT_ADV` support
- **ZMK Main Branch**: Full compatibility with latest ZMK

#### **2. Dynamic Battery Display** 🔋
- **Adaptive Layout**: Automatically adjusts to 1-4 keyboards
- **Smart Labels**:
  - 1 keyboard: No label (wide bar)
  - 2 keyboards: L / R
  - 3 keyboards: L / R / Aux
  - 4 keyboards: L / R / A1 / A2
- **Wider Single Bar**: 165px width (1.5x) for single keyboard

#### **3. Channel Feature** 📡
- **Keyboard Side**: `CONFIG_PROSPECTOR_CHANNEL` (0=broadcast, 1-255=specific)
- **Scanner Side**: Channel filter in keyboard list screen
- **Use Case**: Filter specific keyboards in multi-keyboard environments
- **Badge Display**: Shows channel number on keyboard list

#### **4. Layer Slide Mode** 🎡
- **Dial-Style Animation**: Active layer slides to center position
- **Direction-Based**: Animates based on layer change direction
- **Config Option**: `CONFIG_PROSPECTOR_LAYER_SLIDE_DEFAULT=y`
- **Touch Toggle**: Change mode via Display Settings (touch mode)

#### **5. Over-Max Layer Display** 🔢
- **Large Number Mode**: Layers >= max_layers show single large digit
- **Slide Animations**:
  - Layer increase: slides from right
  - Layer decrease: slides from left
- **Smooth Transitions**: Between list mode and over-max mode

#### **6. Peripheral Battery Slot Mapping** 🔄
- **Configurable Slots**: Remap peripheral indices to display slots
- **Config Options**:
  - `CONFIG_ZMK_STATUS_ADV_HALF_PERIPHERAL` (default: 0)
  - `CONFIG_ZMK_STATUS_ADV_AUX1_PERIPHERAL` (default: 1)
  - `CONFIG_ZMK_STATUS_ADV_AUX2_PERIPHERAL` (default: 2)
- **Use Case**: When connection order doesn't match physical layout

#### **7. Pong Wars Screensaver** 🎮 *(Touch Mode)*
- **Visual Entertainment**: Two-color balls paint the screen
- **Access**: LEFT swipe from main screen
- **Exit**: RIGHT swipe to return

### 🏗️ **Improvements**

#### **Latency Optimization** ⚡
- **Faster Display Updates**: Optimized work queue scheduling (~50ms vs ~200ms)
- **RX Hz Calculation Fix**: Moving average (4 samples) for stable display
- **Atomic Counters**: Proper BLE advertisement counting

#### **Layer Display** 🔢
- **Removed Layer 15 Limit**: Now supports any layer number (0-255)
- **10+ Layer Support**: Better handling of complex keyboard layouts

#### **Sleep/Wake Recovery** 💤
- **Activity State Listener**: Proper cleanup on sleep entry
- **Improved Advertising Recovery**: Keyboard reconnects faster after PC sleep
- **Error Retry Logic**: 3 retries before advertising reset

#### **Extended Advertising** 📡
- **Scan Response Handling**: Fixed `BT_LE_ADV_OPT_SCANNABLE` flag
- **Device Identity**: `USE_IDENTITY` for consistent OS device listing
- **Burst Mode Fixes**: Resolved "Unknown" device name issues

#### **Timeout Feature** ⏱️
- **0 = Disabled**: Setting timeout to 0 now properly disables the feature
- **"Scanning..." Restoration**: Screen correctly returns to scanning state
- **Timeout Brightness**: Proper brightness reduction on timeout

## 📊 Version Comparison

| Feature | v2.0.0 | v2.1.0 | Improvement |
|---------|--------|--------|-------------|
| **ZMK Version** | ZMK ≤0.3 (Zephyr 3.x) | ZMK main (Zephyr 4.x) | ✅ Latest ZMK |
| **Board Name** | `seeeduino_xiao_ble` | `xiao_ble/nrf52840` | ✅ New naming |
| **Battery Display** | Fixed layout | Dynamic 1-4 keyboards | ✅ Adaptive |
| **Layer Limit** | 0-15 | 0-255 | ✅ No limit |
| **Layer Animation** | Static list only | Slide mode option | ✅ Animated |
| **Channel Filter** | Not supported | Keyboard & scanner | ✅ New feature |
| **Display Latency** | ~200ms | ~50ms | ✅ 4x faster |
| **RX Hz Display** | Unstable/garbage | Moving average | ✅ Stable |
| **Sleep Recovery** | Manual workaround | Automatic | ✅ Reliable |
| **Peripheral Mapping** | Hardcoded | Configurable | ✅ Flexible |
| **Screensaver** | None | Pong Wars | ✅ Fun! |

## 🔧 Bug Fixes

### **Display & UI**
1. ✅ **Layer 15 Limit**: Removed artificial limitation in advertisement parsing
2. ✅ **"Unknown" Device Name**: Fixed burst advertising causing name loss
3. ✅ **Timeout Screen**: Properly returns to "Scanning..." when keyboards timeout
4. ✅ **RX Hz Garbage Values**: Fixed float corruption on ARM with int32 storage
5. ✅ **Widget Update on Wrong Screen**: Prevent layer updates on non-main screens

### **BLE & Connectivity**
1. ✅ **Extended Advertising**: Proper scan response flag handling
2. ✅ **Device Identity**: Single device entry in OS Bluetooth list
3. ✅ **Sleep/Wake Recovery**: Improved advertising restart after PC sleep

### **Build & Compilation**
1. ✅ **Format Truncation**: Fixed buffer size warnings
2. ✅ **Forward Declarations**: Added missing function declarations
3. ✅ **Deprecated Headers**: Updated `zephyr/net/buf.h` to `zephyr/net_buf.h`

### **Configuration**
1. ✅ **Timeout 0 = Disabled**: Setting works correctly now
2. ✅ **Timeout Brightness**: Proper application on keyboard timeout

## 📋 New Configuration Options

### Keyboard Side (status_advertisement)

```conf
# Channel broadcasting (0 = all scanners, 1-255 = specific channel)
CONFIG_PROSPECTOR_CHANNEL=0

# Peripheral battery slot mapping (for split keyboards)
CONFIG_ZMK_STATUS_ADV_HALF_PERIPHERAL=0   # Keyboard half slot
CONFIG_ZMK_STATUS_ADV_AUX1_PERIPHERAL=1   # Aux1 (e.g., trackball)
CONFIG_ZMK_STATUS_ADV_AUX2_PERIPHERAL=2   # Aux2 (e.g., numpad)
```

### Scanner Side

```conf
# Layer slide animation mode (works in both touch and non-touch)
CONFIG_PROSPECTOR_LAYER_SLIDE_DEFAULT=y
```

## 🛠️ Hardware Requirements

No hardware changes from v2.0.0 - same components work with both versions.

### Scanner Device
- **MCU**: Seeeduino XIAO BLE (nRF52840)
- **Display**: Waveshare 1.69" Round LCD (ST7789V, 240x280)
- **Optional**: APDS9960 ambient light sensor
- **Optional**: LiPo battery for portable operation
- **Optional**: CST816S touch panel (Touch Mode)

## 🔄 Migration Guide

### **From v2.0.0 → v2.1.0**

#### **1. Update Module Reference**

Edit `config/west.yml`:
```yaml
- name: prospector-zmk-module
  remote: prospector
  revision: v2.1.0  # Update from v2.0.0
  path: modules/prospector-zmk-module
```

#### **2. Update ZMK to Main Branch** (if not already)

```yaml
- name: zmk
  remote: zmkfirmware
  revision: main  # Ensure using main branch
  import: app/west.yml
```

#### **3. Update Build Command**

Change board name in your build scripts:
```bash
# Old
-b seeeduino_xiao_ble

# New
-b xiao_ble/nrf52840
```

#### **4. Rebuild and Flash**

```bash
west update
rm -rf build
west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"
```

### **Keyboard Firmware Compatibility**

v2.1.0 scanner is compatible with both:
- **v2.0.0 keyboard module** (Zephyr 3.x)
- **v2.1.0 keyboard module** (Zephyr 4.x)

The BLE advertisement protocol is unchanged - only the Zephyr/ZMK version differs.

## ⚠️ Known Issues & Limitations

### Zephyr 4.x Specific
1. **Board Name**: Must use `xiao_ble/nrf52840` (old name won't work)
2. **West Update**: Required after changing west.yml

### General
1. **Multi-Keyboard Limit**: Maximum 5 keyboards
2. **Settings Persistence**: Touch mode settings not persistent across reboots
3. **Sleep/Wake**: First keypress wakes MCU; second press may be needed for faster reconnection
4. **Channel Feature**: Requires keyboard firmware update to use channel broadcasting

## 🧹 Cleanup

### Removed Files
- Legacy widget files from v1.x
- Unused prospector_adapter code
- CHANGELOG.md (consolidated into release notes)

### Code Quality
- Forward declarations added for all public functions
- Deprecated header includes updated
- Buffer sizes corrected for compiler warnings

## 📈 Statistics

- **Commits**: 30+ commits from v2.0.0
- **Files Changed**: 15+ source files updated
- **New Features**: 7 major features
- **Build Size**: ~920KB (touch mode), ~900KB (non-touch)

## 🙏 Acknowledgments

### Technical Foundation
- **ZMK Team**: Zephyr 4.x migration work
- **LVGL Team**: LVGL 9 improvements

### Community
- Users who reported sleep/wake issues
- Testers who verified Zephyr 4.x compatibility
- Feature requests for channel filtering and dynamic battery display

---

**Full Changelog**: [v2.0.0...v2.1.0](https://github.com/t-ogura/prospector-zmk-module/compare/v2.0.0...v2.1.0)

**Download Firmware**: [GitHub Actions Builds](https://github.com/t-ogura/zmk-config-prospector/actions)

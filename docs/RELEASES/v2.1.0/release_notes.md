# Prospector Scanner v2.1.0 "Zephyr 4.x Ready"

**Release Date**: January 2026
**Status**: ✅ **Production Ready** - Compatibility & Stability Release
**Version Type**: Minor version with Zephyr 4.x compatibility and stability improvements

## 🌟 Release Highlights

v2.1.0 brings **Zephyr 4.x / ZMK main branch compatibility** along with significant stability improvements, latency optimizations, and bug fixes discovered during the migration process.

### 🏆 **Key Changes**

#### **1. Zephyr 4.x Compatibility** 🔧
- **Board Name Change**: `seeeduino_xiao_ble` → `xiao_ble/nrf52840`
- **LVGL 9 Migration**: Updated for new LVGL API
- **API Updates**: Adapted to Zephyr 4.x breaking changes
- **ZMK Main Branch**: Full compatibility with latest ZMK

**Build Command (New)**:
```bash
west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"
```

#### **2. Latency Optimization** ⚡
- **Faster Display Updates**: Optimized work queue scheduling
- **Reduced Input Lag**: Improved response to keyboard status changes
- **Sub-second Updates**: Near-instant layer and modifier display

#### **3. Layer Display Improvements** 🔢
- **Removed Layer 15 Limit**: Now supports any layer number (0-255)
- **Layer Slide Mode**: Optional dial-style animated display
- **10+ Layer Support**: Better handling of complex keyboard layouts

#### **4. Sleep/Wake Recovery** 💤
- **Improved Advertising Recovery**: Keyboard reconnects faster after PC sleep
- **Activity State Listener**: Proper cleanup on sleep entry
- **Error Retry Logic**: 3 retries before advertising reset

### 🏗️ **Architecture Improvements**

#### **Extended Advertising Support** 📡
- **Scan Response Handling**: Fixed `BT_LE_ADV_OPT_SCANNABLE` flag usage
- **Device Identity**: Added `USE_IDENTITY` for consistent OS device listing
- **Burst Mode Fixes**: Resolved "Unknown" device name issues

#### **Timeout Feature Improvements** ⏱️
- **0 = Disabled**: Setting timeout to 0 now properly disables the feature
- **"Scanning..." Restoration**: Screen correctly returns to scanning state when keyboards timeout
- **Timeout Brightness**: Proper brightness reduction on timeout

## 📊 Version Comparison

| Feature | v2.0.0 | v2.1.0 | Improvement |
|---------|--------|--------|-------------|
| **ZMK Version** | ZMK ≤0.3 (Zephyr 3.x) | ZMK main (Zephyr 4.x) | ✅ Latest ZMK |
| **Board Name** | `seeeduino_xiao_ble` | `xiao_ble/nrf52840` | ✅ New naming |
| **Layer Limit** | 0-15 | 0-255 | ✅ No limit |
| **Display Latency** | ~200ms | ~50ms | ✅ 4x faster |
| **Sleep Recovery** | Manual workaround | Automatic | ✅ Reliable |
| **Timeout 0** | Not working | Properly disabled | ✅ Fixed |

## 🔧 Bug Fixes

### **Display & UI**
1. ✅ **Layer 15 Limit**: Removed artificial limitation in advertisement parsing
2. ✅ **"Unknown" Device Name**: Fixed burst advertising causing name loss
3. ✅ **Timeout Screen**: Properly returns to "Scanning..." when keyboards timeout

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

## 🛠️ Hardware Requirements

No hardware changes from v2.0.0 - same components work with both versions.

### Scanner Device
- **MCU**: Seeeduino XIAO BLE (nRF52840)
- **Display**: Waveshare 1.69" Round LCD (ST7789V, 240x280)
- **Optional**: APDS9960 ambient light sensor
- **Optional**: LiPo battery for portable operation
- **Optional**: CST816S touch panel (Touch Mode)

## 📋 Configuration Changes

### Build Command Update

**Old (Zephyr 3.x / ZMK ≤0.3)**:
```bash
west build -b seeeduino_xiao_ble -s zmk/app -- ...
```

**New (Zephyr 4.x / ZMK main)**:
```bash
west build -b xiao_ble/nrf52840 -s zmk/app -- ...
```

### No Configuration File Changes Required

`prospector_scanner.conf` remains compatible - no changes needed.

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

### General (Same as v2.0.0)
1. **Multi-Keyboard Limit**: Maximum 5 keyboards
2. **Settings Persistence**: Touch mode settings not persistent across reboots
3. **Sleep/Wake**: First keypress wakes MCU; second press may be needed for faster reconnection

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

- **Commits**: 17 commits from v2.1b to v2.1c
- **Files Changed**: 10+ source files updated
- **Build Size**: ~920KB (touch mode), ~900KB (non-touch)

## 🙏 Acknowledgments

### Technical Foundation
- **ZMK Team**: Zephyr 4.x migration work
- **LVGL Team**: LVGL 9 improvements

### Community
- Users who reported sleep/wake issues
- Testers who verified Zephyr 4.x compatibility

---

**Full Changelog**: [v2.0.0...v2.1.0](https://github.com/t-ogura/prospector-zmk-module/compare/v2.0.0...v2.1.0)

**Download Firmware**: [GitHub Actions Builds](https://github.com/t-ogura/zmk-config-prospector/actions)

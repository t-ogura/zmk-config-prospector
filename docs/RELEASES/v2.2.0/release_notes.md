# Prospector Scanner v2.2.0

**Release Date**: April 2026
**Status**: ✅ **Production Ready**
**Version Type**: Minor version with new layouts, BLE ADV rebuild, stability improvements, and version protocol

## 🌟 Release Highlights

v2.2.0 brings **3 new display layouts**, a **rebuilt BLE advertising system**, **NVS settings persistence**, **firmware version protocol**, and significant **stability improvements** including a complete scanner freeze fix.

### 🏆 **Major New Features**

#### **1. New Display Layouts** 🎨
3 new display layouts inspired by [carrefinho/prospector-zmk-module](https://github.com/carrefinho/prospector-zmk-module):

- **Field**: Animated line segments driven by WPM
- **Operator**: Minimalist design with dot indicators and arc/bar battery display
- **Radii**: Circular wheel indicator with color-coded layers

In touch mode, swipe LEFT/RIGHT to cycle through layouts. In non-touch mode, set the default layout via `CONFIG_PROSPECTOR_DEFAULT_LAYOUT`.

#### **2. NVS Settings Persistence** 💾
Settings now survive reboots:
- Channel filter
- Display brightness
- Selected layout
- Layer slide mode toggle

Uses dirty flag + screen transition flush to prevent excessive NVS writes.

#### **3. BLE Advertisement Rebuild** 📡
Complete rewrite of the advertising system with 3-mode Hybrid ADV:

- **Piggyback**: When ZMK is advertising (profile not connected) — injects manufacturer data into ZMK's advertising
- **Non-connectable ADV**: When profile is connected — own advertising set for status display
- **Connectable Proxy ADV**: When profile is not connected and ZMK isn't advertising — enables PC reconnection through Prospector

Additional improvements:
- **Burst on events**: Layer, modifier, and profile changes trigger burst (5×15ms) for reliable delivery
- **Boot burst**: Keyboard sends burst on startup for immediate scanner detection
- **Name-in-AD**: Periodic keyboard name broadcast in advertisement data (in addition to scan response)
- **Activity-based intervals**: Configurable active/idle intervals with automatic transitions

#### **4. Version Protocol** 🏷️
Module firmware version is now embedded in the BLE advertisement:

- Scanner Quick Actions screen shows both scanner and keyboard firmware versions
- Scanner displays exact git position (e.g., `Scanner v2.2.0` or `Scanner v2.2b-7-gc6ab274` for dev builds)
- Backward compatible: old keyboards (< v2.2) display as `KB: name (< v2.2)`

Version encoding uses the existing `version` and `profile_slot` bytes with bit packing — no protocol size change.

#### **5. High-Priority Change Detection** ⚡
Scanner-side display update prioritization:

- **High-priority** (layer, modifier, profile, connection): immediate display update
- **Low-priority** (WPM, battery): 1Hz periodic update

This reduces unnecessary display refreshes while ensuring responsive feedback for user-facing changes.

#### **6. CONFIG_PROSPECTOR_DEFAULT_LAYOUT** ⚙️
Select startup display layout via Kconfig:

```conf
CONFIG_PROSPECTOR_DEFAULT_LAYOUT=0  # 0=YADS, 1=Field, 2=Operator, 3=Radii
```

Works in both touch and non-touch modes.

### 🏗️ **Improvements**

#### **Stability** 🛡️
- **Scanner freeze fix**: Restored mutex + work handler architecture. Data processing (work handler) is separated from display rendering (LVGL timer), with lock-free ring buffer for BT RX path.
- **HWINFO Keyboard ID**: Hardware-unique ID generation prevents duplicate keyboard entries on BLE profile switch
- **Keyboard name fixes**: Stuck name, flicker, and same-name collision resolved
- **Name-in-AD burst suppression**: Name broadcast suppressed during burst to ensure layer data delivery

#### **Operator Layout Battery Display** 🔋
- 2 peripherals: Arc (circular) display
- 3+ peripherals: Automatically switches to bar display
- Proper label positioning and color coding

#### **Layer Name** 🔤
- Increased from 3 to 4 characters in advertisement protocol

#### **Build Compatibility** 🔧
- **Zephyr 3.5 / 4.x dual support**: SYS_INIT compatibility macros for keyboard-side code
- **LVGL 9**: Compatible via LVGL 8 API compatibility layer
- **GitHub Actions**: Board name updated to `xiao_ble/nrf52840/zmk` for ZMK board variant
- **Kconfig fixes**: FADE_STEPS duplicate removed, MIPI_DBI type added, I2C0 overlay moved to shield-level

## 📊 Version Comparison

| Feature | v2.1.0 | v2.2.0 | Improvement |
|---------|--------|--------|-------------|
| **Display Layouts** | YADS only | YADS + Field + Operator + Radii | ✅ 4 layouts |
| **Settings Persistence** | Lost on reboot | NVS persistence | ✅ Survives reboot |
| **BLE ADV** | Simple Legacy ADV | 3-mode Hybrid | ✅ Profile-aware |
| **Event Response** | Periodic only | Burst on changes | ✅ Instant feedback |
| **Keyboard ID** | Name hash | HWINFO (hardware-unique) | ✅ No collisions |
| **Version Display** | Not available | Quick Actions screen | ✅ KB + Scanner versions |
| **Display Priority** | All updates equal | High/Low priority | ✅ Reduced load |
| **Freeze Stability** | Occasional freeze | mutex + work handler | ✅ Stable |
| **ZMK 0.3 Support** | No (keyboard-side) | Yes (keyboard-side) | ✅ Dual support |

## 🔧 Bug Fixes

### **Stability**
1. ✅ **Scanner freeze**: Restored mutex for keyboards[] access + work handler for data processing, separated from LVGL timer
2. ✅ **Keyboard ID collision**: HWINFO-based ID prevents duplicate entries on profile switch
3. ✅ **Keyboard name stuck/flicker**: Improved name cache with LRU eviction and update conditions

### **Display & UI**
1. ✅ **Display refresh after screen transition**: Proper redraw when returning to main screen
2. ✅ **3-battery bar positioning**: Fixed bar/arc auto-switch in Operator layout
3. ✅ **Channel filter**: Keyboard list filtering works correctly

### **BLE**
1. ✅ **Name-in-AD during burst**: Suppressed to ensure layer data delivery
2. ✅ **Same-name keyboard collision**: BLE address verification added
3. ✅ **Profile change detection**: Immediate burst on profile switch

### **Build**
1. ✅ **Kconfig FADE_STEPS duplicate**: Removed
2. ✅ **MIPI_DBI Kconfig type**: Added bool type
3. ✅ **I2C0 overlay conflict**: Moved to shield-level
4. ✅ **GitHub Actions board name**: Updated for ZMK board variant

## 📋 New Configuration Options

### Keyboard Side

```conf
# Required
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"

# Recommended: Activity-based intervals
CONFIG_ZMK_STATUS_ADV_ACTIVITY_BASED=y
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=1000   # 1Hz when typing
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000     # 30s when idle
CONFIG_ZMK_STATUS_ADV_ACTIVITY_TIMEOUT_MS=5000   # 5s to idle transition

# Optional: Channel
CONFIG_PROSPECTOR_CHANNEL=0                      # 0=all, 1-9=specific
```

### Scanner Side

```conf
# Display layout (0=YADS, 1=Field, 2=Operator, 3=Radii)
CONFIG_PROSPECTOR_DEFAULT_LAYOUT=0

# Layer slide animation
CONFIG_PROSPECTOR_LAYER_SLIDE_DEFAULT=y
```

## 🛠️ Hardware Requirements

No hardware changes from v2.1.0.

## 🔄 Migration Guide

### **From v2.1.0 → v2.2.0**

#### **1. Update Module Reference**

Edit your keyboard's `config/west.yml`:
```yaml
- name: prospector-zmk-module
  remote: prospector
  revision: v2.2.0  # Update from v2.1.0
  path: modules/prospector-zmk-module
```

#### **2. Update Keyboard Config** (recommended)

```conf
# Remove these (no longer needed):
# CONFIG_ZMK_STATUS_ADV_PERIODIC=y
# CONFIG_BT_PER_ADV=y
# CONFIG_BT_EXT_ADV=y

# Recommended intervals:
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=1000
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000
CONFIG_ZMK_STATUS_ADV_ACTIVITY_TIMEOUT_MS=5000
```

#### **3. Rebuild and Flash**

```bash
west update
rm -rf build
west build -b xiao_ble/nrf52840 -s zmk/app -- \
  -DSHIELD=prospector_scanner \
  -DZMK_CONFIG="$(pwd)/config"
```

### **Keyboard Firmware Compatibility**

- v2.2.0 scanner works with v2.1.0 and v2.0.0 keyboard modules
- For full version display and burst features, update keyboard firmware to v2.2.0
- Old keyboards display as `KB: name (< v2.2)` on Quick Actions screen

## ⚠️ Known Issues & Limitations

1. **ZMK 0.3 (keyboard-side only)**: Keyboard firmware builds on both ZMK 0.3 and main. Scanner requires ZMK main.
2. **Old keyboard compatibility**: v2.1.0 or earlier keyboards work with v2.2.0 scanner, but profile switching may cause duplicate entries. Update keyboard firmware for best results.
3. **Settings reset**: After flashing v2.2.0 for the first time, NVS settings from previous versions may need to be reset via System Settings > System Reset.

## 📈 Statistics

- **Commits**: 45+ commits from v2.1.0
- **Files Changed**: 20+ source files
- **New Features**: 6 major features
- **Build Size**: ~686KB (touch mode), ~668KB (non-touch)

## 🙏 Acknowledgments

### Design Inspiration
- **[carrefinho](https://github.com/carrefinho/prospector-zmk-module)** - Operator, Radii, Field layout designs
- **[YADS](https://github.com/janpfischer/zmk-dongle-screen)** - Original UI design

### Community
- Users who reported freeze issues and provided detailed reproduction steps
- [jacobsamlarose](https://github.com/jacobsamlarose) - Build compatibility feedback (Issues #18, #19)

---

**Full Changelog**: [v2.1.0...v2.2.0](https://github.com/t-ogura/prospector-zmk-module/compare/v2.1.0...v2.2.0)

**Download Firmware**: [GitHub Actions Builds](https://github.com/t-ogura/zmk-config-prospector/actions)

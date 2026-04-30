# Prospector Scanner v2.2.0

**🎨 New Layouts • 📡 BLE ADV Rebuild • 🛡️ Stability Fix • 🏷️ Version Protocol • 💾 NVS Persistence**

---

## 🌟 **What's New**

### 🎨 **3 New Display Layouts**
Field, Operator, and Radii layouts join the original YADS display!

- **Field**: Animated line segments driven by WPM
- **Operator**: Minimalist with arc/bar battery display (auto-switches for 3+ peripherals)
- **Radii**: Circular wheel indicator
- Touch mode: swipe to cycle. Non-touch: `CONFIG_PROSPECTOR_DEFAULT_LAYOUT`

### 💾 **NVS Settings Persistence**
Channel filter, brightness, layout, and layer slide mode now survive reboots.

### 📡 **BLE Advertisement Rebuild**
Complete rewrite with 3-mode Hybrid ADV:
- **Piggyback** on ZMK's ADV when available
- **Own non-connectable ADV** when profile connected
- **Connectable proxy** when profile not connected (enables PC reconnection)
- **Burst** on layer/modifier/profile changes (5×15ms) for instant response
- **Boot burst** for immediate scanner detection on keyboard startup

### 🏷️ **Version Protocol**
Keyboard firmware version embedded in BLE advertisement. Quick Actions screen shows:
- Scanner version (with exact git position for dev builds)
- Keyboard version (or `< v2.2` for older firmware)

### ⚡ **High-Priority Display Updates**
Layer/modifier/profile changes update instantly. WPM/battery update at 1Hz. Reduces display load while keeping responsive feedback.

### 🛡️ **Scanner Freeze Fix**
Restored mutex + work handler architecture with lock-free ring buffer for BT RX path. Data processing separated from LVGL rendering.

---

## 📊 **Key Improvements from v2.1.0**

| Feature | v2.1.0 | v2.2.0 |
|---------|--------|--------|
| **Layouts** | YADS only | + Field, Operator, Radii |
| **Settings** | Lost on reboot | NVS persistent |
| **BLE ADV** | Simple Legacy | 3-mode Hybrid |
| **Event Response** | Periodic | Burst on changes |
| **Keyboard ID** | Name hash | HWINFO unique |
| **Version Display** | None | Quick Actions screen |
| **ZMK 0.3 (keyboard)** | No | Yes |

---

## 📦 **Downloads**

### **Pre-Built Firmware**
- `prospector_scanner-xiao_ble_nrf52840_zmk-zmk.uf2` - Non-touch mode
- `prospector_scanner_touch-xiao_ble_nrf52840_zmk-zmk.uf2` - Touch mode
- `settings_reset-xiao_ble_nrf52840_zmk-zmk.uf2` - Settings reset

### **Hardware Requirements**
- **MCU**: Seeeduino XIAO BLE (nRF52840)
- **Display**: Waveshare 1.69" Round LCD (ST7789V)
- **Touch** *(optional)*: +4 pins (TP_SDA/SCL/INT/RST)

---

## 🚀 **Quick Start**

### **1. Flash Scanner**
1. Download firmware above
2. Double-press RESET on XIAO BLE (bootloader mode)
3. Copy `.uf2` file to XIAO-SENSE drive

### **2. Update Keyboard**
Add to your keyboard's `config/west.yml`:
```yaml
- name: prospector-zmk-module
  remote: prospector
  revision: v2.2.0
  path: modules/prospector-zmk-module
```

Add to your keyboard's `.conf`:
```conf
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"

# Recommended intervals
CONFIG_ZMK_STATUS_ADV_ACTIVE_INTERVAL_MS=1000
CONFIG_ZMK_STATUS_ADV_IDLE_INTERVAL_MS=30000
CONFIG_ZMK_STATUS_ADV_ACTIVITY_TIMEOUT_MS=5000
```

### **3. Test**
1. Power on scanner → "Scanning..."
2. Power on keyboard → Detected within seconds
3. Touch mode: Try swipe gestures and Quick Actions screen!

---

## 🔄 **Migration from v2.1.0**

1. Update `west.yml`: `revision: v2.1.0` → `revision: v2.2.0`
2. Remove deprecated configs: `CONFIG_ZMK_STATUS_ADV_PERIODIC`, `CONFIG_BT_PER_ADV`, `CONFIG_BT_EXT_ADV`
3. Run `west update`, rebuild, and flash

**Keyboard compatibility**: v2.2.0 scanner works with v2.1.0 and v2.0.0 keyboard modules. For full features, update keyboard firmware too.

---

## 🐛 **Bug Fixes**

- ✅ Scanner freeze (mutex + work handler architecture)
- ✅ Keyboard ID collision (HWINFO-based unique ID)
- ✅ Keyboard name stuck/flicker/collision
- ✅ Name-in-AD during burst suppression
- ✅ Display refresh after screen transitions
- ✅ Kconfig duplicates and type issues
- ✅ GitHub Actions board name for ZMK variant

---

## 📖 **Documentation**

- **[README](../../README.md)** - Complete setup guide
- **[Touch Mode Guide](../../docs/TOUCH_MODE.md)** - Touch panel setup & gestures
- **[Release Notes](release_notes.md)** - Full changelog

---

## 🙏 **Credits**

- **[carrefinho](https://github.com/carrefinho/prospector)** - Original Prospector hardware & layout inspiration
- **[YADS](https://github.com/janpfischer/zmk-dongle-screen)** - UI design foundation
- **[jacobsamlarose](https://github.com/jacobsamlarose)** - Build compatibility feedback

---

**🏷️ Version**: v2.2.0
**📅 Released**: April 2026
**🛡️ Status**: **STABLE - PRODUCTION READY**
**📜 License**: MIT License

# Prospector Scanner v2.1.0 "Zephyr 4.x Ready"

**🔧 Zephyr 4.x / ZMK Main Branch Compatible • 🔋 Dynamic Battery Display • 📡 Channel Filter • 🎡 Layer Slide Mode**

---

## 🌟 **What's New**

### 🔧 **Zephyr 4.x Compatibility**
Full compatibility with ZMK main branch and Zephyr 4.x!

- **Board Name Change**: `seeeduino_xiao_ble` → `xiao_ble/nrf52840`
- **LVGL 9 Migration**: Updated for new LVGL API
- **Extended Advertising**: Native `BT_EXT_ADV` support
- **ZMK Main Branch**: Works with latest ZMK

> ⚠️ For ZMK ≤0.3 (Zephyr 3.x), use `v2.0.0`

### 🔋 **Dynamic Battery Display**
Automatically adapts to connected keyboard count (1-4 keyboards)!

- **1 keyboard**: Wide single bar (no label)
- **2 keyboards**: L / R labels
- **3 keyboards**: L / R / Aux
- **4 keyboards**: L / R / A1 / A2

### 📡 **Channel Filter** *(Touch Mode)*
Filter specific keyboards in multi-keyboard environments!

- **Keyboard Side**: `CONFIG_PROSPECTOR_CHANNEL=N` (0=all, 1-9=specific)
- **Scanner Side**: Swipe or tap "Ch" label on Keyboard List screen
- **Use Case**: Home/office separation, multi-user environments

### 🎡 **Layer Slide Mode**
Dial-style animated layer display!

- **Slide Animation**: Active layer slides to center position
- **Direction-Based**: Animates based on layer change direction
- **Over-Max Display**: Large single digit for layers exceeding max
- **Toggle**: `CONFIG_PROSPECTOR_LAYER_SLIDE_DEFAULT=y` or via Display Settings

### 🎮 **Pong Wars** *(Touch Mode)*
Fun visual screensaver with two-color balls painting the screen!

- **Access**: LEFT swipe from main screen
- **Exit**: RIGHT swipe to return

### ⚡ **Latency Optimization**
4x faster display updates!

- **Before**: ~200ms latency
- **After**: ~50ms latency
- **RX Hz**: Stable moving average display

---

## 📊 **Version Comparison**

| Feature | v2.0.0 | v2.1.0 |
|---------|--------|--------|
| **ZMK Version** | ZMK ≤0.3 (Zephyr 3.x) | ZMK main (Zephyr 4.x) |
| **Board Name** | `seeeduino_xiao_ble` | `xiao_ble/nrf52840` |
| **Battery Display** | Fixed layout | Dynamic 1-4 keyboards |
| **Layer Limit** | 0-15 | 0-255 |
| **Layer Animation** | Static list only | Slide mode option |
| **Channel Filter** | Not supported | Keyboard & scanner |
| **Display Latency** | ~200ms | ~50ms |

---

## 📦 **Downloads**

### **Pre-Built Firmware**
- `prospector_scanner-xiao_ble_nrf52840-zmk.uf2` - Non-touch mode
- `prospector_scanner_touch-xiao_ble_nrf52840-zmk.uf2` - Touch mode
- `settings_reset-xiao_ble_nrf52840-zmk.uf2` - Settings reset

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
  revision: v2.1.0  # Use v2.0.0 for ZMK ≤0.3
  path: modules/prospector-zmk-module
```

Add to your keyboard's `.conf`:
```conf
CONFIG_ZMK_STATUS_ADVERTISEMENT=y
CONFIG_ZMK_STATUS_ADV_KEYBOARD_NAME="MyKeyboard"

# Optional: Channel (for multi-keyboard filtering)
# CONFIG_PROSPECTOR_CHANNEL=1
```

### **3. Test**
1. Power on scanner → "Waiting for keyboards..."
2. Power on keyboard → Detected within 1-2 seconds
3. Touch mode: Try swipe gestures!

---

## 🔄 **Migration from v2.0.0**

### **Breaking Change: Board Name**
Update build commands:
```bash
# Old
-b seeeduino_xiao_ble

# New
-b xiao_ble/nrf52840
```

### **Migration Steps**
1. Update `west.yml`: `revision: v2.0.0` → `revision: v2.1.0`
2. Run `west update`
3. Update build command with new board name
4. Rebuild and flash

### **Keyboard Compatibility**
- v2.1.0 scanner works with both v2.0.0 and v2.1.0 keyboard modules
- BLE advertisement protocol unchanged

---

## 🐛 **Bug Fixes**

- ✅ **Layer 15 Limit**: Now supports any layer (0-255)
- ✅ **"Unknown" Device Name**: Fixed burst advertising issue
- ✅ **RX Hz Garbage Values**: Fixed with int32 storage
- ✅ **Sleep/Wake Recovery**: Improved advertising restart
- ✅ **Timeout Screen**: Properly returns to "Scanning..."

---

## 📖 **Documentation**

- **[README](../../README.md)** - Complete setup guide
- **[Touch Mode Guide](../TOUCH_MODE.md)** - Touch panel setup & gestures
- **[Release Notes](release_notes.md)** - Full changelog

---

## 🙏 **Credits**

- **carrefinho** - Original Prospector hardware platform
- **kot149** - Touch panel integration pioneering work
- **ZMK Team** - Zephyr 4.x migration support

---

**🏷️ Version**: v2.1.0 "Zephyr 4.x Ready"
**📅 Released**: January 2026
**🛡️ Status**: **STABLE - PRODUCTION READY**
**📜 License**: MIT License

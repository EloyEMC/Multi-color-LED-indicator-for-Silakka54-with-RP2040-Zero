# Troubleshooting Guide

## Common Issues and Solutions

### LED Not Working At All

**Symptoms:** LED stays completely off

**Possible Causes:**
1. Wrong GPIO pin configuration
2. Missing RGBLIGHT enable
3. Incorrect WS2812 driver

**Solutions:**
```c
// In config.h - verify GPIO16 is correct for your RP2040 Zero
#define WS2812_DI_PIN GP16

// In rules.mk - ensure these are not commented out
RGBLIGHT_ENABLE = yes
WS2812_DRIVER = vendor
```

### LED Stays One Color Only

**Symptoms:** LED changes once but gets stuck

**Possible Causes:**
1. Layer state not being cleared properly
2. Priority system not working
3. EEPROM retention issues

**Solutions:**
```c
// In layer_state_set_user - ensure states are cleared first
rgblight_set_layer_state(0, false);
rgblight_set_layer_state(1, false);
rgblight_set_layer_state(2, false);
```

### Colors Don't Match Expected

**Symptoms:** Wrong colors or no color change

**Possible Causes:**
1. HSV value errors
2. Layer index mismatch
3. Priority conflicts

**Solutions:**
```c
// Verify HSV constants are correct
{0, 1, HSV_RED}    // Red
{0, 1, HSV_BLUE}   // Blue
{0, 1, HSV_GREEN}  // Green
{0, 1, HSV_PURPLE} // Purple

// Verify layer indices match your keymap
case 1: // Layer 1
case 2: // Layer 2
case 3: // Layer 3
```

### Caps Lock Not Working

**Symptoms:** Caps lock doesn't change LED color

**Possible Causes:**
1. Missing led_update_user function
2. Wrong layer index for caps lock
3. Function not returning true

**Solutions:**
```c
bool led_update_user(led_t led_state) {
    rgblight_set_layer_state(3, led_state.caps_lock); // Index 3 = highest priority
    return true; // Important: must return true
}
```

### State Not Persistent After Reboot

**Symptoms:** LED state resets to default after power cycle

**Possible Causes:**
1. Missing RGBLIGHT_LAYERS_RETAIN_STATE
2. Missing rgblight_reload_from_eeprom()

**Solutions:**
```c
// In config.h
#define RGBLIGHT_LAYERS_RETAIN_STATE

// In keyboard_post_init_user
void keyboard_post_init_user(void) {
    rgblight_layers = my_rgb_layers;
    rgblight_reload_from_eeprom(); // Restore saved state
}
```

### Compilation Errors

**Common Error Messages:**

1. **"implicit declaration of function 'rgblight_set_layer_state'"**
   - Solution: Ensure `RGBLIGHT_ENABLE = yes` in rules.mk

2. **"too many arguments to function 'layer_state_is'"**
   - Solution: Use `layer_state_is(1)` instead of `layer_state_is(state, 1)`

3. **"'WS2812_DI_PIN' undeclared"**
   - Solution: Ensure `#define WS2812_DI_PIN GP16` in config.h

### Flashing Issues

**Symptoms:** UF2 file won't flash or keyboard not recognized

**Solutions:**
1. Ensure keyboard is in bootloader mode (press reset button)
2. Check that RPI-RP2 drive appears
3. Try a different USB cable
4. Recompile firmware before flashing

### Advanced Debugging

If issues persist, add debug output:

```c
// Add to keyboard_post_init_user
void keyboard_post_init_user(void) {
    rgblight_layers = my_rgb_layers;
    rgblight_reload_from_eeprom();
    
    // Debug: Test LED on startup
    rgblight_sethsv(HSV_WHITE);
    rgblight_enable();
    wait_ms(1000); // Show white for 1 second
    rgblight_disable();
}
```

### Hardware Issues

**RP2040 Zero Variants:**
- Some RP2040 Zero clones may use different GPIO pins
- Verify your board's pinout diagram
- Common alternatives: GP25, GP16, GP17

**LED Type:**
- Ensure your board has WS2812 (NeoPixel), not a simple LED
- WS2812 requires data line, simple LEDs need different approach

## Getting Help

If you're still having issues:

1. Check the [QMK Documentation](https://docs.qmk.fm/)
2. Search [QMK Issues](https://github.com/qmk/qmk_firmware/issues)
3. Post on [r/olkb](https://reddit.com/r/olkb) with:
   - Your config.h and rules.mk
   - Error messages
   - Hardware details
   - What you've tried

## Recovery

If your keyboard becomes unresponsive:

1. Enter bootloader mode (reset button)
2. Flash a known working firmware
3. Start with minimal configuration and add features gradually

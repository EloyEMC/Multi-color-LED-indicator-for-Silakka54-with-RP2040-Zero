# Technical Documentation

## Architecture Overview

This LED indicator system uses QMK's RGB Lighting Layers feature to provide visual feedback for keyboard state changes.

## Core Components

### RGB Lighting Layers

QMK's RGBLIGHT_LAYERS system allows:
- Multiple overlay layers with priority system
- Persistent state storage in EEPROM
- Efficient state management
- Hardware abstraction

### Layer Priority System

```
Index 0: Layer 1 (Blue)    - Priority 1 (lowest)
Index 1: Layer 2 (Green)   - Priority 2
Index 2: Layer 3 (Purple)  - Priority 3
Index 3: Caps Lock (Red)   - Priority 4 (highest)
```

Later indices in `RGBLIGHT_LAYERS_LIST` override earlier ones.

## Key Functions

### keyboard_post_init_user()

Called once during keyboard initialization:

```c
void keyboard_post_init_user(void) {
    rgblight_layers = my_rgb_layers;           // Set up layer definitions
    rgblight_reload_from_eeprom();             // Restore saved states
}
```

### led_update_user()

Called when host LED state changes (caps lock, num lock, etc.):

```c
bool led_update_user(led_t led_state) {
    rgblight_set_layer_state(3, led_state.caps_lock); // Index 3 = caps lock
    return true; // Must return true for proper operation
}
```

### layer_state_set_user()

Called when keyboard layer changes:

```c
layer_state_t layer_state_set_user(layer_state_t state) {
    // Clear all layer states first to prevent "stuck" colors
    rgblight_set_layer_state(0, false);
    rgblight_set_layer_state(1, false);
    rgblight_set_layer_state(2, false);
    
    // Set only the current active layer
    switch (get_highest_layer(state)) {
        case 1: rgblight_set_layer_state(0, true); break;  // Blue
        case 2: rgblight_set_layer_state(1, true); break;  // Green
        case 3: rgblight_set_layer_state(2, true); break;  // Purple
        default: break; // Layer 0 - no color
    }
    return state;
}
```

## Memory Management

### EEPROM Usage

With `RGBLIGHT_LAYERS_RETAIN_STATE` enabled:
- Layer states are stored in EEPROM
- Automatically restored on boot
- Persists across power cycles
- Minimal EEPROM wear (only writes on state changes)

### RAM Usage

- Layer definitions stored in PROGMEM (flash memory)
- Runtime state managed by QMK core
- Minimal RAM footprint

## Hardware Interface

### WS2812 Protocol

The RP2040 Zero uses the vendor driver for WS2812 communication:
- Hardware-specific timing
- DMA-based data transfer
- Non-blocking operation

### GPIO Configuration

```c
#define WS2812_DI_PIN GP16  // Data line for WS2812 LED
#define RGBLIGHT_LED_COUNT 1 // Single LED configuration
```

## Timing Considerations

### State Updates

- Layer changes: Immediate (within key processing cycle)
- Caps lock: Host-driven, may have slight delay
- EEPROM restore: During boot (~100ms)

### Performance Impact

- Minimal CPU overhead
- No impact on key scanning
- Efficient state management

## Compatibility

### QMK Version Requirements

- Minimum QMK version with RGBLIGHT_LAYERS support
- RP2040 platform support
- WS2812 vendor driver

### Hardware Variants

**RP2040 Zero Clones:**
- Waveshare RP2040-Zero: GP16 (confirmed)
- Some clones may use GP25 or other pins
- Verify pinout before implementation

**LED Types:**
- WS2812 (NeoPixel): Supported
- SK6812: Compatible (same protocol)
- Simple LEDs: Different approach required

## Customization Guidelines

### Adding More Layers

1. Define new layer segment:
```c
const rgblight_segment_t PROGMEM my_layer4_layer[] = RGBLIGHT_LAYER_SEGMENTS(
    {0, 1, HSV_ORANGE}
);
```

2. Add to layers list:
```c
const rgblight_segment_t* const PROGMEM my_rgb_layers[] = RGBLIGHT_LAYERS_LIST(
    my_layer1_layer,
    my_layer2_layer,
    my_layer3_layer,
    my_layer4_layer,    // New layer
    my_capslock_layer
);
```

3. Update state logic:
```c
case 4:
    rgblight_set_layer_state(3, true);  // New index
    break;
```

### Multiple LEDs

For keyboards with multiple LEDs:

```c
#define RGBLIGHT_LED_COUNT 5

// Define ranges for different LEDs
const rgblight_segment_t PROGMEM my_layer1_layer[] = RGBLIGHT_LAYER_SEGMENTS(
    {0, 2, HSV_BLUE},   // LEDs 0-1
    {3, 2, HSV_CYAN}    // LEDs 3-4
);
```

### Color Customization

Use HSV color space:
```c
HSV_RED    // Hue: 0, Sat: 255, Val: 255
HSV_BLUE   // Hue: 170, Sat: 255, Val: 255
HSV_GREEN  // Hue: 85, Sat: 255, Val: 255
HSV_PURPLE // Hue: 213, Sat: 255, Val: 255

// Custom colors
HSV_TEAL   // Hue: 128, Sat: 255, Val: 255
HSV_GOLD   // Hue: 43, Sat: 255, Val: 255
```

## Debug Information

### Layer State Monitoring

Add debug output to track state changes:

```c
layer_state_t layer_state_set_user(layer_state_t state) {
    // Debug output
    xprintf("Layer changed to: %d\n", get_highest_layer(state));
    
    // ... existing logic ...
}
```

### EEPROM State

Check saved states:
```c
void keyboard_post_init_user(void) {
    // Debug: Show saved states
    for (uint8_t i = 0; i < 4; i++) {
        bool state = rgblight_get_layer_state(i);
        xprintf("Layer %d state: %d\n", i, state);
    }
}
```

## Limitations

### Maximum Layers

- Default: 8 layers maximum
- Can be expanded with `#define RGBLIGHT_MAX_LAYERS 32`
- More layers = increased firmware size

### EEPROM Wear

- EEPROM has limited write cycles (~100,000)
- Frequent layer changes may impact longevity
- Consider using `noeeprom` functions for rapid changes

### Power Consumption

- WS2812 LEDs draw significant current
- Full brightness white: ~60mA per LED
- Consider battery life for portable devices

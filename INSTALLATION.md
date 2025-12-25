# Installation Guide

## Prerequisites

- QMK Firmware environment set up
- Silakka54 keyboard with RP2040 Zero microcontroller
- Basic understanding of QMK configuration

## Step 1: Backup Current Configuration

Before making changes, backup your existing keymap:

```bash
cp -r keyboards/silakka54/keymaps/default keyboards/silakka54/keymaps/default_backup
```

## Step 2: Copy Configuration Files

Copy the provided configuration files to your Silakka54 keymap:

```bash
# Copy config files
cp config.h keyboards/silakka54/config.h
cp rules.mk keyboards/silakka54/keymaps/default/rules.mk
cp keymap.c keyboards/silakka54/keymaps/default/keymap.c
```

## Step 3: Compile Firmware

```bash
qmk compile -kb silakka54 -km default
```

## Step 4: Flash Firmware

1. Put your Silakka54 in bootloader mode:
   - Press and hold the reset button on the RP2040 Zero
   - The keyboard should appear as "RPI-RP2" drive

2. Flash the firmware:
   ```bash
   qmk flash -kb silakka54 -km default
   ```
   
   Or manually copy the `.uf2` file to the RPI-RP2 drive

## Step 5: Test

After flashing, test the LED indicators:

1. **Default Layer (0):** LED should be off
2. **Press TG(1):** LED should turn blue (Layer 1)
3. **Press TG(2):** LED should turn green (Layer 2)
4. **Press TG(3):** LED should turn purple (Layer 3)
5. **Toggle Caps Lock:** LED should turn red (overriding layer color)
6. **Disable Caps Lock:** LED returns to layer color or off

## Troubleshooting

If the LED doesn't work:

1. Check that GPIO16 is correct for your RP2040 Zero variant
2. Verify `WS2812_DRIVER = vendor` is set in rules.mk
3. Ensure `RGBLIGHT_ENABLE = yes` is not commented out
4. Try recompiling and reflashing

## Customization

### Changing Colors

Edit the HSV values in `keymap.c`:

```c
{0, 1, HSV_RED}    // Change HSV_RED to HSV_BLUE, HSV_GREEN, etc.
```

### Adding More Layers

1. Add new layer definitions
2. Update the `RGBLIGHT_LAYERS_LIST`
3. Modify the `layer_state_set_user` function
4. Update `RGBLIGHT_MAX_LAYERS` if needed

### Different Pin Configuration

If your RP2040 Zero variant uses a different pin:

1. Change `WS2812_DI_PIN GP16` in config.h
2. Update documentation accordingly

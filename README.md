# LED Indicator Setup for Silakka54 with RP2040 Zero

This guide explains how to configure the internal WS2812 LED on RP2040 Zero as a multi-color indicator for keyboard layers and caps lock on the Silakka54 keyboard.

[Ver demostración del LED](multimedia/Silakka54_Layout_LED_RGB.mp4)


## Hardware

- **Keyboard:** Silakka54
- **Microcontroller:** RP2040 Zero
- **LED:** Internal WS2812 (NeoPixel) connected to GPIO16

## Features

- **Layer Indication:** Different colors for each keyboard layer
- **Caps Lock Indicator:** Red LED when caps lock is active
- **Persistent State:** LED states are saved to EEPROM and restored on reboot
- **Priority System:** Caps lock has highest priority, followed by layer colors

## Color Scheme

| Layer/Function | Color | Description |
|----------------|-------|-------------|
| Layer 0 (Default) | Off | No illumination |
| Layer 1 | Blue | First function layer |
| Layer 2 | Green | Second function layer |
| Layer 3 | Purple | Third function layer |
| Caps Lock | Red | When caps lock is active (highest priority) |

## Quick Start

1. Copy the configuration files to your Silakka54 keymap
2. Compile and flash the firmware
3. Enjoy your new LED indicators!

## Installation

See [INSTALLATION.md](INSTALLATION.md) for detailed setup instructions.

## Configuration Files

- `config.h` - Hardware configuration
- `rules.mk` - Build configuration  
- `keymap.c` - LED logic implementation

## Troubleshooting

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues and solutions.

## Technical Details

See [TECHNICAL.md](TECHNICAL.md) for in-depth technical information.

## License

This project is open source and available under the MIT License.

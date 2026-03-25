# GenBook U-Boot Patches

U-Boot patches for eDP display and HID-over-I2C keyboard support on the CoolPi CM5 GenBook (RK3588). I plan to incorporate usage of [TF-A firmware](https://github.com/TrustedFirmware-A/trusted-firmware-a) to enable sleep and suspend on the GenBook. Another word of note: my GenBook is not stable using rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.19.bin for DDR timings, so I recommend using rk3588_ddr_lp4_1848MHz_lp5_2112MHz_v1.19.bin.

**Note: These patches have not been tested yet.**

## Patches

### eDP Display Support
- `0001` - Add DisplayPort PHY configuration header (`include/phy-dp.h`)
- `0002` - Add Samsung HDPTX combo PHY driver for Rockchip
- `0003` - Add RK3588 eDP support to rk_edp video driver
- `0004` - Add minimal RK3588 VOP2 video output driver

### Internal Keyboard Support
- `0005` - Enable eDP display, USB keyboard, HID keyboard, and boot menu on CoolPi CM5 GenBook
- `0006` - Add HID-over-I2C keyboard driver (for the internal pct13x2 keyboard on I2C5)

## Display Pipeline

The eDP display pipeline is: VOP2 VP2 -> eDP1 controller -> HDPTX PHY1 -> panel

## Keyboard Support

The GenBook has two input methods supported by these patches:

- **Internal keyboard** via HID-over-I2C (I2C5, address 0x2c, HID descriptor at 0x0020) - uses the new `CONFIG_HID_I2C_KEYBOARD` driver
- **External USB keyboard** via `CONFIG_USB_KEYBOARD`

Both can be used to interact with the U-Boot boot menu (`CONFIG_CMD_BOOTMENU`).

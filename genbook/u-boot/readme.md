# GenBook U-Boot Patches

> **WARNING: These patches have been tested and DO NOT WORK yet.** Enabling the
> eDP display (patches 0001-0005) causes the board to fail to boot — U-Boot
> hangs during initialization with no serial output, no power LED, and no kernel
> load. The USB ACM serial gadget (patch 0007) compiles and the non-blocking
> fix is correct, but U-Boot's DWC3 gadget driver does not appear to enumerate
> on the USB-C port (Linux userspace configfs gadget on the same port works
> fine). **A 3.3V UART serial adapter connected to the UART2 debug pads
> (1.5Mbaud) is required to debug further.** Do not flash these patches
> without a way to recover via maskrom mode.

U-Boot patches for eDP display, HID-over-I2C keyboard, and USB serial console support on the CoolPi CM5 GenBook (RK3588). I plan to incorporate usage of [TF-A firmware](https://github.com/TrustedFirmware-A/trusted-firmware-a) to enable sleep and suspend on the GenBook. Another word of note: my GenBook is not stable using rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.19.bin for DDR timings, so I recommend using rk3588_ddr_lp4_1848MHz_lp5_2112MHz_v1.19.bin.

## Patches

### eDP Display Support (NOT WORKING — causes boot failure)
- `0001` - Add DisplayPort PHY configuration header (`include/phy-dp.h`)
- `0002` - Add Samsung HDPTX combo PHY driver for Rockchip
- `0003` - Add RK3588 eDP support to rk_edp video driver
- `0004` - Add minimal RK3588 VOP2 video output driver
- `0005` - Enable eDP display on CoolPi CM5 GenBook (defconfig + DTS)

### Internal Keyboard Support (untested — depends on eDP patches)
- `0006` - Add HID-over-I2C keyboard driver (for the internal pct13x2 keyboard on I2C5)

### USB Serial Console (compiles but gadget does not enumerate)
- `0007` - Make USB ACM stdio start non-blocking (fix for `f_acm.c`)
- `0008` - Add default console environment file

## Known Issues

1. **eDP display causes boot hang**: When patches 0001-0005 are applied and the
   video configs are enabled in the defconfig, U-Boot fails to boot. The failure
   occurs after SPL (SPL itself works), likely during VOP2 or eDP driver probe.
   Without UART2 serial debug output, the exact failure point is unknown.

2. **USB ACM gadget does not enumerate**: Despite `CONFIG_USB_FUNCTION_ACM=y`,
   `CONFIG_DM_USB_GADGET=y`, and the DWC3 controller being at `fc000000` with
   `dr_mode = "peripheral"`, the USB-C port does not present a CDC ACM device
   to a connected host laptop. The same hardware works correctly when configured
   via Linux userspace configfs, proving the hardware path is functional. The
   issue is in U-Boot's DWC3 DM gadget initialization.

3. **USB ACM blocks boot if set as default console**: The original `f_acm.c`
   `acm_stdio_start()` blocks indefinitely waiting for a host to open the serial
   port. Patch 0007 fixes this by removing the blocking wait. However, this is
   moot until issue 2 is resolved.

## Display Pipeline (intended)

VOP2 VP2 -> eDP1 controller -> HDPTX PHY1 -> panel

## Keyboard Support (intended)

- **Internal keyboard** via HID-over-I2C (I2C5, address 0x2c, HID descriptor at 0x0020)
- **External USB keyboard** via `CONFIG_USB_KEYBOARD`

## Fix Patch (untested)

-  - Fix eDP boot hang: enable pd_vo1 power domain in VOP2 and HDPTX PHY probe, enable PHY ref/apb clocks, abort on poll timeout, add missing DTS status and bootph tags. **Analysis + OEM DTS comparison — NOT YET TESTED. Now includes force-hpd, power domains for all nodes, VOP clock parent from OEM.** Created by analyzing the probe sequence; the root cause is almost certainly the missing power domain enable.

## Next Steps

- Obtain a 3.3V USB-to-serial adapter supporting 1.5Mbaud
- Connect to UART2 debug pads on the GenBook PCB
- Re-apply eDP patches and capture serial output to identify the failure point
- Debug USB ACM gadget initialization via serial console

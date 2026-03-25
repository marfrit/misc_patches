# GenBook Kernel Patches

Linux kernel patches for the CoolPi CM5 GenBook (RK3588).

## Patches

- `0001` - Add PWM15 pinctrl entries to RK3588 device-tree
- `0002` - Enable PWM fan on CoolPi CM5 GenBook. The GenBook does have a fan header. Originally, the RK3588 CPU is just connected with a heat pad to a copper plate. The heat pad should be replaced. I also added an additional copperplate being connected to the original one with a heat pad and a fan. The additional copper plate has holes so the fan air flow cools the plate.
- `0003` - Fix power-off by enabling RK806 as system power controller
- `0004` - Enable speaker output via audio graph card
- `0005` - Enable USB-C PD charging via FUSB302
- `0006` - Disable HAVE_GCC_PLUGINS selection on arm64
- `0008` - Add lid switch and USB3 PHY lane configuration
- `0009` - Make RTL_SEC_PROJ read non-fatal in btrtl Bluetooth driver
- `0010` - Fix suspend/resume and wakeup (rk8xx-spi, pwrkey, analogix-dp, gpio)

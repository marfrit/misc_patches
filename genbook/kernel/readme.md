# GenBook Kernel Patches

Linux kernel patches for the CoolPi CM5 GenBook (RK3588).

## Patches

- `0001` - Add PWM15 pinctrl entries to RK3588 device-tree
- `0002` - Enable PWM fan on CoolPi CM5 GenBook. The GenBook does have a fan header. Originally, the RK3588 CPU is just connected with a heat pad to a copper plate. The heat pad should be replaced. I also added an additional copperplate being connected to the original one with a heat pad and a fan. The additional copper plate has holes so the fan air flow cools the plate.
- `0003` - Fix power-off by enabling RK806 as system power controller
- `0004` - Enable speaker output via audio graph card
- `0005` - Enable USB-C PD charging via FUSB302
- `0006` - Disable HAVE_GCC_PLUGINS selection on arm64
- `0008` - Add lid switch (MH248 hall-effect sensor) and USB3 PHY lane mux configuration for the USB-A port and DP output
- `0009` - Make RTL_SEC_PROJ read non-fatal in btrtl Bluetooth driver (fixes RTL8852BE BT init crash)
- `0010` - Fix suspend/resume and wakeup: GPIO wake propagation to GIC, analogix eDP IRQ disable during suspend, rk8xx-spi PM ops, rk805-pwrkey wake IRQ, NPU power domain, touchpad wakeup-source

Note: `0007` (BCM43438 UART Bluetooth) was dropped — the GenBook uses RTL8852BE (WiFi via PCIe, BT via USB), not a Broadcom UART chip.

## Suspend/Resume Status

Patches 0010 provides the kernel-side fixes for suspend/resume. Full suspend also requires a PSCI firmware (BL31) that implements SYSTEM_SUSPEND. The Rockchip rkbin prebuilt BL31 (`rk3588_bl31_v1.51.elf`) does **not** support this; mainline [ARM Trusted Firmware (TF-A)](https://github.com/TrustedFirmware-A/trusted-firmware-a) for RK3588 does. Until TF-A is integrated, suspend is inhibited via systemd (see `misc/` directory).

## Audio

Patch 0004 adds the DTS configuration. The speaker also requires an ALSA UCM2 patch (see `misc/` directory) that adds a `SectionVerb` with the DAC mixer switch initialization and a `Speaker` device to the `rk3588-es8316` HiFi profile.

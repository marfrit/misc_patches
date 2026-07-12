# GenBook Kernel Patches

Linux kernel patches for the CoolPi CM5 GenBook (RK3588). These patches apply cleanly to **linux-6.19.9** using `git apply`.

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

## Kernel 7.1 Forward-Port Status (2026-07-13)

Forward-ported to **linux-7.1** (`marfrit/linux-7.1-rockchip`, v7.1-rc5+). Builds and **boots to a
working desktop**: panthor GPU comes up, the internal eDP panel lights at 1920x1080, and the SDDM
greeter is reached.

Two boot-log warnings look alarming but are **benign** and do **not** need patches:

- `drm_bridge.c: Missing drm_bridge_add() before attach` (rockchip eDP path). `analogix_dp` already
  carries the upstream `devm_drm_bridge_alloc()` conversion (it missed the bulk commit `9c399719cfb9`
  and was fixed in a follow-up; the conversion is present in this tree). The bridge still attaches and
  the eDP comes up at 1920x1080 — cosmetic.
- `panel-edp.c:814: Unknown panel CSO 0x144a, using conservative timings`. The GenBook eDP EDID isn't
  in panel-edp's timing table, so it falls back to conservative timings that work fine. Optional:
  add a table entry for panel `CSO 0x144a`.

**The real 7.1 gotcha is a module-install issue, not a kernel change.** A 7.1 build deployed with an
*incomplete* `/lib/modules` — missing the whole `drivers/input/` subtree (incl. `uinput.ko`) despite
`CONFIG_INPUT_UINPUT=m` in the built image — presents as "won't boot": display + SDDM come up fine, but
with no `/dev/uinput`, kmonad restart-loops and the **keyboard is dead**, so you can't log in at the
greeter. Verify the module deploy is complete before blaming the kernel:
`find /lib/modules/<ver>/kernel/drivers/input -name '*.ko*'` must be non-empty, then run `depmod <ver>`.

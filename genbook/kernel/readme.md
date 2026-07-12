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

Forward-ported to **linux-7.1** (`marfrit/linux-7.1-rockchip`, v7.1-rc5+). It builds and reaches
userspace, but is **NOT usable yet — the internal eDP panel stays black** (backlight on, no image).

**Blocker — eDP video stream clock never locks:**
`rockchip-dp fded0000.edp: Ignoring timeout of video streamclk ok`. VOP2 sets its dclk, the eDP-1
connector reports connected + enabled at 1920x1080, and a plasma/kwin session actually runs — but the
eDP link carries no video (the streamclk timeout is silently ignored), so the panel shows nothing. This
is a real RK3588 eDP clock/PHY regression vs the working fourier (7.0-rc3) kernel. **Open — needs the eDP
video-clock/PHY path debugged and compared against fourier.** (GenBook reverted to fourier meanwhile.)

**Benign — NOT the cause (do not chase these):**
- `[drm] Missing drm_bridge_add() before attach` — `analogix_dp` already carries the upstream
  `devm_drm_bridge_alloc()` conversion in this tree; cosmetic, the bridge still attaches.
- `panel-edp.c:814 Unknown panel CSO 0x144a, using conservative timings` — GenBook eDP EDID not in
  panel-edp's table; power-sequencing only, not the black screen. (Optional: add a `CSO 0x144a` entry.)

**Separate, already fixed — incomplete module install (dead keyboard).** A 7.1 build deployed with a
truncated `/lib/modules` (only 896 of 3550 modules; `drivers/input/` incl. `uinput.ko` missing despite
`CONFIG_INPUT_UINPUT=m`) leaves no `/dev/uinput` -> kmonad restart-loops -> dead keyboard at the (black)
greeter. Fixed by completing `make modules_install` + rsync to the GenBook + `depmod`. Verify deploy
completeness: `find /lib/modules/<ver>/kernel/drivers/input -name '*.ko*'` must be non-empty. This was a
second bug stacked on the black screen, not its cause.

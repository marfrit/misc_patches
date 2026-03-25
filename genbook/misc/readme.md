# GenBook Miscellaneous Patches

Userspace patches for the CoolPi CM5 GenBook (RK3588).

## Patches

- `0001` - Add SectionVerb (DAC mixer switch init) and Speaker output device to ALSA UCM2 `rk3588-es8316` HiFi profile. Without the SectionVerb, DAPM clears the mixer switches after card registration, resulting in no audio output. Install to `/usr/share/alsa/ucm2/Rockchip/rk3588-es8316/HiFi.conf`.
- `99-tcpm-debounce.rules` - Suppress rapid TCPM power supply change events during USB-C PD negotiation at boot, preventing repeated charger plug/unplug notification sounds in KDE Plasma. Install to `/etc/udev/rules.d/`.

## Suspend Inhibit Configuration

Until mainline TF-A firmware is integrated, suspend is inhibited system-wide. To apply:

```bash
# Mask sleep targets
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

# Configure logind to lock/ignore instead of suspend
sudo mkdir -p /etc/systemd/logind.conf.d
cat << 'EOF' | sudo tee /etc/systemd/logind.conf.d/no-suspend.conf
[Login]
HandleLidSwitch=lock
HandleLidSwitchExternalPower=lock
HandleLidSwitchDocked=ignore
HandleSuspendKey=lock
HandlePowerKey=ignore
IdleAction=lock
EOF
```

To re-enable suspend later: `sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target`

# GenBook U-Boot Patches

U-Boot patches for eDP display support on the CoolPi CM5 GenBook (RK3588). Please note: once this works, I plan to get the internal keyboard working. Also, I want to incorporate usage of [TF-A firmware](https://github.com/TrustedFirmware-A/trusted-firmware-a) to enable sleep and suspend on the GenBook. Another word of note: my GenBook is not stable using rk3588_ddr_lp4_2112MHz_lp5_2400MHz_v1.19.bin for DDR timings, so I recommend using rk3588_ddr_lp4_1848MHz_lp5_2112MHz_v1.19.bin. 

**Note: These patches have not been tested yet.**

## Patches

- `0001` - Add DisplayPort PHY configuration header
- `0002` - Add Samsung HDPTX combo PHY driver for Rockchip
- `0003` - Add RK3588 eDP support to rk_edp video driver
- `0004` - Add minimal RK3588 VOP2 video output driver
- `0005` - Enable eDP display on CoolPi CM5 GenBook board

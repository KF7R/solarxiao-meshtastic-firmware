# SolarXiao Advanced E22P — Meshtastic 2.8.1

Standalone build repository for the SolarXiao Advanced Plus carrier. Firmware reports **XIAO_NRF52_KIT (88)** and build target **seeed_xiao_nrf52840_kit** so clients can recognize it. The application still uses the SolarXiao-specific E22P, GPS, I2C and PA settings.

This repository contains only the SolarXiao Advanced build workflow and documentation; it does not contain the unrelated Spicy binaries.

The workflow checks out official Meshtastic commit `8e6a88d06f44cad26f1e8d7cd402939ccacb9b4c` (v2.8.1.8e6a88d), patches the XIAO kit variant with the SolarXiao settings, and builds `seeed_xiao_nrf52840_kit`.

## Download and build

[Download the verified USB/BLE package](https://github.com/KF7R/solarxiao-meshtastic-firmware/actions/runs/38088565432/artifacts/11683237160) from the [successful build](https://github.com/KF7R/solarxiao-meshtastic-firmware/actions/runs/38088565432). Compilation and package verification passed on October 10, 2026 (America/Phoenix). Flash usage: 690,332 / 815,104 bytes (84.7%); static RAM: 96,532 / 248,832 bytes (38.8%).

UF2 SHA256: `8bbd71be2fef53b8f3c5f4045cd5a290ba1224240bf7d9fa507a1c1e71107075`.

BLE ZIP SHA256: `91f185bd9beb947ea183b59e98734afbf4011010fe9af44da4df874abde6902b`.

Open [GitHub Actions](https://github.com/KF7R/solarxiao-meshtastic-firmware/actions/workflows/build.yml), select a successful run, and download **SolarXiao-2.8.1-XIAO-NRF-KIT-D0-GPS-USB-BLE**. Extract the outer artifact ZIP.

- `firmware-seeed_xiao_nrf52840_kit-2.8.1.8e6a88d.uf2`: USB application install.
- `firmware-seeed_xiao_nrf52840_kit-2.8.1.8e6a88d-ota.zip`: BLE application DFU package. Select this inner ZIP in the updater.
- `*.mt.json`, `SHA256SUMS.txt`, `README.txt`: identity metadata, checksums and install notes.

A second artifact contains the ELF/HEX, generated board sources, build log and validation record. The workflow verifies XIAO hardware identity, GPS/I2C/PA definitions, UF2 family and address bounds, application-only DFU manifest, S140 7.3.0 requirement (0x0123), CRC16, and application bytes against the HEX/UF2.

Builds run when the workflow changes or via **Run workflow** on main. Compilation and file checks do not establish on-device operation or successful BLE transfer.

## Wiring

Based on the designer's Advanced Plus schematic dated 2026-10-03:

| Function | XIAO assignment |
| --- | --- |
| Radio CS / IRQ | D4 / D1 |
| Radio RESET / BUSY | D2 / D3 |
| Radio SPI SCK / MISO / MOSI | D8 / D9 / D10 |
| GPS module RX / TX | D6 / D7 |
| I2C SDA / SCL | logical 30 / 31, NFC1 / NFC2 |
| GPS enable | D0, active high |

Radio EN is tied to 3.3 V on the carrier; D5 is not used. DIO2 controls T/RCTRL; DIO3 supplies the 1.8 V TCXO reference. NFC GPIO configuration is enabled. UART GPS starts at 9600 baud with no L76K-specific default. D0 provides active-high GPS enable control: high when active, low during hard sleep. The build patches the upstream GPS enable-pin check to support valid GPIO number zero.

U17 GPS connector schematic numbering: 1 D0 (active-high GPS enable), 2 GPS RX (MCU D6), 3 GPS TX (MCU D7), 4 GND, 5 3.3 V. Verify the chosen GPS breakout's supply requirements before using pin 5; connector numbering is not a physical viewing-order diagram.

U14 I2C connector numbering: 1 GND, 2 3.3 V, 3 SDA, 4 SCL.

The Adafruit Ultimate GPS breakout (product 746) uses all five connections: EN to pin 1, RX to pin 2, TX to pin 3, GND to pin 4, and VIN to pin 5. Its active-high enable matches this build. Verify physical pin orientation before applying power. Firmware can be installed without a GPS; leave fixed position enabled until live GPS coordinates are wanted.

## Power and limitations

The designer recommends 18 dBm chip drive for approximately 30 dBm RF output. This build caps chip drive at 18 dBm and applies an estimated 12 dB PA gain correction in normal mode. **Set requested transmit power to 30 dBm for the full-power operating point in normal mode**; requested 18 maps to chip drive 6. Licensed mode bypasses gain correction but retains the 18 dBm cap.

Lower power settings are not calibrated. The solar charger's status outputs are not connected to the MCU; the inherited XIAO charge detector cannot reliably indicate solar charging. Battery ADC settings remain inherited from the XIAO kit.

A successful build and UF2 checks establish compilation and file structure, not on-device performance. The previous application was checked over USB for radio initialization and power calculation. This D0 revision has not been flashed or tested with a GPS receiver.

## Flash

Extract the artifact ZIP. Double-tap the XIAO reset button to enter its UF2 bootloader and copy only the SolarXiao `.uf2` to the bootloader drive. Reconnect after it restarts, select the appropriate region, enable GPS, and inspect GPS/I2C detection.

## BLE OTA

The verified device bootloader is **0.9.2-OTAFIX2.3-BP1.4**, Board-ID **nRF52840-SeeedXiaoSense-v1**, SoftDevice **S140 7.3.0**. The package targets S140 7.3.0 and starts the application at **0x27000**.

First install this revised application over USB so the node reports XIAO_NRF52_KIT and the stock XIAO build-target name. For subsequent updates, connect in Meshtastic Android, open Firmware Update, choose the local/custom firmware option and BLE, and select the application `-ota.zip`. Keep the device powered and phone nearby until the update and reboot finish.

Use the custom package from this repository. An official stock XIAO firmware download lacks the SolarXiao-specific settings. The outer GitHub artifact ZIP and OTAFIX bootloader ZIP are not application DFU packages.

If your Android version cannot complete the update, [OTAFIX documents Nordic DFU apps](https://github.com/oltaco/Adafruit_nRF52_Bootloader_OTAFIX#recommended-ota-dfu-settings); the device advertises as `XIAO_DFU` in BLE DFU mode.

## References

- [Designer hardware](https://github.com/wehooper4/Meshtastic-Hardware/tree/main/XaioSeries/SolarXaio)
- [Advanced Plus schematic](https://github.com/wehooper4/Meshtastic-Hardware/blob/main/XaioSeries/SolarXaio/Advanced/Plus/Schematic_SolarXaio_ADV%2B_2026-10-03.pdf)
- [XIAO I2C/GPS upstream PR](https://github.com/meshtastic/firmware/pull/8866)

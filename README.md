# rapture releases

Firmware files of rapture (Tears Of Rapture boards). Boards download their update
from here themselves: the app sends `{"cmd":"ota_url","url":...}` over Wi-Fi or the cloud.

- `latest.json` — the newest version and the update file for each build target.
- `<version>/rapture-<chip>-<version>-ota.bin` — update over Wi-Fi / internet.
- `<version>/rapture-<chip>-<version>-full.bin` — by cable at 0x0 (new or erased board).

Chips: esp32, esp32s2, esp32s3, esp32s3_n16r8 (16 MB flash, 8 MB PSRAM), esp32c3, esp32c6.

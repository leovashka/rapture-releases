rapture 1.0.16, ESP-IDF 5.5.6

New in 1.0.16:
- "Защита насоса (ULP)" (pump_guard, S2 / S3): while the low-level signal stays
  below its threshold, the pause after a shutdown does not run out. The pump
  restarts a full pause after the level is back.
  - Example: a photo sensor as the level. In the dark the pump does not start;
    after dawn it waits the whole pause (as the MicroPython "dacia Codru" pump did).
  - Boards whose level threshold is 0 (off) work as before.
  - The ULP program version is 8: after the update the ULP is loaded again
    (the pump output follows the new program within a second).

- New command "ota_url": the board downloads its update itself, so it also works
  through the cloud (from anywhere):
    {"cmd":"ota_url","url":"https://.../rapture-esp32s2-1.0.16-ota.bin"}
  - http or https (certificates checked), redirects followed (GitHub release files).
  - The reply comes at once; progress and the result are in info.ota.progress and
    info.ota.url_state. Same checks as the update over Wi-Fi (chip, program), then
    a reboot and the usual health check with rollback.
  - The link must be reachable without a login (a public repository or server).
- Fixed: a needless restart "завис: сеть (net_mon), молчит 4294967 с". The task
  watchdog read the time before the task's mark; when the mark came in between,
  the difference went negative and looked like 49 days of silence.

Files for each chip (esp32, esp32s2, esp32s3, esp32s3_n16r8, esp32c3, esp32c6):
  rapture-<chip>-1.0.16-ota.bin   update over Wi-Fi from the app (board already runs rapture)
  rapture-<chip>-1.0.16-full.bin  by cable at address 0x0 (new or erased board;
                                  also clears Wi-Fi and cloud settings at 0x9000)
  esp32s3_n16r8: 16 MB flash layout, 3 MB app slots (see release/1.0.15).

Free space in the 1.5 MB app slot: esp32 22 %, s2 6 %, c3 6 %, s3 1 % (16 KB), c6 1 % (15 KB);
3 MB slot on s3_n16r8: 50 %.

Tested on S2 tor-556270 (well pump, preset "Насос в колодце (dacia Codru)"):
- updated by "ota_url" sent through the cloud (flespi), file served over http;
- https with a GitHub redirect: downloaded and checked (a non-firmware file was
  refused before anything was written);
- in the dark the pause stays at 180 min and the pump is off.
Preset for the well pump (S2 tor-556270):
  /Users/lev/Documents/tor-556270_s2_2026-11-21/well_pump_dacia_preset.json
Older releases stay in release/1.0.7 ... release/1.0.15.

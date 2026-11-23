rapture 1.0.17, ESP-IDF 5.5.6

New in 1.0.17: raw inverter packets, and an inverter module fed by another board.

- Inverter module ("Инвертор PI18"), new output "packet" ("Принятый пакет"):
  - the last PI18 frame exactly as it came, byte for byte, CRC and CR included
    ("^D106...<crc hi><crc lo>\r"); COM port, simulator or another board;
  - a signal from this output has the frame number as its value; the frame is
    in the signal's "packet" text (app: printable characters as they are, other
    bytes as \xHH) and in the module's info ("Последний пакет");
  - the app (1.0.5+) adds the signal "inv_packet" by itself when an inverter
    module on the COM port is added or saved.

- Inverter module, new mode "remote" ("Пакеты с другой платы"):
    {"type":"inverter","mode":"remote","board":"tor-xxxxxx","signal":"inv_packet"}
  - takes the raw frames from the "packet" signal of another board whose inverter
    module is on the UART, and decodes them here as if they came from the COM port
    (same CRC check, same outputs: pv_w, bat_v, out_w, grid, ...);
  - for now only between boards on the same Wi-Fi network: directly over Wi-Fi
    (HTTP once a second); packets never go through the cloud broker;
  - each transfer holds the newest frame of each kind ("^D106" GS, "^D005", "^D020",
    ... up to 4), so the GS gets through even when another device asks the inverter
    many other things; the frame numbers make sure none is decoded twice (also after
    a reboot of either board). Polled once a second: when two GS frames come within
    one second, only the newer one is decoded;
  - the module's info shows the link: "связь есть · Wi-Fi · N с назад", or why not
    ("другая плата не найдена в этой сети Wi-Fi", "нет ответа по Wi-Fi N с").
  - Both boards need 1.0.17.

- No extra broker traffic: a packet signal read by another board's ordinary
  "remote" module through the cloud gives only the frame number.

Files for each chip (esp32, esp32s2, esp32s3, esp32s3_n16r8, esp32c3, esp32c6):
  rapture-<chip>-1.0.17-ota.bin   update over Wi-Fi from the app, or "ota_url"
  rapture-<chip>-1.0.17-full.bin  by cable at address 0x0 (clears Wi-Fi and cloud
                                  settings at 0x9000)

Free space in the 1.5 MB app slot: esp32 22 %, s2 6 %, c3 6 %, s3 1 % (11 KB), c6 1 % (11 KB);
3 MB slot on s3_n16r8: 50 %.

Also fixed: with "check_crc" off, broken bytes of a frame went into "last_raw" and made
the status reply invalid JSON; they show as "?" now.

Tested (2026-11-23) with the real inverter: S3 mini tor-f17298 "Инвертор Дома"
(listen-only on the UART, another device polls the inverter) -> S3 N16R8 tor-eef350
(mode "remote"): same pv_w / out_w / bat_v / bat_cap / grid_v, no CRC errors, about
85 % of the GS frames decoded (the rest replaced by a newer one within a second).
Not for C3 (no PSRAM): with Bluetooth, Wi-Fi and the cloud TLS a C3 has ~36 KB free;
as a receiver it ran out of memory and restarted within 1-2 minutes.

Earlier test: C3 tor-11e5a8 (sender, simulator) -> S3 N16R8 tor-eef350
(receiver, mode "remote"): frames identical byte for byte, frame counters and all
values equal, no CRC errors, over Wi-Fi; checked that nothing goes to the broker. The test
presets were removed afterwards; both boards run 1.0.17 with their old presets.
Older releases stay in release/1.0.7 ... release/1.0.16.

rapture 1.0.18, ESP-IDF 5.5.6

New in 1.0.18: the inverter module decodes every PI18 reply, not only the status (GS).

Frames (a device polling the inverter asks for them; rapture with "poll" asks too):
  ^D106  GS    current state (as before) + new outputs below
  ^D005  MOD   working mode
  ^D037  FWS   fault code + 16 warning flags
  ^D088  PIRI  rated information = the settings
  ^D020  VFW   firmware versions ("05210,00000,00000")
  ^D020  FLAG  enable / disable flags ("1,1,0,1,1,1,0,1,2": same header, other shape)
  ^D034 / ^D038  MCHGCR / MUCHGCR  allowed charging currents (in the info only)
  ^D012  ACCT / ACLT  AC charging / AC load time windows "HHMM,HHMM"
  ^D011  ET / EY / EM / ED  energy counters
When rapture polls itself: GS, MOD and FWS every 6th query, PIRI every 30th, VFW,
FLAG and ET every 60th, ACCT and ACLT every 120th (5 s period: faults every 30 s,
settings every 2.5 min, flags and the total every 5 min, time windows every 10 min).
A listener cannot tell the two ^D012 replies apart, nor which day a ^D011 counter is
(SolarPower asks every day of a month for its chart): those come only from rapture's
own queries.

New outputs of the inverter module (for signals):
- state (GS): pv2_w, pv2_v, temp_mppt, mppt1_st / mppt2_st (0 off, 1 working, 2 charging),
  mppt_charging, charging, discharging, load_on, dcac_dir (0, 1 AC->DC, 2 DC->AC),
  cfg_changed
- mode (MOD): work_mode (0 power on, 1 standby, 2 bypass, 3 battery, 4 fault, 5 line /
  hybrid), on_battery, line_mode, bypass, mode_fault
- faults (FWS): fault (code, 0 none), fault_any, warning (any flag), w_line_fail,
  w_out_short, w_over_temp, w_fan, w_bat_high, w_bat_low, w_bat_under, w_overload,
  w_eeprom, w_power_limit, w_pv1_high, w_pv2_high, w_mppt1_over, w_mppt2_over,
  w_scc1_bat_low, w_scc2_bat_low
- settings (PIRI): grid_tie (on = Hybrid / grid-tie, off = Off-grid), set_out_prio,
  set_chg_prio, set_solar_prio, set_ac_chg_a, set_max_chg_a, set_bulk_v, set_float_v,
  set_recharge_v, set_redisch_v, set_cutoff_v, set_bat_type, set_in_range,
  set_out_model, rated_va, rated_w, rated_bat_v (-1 until the inverter sent them)
- flags (FLAG): f_buzzer, f_ovl_bypass, f_lcd_escape, f_ovl_restart, f_temp_restart,
  f_backlight, f_src_alarm, f_fault_record (f_buzzer is on when SolarPower shows the
  buzzer "Отключить": the flag probably means "muted")
- ac_chg_from / ac_chg_to / ac_load_from / ac_load_to: HHMM, -1 unknown (rapture polls)
- energy_total: ET, kWh (rapture polls); pv_today: solar energy today in kWh, counted
  by rapture from the PV power of each GS frame (any mode; from 0 after a restart and at
  midnight, needs the clock)
- inv_fw (main CPU firmware, e.g. 5210)
Mode and faults count for 3 minutes after their last frame (then -1 / off).
The module info has them too: data.work_mode / fault / warnings / fw_main / energy and
"settings" with every PIRI field.
All of them also work on a board in mode "remote" (frames from another board) and in
the simulator (it now also produces FWS with "line fail" while its grid is off, PIRI
and VFW).
When the module's source changes (COM port / simulator / another board), the old
settings, faults and counters are cleared.

Space: esp32s3 (and s3_n16r8) now build without the error-name table
(CONFIG_ESP_ERR_TO_NAME_LOOKUP=n) and without NimBLE logs, as the C6 already did
(error codes show as numbers). The inverter output labels in the catalog are short.
The C6 is full: 0x320 bytes (800 B) left.

Free space in the 1.5 MB app slot: esp32 21 %, s2 5 %, c3 5 %, s3 2 % (25 KB), c6 0 % (800 B);
3 MB slot on s3_n16r8: 50 %.

Tested (2026-11-24) with the real inverter (5 kW, firmware 05210, Off-grid):
- S3 mini tor-f17298 "Инвертор Дома" (listen-only, another device polls) and S3 N16R8
  tor-eef350 (mode "remote") show the same: work_mode 5, fault 0, no warnings, settings
  as in the inverter's own program (bulk 58.4 V, float 54 V, flooded, AC charge 2 A,
  max charge 10 A, solar first, appliance, single, off-grid);
- in Hybrid the PIRI field 21 became 1 and grid_tie came on (the "02" is the AC charging
  current, 2 A, unchanged);
- FLAG from the simulator: 1,1,0,1,1,1,0,1 -> the eight f_* outputs;
- not tested with a real inverter: rapture's own polling of FLAG / ET / ACCT / ACLT
  (no board polls the inverter now: tor-f17298 only listens);
- simulator with the grid off: work_mode 3 (battery), w_line_fail on, grid_tie follows
  "feed_grid".
Older releases stay in release/1.0.7 ... release/1.0.17.

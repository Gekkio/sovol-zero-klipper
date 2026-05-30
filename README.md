# Klipper with Sovol Zero changes

This repo attempts to clean up Sovol's Klipper changes made for the Sovol Zero 3D printer.

The [sovol-img](https://github.com/Gekkio/sovol-zero-klipper/tree/sovol-img) branch contains the git commits on the actual disk images on the printers. However, Sovol basically did a massive initial commit instead of applying changes on top of the upstream history. By comparing files and their history, I think Sovol used the upstream commit `faa89be816064b42bff1ba81478405490e49289c` as their base. The initial commit is basically the state of the repo at this particular upstream commit + a lot of Sovol changes in one large commit.

The [sovol-clean](https://github.com/Gekkio/sovol-zero-klipper/tree/sovol-clean) branch contains the changes cleaned up slightly and applied *on top of the upstream base*. This makes it much easier to study the differences for research purposes, since now all git tools (e.g. git blame) work properly.

## Summary by Claude Code

The following is a summary of the commits on the `sovol-clean` branch that sit on top of the
upstream base (`faa89be816064b42bff1ba81478405490e49289c`). 

### Context

This branch is Sovol's vendor customization of Klipper for the **Sovol Zero** printer (internal
codename "sv08mini"). The work clusters around a handful of themes:

- A custom **on-screen menu/display system** (UC1701 LCD, custom fonts, WS2811 status LED)
- An **eddy-current inductive probe** (LDC1612 sensor; Chinese 涡流 = "eddy/vortex") used for Z
  calibration, plus a separate **pressure probe**
- A **Z-offset calibration** routine that gets heavily iterated
- A scheme of numeric **"tip codes" / "error codes"** (提示码 / 错误码) surfaced on the screen and
  web UI
- **I²C reliability fixes** for the STM32F1 hardware I²C talking to the LDC1612
- Sovol Zero **config files** (printer.cfg, macros, moonraker, etc.)

### Translation glossary

涡流 = eddy-current probe · 提示码 = tip/prompt code · 错误码 = error code · 校准 = calibration ·
断电续打 = power-loss resume · 腔室温度 = chamber temperature · 输入整形 = input shaping ·
冗余 = redundant.

### Chronological walkthrough

The "Original ID" column is the SHA recorded in each commit's message body (`Commit ID:`),
identifying the corresponding commit in Sovol's original repository.

| # | Hash | Original ID | Original message (translated) | What it does |
|---|------|-------------|-------------------------------|--------------|
| 1 | `d81845c2` | `a6dc07c8da` | "first commit for sv08mini" | Foundational drop. Adds the UC1701 LCD driver + fonts, WS2811 LED driver, `gcode_shell_command`, `probe_pressure.py`, `z_offset_calibration.py`, a big rewrite of the display `menu.cfg`/`menu.py`, plus tweaks across fan, gcode_move, idle_timeout, eddy-current probe, MCU/scheduler/chipid. (33 files) |
| 2 | `1e967130` | `ad90d601b7` | "Add eddy-current function" | Wires up the eddy-current (涡流) probe: ldc1612, probe_eddy_current, z-offset calibration, menu, I²C. |
| 3 | `1742f3b2` | `1b1618962e` | "Modify eddy-current motion" | Reworks the probing motion sequence in z_offset_calibration. |
| 4 | `cebf1416` | `d8d51937a7` | "1" | Trivial: float literal `130.` → `130.0` in z_offset_calibration. |
| 5 | `f83ef3d0` | `b775d99e28` | "1" | Trivial: float literal `65.` → `65.0`. |
| 6 | `49b3e398` | `9312a2756e` | "Fix eddy-current logic" | Small correction to z_offset calibration logic. |
| 7 | `8c4cafc2` | `4eba63f19f` | "fix canbus_uuid" | Adds Sovol Zero board `.config` build files; fixes chip-ID/CAN UUID handling in `chipid.c` and the WS2811 driver. |
| 8 | `f3ad1a09` | `d52b4a86e1` | "fix id" | One-char tweak to a chipid.c comment (CAN UUID note). |
| 9 | `c2da21bb` | `f895b5ee5b` | "1. Rework z_offset calibration logic, add up-to-5-loop pre/post pressure verification; 2. Improve how Z-max is obtained" | Adds a retry loop (max 5) that re-checks pressure before/after, plus better Z-axis max detection. |
| 10 | `7005adf0` | `26d26645f5` | "Modify UI, fan-alarm logic, boot-screen delay" | Menu, fan.py, and WS2811 LED tweaks. |
| 11 | `32c34745` | `e2bd3d3874` | "Add chamber-temperature display" | Adds chamber temp to the menu. |
| 12 | `25264a4f` | `6a5dd47d99` | "1.2.0" | Version bump; board `.config`, menu, mcu.py tweaks. |
| 13 | `5fd5b0ab` | `bf122d4b81` | "Remove redundant code, change internal_endstop_offset compensation logic" | Simplifies z_offset_calibration (−32 lines). |
| 14 | `e38a0111` | `93b121b0ff` | "1. Improve `_start_measurements` device-ID read flow with a retry/probe loop — when I²C is disturbed and returns bad data, recover I²C and re-read to ensure valid data; 2. Add a handler that logs I²C error-report register info" | LDC1612 robustness against I²C glitches. |
| 15 | `fa9274fd` | `dbfbb065f2` | "1. Remove redundant code; 2. Optimize eddy calibration height/frequency sampling — drop the lift-0.5mm-then-move-back-down step to shorten probe time" | Faster eddy calibration. |
| 16 | `00cc8ee0` | `060eada7dc` | "1. Disable i2c_shutdown_on_err for STM32F1 (to be optimized)" | i2ccmds.c change. |
| 17 | `acdb0efe` | `79c9992c52` | "1. Add i2c_bus error code" | New error code in i2ccmds.h. |
| 18 | `a4b711d5` | `738006c948` | "1. Disable i2c_shutdown_on_err for STM32F1 (to be optimized)" | Same intent applied in sensor_ldc1612.c. |
| 19 | `f45b42e0` | `ee6f394f1a` | "1. Improve i2c_busy_errata so it can recover STM32F1 hardware I²C; 2. Change the i2c_wait handling mechanism; 3. Add error-exit handling to the remaining functions" | Major STM32 `i2c.c` reliability rework (+147 lines). |
| 20 | `95e3a3ec` | `42757b9a39` | "z_offset_calibration.py: handle the case internal_endstop_offset=0 so calibration accept passes reliably; i2c.c: remove comments" | (The `\n` in the message is literal text, not a newline.) |
| 21 | `bdb81835` | `ea9486db62` | "Revert '…internal_endstop_offset=0…'" | Reverts the previous commit. |
| 22 | `b0847ff0` | `de564adbe7` | "Restore the changes from the last revert" | Un-reverts #20 — net result re-applies it. |
| 23 | `4b2f4e25` | `265b3f75dd` | "Add delay in i2c_busy_errata" | STM32 i2c.c timing. |
| 24 | `caf96dbb` | `54f10c5ad7` | "1. Modify menu list; 2. Add tip codes" | menu, fan, homing, z_offset, basecmd.c, Kconfig. |
| 25 | `43454cf6` | `14d7b1831d` | "Fix OTA-screen back/return function" | menu.cfg/menu.py navigation fix. |
| 26 | `79f371bc` | `0d9635111d` | "fix c_helper.so" | Adjusts the chelper library loader (`__init__.py`). |
| 27 | `0fd09929` | `d436c9febb` | "i2c.c: fix compile warning; homing.py: fix error when running `self.gcode._process_commands(['M117 Tip code: 102'])`" | |
| 28 | `c1d8dbab` | `fe4df8bb4d` | "i2c.c: cut duplicated code, reduce redundancy; ldc1612: add I²C error-code classification handling" | |
| 29 | `6a33dfd2` | `92beb4c9a1` | "Add tip codes" | Spreads tip/error codes across fan, homing, ldc1612, lis2dw, z_offset, corexy, extruder. |
| 30 | `7d553676` | `60f5f3b0d0` | "Optimize screen display and menu" | menu + stm32h7.c. |
| 31 | `a909ec20` | `99c9e84096` | "Version number change → 1.2.7" | menu.cfg version string. |
| 32 | `5f409b85` | `3a6d0f922c` | "Add eddy-current sensor tip codes" | probe_eddy_current.py. |
| 33 | `f2502dcd` | `c05d7af3fe` | "Change the on-screen time display to actual elapsed time since print start" | display.cfg. |
| 34 | `1f26edd6` | `02792f8bc9` | "Fan warning message, also shown synchronized on the web page" | fan.py. |
| 35 | `a1bdf0ca` | `5e947f8200` | "Add tip code 124, version → 1.2.8" | menu + probe_pressure. |
| 36 | `9e6592fa` | `ff3dd6c551` | "add error code from 60" | klippy.py; also resets file modes on several scripts. |
| 37 | `0a75d053` | `fe6036f703` | "Power-loss-resume (PLR) screen fix" | 断电续打 = resume after power outage; menu fix. |
| 38 | `69a5f393` | `651f2bb1fd` | "Fix Obico connection problem" | menu.cfg/menu.py. |
| 39 | `ae5f2254` | `0ffea31087` | "Add power-on (startup) calibration feature" | menu-driven auto-calibrate at boot. |
| 40 | `ff312671` | `112b04730d` | "Subdivide error code 1" | Splits error code 1 into finer cases (klippy.py, basecmd.c, menu). |
| 41 | `9984988d` | `1c0a784e63` | "Add errors 69, 70" | klippy.py. |
| 42 | `87c4dc2b` | `3afc384eda` | "Loosen buffer processing time" | toolhead.py move-buffer timing + menu. |
| 43 | `d2ae6fe3` | `a95bbe3f37` | "Add calibration skip button" | menu. |
| 44 | `b2625bb0` | `e6462b6f68` | "1" | Just reorders the "Skip Calibrate" menu entry (moves it below the reset menu). |
| 45 | `6d8fbb41` | `9366ec44fe` | "Refine tip codes 101 and 103" | homing.py, corexy.py, menu. |
| 46 | `415d5fa3` | `e88b61ca30` | "Add Z-axis lift to eddy-current calibration" | menu + klippy.py. |
| 47 | `5dec0f5a` | `cc8afd89ab` | "Version 1.3.7" | menu, fan, klippy, stm32h7, ws2811. |
| 48 | `eb90783f` | `b920e83d1c` | "1.3.8" | Large pass touching the whole probe stack: ldc1612, probe, probe_eddy_current, probe_pressure, z_offset_calibration, toolhead, sensor_ldc1612.c, fan, homing, menu. (10 files, +397/−162) |
| 49 | `3c6331fe` | `503f61d64b` | "Open up all input-shaper types; fix the 101 Python error" | 输入整形 = input shaping; shaper_calibrate.py + z_offset + menu. |
| 50 | `3c7bc2fe` | `a471c3aead` | "Keep the ZERO config files" | Deletes all the upstream generic/printer example configs and adds the Sovol Zero set (printer.cfg, Macro.cfg, chamber_hot.cfg, moonraker/obico/crowsnest confs, plr.cfg, etc.). (238 files, −31.6k lines) |
| 51 | `e67ace9e` | `36d4468930` | "Fix calibration showing a 105 Z tip code" | printer.cfg, menu, z_offset_calibration. |
| 52 | `83bbce1c` | `056345db31` | "1.4.3" | Macro.cfg + chamber_hot, menu, fan, heater_fan. |
| 53 | `b60d8185` | `901ee7a896` | "1" | Bug fix in fan.py: `self.start_check_time == 0` (comparison) corrected to `= 0.` (assignment). |
| 54 | `243dc182` | `8a8b5e881f` | "1.4.5" | Macro.cfg, chamber_hot, printer.cfg, bed_mesh, menu, fan, probe_eddy_current. |
| 55 | `429f3b5e` | — | "Uncommitted changes on the 1.4.7 image" | Final HEAD. Per the message, these are changes found on the 1.4.7 disk image but never committed to Sovol's git — just a 2-line menu.cfg tweak. |

### Notes / observations on the history itself

- The `Commit ID:` and "Ignored changes/Conflicts" notes in several message bodies (e.g. #1, #7,
  #20, #26, #36) document what was deliberately dropped during the cleanup import (mode changes,
  `out/`, `__pycache__/`, `.gitignore` churn), and are not part of Sovol's originals.
- A large fraction of commits (#16–#23, #27, #28) are I²C-bus reliability firefighting for the
  STM32F1↔LDC1612 link — clearly the hardest problem in this firmware.
- The Z-offset calibration routine (`z_offset_calibration.py`) is the single most-churned file,
  edited in ~15 of the 55 commits.
- Two commits (#21/#22) are a revert immediately followed by an un-revert, so that pair is
  effectively a no-op against #20.

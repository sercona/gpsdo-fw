# Sercona / Linux-Works Fork — UI Changes

Forked from ESS-1/gpsdo-fw (v0.1.18 / 0.1.22).

This fork is available at: https://github.com/sercona/gpsdo-fw

## Version string

The version screen shows `1.22/Slw` (autosave build) or `1.22/Nlw` (no autosave).
Leading `0.` removed to save display space. `lw` suffix identifies this Linux-Works
/ Sercona fork.

## Hardware tested

- BH3SAP v1.20 board
- OX256B-T-LU-V-10M OCXO (Chinese, not Isotemp)
- ATGM336H-5N31 GPS module (GPS + BeiDou + GLONASS)
- STM32F103C8T6 bluepill (128KB flash silicon)

## Build system changes

- `CMakeLists.txt`: Release build by default, EEPROM_AUTO_SAVE defaults to 1,
  post-build `arm-none-eabi-objcopy` generates `gpsdo.bin` automatically,
  `make flash` target added using PlatformIO's bundled OpenOCD at
  `~/.platformio/packages/tool-openocd`.
- `STM32F103C8Tx_FLASH.ld`: Flash region set to 128K to match actual silicon
  on all bluepill variants encountered. The original 64K limit caused link
  failure in debug builds and was wrong for all 128K-silicon C8T6 parts.

## Display layout changes

The original firmware used col 0 for an animated radio-wave icon cycling through
custom LCD characters on every PPS pulse, causing visible flicker as LCD character
RAM was rewritten while the character was displayed. The lock indicator was a small
symbol embedded in the top corner of the animated character, too small to read at
a glance.

### Lock/unlock icon

- Replaced animated radio-wave with a static padlock icon in custom char slot 1.
- Icon switches between open (unlocked) and closed (locked) only when PPB lock
  status changes — no more flicker.
- Unlocked: open shackle (right leg dropped). Locked: closed shackle.
- 3-wide arch on 5-wide body with keyhole, full 8 rows tall.
- Only one custom char slot used for the icon (was 5), leaving slots free.

### Top-line layout

All screens now follow a consistent layout:

```
[icon] NN <label>
```

- Col 0: lock/unlock icon (custom char 1)
- Col 1: blank separator  
- Col 2+: sat count (2 digits) then label

Previously all sub-screen labels started at col 1 with no separation from the
icon. All `LCD_Puts(1, 0, ...)` calls shifted to `LCD_Puts(2, 0, ...)`.

### Label cleanup

- Removed trailing colons from all top-line labels: PPB, PWM, PPS, UPTIME,
  GGA FR, CNTRST, Vers., and all sub-menu labels.
- GPS screen top line: sat count at col 2, `GPS` label right-aligned at col 5.
- PPB compact format trimmed from `#.##` to `#.#` to fit within available space.

### Trend screen

- Stuck `-` character at col 0 replaced with `T` (Trend), since the trend screen
  uses all 8 custom char slots for its graph and cannot display the padlock icon.

### Notification messages

- Added `LCD_Clear()` and 1.5s hold before clearing after PWM/PPS save
  notifications so the message is readable and no ghost characters remain.

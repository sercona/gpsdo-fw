# Sercona Fork — UI Changes

Forked from ESS-1/gpsdo-fw (v0.1.18).

## Display layout changes

The original firmware used the first character position (col 0) for an animated
radio-wave icon cycling through custom LCD characters on every PPS pulse. This
caused a visible flicker as the LCD character RAM was rewritten while the
character was being displayed. The lock indicator was a small symbol embedded in
the top corner of the animated character, too small to read at a glance.

### Icon

- Replaced the animated radio-wave with a static padlock icon loaded once into
  custom char slot 1. The icon changes between open (unlocked) and closed
  (locked) only when PPB lock status actually changes, eliminating all flicker.
- Unlocked: open shackle (right leg dropped). Locked: closed shackle.
- Both icons use a 3-wide arch on a 5-wide body with keyhole, full 8 rows.

### Top-line layout

All screens now follow a consistent layout:

```
[lock] NN <label>
```

- Col 0: lock/unlock icon (custom char 1)
- Col 1: blank separator
- Col 2+: content

Previously all sub-screen labels started at col 1, leaving no visual separation
from the lock icon. All `LCD_Puts(1, 0, ...)` calls shifted to `LCD_Puts(2, 0, ...)`.

### Labels

- Removed trailing colons from all top-line labels (PPB, PWM, PPS, UPTIME,
  GGA FR, CNTRST, Vers., and all sub-menu labels).
- GPS screen top line reformatted: sat count at col 2, GPS label right-aligned.
- PPB compact format trimmed from `#.##` to `#.#` to fit the wider layout.

### Trend screen

- Stuck `-` character replaced with `T` (for Trend) since the icon can no
  longer animate while the trend screen owns all 8 custom char slots.

### Notification messages

- Added `LCD_Clear()` and 1.5s delay before and after PWM/PPS save notifications
  so the message is readable and no ghost characters are left on screen afterward.

## Build system changes

- `CMakeLists.txt`: Release build by default, EEPROM_AUTO_SAVE defaults to 1,
  post-build `arm-none-eabi-objcopy` step generates `gpsdo.bin` automatically,
  `make flash` target added using PlatformIO's bundled OpenOCD.
- `STM32F103C8Tx_FLASH.ld`: Flash region set to 128K to match actual silicon
  on all bluepill variants encountered in the wild.

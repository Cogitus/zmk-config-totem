<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/docs/images/TOTEM_logo_dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="/docs/images/TOTEM_logo_bright.svg">
  <img alt="TOTEM logo font" src="/docs/images/TOTEM_logo_bright.svg">
</picture>

# ZMK CONFIG FOR THE TOTEM SPLIT KEYBOARD

[Here](https://github.com/GEIGEIGEIST/totem) you can find the hardware files and build guide.\
[Here](https://github.com/GEIGEIGEIST/qmk-config-totem) you can find the QMK config for the TOTEM.

TOTEM is a 38 key column-staggered split keyboard running [ZMK](https://zmk.dev/) or [QMK](https://docs.qmk.fm/). It's meant to be used with a SEEED XIAO BLE or RP2040.


![TOTEM layout](/docs/images/TOTEM_layout.svg)



## HOW TO USE

- fork this repo
- `git clone` your repo, to create a local copy on your PC (you can use the [command line](https://www.atlassian.com/git/tutorials) or [github desktop](https://desktop.github.com/))
- adjust the totem.keymap file (find all the keycodes on [the zmk docs pages](https://zmk.dev/docs/codes/))
- `git push` your repo to your fork
- on the GitHub page of your fork navigate to "Actions"
- scroll down and unzip the `firmware.zip` archive that contains the latest firmware
- connect the left half of the TOTEM to your PC, press reset twice
- the keyboard should now appear as a mass storage device
- drag'n'drop the `totem_left-seeeduino_xiao_ble-zmk.uf2` file from the archive onto the storage device
- repeat this process with the right half and the `totem_right-seeeduino_xiao_ble-zmk.uf2` file.

## Prospector Dongle setup (branch `zmk-dongle`)

This repo also builds firmware for a [Prospector](https://github.com/carrefinho/prospector) dongle — a Xiao BLE with a round ST7789 LCD that becomes the wireless central for the keyboard. Both halves become peripherals that connect to the dongle; the PC pairs with the dongle over BLE.

Integration uses [`carrefinho/prospector-zmk-module`](https://github.com/carrefinho/prospector-zmk-module) on branch `feat/new-status-screens` (the Zephyr 4.1 / ZMK main branch, required for the `xiao_ble//zmk` board used here).

### What the display shows
- Active layer (reads `display-name` from each keymap layer)
- Per-peripheral battery bar
- Peripheral connection status
- BLE profile + output indicator
- Active modifiers
- Caps word indicator

### Repo changes that enable the dongle build
- `config/west.yml` — adds `carrefinho` remote and `prospector-zmk-module` project on `feat/new-status-screens`
- `build.yaml` — adds `- board: xiao_ble//zmk` with `shield: totem_dongle prospector_adapter`
- `config/boards/shields/totem/Kconfig.shield` — registers `SHIELD_TOTEM_DONGLE`
- `config/boards/shields/totem/Kconfig.defconfig` — **drops `ZMK_SPLIT_ROLE_CENTRAL` from `totem_left`** (left is now a peripheral) and adds a `SHIELD_TOTEM_DONGLE` block: central role, `ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2`, `BT_MAX_CONN=7`, `BT_MAX_PAIRED=7`
- `config/boards/shields/totem/totem.zmk.yml` — lists `totem_dongle` as a sibling shield
- `config/boards/shields/totem/totem_dongle.overlay` — mock kscan + matrix transform + `physical_layout0` from `totem-layouts.dtsi`
- `config/boards/shields/totem/totem_dongle.conf` — shield-level placeholder
- `config/totem_dongle.conf` — user-level dongle config: sleep, BT tx power, battery proxy/fetching, `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR=n`, `CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=80`
- `config/totem.keymap` — `display-name` added to every layer (Base, Nav/Num, Sym/Fn, Media, BT, Game, Game#)

### Artifacts produced by the GitHub Actions build
- `totem_left-xiao_ble-zmk.uf2` — flash to left half
- `totem_right-xiao_ble-zmk.uf2` — flash to right half
- `totem_dongle_prospector_adapter-xiao_ble-zmk.uf2` — flash to the Prospector dongle
- `settings_reset-xiao_ble-zmk.uf2` — use first to wipe bonds on every device

### Flashing order (first time, or when switching from non-dongle firmware)
1. Flash `settings_reset` onto **all three** devices (left half, right half, dongle). Left was previously central, so both halves hold stale bonds that must be cleared.
2. Flash the dongle with `totem_dongle_prospector_adapter-xiao_ble-zmk.uf2`.
3. Flash left with `totem_left-xiao_ble-zmk.uf2` and right with `totem_right-xiao_ble-zmk.uf2`.
4. Pair the dongle to your host over BLE (no USB HID on the halves anymore).
5. Power the halves **left first, then right** — Prospector arranges the battery widget in connection order.

### Tweakables in `config/totem_dongle.conf`
| Option | Effect | Current |
| ------ | ------ | ------- |
| `CONFIG_PROSPECTOR_USE_AMBIENT_LIGHT_SENSOR` | Auto-brightness via APDS9960 | `n` |
| `CONFIG_PROSPECTOR_FIXED_BRIGHTNESS` | Fixed brightness 1-100 | `80` |
| `CONFIG_PROSPECTOR_ROTATE_DISPLAY_180` | Flip the screen | unset (`n`) |
| `CONFIG_PROSPECTOR_STATUS_SCREEN_{RADII,FIELD,OPERATOR}` | Alternative status screen layouts (default is Classic) | unset |

If the dongle build fails with `region 'RAM' overflowed`, add `CONFIG_LV_Z_VDB_SIZE=25` to `config/totem_dongle.conf`.

## Disclaimer
There are some adaptations of what was changed by the aliexpress vendor on its [private repository](https://github.com/Keycoon/zmk-config-totem)

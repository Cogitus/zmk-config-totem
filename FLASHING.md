# Flashing the Totem + Prospector Dongle

Step-by-step procedure for flashing and pairing the three devices: left half, right half, and Prospector dongle.

> [!IMPORTANT]
> Do the steps in order. The `settings_reset` step is critical because the **left** half still holds old "I'm the central" bonds from the non-dongle firmware. Without wiping them it will fight the dongle.

## 1. Build the firmware

Push the `zmk-dongle` branch to GitHub. GitHub Actions produces `firmware.zip` containing four files:

- `settings_reset-xiao_ble-zmk.uf2`
- `totem_left-xiao_ble-zmk.uf2`
- `totem_right-xiao_ble-zmk.uf2`
- `totem_dongle_prospector_adapter-xiao_ble-zmk.uf2`

Download and unzip.

## 2. Wipe old pairings (all three devices)

For **each** device (left half, right half, dongle):

1. Plug it into USB.
2. Double-tap the reset button — the Xiao BLE mounts as the `XIAO-BOOT` mass storage device.
3. Drag `settings_reset-xiao_ble-zmk.uf2` onto it. It reboots, wipes bonds, then goes to sleep.

Doing this on the brand-new dongle is redundant but harmless.

## 3. Flash the real firmware

Same double-tap-reset + drag-and-drop, one device at a time:

| Device     | File                                              |
| ---------- | ------------------------------------------------- |
| Left half  | `totem_left-xiao_ble-zmk.uf2`                     |
| Right half | `totem_right-xiao_ble-zmk.uf2`                    |
| Dongle     | `totem_dongle_prospector_adapter-xiao_ble-zmk.uf2`|

## 4. Power-on / pairing sequence

Order matters — Prospector arranges the battery widgets in connection order.

1. **Power off** both halves (unplug USB / flip the power switch).
2. **Plug the dongle** into the PC's USB. The LCD lights up. The dongle advertises itself as `Totem` over BLE.
3. On the PC, pair with `Totem` (Windows Bluetooth / macOS / Linux — same flow as any BT keyboard). This pairs **host ↔ dongle**.
4. **Turn on the left half first.** After ~2 s, the left battery bar appears on the LCD.
5. **Turn on the right half.** The right battery bar appears next to left.
6. Type on either half — keys flow `half → dongle → PC`.

## 5. Sanity check

- LCD shows a layer name (`Base`) → dongle is working.
- Both battery bars visible → halves paired to the dongle.
- Typing from both halves works → split matrix transform is correct.

## Troubleshooting

| Symptom                                         | Fix                                                                                                                                                            |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A half doesn't connect to the dongle            | Double-tap reset on that half so it re-advertises. If still stuck, re-flash `settings_reset` on just that half, then re-flash its firmware.                   |
| Dongle doesn't appear in host's Bluetooth list  | Re-flash `settings_reset` on the dongle, re-flash dongle firmware, retry pairing.                                                                              |
| Right half types as left (or vice-versa)        | Wrong UF2 flashed to that side. Re-flash with the correct file.                                                                                                |
| Battery bars swapped L/R on the LCD             | Right was turned on before left. Power both halves off, then on — left first, right second.                                                                   |
| Build fails with `region 'RAM' overflowed`      | Add `CONFIG_LV_Z_VDB_SIZE=25` to `config/totem_dongle.conf` and rebuild.                                                                                       |
| Want to switch to another paired host           | The dongle owns BT profiles now, not the halves. Use the BT layer on the keymap (combo `21 30 22 29` → `profile_management`) and press `&bt BT_SEL 0..4`.     |

## Daily use

- **Dongle** stays plugged into the PC.
- **Halves** power on/off as usual. They reconnect to the dongle automatically.
- The host only ever sees `Totem` (the dongle) — never the halves directly.

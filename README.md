# Galaxy Book4 Pro — Fingerprint reader on Arch Linux

Working fingerprint unlock for the **Samsung Galaxy Book4 Pro** with the **Egis/LighTuning ETU905A80-E** sensor (`lsusb`: `1c7a:05a1`).

Tested on: Arch Linux (kernel 7.2.6-zen), Hyprland + Noctalia 5.1 lock screen, fprintd 1.94.5, base libfprint 1.94.100.

## The problem

With the **official** libfprint (≤ 1.94.100), the `egismoc` driver for this sensor has a bug:

1. `fprintd-enroll` appears to succeed (`enroll-completed`).
2. The **first** verify / identify / fingerprint use fails with `verify-no-match`, and fprintd logs:

   ```
   Device reported an error during verify: Print was not found on the devices storage.
   Deleted stored finger X for user ... as it is unknown to device.
   ```

3. fprintd then **deletes the enrolled print** — so every fingerprint use (lock screen, sudo, `fprintd-verify`) wipes the enrollment.

Upstream reports: [Launchpad bug #2146729](https://bugs.launchpad.net/ubuntu/+source/libfprint/+bug/2146729) · [Arch Linux forum thread](https://bbs.archlinux.org/viewtopic.php?id=302751) (same sensor, same behavior).

## The fix

Build libfprint from upstream **[MR #547 “Implement SDCP v2 (including egismoc implementation)”](https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/547)**, which rewrites the egismoc driver and fixes the storage/verify issue. This repository contains an Arch PKGBUILD for it.

### 1. Build & install

```bash
git clone https://github.com/lanlanndn/galaxybook4-fingerprint.git
cd galaxybook4-fingerprint
makepkg -si --nocheck
```

- `-s` installs missing build deps (`gtk-doc`, etc.) — needs your sudo password.
- When pacman asks to remove the conflicting official `libfprint`, answer **y**.
- `--nocheck` skips the unit test suite (experimental code; tests don't cover this hardware bug).

### 2. Restart fprintd and enroll

```bash
sudo systemctl restart fprintd
fprintd-enroll -f right-index-finger $USER
fprintd-verify -f right-index-finger $USER   # must print: verify-match
```

### 3. Use it

Once a print is enrolled, the reader is exposed through fprintd:

- **Noctalia** lock screen: works out of the box (it drives fprintd over D-Bus, fingerprint enabled by default).
- **hyprlock**: Arch's default `/etc/pam.d/hyprlock` already contains `pam_fprintd.so`.
- **sudo (optional)**: add `auth sufficient pam_fprintd.so` at the top of `/etc/pam.d/sudo`.

## Rollback & updates

- Rollback to the official package: `sudo pacman -S libfprint`
- `pacman -Syu` will offer to update the official `libfprint` again — **don't let it replace this package** until MR #547 (or an equivalent fix) lands in an official release. Once it does: `sudo pacman -S libfprint`.

## Notes

- Other Galaxy Book4 variants ship a **FocalTech FT9365** sensor (`2808:6553`) — for those, see the AUR package [`libfprint-ft9365`](https://aur.archlinux.org/packages/libfprint-ft9365) instead.
- `PKGBUILD` is based on the official Arch Linux PKGBUILD, with the source pinned to MR #547 commit `2d7c5277` (branch `feature/sdcp-v2`).

## Credits

- [joshuagrisham](https://gitlab.freedesktop.org/joshuagrisham) — author of MR #547
- The libfprint and fprintd teams

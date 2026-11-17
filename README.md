# openwrt-mr3420-tiny

Trimmed **LEDE 17.01.7** firmware for **TP-Link TL-MR3420 v1** (Atheros AR7241,
4 MB flash, 32 MB RAM, 16 MB? no: 32 MB DDR), built with the official
ImageBuilder on GitHub Actions.

## Why this repo exists

The stock 17.01.7 image fills the 3.9 MB firmware partition almost completely
(3840 KB of 3904 KB), leaving only a 384 KB `rootfs_data` overlay. Installing
anything meaningful — let alone `openconnect` — is impossible, and filling that
overlay to ~90% is what causes the overlay to be wiped after a power cut.

This build:

- drops LuCI/uhttpd (manage over SSH instead)
- drops ppp/pppoe and IPv6 helpers (WAN is plain DHCP here)
- keeps the 802.11n AP stack
- bakes in `kmod-fs-ext4`, `block-mount`, `e2fsprogs`, `kmod-usb-storage`,
  `kmod-scsi-core` **inside squashfs**, so preinit can load them and mount a USB
  stick before pivoting the overlay

The point is **extroot**: after flashing, `/overlay` moves to a USB stick
format 2 GB ext4 partition and the router's flash becomes read-only. Any package
(openconnect, LuCI, ...) then installs to the stick and the 4 MB flash is never
written again.

## Build

Push to `main`, or run the `build-mr3420` workflow manually. Artifact:
`mr3420-firmware` → `lede-17.01.7-ar71xx-generic-tl-mr3420-v1-squashfs-sysupgrade.bin`.

## Flash

```sh
scp <image>.bin root@192.168.1.1:/tmp/
ssh root@192.168.1.1 'sysupgrade -n /tmp/<image>.bin'
```

`-n` keeps no configuration (the current config is backed up separately).
If the image misbehaves, Breed still boots: power off, hold reset, power on,
open http://192.168.1.1 and flash a factory image.

## Note

TL-MR3420 v1 was dropped from ar71xx after 17.01 — 18.06/19.07 have no build
for it, so 17.01.7 is the newest official base that boots on 4 MB.

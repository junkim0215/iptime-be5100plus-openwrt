# OpenWrt for ipTIME BE5100plus (preview)

**English** | [한국어](README.ko.md)

Preview builds of OpenWrt for the ipTIME BE5100plus (MediaTek MT7987A, 512 MiB RAM, 128 MiB SPI-NAND, MT7992 Wi-Fi 7 with 2.4 GHz 2x2 and 5 GHz 3x3, 4× 1G LAN, 2.5G WAN, USB 3.0).

Support for this router has been submitted to OpenWrt ([openwrt/openwrt#26729](https://github.com/openwrt/openwrt/pull/26729)) but is not merged yet. These images are built from that pull request without changes to the device support, plus the image tool change it needs ([openwrt/firmware-utils#91](https://github.com/openwrt/firmware-utils/pull/91)). Once the pull request is merged, switch to the official OpenWrt images (see below); this repository is only meant to bridge the time until then.

> [!WARNING]
> This is unofficial firmware. Installing it may void your warranty. Use it at your own risk.

## Status

| Feature | Status |
|---|---|
| Boot, sysupgrade (keeping settings) | Working |
| 4 LAN ports, 2.5G WAN | Working |
| Wired hardware routing offload | Working |
| Wi-Fi access point, 2.4 GHz (2x2) and 5 GHz (3x3) | Working |
| WireGuard | Working |
| CPU LED, reset button, failsafe | Working |
| WPS button | Working |
| LuCI in Korean (chosen automatically from the browser language) | Working |
| Wi-Fi throughput, 160 MHz | Not tested yet |
| USB 3.0 storage (ext4, exFAT, vfat, NTFS) | Detected; storage mount not tested yet |
| Wi-Fi offload (WED) | Off by default (OpenWrt default), not tested |
| Long-term stability | Not tested yet |

Tested on a single unit.

### What is in the image

- OpenWrt main (snapshot) with LuCI and its Korean translation
- USB storage: ext4, exFAT, vfat and NTFS (`kmod-usb-storage`, `kmod-usb-storage-uas`, `block-mount`, `kmod-fs-*`, `kmod-nls-*`)
- WireGuard (`kmod-wireguard`, `wireguard-tools`, `luci-proto-wireguard`)

## Downloads

Get the files from [Releases](../../releases).

| File | Use |
|---|---|
| `…-squashfs-factory.bin` | first install through the U-Boot recovery page |
| `…-squashfs-sysupgrade.bin` | install / upgrade from OpenWrt |
| `…-initramfs-kernel.bin` | RAM boot (for testing and recovery) |

## Installing

Install through the recovery page of the stock U-Boot. It works from the stock firmware and from OpenWrt. Installing from the stock web UI is not supported.

1. Download the `*-squashfs-factory.bin` and `*-squashfs-sysupgrade.bin` files.
2. Press and hold the reset button, then power up the router.
3. Keep holding the button until the CPU LED stops blinking and stays on (about 10 seconds).
4. Connect the PC to a LAN port and set its address to `192.168.0.100/24`.
5. Open `http://192.168.0.1`, upload the `*-squashfs-factory.bin` file and wait until the page shows that the upgrade is complete.
6. The router reboots into an OpenWrt initramfs at `192.168.1.1`.
7. Install the `*-squashfs-sysupgrade.bin` file in either way:
   - LuCI: open `http://192.168.1.1` (user `root`, no password), go to System → Backup / Flash Firmware → Flash image, choose the file and continue, or
   - SSH: copy the file to `/tmp` and run:
     ```
     sysupgrade -n /tmp/*squashfs-sysupgrade.bin
     ```
8. Log in to LuCI at `http://192.168.1.1` (user `root`, no password) and set a root password.

## Upgrading

From inside OpenWrt, use LuCI → System → Backup / Flash Firmware and upload a `sysupgrade.bin` (keeping settings is fine). Do not use `-F` (force).

sysupgrade writes the same image to both stock slots. The stock U-Boot may switch between them after warm reboots; both hold the same OpenWrt.

## Switching to official OpenWrt (once the device becomes supported)

These images use the same device name (`iptime,be5100p`) and flash layout as the official support in #26729. After it is merged:

1. Get the official `…-iptime_be5100p-squashfs-sysupgrade.bin` from the [OpenWrt firmware selector](https://firmware-selector.openwrt.org/) (snapshot builds first, releases later).
2. Upload it in LuCI → System → Backup / Flash Firmware. Keeping settings should work; if anything looks wrong afterwards, upgrade again without keeping settings.
3. Official images may not include LuCI, its Korean translation or the extra packages of this preview. Install what you use from the official feeds, for example:
   ```
   apk update
   apk add luci luci-i18n-base-ko luci-i18n-firewall-ko luci-i18n-package-manager-ko
   apk add kmod-usb-storage kmod-usb-storage-uas block-mount kmod-fs-ext4 kmod-fs-exfat kmod-fs-vfat kmod-fs-ntfs3 kmod-nls-cp437 kmod-nls-iso8859-1 kmod-nls-utf8 kmod-wireguard wireguard-tools luci-proto-wireguard
   ```
   You can also pick the packages before downloading with "Customize installed packages" in the firmware selector.
4. From then on, use official images and the official package feeds.

## Returning to stock

1. Boot OpenWrt into failsafe mode (press the reset button while the CPU LED blinks during boot).
2. Connect the PC to LAN1 (failsafe only uses this port), set its address to `192.168.1.2/24` and log in with `ssh root@192.168.1.1`.
3. Delete the `rootfs_data` volume:
   ```
   ubirmvol /dev/ubi0 -N rootfs_data
   ```
4. Power off the router and follow steps 2–5 of the installation with the stock firmware file from the [ipTIME download page](https://iptime.com/support/downloads).

The recovery page does not remove `rootfs_data`, which OpenWrt grows to fill the flash, so the stock image does not fit without step 3. If it fails anyway, OpenWrt still boots from the other slot.

## Known limitations

- **Official kernel modules cannot be installed.** The image is not built by OpenWrt, so `apk add kmod-…` from the official feeds does not work (the kernel does not match). User-space packages from the official feeds can be installed. Kernel features that are not in the image need a newer release here, or the official images once #26729 is merged.
- The power LED is always on, whatever its brightness setting; it seems to be wired straight to the power supply on the board.
- Installing from the stock web UI is not supported; use the U-Boot recovery page.
- The stock reset-button recovery page has been tested for installing OpenWrt and going back to stock only.

## Source and building

The source is the `be5100plus-preview` branch of [junkim0215/openwrt](https://github.com/junkim0215/openwrt): the branch of #26729 (`iptime-be5100p`) plus preview-only commits that do not change the device support. Each release is tagged at the commit it was built from; the tag and commit are listed in the release notes.

```
git clone https://github.com/junkim0215/openwrt.git && cd openwrt
git checkout <tag>
./scripts/feeds update -a && ./scripts/feeds install -a
cp be5100plus-preview.config .config && make defconfig && make -j$(nproc)
```

## Other ipTIME devices

- [ipTIME BE5100M](https://github.com/junkim0215/iptime-be5100m-openwrt) (Airoha AN7563)
- [ipTIME BE3600plus](https://github.com/junkim0215/iptime-be3600plus-openwrt) (Airoha AN7563)

## Credits

- The OpenWrt project and the MediaTek Filogic target maintainers
- This port was written with the help of Claude (Anthropic), and the changes were tested on real hardware.

Licensed under GPL-2.0.

The image also contains binary firmware for Wi-Fi and the 2.5G PHY. See the license information of the corresponding OpenWrt packages.

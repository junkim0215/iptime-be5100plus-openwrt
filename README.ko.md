# OpenWrt for ipTIME BE5100plus (미리보기)

[English](README.md) | **한국어**

ipTIME BE5100plus(MediaTek MT7987A, 512 MiB RAM, 128 MiB SPI-NAND, MT7992 Wi-Fi 7(2.4 GHz 2x2, 5 GHz 3x3), 1G LAN 4개, 2.5G WAN, USB 3.0)용 OpenWrt 미리보기 빌드입니다.

이 공유기 지원은 OpenWrt에 제출되어 있지만([openwrt/openwrt#26729](https://github.com/openwrt/openwrt/pull/26729)) 아직 머지되지 않았습니다. 이 이미지는 그 PR의 기기 지원을 바꾸지 않고 그대로 빌드하고, 필요한 이미지 도구 수정([openwrt/firmware-utils#91](https://github.com/openwrt/firmware-utils/pull/91))만 더한 것입니다. PR이 머지되면 공식 OpenWrt 이미지로 갈아타세요(아래 참고). 이 저장소는 그때까지만 쓰는 용도입니다.

> [!WARNING]
> 비공식 펌웨어입니다. 설치하면 제조사 보증이 사라질 수 있습니다. 모든 책임은 사용자 본인에게 있습니다.

## 상태

| 기능 | 상태 |
|---|---|
| 부팅, sysupgrade(설정 유지) | 동작 |
| LAN 4포트, 2.5G WAN | 동작 |
| 유선 하드웨어 라우팅 오프로드 | 동작 |
| Wi-Fi AP, 2.4 GHz(2x2)·5 GHz(3x3) | 동작 |
| WireGuard | 동작 |
| CPU LED, 리셋 버튼, failsafe | 동작 |
| WPS 버튼 | 미확인 |
| LuCI 한국어(브라우저 언어에 따라 자동) | 동작 |
| Wi-Fi 처리량, 160 MHz | 미확인 |
| USB 3.0 저장장치(ext4, exFAT, vfat, NTFS) | 미확인 |
| Wi-Fi 오프로드(WED) | 기본 꺼짐(OpenWrt 기본값), 미시험 |
| 장시간 안정성 | 미확인 |

한 대의 기기에서만 확인했습니다.

### 이미지에 들어 있는 것

- OpenWrt main(snapshot)과 LuCI, LuCI 한국어 번역
- USB 저장장치: ext4, exFAT, vfat, NTFS (`kmod-usb-storage`, `kmod-usb-storage-uas`, `block-mount`, `kmod-fs-*`, `kmod-nls-*`)
- WireGuard (`kmod-wireguard`, `wireguard-tools`, `luci-proto-wireguard`)

## 다운로드

[Releases](../../releases)에서 받습니다.

| 파일 | 용도 |
|---|---|
| `…-squashfs-factory.bin` | U-Boot 복구 페이지로 첫 설치 |
| `…-squashfs-sysupgrade.bin` | OpenWrt에서 설치·업그레이드 |
| `…-initramfs-kernel.bin` | RAM 부팅(시험·복구용) |

## 설치

제조사 웹 UI는 OpenWrt 이미지를 거부합니다. 대신 제조사 U-Boot의 복구 페이지를 씁니다. 제조사 펌웨어에서도, OpenWrt에서도 됩니다.

1. `*-squashfs-factory.bin`과 `*-squashfs-sysupgrade.bin`을 받습니다.
2. 리셋 버튼을 누른 채로 전원을 켭니다.
3. CPU LED가 깜박임을 멈추고 계속 켜질 때까지(약 10초) 버튼을 누르고 있습니다.
4. PC를 LAN 포트에 연결하고 주소를 `192.168.0.100/24`로 설정합니다.
5. `http://192.168.0.1`을 열고 `*-squashfs-factory.bin`을 올린 뒤, 업그레이드 완료가 표시될 때까지 기다립니다.
6. 공유기가 OpenWrt initramfs(`192.168.1.1`)로 재부팅됩니다.
7. `*-squashfs-sysupgrade.bin`을 설치합니다. 다음 둘 중 하나로 합니다.
   - LuCI: `http://192.168.1.1`(사용자 `root`, 비밀번호 없음) → 시스템 → 백업/펌웨어 업그레이드 → 이미지 플래시에서 파일을 고르고 진행합니다.
   - SSH: 파일을 `/tmp`에 복사하고 실행합니다.
     ```
     sysupgrade -n /tmp/*squashfs-sysupgrade.bin
     ```
8. `http://192.168.1.1`의 LuCI에 로그인(사용자 `root`, 비밀번호 없음)해 root 비밀번호를 설정합니다.

## 업그레이드

OpenWrt 안에서 LuCI → 시스템 → 백업/펌웨어 업그레이드로 `sysupgrade.bin`을 올립니다(설정 유지 가능). `-F`(강제)는 쓰지 마세요.

sysupgrade는 제조사의 두 슬롯에 같은 이미지를 씁니다. 제조사 U-Boot가 재부팅 중 슬롯을 바꿀 수 있지만 두 슬롯 모두 같은 OpenWrt입니다.

## 공식 OpenWrt로 갈아타기

이 이미지는 #26729의 공식 지원과 같은 기기 이름(`iptime,be5100p`)과 플래시 배치를 씁니다. PR이 머지되면:

1. [OpenWrt firmware selector](https://firmware-selector.openwrt.org/)에서 공식 `…-iptime_be5100p-squashfs-sysupgrade.bin`을 받습니다(처음에는 snapshot, 나중에 정식 릴리스).
2. LuCI → 시스템 → 백업/펌웨어 업그레이드에서 올립니다. 설정 유지로 될 것입니다. 이후 이상하면 설정 유지 없이 한 번 더 업그레이드하세요.
3. 공식 이미지에는 LuCI, LuCI 한국어 번역, 이 미리보기에 더한 패키지가 없을 수 있습니다. 쓰는 것을 공식 저장소에서 설치하세요. 예:
   ```
   apk update
   apk add luci luci-i18n-base-ko luci-i18n-firewall-ko luci-i18n-package-manager-ko
   apk add kmod-usb-storage kmod-usb-storage-uas block-mount kmod-fs-ext4 kmod-fs-exfat kmod-fs-vfat kmod-fs-ntfs3 kmod-nls-cp437 kmod-nls-iso8859-1 kmod-nls-utf8 kmod-wireguard wireguard-tools luci-proto-wireguard
   ```
   firmware selector의 "Customize installed packages"에서 받기 전에 패키지를 고를 수도 있습니다.
4. 그 뒤로는 공식 이미지와 공식 패키지 저장소를 씁니다.

## 제조사 펌웨어로 되돌리기

1. OpenWrt를 failsafe 모드로 부팅합니다(부팅 중 CPU LED가 깜박일 때 리셋 버튼을 누름).
2. PC를 LAN1에 연결하고(failsafe는 이 포트만 씀) 주소를 `192.168.1.2/24`로 설정한 뒤 `ssh root@192.168.1.1`로 접속합니다.
3. `rootfs_data` 볼륨을 지웁니다.
   ```
   ubirmvol /dev/ubi0 -N rootfs_data
   ```
4. 전원을 끄고, [ipTIME 다운로드 페이지](https://iptime.com/support/downloads)에서 받은 제조사 펌웨어로 설치 2~5단계를 따라 합니다.

복구 페이지는 `rootfs_data`를 지우지 않는데, OpenWrt가 이 볼륨을 플래시 끝까지 키워 두기 때문에 3단계 없이는 제조사 이미지가 들어가지 않습니다. 그래도 실패하면 OpenWrt는 다른 슬롯에서 그대로 부팅됩니다.

## 알려진 제한 사항

- **공식 커널 모듈을 설치할 수 없습니다.** OpenWrt가 빌드한 이미지가 아니어서 공식 저장소의 `apk add kmod-…`는 동작하지 않습니다(커널이 맞지 않음). 공식 저장소의 일반 패키지는 설치할 수 있습니다. 이미지에 없는 커널 기능은 이 저장소의 새 릴리스나, #26729 머지 후 공식 이미지가 필요합니다.
- 전원 LED는 밝기 설정과 상관없이 항상 켜져 있습니다. 보드에서 전원에 바로 연결된 것으로 보입니다.
- 제조사 웹 UI로는 OpenWrt를 설치할 수 없습니다. U-Boot 복구 페이지를 쓰세요.
- 리셋 버튼 복구 페이지는 OpenWrt 설치와 제조사 펌웨어 복귀에만 시험했습니다.

## 소스와 빌드

소스는 [junkim0215/openwrt](https://github.com/junkim0215/openwrt)의 `be5100plus-preview` 브랜치입니다. #26729의 브랜치(`iptime-be5100p`)에 기기 지원을 바꾸지 않는 미리보기 전용 커밋만 더했습니다. 릴리스마다 빌드한 커밋에 태그가 있고, 태그와 커밋은 릴리스 노트에 적혀 있습니다.

```
git clone https://github.com/junkim0215/openwrt.git && cd openwrt
git checkout <tag>
./scripts/feeds update -a && ./scripts/feeds install -a
cp be5100plus-preview.config .config && make defconfig && make -j$(nproc)
```

## 다른 ipTIME 기기

- [ipTIME BE5100M](https://github.com/junkim0215/iptime-be5100m-openwrt) (Airoha AN7563)
- [ipTIME BE3600plus](https://github.com/junkim0215/iptime-be3600plus-openwrt) (Airoha AN7563)

## 크레딧

- OpenWrt 프로젝트와 MediaTek Filogic 타겟 메인테이너
- 이 포트는 Claude(Anthropic)의 도움을 받아 작성했고, 변경 사항은 실제 기기에서 시험했습니다.

라이선스는 GPL-2.0입니다.

이미지에는 Wi-Fi와 2.5G PHY용 바이너리 펌웨어가 포함되어 있습니다. 라이선스는 해당 OpenWrt 패키지의 정보를 따릅니다.

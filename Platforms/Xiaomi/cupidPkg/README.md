# cupid firmware platform

UEFI firmware for the **Xiaomi 12** (`cupid`, Qualcomm **SM8450**), built from
this Mu-Silicium fork for UniXNU. It is the firmware that carries UniXNU,
DarwinBoot and the payload ramdisk into the boot image.

## State

| Item | Value |
| --- | --- |
| SoC package | `Silicon/Qualcomm/WaipioPkg` (SM8450, 8 cores, GIC 0x17100000) |
| Vendor hardware | **off** (`CUPID_VENDOR_HW = 0`), no `Binaries/cupid` needed |
| Display | `SimpleFbDxe` on ABL's scan-out, 1080x2400 |
| FD window | `0xA7100000` + `0x300000` (stock 4 MiB window at `0xA7000000`) |
| Boot image header | v0, 4096-byte pages |

The minimal configuration is deliberate: it produces a firmware volume with the
UniXNU menu, the XNU loader, the payload FAT driver and the shell without any
stock vendor module. `Binaries/cupid` is only required for
`CUPID_VENDOR_HW = 1` (stock clock, RPMh, UFS, USB, charger and display
drivers) and for the `SecParti.cfg` raw file.

## Memory map provenance

`Library/MemoryMapLib/MemoryMapLib.c` and
`Library/ConfigurationMapLib/ConfigurationMapLib.c` are copied from the
**ingres** platform, the other Xiaomi SM8450 (waipio) board in this tree:
same DRAM base `0x80000000`, same FD window and the same SoC carveouts
(HYP, SMEM, PIL, TZApps, DXE heap). `ingres` is 1080x2400 as well.

**REQUIRED before flashing**: replace both tables with cupid's own map from the
stock `uefiplat.cfg` (HyperOS `3.0.4.0.VLCMIXM`, `uefi.img`), and confirm the
FD base/size in `Resources/Configs/cupid.toml`. Board reservations differ even
between boards of the same SoC.

## Build

The device repository drives the build:

```sh
cd ../UniXNU-cupid
scripts/install-os-profile.sh ../UniXNU
cd ../UniXNU
./unixnu defconfig cupid_ios27_defconfig -D ../UniXNU-cupid
./unixnu fetch -D ../UniXNU-cupid
./unixnu build firmware -D ../UniXNU-cupid --uefi ../uefi-cupid
```

The expected output is
`Build/cupidPkg/{DEBUG,RELEASE}_CLANGPDB/FV/SILICIUM_UEFI.fd`.
The checkout needs its submodules (`Common/Mu`, `Common/Mu_OEM_Sample`,
`Mu_Basecore`, `Silicium-ACPI`, the OpenSSL submodule, and `Binaries` when
vendor hardware is enabled).

## Not yet done

- Import cupid's stock UEFI modules into `Binaries/cupid`
  (`QcomPkg/Drivers/*`, `RawFiles/SecParti.cfg`, panel XMLs).
- Verify the memory map and FD base on the phone.
- Vendor display chain (`USE_CUSTOM_DISPLAY_DRIVER = 1`) with the L3 panel XML.
- ACPICA/ACPI tables for cupid (the minimal build omits them).

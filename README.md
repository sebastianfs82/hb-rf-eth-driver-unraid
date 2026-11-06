# hb-rf-eth-driver-unraid

Unraid plugin for the `hb_rf_eth` / `generic_raw_uart` kernel modules, built from
[alexreinert/piVCCU](https://github.com/alexreinert/piVCCU) for use with the
HB-RF-ETH network radio module adapter (e.g. with [OpenCCU](https://github.com/jens-maus/OpenCCU)).

Current build targets kernel `6.18.58-Unraid` (Unraid 7.3.4). `vermagic` of
both `.ko` files matches that kernel exactly.

## Install

In the Unraid WebUI: **Plugins → Install Plugin → Enter URL of remote plugin
file**, paste:

```
https://raw.githubusercontent.com/sebastianfs82/hb-rf-eth-driver-unraid/main/hb-rf-eth-driver.plg
```

This downloads and installs `hb-rf-eth-driver-6.18.58u-x86_64-1.txz`, which
drops the two `.ko` files into `/lib/modules/6.18.58-Unraid/kernel/extra/` and
runs `depmod`/`modprobe` automatically. Since Unraid re-applies installed
plugins on every boot, this persists across reboots without any manual
`/boot/config/go` edits.

Verify after install:
```sh
lsmod | grep hb_rf_eth
cat /sys/class/hb-rf-eth/hb-rf-eth/is_connected
```

## Configure OpenCCU

**Settings → Control Panel → Advanced Settings → IP-Adresse (HB-RF-ETH)** —
enter the HB-RF-ETH's IP address and restart OpenCCU. Its init script
(`S47InitRFHardware`) handles `modprobe` and connecting automatically.

## Rebuilding for a different kernel version

These modules only load on the exact kernel they were built against. To
rebuild for a new Unraid kernel version:

1. Download the matching kernel tree from `ich777/unraid_kernel` GitHub
   releases (tag = target `uname -r`), verify its `.sha256`, extract it.
2. Install the matching gcc version (shown in that release's build warning).
3. Build `generic_raw_uart.c` + `hb_rf_eth.c` (+ `generic_raw_uart.h`,
   `devkey.inc`, `stack_protector.include` from piVCCU's `kernel/` folder)
   with `obj-m += generic_raw_uart.o hb_rf_eth.o` against that kernel tree.
4. Package the two `.ko` files into a new `.txz` (same layout:
   `lib/modules/<kver>/kernel/extra/*.ko`, plus `install/doinst.sh` running
   `depmod -a`/`modprobe`), bump version/kver/pkgver in `hb-rf-eth-driver.plg`,
   update the `<MD5>`, commit and push.

## License

The kernel module sources are from [alexreinert/piVCCU](https://github.com/alexreinert/piVCCU)
(GPL). This repo redistributes unmodified-source builds; see upstream for the
original source and license.

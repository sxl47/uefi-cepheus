# uefi-cepheus

小米 9 (cepheus, SM8150) 的 UniXNU UEFI 固件构建仓库。

## 它做什么

uefi-sheng(UniXNU 官方 Mu-Silicium fork)**自带** `cepheusPkg`(Windows 移植向,
Daniel224455 维护,Display/Internal Storage/USB Device Mode/Mass Storage 均 ✅),
但它的 FV 里没有 UniXNU 引导组件。本仓库在 CI 里:

1. 按 pin 的 commit 克隆 `UniXNU/uefi-sheng`
2. 先构建 **stock** 固件(上游原样 cepheusPkg)→ `Mu-cepheus-stock.img`
3. 应用 `overlay/unixnu-fv.patch`(把 UniXNU 菜单 + UniXNUDxe + DarwinBoot 注入 FV,
   依据 shengPkg 同款接线 + nabu 的 APRIORI 经验)→ `Mu-cepheus-unixnu.img`

## overlay/unixnu-fv.patch 内容(5 处)

| 文件 | 改动 |
|---|---|
| `cepheus.dsc` | `[LibraryClasses]` 加 UniXNULib + FdtLib;`[Components]` 加 UniXNU/UniXNUDxe/DarwinBoot 三个 INF |
| `cepheus.fdf` | FvMain 加 UniXNU 菜单应用 + UniXNUDxe |
| `Include/DXE.inc` | 加 DarwinBoot(firmware volume 内,卷加载的驱动会被镜像校验拒绝) |
| `Include/APRIORI.inc` | 第一个 APRIORI DXE 块加 DarwinBoot(nabu 经验:否则 LocateProtocol(UniXNULoader) 找不到) |
| `DeviceBuild.py` | PackagesPath 加 `UniXNU-Pkg-Tree` / `DarwinBoot-Tree`(CI 里是指向 UniXNU 伞仓库 package/ 与 drivers/DarwinBoot 的符号链接) |

## 真机使用

```
fastboot boot Mu-cepheus-unixnu.img
```

只进内存不刷分区,断电即恢复。预期现象:屏幕出 UniXNU 菜单(FV 内置)或
stock 固件的 UEFI 界面。UEFI 阶段日志看串口(fastboot boot 的内核日志无 UART 时靠
帧缓冲 DEBUG 输出,本 CI 构 DEBUG 版)。

## FD 布局(来自 Resources/Configs/cepheus.toml)

- base `0x9FC00000`,size `0x300000`(3MB,同 nabu;不要放大)
- boot image: gzip 压缩 + BootShim + 附带 `Resources/DTBs/cepheus.dtb`

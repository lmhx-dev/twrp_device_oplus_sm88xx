# TWRP device tree for OPLUS sm88xx series (SM8850)

高通 **SM8850**（平台代号 `canoe`）欧加系设备的 TWRP 设备树。
由 `https://github.com/kmiit/twrp_device_oplus_sm87xx`（SM87xx / `sun`）复制并适配而来。

## 支持机型

- Realme GT8 Pro（国行）— RMX5200 / `lafa` / RE6030L1

新增机型：在 `libinit/libinit_oplus_sm88xx.cpp` 的 `kModelInfoMap` 里追加条目，
key 用设备的 `prjname`（取自 `ro.boot.prjname` / 内核 cmdline 的 `oplusboot.prjname`）。

## Build it yourself?

```shell
mkdir twrp && cd twrp
repo init --depth=1 -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp.git -b twrp-16.0
repo sync
git clone --depth=1 <this repo> device/oplus/sm88xx
```

```shell
source build/envsetup.sh
lunch twrp_sm88xx
make recoveryimage
```

If there is no error, `recovery.img` will be found in
`out/target/product/sm88xx/recovery.img`

## Features

Works:

- [X] ADB
- [X] Display
- [X] Decryption
- [X] Fastbootd
- [X] Flashing
- [X] MTP
- [X] Sideload
- [X] Touch
- [X] USB OTG
- [X] Vibrator

## To use it:

```shell
fastboot flash recovery recovery.img
```

or

```shell
fastboot flash recovery_a recovery.img
fastboot flash recovery_b recovery.img
```

---

# 改动记录（相对 `device/oplus/sm87xx`）

## 1. 重命名

| sm87xx | sm88xx |
| --- | --- |
| `sm87xx/` | `sm88xx/` |
| `twrp_sm87xx.mk` | `twrp_sm88xx.mk` |
| `libinit/libinit_oplus_sm87xx.cpp` | `libinit/libinit_oplus_sm88xx.cpp` |
| `recovery/root/vendor/bin/init.kernel.post_boot-sun_default_6_2.sh` | `...-canoe_default_6_2.sh` |

## 2. `BoardConfig.mk`

| 配置项 | sm87xx | sm88xx | 数据来源 |
| --- | --- | --- | --- |
| `TARGET_CPU_VARIANT` | `oryon` | `generic` | — |
| `TARGET_CPU_VARIANT_RUNTIME` | *(无)* | `oryon` | — |
| `PRODUCT_PLATFORM` | `sun` | `canoe` | 内核 cmdline |
| `TARGET_BOOTLOADER_BOARD_NAME` | `sun` | `canoe` | 内核 cmdline |
| `TARGET_BOARD_PLATFORM` | `sm87xx` | `sm8850` | 内核 cmdline |
| `TARGET_BOARD_PLATFORM_GPU` | *(无)* | `qcom-adreno840` | — |
| `QCOM_BOARD_PLATFORMS` | `sm87xx` | `sm8850` | — |
| `TARGET_INIT_VENDOR_LIB` | `libinit_oplus_sm87xx` | `libinit_oplus_sm88xx` | — |
| `TARGET_RECOVERY_DEVICE_MODULES` | `libinit_oplus_sm87xx` | `libinit_oplus_sm88xx` | — |
| `BOARD_SUPER_PARTITION_SIZE` | `15569256448` | `18907922432` | `/proc/partitions` |
| `BOARD_QTI_DYNAMIC_PARTITIONS_SIZE` | `15565062144` | `18903728128` | super − 4 MiB |
| `BOARD_QTI_DYNAMIC_PARTITIONS_PARTITION_LIST` | *(无 `system_dlkm`)* | `+ system_dlkm` | `lpdump` |
| `BOARD_ODMIMAGE_FILE_SYSTEM_TYPE` | `ext4` | `erofs` | `mount` |
| `TARGET_COPY_OUT_VENDOR_DLKM` | *(无)* | `vendor_dlkm` | 见注意事项 4 |
| `BOARD_VENDOR_DLKMIMAGE_FILE_SYSTEM_TYPE` | *(无)* | `ext4` | `mount` |
| `TW_DEFAULT_BRIGHTNESS` | `1000` | `2047` | — |
| `TW_MAX_BRIGHTNESS` | `2047` | `4094` | `/sys/class/backlight/panel0-backlight/max_brightness` |
| `TW_DEVICE_VERSION` | `OPLUS-SM87XX` | `Realme_GT_8_Pro` | — |
| `TW_CUSTOM_CPU_TEMP_PATH` | `thermal_zone45` | `thermal_zone5` | 见注意事项 5 |
| `TW_LOAD_VENDOR_MODULES` | 4 个模块 | 32 个模块（lafa 触摸/振动集合） | — |

`BOARD_RECOVERYIMAGE_PARTITION_SIZE`（`0x6400000`）**未改动** —— 与 lafa `recovery` 的 104857600 字节相同。

`device.mk`、`system.prop`、`recovery.fstab`、`Android.bp` 和 OTA 证书与 sm87xx
**逐字节相同**，无需改动。fstab 里的 `sysfs_path=/sys/devices/platform/soc/1d84000.ufshc`
本来就与 lafa 一致。

## 3. `libinit/libinit_oplus_sm88xx.cpp`

在 `kModelInfoMap` 中新增 lafa 条目（prjname `24624`）：

```cpp
{24624, {"realme",  "RE6030L1", "realme",  "RMX5200", "RMX5200", "Realme_GT_8_Pro",     "0"}}, // RMX5200 CN (lafa)
```

default 条目由 `SM87XX` 改名为 `SM88XX`。`libinit/Android.bp` 同步更新了库名与源文件名。

lafa 沿用默认的 `twrp.se.no_sb=false`（该机确实带 strongbox HAL）。

`SetupModelProperties()` 的 `props[]` 里新增了一条（见注意事项 7）：

```cpp
{"ro.color597.product_name",   info.model},   // TWRP MTP reads this for the model name
```

## 4. `recovery/root/init.recovery.qcom.rc`

- `post_boot-sun_default_6_2.sh` → `post_boot-canoe_default_6_2.sh`
- 在 `on property:vendor.sys.listeners.registered=true` 下新增
  `start vendor.keymint-strongbox`（见注意事项 3）

## 5. `recovery/root/` 下的设备资源

替换（机型相关）：

| sm87xx | sm88xx |
| --- | --- |
| `vendor/odm/firmware/tp/ktm/`、`.../vw/` | `vendor/odm/firmware/tp/lafa/` |
| `vendor/odm/etc/vibrator/809/` | `vendor/odm/etc/vibrator/816/` |
| `vendor/odm/etc/vibrator/vibrator_effect.json` | lafa 原厂版本（34789 字节） |

新增：

- `vendor/odm/lib64/android.hardware.secure_element-V1-ndk.so`
- `vendor/odm/lib64/android.hardware.security.sharedsecret-V1-ndk.so`
- `vendor/odm/lib64/vendor.oplus.hardware.olc2-V2-ndk.so`（见注意事项 2）

上面两个 `android.hardware.*` 库在 lafa 上存在，但 sm87xx 的 blob 集合里没有 ——
secure-element / strongbox 这条链路需要它们。

## 6. keymint 替换为 OneKeyMint

删除：

- `recovery/root/vendor/bin/hw/android.hardware.security.keymint-service-qti`
- `recovery/root/vendor/etc/init/android.hardware.security.keymint-service-qti.rc`

新增：

- `recovery/root/vendor/bin/hw/android.hardware.security.onekeymint-service-qti`
- `recovery/root/vendor/etc/init/android.hardware.security.onekeymint-service-qti.rc`

见注意事项 1 —— 这是解密能工作的关键。

## 7. VINTF 片段复制进 `/system`

新增 `recovery/root/system/etc/vintf/manifest/`，含 14 个片段：

```
android.hardware.gatekeeper.xml
android.hardware.health-service.qti.xml
android.hardware.secure_element.xml
android.hardware.security.keymint-service-qti.xml
android.hardware.security.keymint3-service.strongbox.nxp.xml
android.hardware.security.sharedsecret3-service.strongbox.nxp.xml
android.hardware.weaver-service.nxp.xml
boot-service.qti.xml
manifest_oplus_keymint_aidl.xml
manifest_oplus_weaver_aidl.xml
manifest_touch_aidl.xml
secure_element-service.xml
vendor.qti.hardware.qseecom@1.0-service.xml
vibrator-default.xml
```

其中 `android.hardware.gatekeeper.xml` 是新写的（gatekeeper 的 HAL 原本内联声明在
`/vendor/etc/vintf/manifest.xml` 里，不是独立片段），其余 13 个是 vendor/odm 片段的副本。
见注意事项 6。

---

# 注意事项

## 注意事项 1 —— 解密依赖 OneKeyMint HAL

原厂 `android.hardware.security.keymint-service-qti`（那个约 85 KB、包装
`libqtikeymint.so` 的 C++ 版本）在 recovery 里会失败：

```
E android.hardware.security.keymint-service-qti: deserializeClientBegin
E android.hardware.security.keymint-service-qti: ret: -1
E KeyMasterHalDevice: keymint_begin_operation
E KeyMasterHalDevice: ret: -1
E keystore2: Error::Km(r#ROOT_OF_TRUST_ALREADY_SET)
E recovery: decryptWithKeystoreKey failed
```

随后 `/metadata` 在开机时解密失败，`/data` 保持加密。

解法是改用高通的 **OneKeyMint** HAL
（`vendor/qcom/proprietary/securemsm/onekeymint/`）。它是自包含的 Rust 实现，
会**等 `vendor.sys.listeners.registered` 之后才建立 TEE 通道** —— 原厂包装建得太早，
TEE 握手会失败。它注册的**服务名完全相同**，因此只需改可执行文件路径：

```
service vendor.keymint-qti /vendor/bin/hw/android.hardware.security.onekeymint-service-qti
```

它只依赖 `liblog`、`libbinder_ndk`、`libc` 和 `libminkdescriptor.so`。

**不要换回原厂 keymint 二进制。**

## 注意事项 2 —— `vendor.oplus.hardware.olc2-*-ndk.so`

`libolc_vnd.so` 链接特定版本的 olc2 AIDL。本树继承自 sm87xx 的 `libolc_vnd.so`
需要 **V3**；而 OrangeFox 的 lafa 树里那份 `libolc_vnd.so` 需要 V2。
两个版本都保留，这样触摸 HAL 无论哪种都能链接上。删掉任意一个都会导致触摸报
`CANNOT LINK EXECUTABLE`。

## 注意事项 3 —— strongbox

`vendor.keymint-strongbox` 在其 rc 里是 `disabled`，而 `twrp_secure_element` 触发器里的
`enable` 不能可靠地拉起它，所以 `init.recovery.qcom.rc` 里额外显式启动了一次。
没有这一行，keystore2 会反复打印
`Unable to connect "android.hardware.security.sharedsecret.ISharedSecret/strongbox"`。

已知限制：strongbox 的 javacard 初始化在 recovery 里会失败，因为 eSE 通道不可用
（`GPQeSE-HAL: Condition of use not satisfied`、`openLogicalChannel failed`）。
这**不影响解密** —— 解密走的是默认 keymint 实例 —— 而且 recovery 里没有任何场景
会用到 strongbox 密钥。纯粹是日志噪音。

## 注意事项 4 —— `TARGET_COPY_OUT_VENDOR_DLKM` 必须一起加

设置 `BOARD_VENDOR_DLKMIMAGE_FILE_SYSTEM_TYPE` 会让
`build/make/core/board_config.mk` 执行 `check_image_config`，它要求
`TARGET_COPY_OUT_VENDOR_DLKM` 的值是**整词** `vendor_dlkm`。该变量的默认值是
`$(TARGET_COPY_OUT_VENDOR)/vendor_dlkm`，也就是 `vendor/vendor_dlkm`，
无法通过整词匹配的 `$(filter ...)`，于是整个构建中断：

```
error: TARGET_COPY_OUT_VENDOR_DLKM must be set to 'vendor_dlkm' to use a vendor_dlkm image.
```

所以这两项必须成对添加。

## 注意事项 5 —— CPU 温度 zone

lafa 的 `cpu-0-0-0` 是 `thermal_zone5`。从 OrangeFox lafa 树继承来的值
（`thermal_zone45`）指向的是 `gpuss-9`，即 GPU，所以这里做了修正。

## 注意事项 6 —— VINTF manifest 必须放在 `/system/etc/vintf/manifest/`

recovery 构建的 `servicemanager` 带 `__ANDROID_RECOVERY__` 宏，会切到
`vintf::VintfObjectRecovery`。该类**只读**：

- `/system/etc/vintf/manifest.xml`
- `/system/etc/vintf/manifest/*.xml`

它**不读** `/vendor/etc/vintf/` 和 `/odm/etc/vintf/`
（源码见 `system/libvintf/VintfObjectRecovery.cpp`）。

后果：凡是只在 vendor/odm manifest 里声明的 AIDL HAL 都注册不上，
`servicemanager` 会拒绝并打印

```
Could not find <hal>/<instance> in the VINTF manifest. No alternative instances declared in VINTF.
```

HAL 随即 abort（`SIGABRT`，常伴随
`assertion "SharedRefBase: no ref created during lifetime" failed`），
服务陷入 init 重启循环。本机当时同时中招的有：振动器、触摸、gatekeeper、
health、secure_element、weaver、boot、keystore2。

这就是为什么本树需要的**每一个** HAL 片段都在
`recovery/root/system/etc/vintf/manifest/` 里复制了一份。
`recovery/root/vendor/etc/vintf/` 和 `recovery/root/vendor/odm/etc/vintf/` 下的副本
仅作参考保留，在 recovery 里不生效。

**新增 HAL 时，记得把它的片段也放进 `system/etc/vintf/manifest/`。**

## 注意事项 7 —— MTP 显示 `unknown model`

TWRP 的 MTP 实现没有用标准属性，而是用了两个非常规属性
（`bootable/recovery/mtp/twrp/TwrpMtpServer.cpp:42-46`）：

```cpp
const std::string manufacturer = base::GetProperty("ro.build.product", "unknown manufacturer");
const std::string model        = base::GetProperty("ro.color597.product_name", "unknown model");
const std::string serial_number = base::GetProperty("ro.serialno", "unknown serial number");
```

| MTP 字段 | 用的属性 | 本机值 |
| --- | --- | --- |
| 型号 | `ro.color597.product_name` | 原先未设置 → 回退成字面量 `unknown model` |
| 厂商 | `ro.build.product` | `sm88xx`（构建系统自动生成的产品名） |
| 序列号 | `ro.serialno` | 正常 |

`ro.color597.product_name` 是 TWRP 上游引用的一个厂商专有属性，
**sm87xx / sm88xx / OrangeFox lafa 三棵树都没有设置过它**，所以 PC 上会显示
`unknown model`。该属性在 MTP 里同时用于型号和设备描述两个字段
（`TwrpMtpDatabase.cpp:528`）。

修法是在 `libinit` 的 `SetupModelProperties()` 里按机型设置它：

```cpp
{"ro.color597.product_name",   info.model},
```

这样每个机型（按 `prjname` 分派）都会拿到自己正确的型号名。

已知遗留：**厂商字段仍显示 `sm88xx`**，因为它取的是 `ro.build.product`。
该属性是构建系统生成的标准属性（语义上是产品名，不是厂商），在 recovery 里只被
MTP 用到，覆盖它语义上不规范，因此未做处理。若确实需要显示 `realme`，
有两种做法：在 `props[]` 里一并覆盖 `ro.build.product`，或把
`TwrpMtpServer.cpp` 的两行改用标准的 `ro.product.manufacturer` / `ro.product.model`
（后者语义最正确，但改动在 TWRP 源码里，重新 `repo sync` 会被覆盖）。

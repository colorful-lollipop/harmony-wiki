# HiLog 编译产物文档

> 生成时间: 2026-02-06
> 相关证据: `bundle.json`, 各 `BUILD.gn` 文件的 install/install_images 配置

---

## 目的

本文档详细列出 HiLog 模块编译生成的所有产物及其安装路径、运行时加载关系。

## 适用范围

涵盖 HiLog 模块的所有编译产物（可执行文件、共享库、配置文件等）。

---

## 可执行文件

### hilogd

| 属性 | 值 |
|------|------|------|
| **Target** | `//services/hilogd:hilogd` |
| **Target 类型** | ohos_executable |
| **输出文件名** | `hilogd` |
| **安装路径** | `/system/bin/hilogd` |
| **启动方式** | init 服务（`hilogd.cfg`） |
| **权限** | -r-xr-xr-x（755） |
| **所属用户** | logd:log (UID 1036, GID 1036） |
| **能力** | SYSLOG（读取内核日志） |

**功能**: 日志常驻服务，接收并存储日志到环形缓冲区，处理控制命令，执行日志落盘。

**证据**:
- Target: `services/hilogd/BUILD.gn:22-30`
- Install images: `services/hilogd/BUILD.gn:61-65`
- 服务配置: `services/hilogd/etc/hilogd.cfg` - 用户:logd, 组:log

### hilog

| 属性 | 值 |
|------|------|------|
| **Target** | `//services/hilogtool:hilog` |
| **Target 类型** | ohos_executable |
| **输出文件名** | `hilog` |
| **安装路径** | `/system/bin/hilog` |
| **符号链接** | `/usr/bin/hilog`（如果 `hilog_feature_support_usr_symlink`） |
| **权限** | -r-xr-xr-x（755） |
| **所属用户** | root:root（UID 0, GID 0） |

**功能**: 命令行工具，用于查询、过滤、显示日志，发送控制命令到 hilogd。

**证据**:
- Target: `services/hilogtool/BUILD.gn:23-30`
- Install images: `services/hilogtool/BUILD.gn:47-50`
- Symlink: `services/hilogtool/BUILD.gn:51-55`

---

## 共享库

### libhilog.so

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/native/innerkits:libhilog` |
| **Target 类型** | ohos_shared_library |
| **输出文件名** | `libhilog.so` |
| **安装路径** | `/system/lib/libhilog.so` |
| **符号版本** | 使用 `libhilog.map` |
| **Inner API Tags** | chipsetsdk, platformsdk, sasdk |
| **安装条件** | `!hilog_native_feature_ohcore` |
| **安装镜像** | `system`, `updater` |
| **依赖库** | 无静态链接依赖 |

**功能**: 对外 Native 日志库，提供 C/C++ API，通过 Unix Domain Socket 与 hilogd 通信。

**证据**:
- Target: `interfaces/native/innerkits/BUILD.gn:33-46`
- Version script: `interfaces/native/innerkits/BUILD.gn:47`

### libhilog_ndk.so

| 属性 | 值 |
|------|------|------|
| **Target** | `//frameworks/hilog_ndk:hilog_ndk` |
| **Target 类型** | ohos_shared_library |
| **输出文件名** | `libhilog_ndk.so` |
| **安装路径** | `/system/lib/libhilog_ndk.so` |
| **Inner API Tags** | ndk |
| **安装镜像** | `system_base_dir`, `updater` |

**功能**: NDK 接口封装库，为 Native 应用提供简化的日志 API。

**证据**:
- Target: `frameworks/hilog_ndk/BUILD.gn:19-35`
- NDK 定义: `interfaces/native/kits/libhilog.ndk.json`

### libhilog_napi.so

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/js/kits/napi:libhilognapi` |
| **Target 类型** | ohos_shared_library |
| **输出文件名** | `libhilog_napi.so` |
| **安装路径** | `/system/lib/module/libhilog_napi.so` |
| **相对安装目录** | `module/` |
| **依赖库** | `libhilog.so`（运行时动态链接） |

**功能**: JavaScript N-API 绑定，为 JS 应用提供 HiLog 接口。

**证据**:
- Target: `interfaces/js/kits/napi/BUILD.gn:67-82`

### libhilog_rust.so

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/rust:hilog_rust` |
| **Target 类型** | ohos_rust_shared_library |
| **Crate 名称** | `hilog_rust` |
| **Crate 类型** | dylib |
| **输出文件名** | `libhilog_rust.so` |
| **安装路径** | `/system/lib/libhilog_rust.so` |
| **依赖库** | `libhilog.so`（运行时动态链接） |
| **栈保护** | `-Zstack-protector=all` |

**功能**: Rust FFI 绑定，为 Rust 应用提供 HiLog 接口。

**证据**:
- Target: `interfaces/rust/BUILD.gn:14-23`

### libhilog_ani.so

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/ets/ani/hilog:hilog_ani` |
| **Target 类型** | ohos_shared_library |
| **输出文件名** | `libhilog_ani.so` |
| **安装路径** | `/system/lib/libhilog_ani.so` |
| **依赖库** | `libhilog.so`（运行时动态链接） |
| **运行时依赖** | `libarkruntime.so` |

**功能**: ArkTS Native Interface (ANI) 绑定，为 ArkTS 应用提供 HiLog 接口。

**证据**:
- Target: `interfaces/ets/ani/hilog/BUILD.gn:19-65`

---

## 静态库

### libhilog_base.a

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/native/innerkits:libhilog_base` |
| **Target 类型** | ohos_static_library |
| **用途** | 基础日志功能（无动态分配） |
| **安装路径** | 不安装（链接到使用它的目标） |
| **依赖** | `libsec_static`（静态链接） |

**功能**: 提供基础日志功能，禁止动态内存分配，用于 musl libc 等场景。

**证据**:
- Target: `interfaces/native/innerkits/BUILD.gn:169-190`

### libhilog_snapshot.a

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/native/innerkits:libhilog_snapshot` |
| **Target 类型** | ohos_static_library |
| **用途** | 崩溃日志快照 |
| **安装路径** | 不安装（链接到使用它的目标） |

**功能**: 提供崩溃时日志快照功能。

**证据**:
- Target: `interfaces/native/innerkits/BUILD.gn:200-218`

### libhilog_base_for_musl.a

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/native/innerkits:libhilog_base_for_musl` |
| **Target 类型** | ohos_static_library |
| **特殊标志** | `-nostdlib`（不链接标准 C 库） |
| **用途** | musl libc 专用基础库 |

**功能**: 为 musl libc 提供特殊版本的基础日志功能。

**证据**:
- Target: `interfaces/native/innerkits/BUILD.gn:158-167`

---

## 字节码文件

### hilog.abc

| 属性 | 值 |
|------|------|------|
| **Target** | `//interfaces/ets/ani/hilog:hilog` |
| **Target 类型** | `generate_static_abc` |
| **输出文件名** | `hilog.abc` |
| **安装路径** | `/system/framework/hilog.abc` |
| **Is Boot ABC** | `True` |
| **源文件** | `./ets/@ohos.hilog.ets` |
| **生成工具** | ArkTS 编译器 |

**功能**: ArkTS 字节码文件，定义 HiLog 模块的类型信息。

**证据**:
- Target: `interfaces/ets/ani/hilog/BUILD.gn:56-62`

---

## 配置文件

### hilogd.cfg

| 属性 | 值 |
|------|------|------|
| **Target** | `//services/hilogd/etc:hilogd.cfg` |
| **类型** | ohos_prebuilt_etc |
| **源文件** | `services/hilogd/etc/hilogd.cfg` |
| **安装路径** | `/system/etc/init/hilogd.cfg` |
| **目的** | init 服务启动配置 |

**内容关键部分**:
```json
{
  "name" : "hilogd",
  "uid" : "logd",
  "gid" : "logd",
  "caps" : [ "SYSLOG" ],
  "socket" : {
    "hilogInput" : {
      "mode" : "0222"
    },
    "hilogOutput" : {
      "mode" : "0666"
    },
    "hilogControl" : {
      "mode" : "0660"
    }
  }
}
```

**证据**: `services/hilogd/etc/hilogd.cfg`（内容）

### hilog.para

| 属性 | 值 |
|------|------|------|
| **Target** | `//services/hilogd/etc:hilog.para` |
| **类型** | ohos_prebuilt_etc |
| **源文件** | `services/hilogd/etc/hilog.para` |
| **安装路径** | `/system/etc/param/hilog.para` |

**功能**: 参数配置文件。

### hilog.para.dac

| 属性 | 值 |
|------|------|------|
| **Target** | `//services/hilogd/etc:hilog.para.dac` |
| **类型** | ohos_prebuilt_etc |
| **源文件** | `services/hilogd/etc/hilog.para.dac` |
| **安装路径** | `/system/etc/param/hilog.para.dac` |

**功能**: 参数 DAC 配置文件。

---

## 运行时加载关系

### 库依赖链

```
libhilog_napi.so
├─→ libhilog.so (动态链接）
│   ├─→ libsec_shared.so (符号解析）
│   ├─→ libc.so (系统 C 库）
│   └─→ libz.so（压缩，落盘时使用）
└─→ libace_napi.so（N-API 运行时）

libhilog_rust.so
├─→ libhilog.so (动态链接，FFI）
└─→ libc.so（系统 C 库）

libhilog_ani.so
├─→ libhilog.so (动态链接）
├─→ libarkruntime.so（ANI 运行时）
└─→ libc.so（系统 C 库）

libhilog_ndk.so
└─→ libhilog.so (动态链接）

应用（使用 N-API）
└─→ libhilog_napi.so → libhilog.so → Unix Domain Socket → hilogd

应用（使用 NDK）
└─→ libhilog_ndk.so → libhilog.so → Unix Domain Socket → hilogd

hilog 工具
└─→ libhilog.so → LogIoctl → Unix Domain Socket → hilogd
```

### Socket 文件

| Socket | 路径 | 类型 | 用途 |
|--------|------|------|------|
| hilogInput | `/dev/unix/socket/hilogInput` | SOCK_DGRAM | 应用提交日志 |
| hilogOutput | `/dev/unix/socket/hilogOutput` | SOCK_SEQPACKET | 工具查询日志 |
| hilogControl | `/dev/unix/socket/hilogControl` | SOCK_SEQPACKET | 工具发送控制命令 |

**证据**: `frameworks/libhilog/include/hilog_common.h:27-28`

---

## 数据目录

### 日志落盘目录

| 路径 | 用途 | 权限 |
|------|------|------|
| `/data/log/hilog/` | 日志落盘根目录 | logd:log |
| `/data/log/hilog/*.gz` | 压缩日志文件 | logd:log |
| `/data/log/hilog/*.zst` | zstd 压缩日志文件 | logd:log |

**文件命名格式**:
- hilog: `hilog.000.YYYYMMDD-HHMMSS.gz`
- hilog_kmsg: `hilog_kmsg.000.YYYYMMDD-HHMMSS.gz`

**证据**:
- 落盘实现: `services/hilogd/log_persister.cpp`
- 文件格式: `services/hilogd/log_persister_rotator.cpp`

### 参数目录

| 路径 | 用途 |
|------|------|
| `/system/etc/init/` | init 服务配置 |
| `/system/etc/param/` | 参数配置 |

---

## 产物总览表

| 产物类型 | 数量 | 列表 |
|----------|------|------|
| 可执行文件 | 2 | hilogd, hilog |
| 共享库 | 5 | libhilog.so, libhilog_ndk.so, libhilog_napi.so, libhilog_rust.so, libhilog_ani.so |
| 静态库 | 3 | libhilog_base.a, libhilog_snapshot.a, libhilog_base_for_musl.a |
| 字节码文件 | 1 | hilog.abc |
| 配置文件 | 3 | hilogd.cfg, hilog.para, hilog.para.dac |

---

## 相关跳转链接

- [GN Targets](06_GN_Targets.md)
- [目录结构](02_Directory_Structure.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| hilogd 安装 | services/hilogd/BUILD.gn:61-65 | install_images |
| hilog 安装 | services/hilogtool/BUILD.gn:47-50 | install_images |
| libhilog 安装 | interfaces/native/innerkits/BUILD.gn:44-46 | install_images, version_script |
| N-API 安装 | interfaces/js/kits/napi/BUILD.gn:78 | relative_install_dir |
| 符号链接 | services/hilogtool/BUILD.gn:51-55 | symlink_target_name |
| hilog.abc 安装 | interfaces/ets/ani/hilog/BUILD.gn:60 | device_destination |
| Socket 路径 | frameworks/libhilog/include/hilog_common.h:27-28 | OUTPUT_SOCKET_NAME, CONTROL_SOCKET_NAME |

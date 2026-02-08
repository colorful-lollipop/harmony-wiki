# 编译产物

本文档详细说明 OpenHarmony 构建系统的编译产物，包括产物类型、输出路径和安装位置。

## 产物类型概览

| 类别 | 产物类型 | 扩展名 | 输出位置 |
|------|---------|--------|----------|
| 库文件 | 共享库 | .so | out/{device}/lib/ |
| 库文件 | 静态库 | .a | out/{device}/obj/ |
| 可执行文件 | 可执行文件 | - | out/{device}/exe/ |
| 应用 | HAP | .hap | out/{device}/packages/ |
| 镜像 | 系统镜像 | .img | out/{device}/packages/phone/images/ |
| SDK | SDK 包 | .zip | out/sdk/packages/ohos-sdk/ |
| NDK | NDK 包 | .zip | out/ndk/ |

## 详细产物说明

### 1. 共享库 (.so)

**模板**: `ohos_shared_library`, `ohos_rust_shared_library`

**输出路径**:
```
out/{device_name}/
├── lib/                          # 未剥离的库
│   └── lib{name}.so
├── lib.unstripped/              # 带符号的库
│   └── lib{name}.so
└── system/lib64/                # 安装路径（实际打包）
    └── lib{name}.so
```

**安装规则**:
- 默认安装到 `system/lib64/` (arm64) 或 `system/lib/` (arm)
- 可通过 `module_install_dir` 指定自定义路径
- 可通过 `relative_install_dir` 指定相对路径

**示例**:
```gn
ohos_shared_library("my_lib") {
  module_install_dir = "vendor/lib"  # 安装到 vendor/lib/
  relative_install_dir = "subdir"     # 安装到 system/lib64/subdir/
}
```

### 2. 静态库 (.a / .rlib)

**模板**: `ohos_static_library`, `ohos_rust_static_library`

**输出路径**:
```
out/{device_name}/
├── obj/{path/to/}lib{name}.a      # C/C++ 静态库
└── obj/{path/to/}lib{name}.rlib   # Rust 静态库
```

**特点**:
- 默认不安装到系统目录
- 仅用于编译时链接

### 3. 可执行文件

**模板**: `ohos_executable`, `ohos_rust_executable`

**输出路径**:
```
out/{device_name}/
├── exe/{name}                     # 可执行文件
├── exe.unstripped/{name}          # 带符号版本
└── system/bin/{name}              # 安装路径
```

**安装规则**:
- 默认不安装，需设置 `install_enable = true`
- 默认安装到 `system/bin/`

**示例**:
```gn
ohos_executable("my_tool") {
  install_enable = true
  module_install_dir = "vendor/bin"  # 安装到 vendor/bin/
}
```

### 4. HAP (HarmonyOS Ability Package)

**模板**: `ohos_hap`

**输出路径**:
```
out/{device_name}/
├── packages/phone/hap/
│   └── {name}.hap
└── entry/build/default/outputs/default/
    └── entry-default-signed.hap
```

**HAP 内容**:
```
{name}.hap
├── ets/                    # ArkTS/ETS 代码
├── js/                     # JavaScript 代码
├── resources/              # 资源文件
├── module.json             # 模块配置
└── ability.json            # Ability 配置
```

### 5. Ark Bytecode (.abc)

**模板**: `ohos_abc`

**输出路径**:
```
out/{device_name}/
├── gen/{path}/
│   └── {name}.abc
└── 打包到 HAP 中
```

### 6. 系统镜像

#### system.img

**配置**: `//build/ohos/images/mkimage/system_image_conf.txt`

**参数**:
- 大小: 1610612224 bytes (1.5GB)
- 文件系统: ext4
- 挂载点: /

**内容**:
```
system/
├── bin/                # 可执行文件
├── lib64/              # 64位共享库
├── lib/                # 32位共享库
├── etc/                # 配置文件
├── usr/
└── ...
```

#### vendor.img

**配置**: `//build/ohos/images/mkimage/vendor_image_conf.txt`

**参数**:
- 大小: 268434944 bytes (256MB)
- 文件系统: ext4
- 挂载点: /vendor

**内容**:
```
vendor/
├── bin/
├── lib64/
├── lib/
└── etc/
```

#### userdata.img

**配置**: `//build/ohos/images/mkimage/userdata_image_conf.txt`

**参数**:
- 大小: 1468006400 bytes (1.4GB)
- 文件系统: f2fs
- 挂载点: /data

#### ramdisk.img

**配置**: `//build/ohos/images/mkimage/ramdisk_image_conf.txt`

**用途**: 启动内存盘

### 7. SDK

**构建命令**:
```bash
./build.sh --product-name ohos-sdk --ccache
```

**输出路径**:
```
out/sdk/packages/ohos-sdk/
├── linux/
│   ├── toolchains/           # 工具链
│   ├── api_version-x/        # API 版本
│   │   ├── toolchains/
│   │   ├── sysroot/
│   │   └── ...
│   └── ohos-sdk-linux.tar.gz
├── darwin/                   # macOS SDK
└── windows/                  # Windows SDK
```

**SDK 组件**:

| 组件 | 说明 |
|------|------|
| toolchains | 编译工具链 (clang, llvm) |
| sysroot | 系统根目录 (libc, headers) |
| js | JavaScript API |
| ets | ArkTS/ETS API (static/dynamic) |
| java | Java API |
| previewer | 预览器工具 |

### 8. NDK

**构建命令**:
```bash
./build.sh --product-name ohos-sdk --build-ohos-ndk
```

**输出路径**:
```
out/ndk/
├── ohos-ndk/
│   ├── toolchains/
│   ├── sysroot/
│   ├── usr/
│   └── build/cmake/
└── ohos-ndk-linux-x64.tar.gz
```

**NDK 内容**:
```
ohos-ndk/
├── toolchains/llvm/prebuilt/linux-x86_64/
│   ├── bin/clang
│   ├── bin/clang++
│   └── ...
├── sysroot/usr/include/       # 头文件
├── sysroot/usr/lib/           # 库文件
└── build/cmake/ohos.toolchain.cmake
```

## 产物映射关系

### Target → 产物映射

| Target 类型 | 中间产物 | 最终产物 | 安装位置 |
|------------|---------|---------|----------|
| `ohos_shared_library` | .o | .so | system/lib64/ |
| `ohos_static_library` | .o | .a | (不安装) |
| `ohos_executable` | .o | 可执行文件 | system/bin/ |
| `ohos_rust_shared_library` | .rlib | .so | system/lib64/ |
| `ohos_rust_static_library` | .rlib | .rlib | (不安装) |
| `ohos_hap` | .abc, .js | .hap | packages/phone/hap/ |
| `ohos_prebuilt_etc` | - | 源文件 | 按 module_install_dir |

### 分区映射

| 安装路径前缀 | 目标分区 | 说明 |
|-------------|---------|------|
| system/ | system.img | 系统分区 |
| vendor/ | vendor.img | 芯片厂商分区 |
| data/ | userdata.img | 用户数据分区 |

## 构建输出目录结构

```
out/{device_name}/
├── build.ninja                  # Ninja 构建文件
├── build_configs/               # 构建配置
│   ├── parts.json              # 部件清单
│   ├── features.json           # 特性配置
│   ├── syscap.json             # 系统能力
│   └── build_config.json       # 构建配置
├── gen/                         # 生成的代码
├── obj/                         # 目标文件
│   └── {path/to/}
│       ├── {target}.o
│       └── {target}.a
├── lib/                         # 共享库（未剥离）
│   └── lib{name}.so
├── lib.unstripped/              # 共享库（带符号）
├── exe/                         # 可执行文件
├── exe.unstripped/              # 可执行文件（带符号）
├── system/                      # 系统目录（打包前）
│   ├── bin/
│   ├── lib64/
│   └── lib/
├── vendor/                      # 厂商目录（打包前）
├── packages/                    # 打包输出
│   └── phone/
│       ├── images/             # 镜像文件
│       │   ├── system.img
│       │   ├── vendor.img
│       │   ├── userdata.img
│       │   └── ramdisk.img
│       └── hap/                # HAP 应用
└── logs/                        # 构建日志
```

## 运行时加载关系

### 动态库加载

```
可执行文件
    │
    ├──→ 链接时: 从 out/{device}/lib/ 链接
    │
    └──→ 运行时: 从 /system/lib64/ 或 /vendor/lib64/ 加载
              │
              ├──→ 依赖库递归加载
              │
              └──→ 使用 ld-musl-aarch64.so.1 动态链接器
```

### 库搜索路径

运行时动态库搜索顺序：
1. `LD_LIBRARY_PATH` 环境变量
2. `/lib64` 和 `/lib` (system 分区)
3. `/vendor/lib64` 和 `/vendor/lib`
4. `/chipset/lib64` 和 `/chipset/lib`

## 产物验证

### 检查库依赖

```bash
# 查看共享库依赖
readelf -d out/{device}/lib/lib{name}.so | grep NEEDED

# 查看符号表
nm out/{device}/lib.unstripped/lib{name}.so
```

### 检查镜像内容

```bash
# 查看 ext4 镜像内容
debugfs out/{device}/packages/phone/images/system.img

# 挂载镜像查看
mkdir -p /mnt/system
mount -o loop system.img /mnt/system
```

### 检查 HAP 内容

```bash
# HAP 是 ZIP 格式
unzip -l {name}.hap

# 解压查看
unzip {name}.hap -d hap_content/
```

---

*文档生成时间: 2025-02-06*

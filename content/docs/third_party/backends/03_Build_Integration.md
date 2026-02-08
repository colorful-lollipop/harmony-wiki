# 03 - OpenHarmony 构建适配

本文档详细说明 SANE-backends 如何从上游的 Autotools 构建系统适配到 OpenHarmony 的 GN 构建系统。

---

## 构建系统对比

| 特性 | 上游 (Autotools) | OpenHarmony (GN) |
|------|-----------------|------------------|
| **配置方式** | `./configure` 脚本 | `BUILD.gn` + `args.gn` |
| **构建命令** | `make` | `gn gen` + `ninja` |
| **依赖管理** | `configure.ac` 检测 | `deps` 显式声明 |
| **跨平台** | autoconf 宏 | GN 条件语句 |
| **Patch 应用** | 手动或 `autoreconf` | Python 脚本自动应用 |

---

## BUILD.gn 结构

### 路径定义（OH 沙箱）

```gn
# OH 沙箱目录定义
SANE_CONFIG_DIR = "/data/service/el1/public/print_service/sane/config"
SANE_DATA_DIR = "/data/service/el1/public/print_service/sane/data"
SANE_TMP_DIR = "/data/service/el2/public/print_service/sane/tmp"
SANE_LOCK_DIR = "/data/service/el1/public/print_service/sane/lock"
SANE_LIB_DIR = "/data/service/el1/public/print_service/sane/backend"

# 版本定义
SANE_V_MAJOR = 1
SANE_V_MINOR = 2
```

### 功能开关

```gn
# 特性开关
enable_hilog = true        # 启用 HiLog 日志系统
enable_thread_pool = true  # 启用线程池并发设备发现
enable_scan_service = true # 启用扫描服务适配
```

这些开关对应以下 C 宏：
- `ENABLE_HILOG` → HiLog 集成
- `HAVE_THREAD_POLL` → 线程池支持
- `HAVE_SCAN_SERVICE` → 扫描服务目录结构

---

## 关键编译配置

### 公共配置 (backends_public_config)

```gn
config("backends_public_config") {
  include_dirs = [
    "${target_dir}/include",
    "${target_dir}/include/sane",
  ]
}
```

**作用**: 对外暴露的头文件搜索路径，使用 `target_gen_dir` 下的生成文件。

### 私有配置 (backends_private_config)

```gn
config("backends_private_config") {
  cflags = [
    "-Wall",
    "-g",
    "-O2",
    "-fPIC",
    "-DPIC",
    "-D_REENTRANT",
    "-DHAVE_CONFIG_H",
    "-Wno-format",                    # 禁用格式警告
    "-Wno-unused-const-variable",     # 禁用未使用常量警告
    "-Wno-unused-variable",           # 禁用未使用变量警告
    "-Wno-unused-but-set-variable",   # 禁用设置未使用变量警告
    "-Wno-unused-function",           # 禁用未使用函数警告
  ]

  # 条件编译
  if (enable_hilog) {
    cflags += [ "-DENABLE_HILOG" ]
  }
  if (enable_thread_pool) {
    cflags += [ "-DHAVE_THREAD_POLL" ]
  }
  if (enable_scan_service) {
    cflags += [ "-DHAVE_SCAN_SERVICE" ]
  }

  # 预定义宏
  defines = [
    "PATH_SANE_CONFIG_DIR=$SANE_CONFIG_DIR",
    "PATH_SANE_DATA_DIR=$SANE_DATA_DIR",
    "PATH_SANE_LOCK_DIR=$SANE_LOCK_DIR",
    "PATH_SANE_TMP_DIR=$SANE_TMP_DIR",
    "V_MAJOR=$SANE_V_MAJOR",
    "V_MINOR=$SANE_V_MINOR",
    "LIBDIR=\"$SANE_LIB_DIR\"",
  ]
}
```

**关键参数说明**:

| 参数 | 说明 |
|------|------|
| `-fPIC -DPIC` | 生成位置无关代码（共享库必需） |
| `-D_REENTRANT` | 启用可重入代码支持（线程安全） |
| `-DHAVE_CONFIG_H` | 启用配置头文件支持 |
| `-Wno-xxx` | 禁用特定警告（上游代码兼容性） |

---

## 构建目标详解

### 1. backends_action (Action)

```gn
action("backends_action") {
  script = "//third_party/backends/install.py"
  inputs = [
    "${source_dir}/install.py",
    "${source_dir}/patches/modifying_driver_search_path.patch",
    "${source_dir}/patches/add_thread_poll.patch",
    "${source_dir}/patches/hilog_debug.patch",
    "${source_dir}/patches/modify_load_function.patch",
  ]
  outputs = []
  outputs += backends_generated_sources
  outputs += backends_generated_header
  
  args = [
    "--source-dir", backends_source_dir,
    "--target-dir", backends_target_dir,
  ]
}
```

**功能**:
- 执行 `install.py` 脚本
- 复制源文件到生成目录
- 按顺序应用 4 个 OH 特有 Patch

**Patch 应用顺序**:
1. `modifying_driver_search_path.patch`
2. `add_thread_poll.patch`
3. `hilog_debug.patch`
4. `modify_load_function.patch`

### 2. lib (Source Set)

```gn
ohos_source_set("lib") {
  sources = [ "${target_dir}/lib/md5.c" ]
  deps = [ ":backends_action" ]
  public_configs = [ ":backends_public_config" ]
  configs = [ ":backends_private_config" ]
}
```

**功能**: MD5 工具函数库。

### 3. sanei_usb (Source Set)

```gn
ohos_source_set("sanei_usb") {
  sources = [ "${target_dir}/sanei/sanei_usb.c" ]
  external_deps = [ "libusb:libusb" ]
  if (enable_hilog) {
    external_deps += [ "hilog:libhilog" ]
  }
  deps = [ ":backends_action" ]
  public_configs = [ ":backends_public_config" ]
  configs = [ ":backends_private_config" ]
}
```

**功能**: USB 设备通信支持。

**依赖**:
- `libusb:libusb` - USB 库
- `hilog:libhilog` - 日志服务（可选）

### 4. sanei (Static Library)

```gn
ohos_static_library("sanei") {
  sources = []
  foreach(name, sanei_names) {
    sources += [ "${target_dir}/sanei/$name.c" ]
  }
  if (enable_hilog) {
    external_deps = [ "hilog:libhilog" ]
  }
  deps = [ ":backends_action" ]
  public_configs = [ ":backends_public_config" ]
  configs = [ ":backends_private_config" ]
}
```

**包含的模块**:
```gn
sanei_names = [
  "sanei_directio",      # 直接 I/O
  "sanei_ab306",         # AB306 芯片支持
  "sanei_constrain_value", # 值约束
  "sanei_init_debug",    # 调试初始化
  "sanei_net",           # 网络支持
  "sanei_wire",          # 协议编解码
  "sanei_codec_ascii",   # ASCII 编解码
  "sanei_codec_bin",     # 二进制编解码
  "sanei_scsi",          # SCSI 支持
  "sanei_config",        # 配置读取
  "sanei_config2",       # 配置读取 v2
  "sanei_pio",           # 并行 I/O
  "sanei_pa4s2",         # PA4S2 芯片
  "sanei_auth",          # 认证
  "sanei_thread",        # 线程支持
  "sanei_pv8630",        # PV8630 芯片
  "sanei_pp",            # 并行端口
  "sanei_lm983x",        # LM983x 芯片
  "sanei_access",        # 访问控制
  "sanei_tcp",           # TCP 网络
  "sanei_udp",           # UDP 网络
  "sanei_magic",         # 图像处理
  "sanei_ir",            # 红外支持
  "sanei_jpeg",          # JPEG 处理
]
```

### 5. threadpool_c (Source Set)

```gn
ohos_source_set("threadpool_c") {
  sources = [
    "${target_dir}/threadpool/src/threadpool_c.cpp",
    "${target_dir}/threadpool/src/threadpool_wrapper.cpp"
  ]
  include_dirs = [ "${target_dir}/threadpool/include" ]
  deps = [ ":backends_action" ]
  external_deps = [
    "c_utils:utils",       # OHOS::ThreadPool
  ]
  if (enable_hilog) {
    external_deps += [ "hilog:libhilog" ]
  }
  public_configs = [ ":backends_public_config" ]
  configs = [ ":backends_private_config" ]
}
```

**功能**: OH 特有线程池组件。

**OH 特有文件**:
- `threadpool/src/threadpool_c.cpp` - C 接口实现
- `threadpool/src/threadpool_wrapper.cpp` - OHOS::ThreadPool 包装器
- `threadpool/include/threadpool_c.h` - C 接口头文件
- `threadpool/include/threadpool_wrapper.h` - 包装器头文件

### 6. sane (Shared Library)

```gn
ohos_shared_library("sane") {
  sources = [
    "${target_dir}/backend/dll.c",
    "${target_dir}/backend/stubs.c",
  ]
  public_configs = [ ":backends_public_config" ]
  configs = [ ":backends_private_config" ]
  defines = [ "BACKEND_NAME=dll" ]
  deps = [
    ":backends_action",
    ":lib",
    ":sane_strstatus",
    ":sanei_config",
    ":sanei_constrain_value",
    ":sanei_init_debug",
    ":sanei_usb",
  ]
  if (enable_thread_pool) {
    deps += [ ":threadpool_c" ]
    include_dirs += [ "${target_dir}/threadpool/include" ]
  }
  if (enable_hilog) {
    external_deps = [ "hilog:libhilog" ]
  }
  ldflags = [ "-ldl" ]  # 动态链接器支持
}
```

**功能**: SANE 核心共享库，对外暴露 SANE API。

### 7. third_sane (Group)

```gn
group("third_sane") {
  deps = [ ":sane" ]
}
```

**功能**: 对外暴露的构建目标，其他模块通过此目标依赖 SANE。

---

## 与上游构建的差异

### 1. 后端驱动处理

**上游**: 
- 所有后端在编译时链接到主库或单独编译为共享库
- 使用 `dlopen` 动态加载

**OH**:
- 精简后端列表（仅保留常用后端）
- 驱动库放置于沙箱目录
- 动态搜索和加载

### 2. 配置生成

**上游**:
- 通过 `configure.ac` 检测系统特性
- 生成 `config.h` 头文件

**OH**:
- 配置硬编码在 BUILD.gn 中
- 通过 `defines` 直接传递给编译器
- 减少运行时检测依赖

### 3. 依赖声明

**上游**:
```bash
# configure.ac
PKG_CHECK_MODULES(LIBUSB, libusb-1.0 >= 1.0.0)
```

**OH**:
```gn
# BUILD.gn
external_deps = [ "libusb:libusb" ]
```

### 4. Patch 管理

**上游**:
- 手动应用 Patch
- 或使用 distribution 维护的 patch 系统

**OH**:
- `install.py` 自动应用
- 构建时自动完成
- 保证一致性

---

## 依赖关系

### 外部依赖（从 bundle.json）

```json
"deps": {
  "components": [
    "hilog",      # 日志服务
    "c_utils",    # 线程池和工具类
    "libusb"      # USB 通信
  ],
  "third_party": [
    "libxml2",    # XML 解析（escl 后端）
    "libpng",     # PNG 图像处理
    "libjpeg-turbo"  # JPEG 图像处理
  ]
}
```

### 构建依赖图

```
third_sane (group)
    │
    └─> sane (shared_library)
            │
            ├─> backends_action (action) [应用 Patch]
            │
            ├─> lib (source_set)
            │       └─> md5.c
            │
            ├─> sane_strstatus (source_set)
            │       └─> sane_strstatus.c
            │
            ├─> sanei_config (source_set)
            │       └─> sanei_config.c
            │
            ├─> sanei_constrain_value (source_set)
            │       └─> sanei_constrain_value.c
            │
            ├─> sanei_init_debug (source_set)
            │       └─> sanei_init_debug.c
            │
            ├─> sanei_usb (source_set)
            │       └─> sanei_usb.c
            │       └─> [external: libusb]
            │
            └─> threadpool_c (source_set) [optional]
                    ├─> threadpool_c.cpp
                    ├─> threadpool_wrapper.cpp
                    └─> [external: c_utils]
```

---

## 构建流程

### 完整构建步骤

```bash
# 1. 生成构建配置
gn gen out --args='target_os="ohos" target_cpu="arm64"'

# 2. 编译 third_party/backends
ninja -C out third_party/backends:third_sane

# 3. 构建流程内部执行：
#    a. 执行 backends_action
#       - 运行 install.py
#       - 复制源文件到 ${target_gen_dir}/sane
#       - 应用 4 个 OH Patch
#    
#    b. 编译各个 source_set
#       - 使用生成后的源文件
#       - 应用 cflags 和 defines
#    
#    c. 链接生成 libsane.so
#       - 链接所有依赖的 source_set
#       - 添加 -ldl 链接器标志
```

### 生成的文件结构

```
${target_gen_dir}/sane/
├── backend/
│   ├── dll.c              # 应用 Patch 后的文件
│   ├── stubs.c
│   └── sane_strstatus.c
├── include/
│   ├── sane/
│   │   └── sanei_debug.h  # 应用 hilog_debug.patch
│   └── sane.h
├── threadpool/
│   ├── src/
│   │   ├── threadpool_c.cpp      # OH 特有
│   │   └── threadpool_wrapper.cpp # OH 特有
│   └── include/
│       ├── threadpool_c.h        # OH 特有
│       └── threadpool_wrapper.h  # OH 特有
└── sanei/
    ├── sanei_usb.c
    └── ...
```

---

## 调优和优化

### 1. 禁用线程池

如果需要禁用线程池功能（例如调试）：

```gn
# 在 BUILD.gn 或 args.gn 中
enable_thread_pool = false
```

**影响**: 设备发现将恢复为串行模式。

### 2. 禁用 HiLog

```gn
enable_hilog = false
```

**影响**: 日志将使用标准错误输出。

### 3. 禁用扫描服务适配

```gn
enable_scan_service = false
```

**影响**: 将使用标准 FHS 路径而非 OH 沙箱路径。

---

## 常见问题

### Q: Patch 应用失败怎么办？

A: 检查 `install.py` 的输出日志：
```bash
# 查看详细日志
ninja -C out -v third_party/backends:backends_action
```

常见原因：
- 上游版本与 Patch 不匹配
- 文件权限问题
- Patch 已部分应用

### Q: 如何添加新的后端？

A: 需要修改：
1. `BUILD.gn` 添加源文件到 `backends_generated_sources`
2. 确保后端配置正确
3. 可能需要新增依赖

### Q: 如何修改沙箱路径？

A: 修改 `BUILD.gn` 开头的路径定义：
```gn
SANE_CONFIG_DIR = "/your/custom/path/config"
# ... 其他路径
```

---

## 参考

- [GN 构建系统文档](https://gn.googlesource.com/gn/+/main/docs/)
- [OpenHarmony 构建指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-gn-coding-style.md)
- [SANE 构建文档](https://gitlab.com/sane-project/backends/-/blob/master/README.md)

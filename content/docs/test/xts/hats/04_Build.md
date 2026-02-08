# 构建系统

## GN 构建概览

### 构建工具链

| 组件 | 说明 |
|------|------|
| **构建系统** | GN (Generate Ninja) |
| **构建工具** | hb (OpenHarmony Build) |
| **编译工具链** | LLVM/clang, GCC |
| **测试框架** | GoogleTest (googletest) |

### 构建文件结构

```
/Volumes/lexar/code/d/work/oh/test/xts/hats/
├── BUILD.gn              # 根构建配置（主入口）
├── build.gni             # 特性开关配置
├── bundle.json           # 组件元数据
├── test_packages.gni     # 测试包选择逻辑
└── [subsystem]/
    ├── BUILD.gn          # 子系统聚合配置
    └── [module]/
        └── BUILD.gn      # 模块级配置
```

---

## 根构建入口

### BUILD.gn

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/BUILD.gn`

```gn
import("//test/xts/tools/lite/build/suite_lite.gni")  # 或 standard build
import("test_packages.gni")

# 合并开源声明
merge_xts_notice("hats_opensource_process") {
  target = "hats"
  deps = selected_packages
}

# 主测试套件
ohos_test_suite("xts_hats") {
  deps = [ ":hats_opensource_process" ]
}

# 产品变体
ohos_test_suite("hats_ivi") {
  deps = selected_packages_ivi
}

ohos_test_suite("hats_intellitv") {
  deps = selected_packages_intellitv
}

ohos_test_suite("hats_wearable") {
  deps = selected_packages_wearable
}
```

---

## 特性开关

### build.gni

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/build.gni`

```gn
declare_args() {
  # 富功能测试开关
  hats_rich = false

  # 神经网络运行时测试
  hats_nnrt = false

  # 电源相关特性
  hats_drivers_peripheral_power_wakeup_cause_path = false
  hats_drivers_peripheral_battery_pc_macro_isolation = false
}
```

### bundle.json 特性声明

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/bundle.json`

```json
{
  "name": "@ohos/hats",
  "component": {
    "name": "hats",
    "subsystem": "xts",
    "features": [
      "hats_rich",
      "hats_nnrt",
      "hats_drivers_peripheral_power_wakeup_cause_path",
      "hats_drivers_peripheral_battery_pc_macro_isolation"
    ],
    "adapted_system_type": [
      "mini",
      "small",
      "standard"
    ]
  }
}
```

---

## GN Target 类型

### 目标类型统计

| 类型 | 数量 | 用途 |
|------|------|------|
| `ohos_moduletest_suite` | 320+ | 模块级测试目标 |
| `ohos_test_suite` | 4 | 顶层测试套件聚合 |
| `group` | ~50 | 依赖分组 |
| `ohos_static_library` | 2 | 静态库 |
| `ohos_source_set` | 1 | 源码集合 |
| `hcpptest_suite` | 5 | LiteOS 小系统测试 |

---

## 主要 Target 清单

### 顶层测试套件

| Target | 类型 | 输出 | 描述 |
|--------|------|------|------|
| `xts_hats` | ohos_test_suite | 测试包 | 标准系统完整测试 |
| `hats_ivi` | ohos_test_suite | 测试包 | 车载信息娱乐系统 |
| `hats_intellitv` | ohos_test_suite | 测试包 | 智能电视 |
| `hats_wearable` | ohos_test_suite | 测试包 | 可穿戴设备 |

### 子系统聚合 Target

| Target | 路径 | 模块数 |
|--------|------|--------|
| `kernel` | `/kernel/BUILD.gn` | 80+ |
| `hatshdftest` | `/hdf/BUILD.gn` | 15+ |
| `powermgr` | `/powermgr/BUILD.gn` | 9 |
| `useriam` | `/useriam/BUILD.gn` | 8 |
| `telephony` | `/telephony/BUILD.gn` | 2 |
| `startup` | `/startup/BUILD.gn` | 2 |
| `hatsaitest` | `/ai/BUILD.gn` | 6 |
| `distributedhardware` | `/distributedhardware/BUILD.gn` | 2 |

---

## 模块级 Target 示例

### 标准系统测试（ohos_moduletest_suite）

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/kernel/syscalls/mem/mmap/BUILD.gn`

```gn
ohos_moduletest_suite("HatsMmapSyscallTest") {
  module_out_path = "hats/kernel/syscalls"  # 输出路径
  sources = [
    "src/mmap_api_test.cpp",
    "src/mmap_test.cpp",
  ]
  deps = [
    "//third_party/googletest:gtest_main",
  ]
  external_deps = [
    "c_utils:utils",
  ]
  include_dirs = [ "include" ]
  cflags = [ "-Wno-error" ]
  subsystem_name = "xts"
  part_name = "hats"
}
```

### HDF 驱动测试（带条件编译）

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/powermgr/battery/hdi_battery/BUILD.gn`

```gn
ohos_moduletest_suite("HatsPowermgrBatteryTest") {
  module_out_path = "hats/powermgr/battery"
  sources = [
    "common/hdi_battery_test.cpp",
  ]

  defines = [ "HDF_CONFIGURATION" ]

  external_deps = [
    "c_utils:utils",
    "drivers_interface_battery:libbattery_proxy_2.0",
    "hdf_core:libhdf_utils",
    "hilog:libhilog",
    "ipc:ipc_single",
  ]
}
```

### 静态库 Target

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/useriam/common/BUILD.gn`

```gn
ohos_source_set("useriam_common") {
  sources = [
    "src/iam_hat_test.cpp",
  ]
  include_dirs = [ "include" ]

  deps = [
    "//third_party/googletest:gtest",
    "//third_party/googletest:gmock",
  ]
}
```

### LiteOS 小系统测试（hcpptest_suite）

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/hdf/hdf_lite/manager/test_pm/BUILD.gn`

```gn
hcpptest_suite("HatsPmTest") {
  sources = [
    "src/pm_test.cpp",
  ]
  public_deps = [
    "//third_party/bounds_checking_function:libsec_shared",
    "//third_party/googletest:gtest",
  ]
  cflags = [ "-Wno-error" ]
}
```

---

## 依赖关系

### 通用依赖模式

```gn
# 测试框架依赖
deps = [
  "//third_party/googletest:gtest_main",  # 必须
]

# 工具库依赖
external_deps = [
  "c_utils:utils",      # 通用工具
  "hilog:libhilog",     # 日志
  "ipc:ipc_single",     # IPC
]

# 驱动接口依赖
external_deps = [
  "drivers_interface_battery:libbattery_proxy_2.0",  # HDI 绑定
  "drivers_interface_sensor:libsensor_proxy",        # 传感器
  "drivers_interface_audio:libaudio_proxy",          # 音频
]

# 安全依赖
external_deps = [
  "access_token:libaccesstoken_sdk",  # 权限检查
]
```

### 层级依赖

```
                    ┌────────────────────────────┐
                    │     googletest:gtest_main  │
                    └─────────────┬──────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
│  c_utils:utils    │   │ hilog:libhilog    │   │ ipc:ipc_single    │
└─────────┬─────────┘   └───────────────────┘   └───────────────────┘
          │
          ▼
┌───────────────────────────────────────────────────────────────┐
│                    驱动接口 (libbattery_proxy_2.0 等)         │
└───────────────────────────────────────────────────────────────┘
```

---

## 编译产物

### 产物清单

| 产物类型 | 文件模式 | 位置 |
|----------|----------|------|
| **测试可执行文件** | `Hats*.bin` | `out/{product}/suites/hats/testcases/` |
| **静态库** | `lib*.a` | `out/{product}/libs/` |
| **配置文件** | `Test.json` | 测试目录 |

### 产物映射表

| Target | 输出 | 路径 |
|--------|------|------|
| `HatsMmapSyscallTest` | `HatsMmapSyscallTest.bin` | `out/hats/kernel/syscalls/mem/mmap/` |
| `HatsPowermgrBatteryTest` | `HatsPowermgrBatteryTest.bin` | `out/hats/powermgr/battery/` |
| `HatsHdfAudioTest` | `HatsHdfAudioTest.bin` | `out/hats/hdf/audio/` |
| `HatsUserAuthTest` | `HatsUserAuthTest.bin` | `out/hats/useriam/` |

### 构建输出结构

```
out/{product}/
└── suites/
    └── hats/
        ├── testcases/           # 测试可执行文件
        │   ├── kernel/
        │   │   └── syscalls/
        │   │       └── mem/
        │   │           └── mmap/
        │   │               └── HatsMmapSyscallTest.bin
        │   ├── hdf/
        │   ├── powermgr/
        │   └── useriam/
        ├── tools/               # 测试工具
        ├── reports/             # 测试报告
        │   └── summary_report.html
        └── run.bat              # 执行脚本
```

---

## 构建命令

### 标准编译

```bash
# 全量编译
./build.sh product_name=hispark_taurus_standard suite=hats system_size=standard

# 指定子系统
./build.sh product_name=hispark_taurus_standard suite=hats target_subsystem=hdf

# 启用特性
./build.sh product_name=hispark_taurus_standard suite=hats hats_nnrt=true
```

### 编译变体

| 产品类型 | system_size | 目标设备 |
|----------|-------------|----------|
| standard | standard | 标准系统设备 |
| ivi | standard | 车载系统 |
| intellitv | standard | 智能电视 |
| wearable | standard | 可穿戴设备 |

---

## 测试套件选择

### test_packages.gni 逻辑

**文件**: `/Volumes/lexar/code/d/work/oh/test/xts/hats/test_packages.gni`

```python
# 伪代码：包选择逻辑
_all_test_packages = [
  "${HATS_ROOT}/distributedhardware:distributedhardware",
  "${HATS_ROOT}/hdf:hatshdftest",
  "${HATS_ROOT}/kernel:kernel",
  "${HATS_ROOT}/powermgr:powermgr",
  "${HATS_ROOT}/useriam:useriam",
  "${HATS_ROOT}/telephony:telephony",
  "${HATS_ROOT}/startup:startup",
  "${HATS_ROOT}/ai:hatsaitest",
]

# 基于产品类型的条件选择
selected_packages = []
if product_type == "standard":
    selected_packages = _all_test_packages
elif product_type == "ivi":
    selected_packages = selected_packages_ivi
elif product_type == "intellitv":
    selected_packages = selected_packages_intellitv
elif product_type == "wearable":
    selected_packages = selected_packages_wearable
```

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 概览 | [00_Overview.md](./00_Overview.md) | 项目定位 |
| 架构 | [01_Architecture.md](./01_Architecture.md) | 系统架构 |
| 模块 | [02_Modules.md](./02_Modules.md) | 子系统详解 |
| N-API | [03_N-API.md](./03_N-API.md) | HDI 接口清单 |
| 安全 | [05_Security.md](./05_Security.md) | 安全风险 |
| 附录 | [06_Appendix.md](./06_Appendix.md) | 调用链与配置 |

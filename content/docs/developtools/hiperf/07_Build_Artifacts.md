# 编译产物

## 目的

本文档说明 hiperf 的编译产物清单、安装路径和运行时加载关系。

## 适用范围

- 系统集成人员
- 需要部署 hiperf 的开发者
- 性能分析工具使用者

## 产物清单

### 1. 可执行文件

#### hiperf (设备端)

| 属性 | 值 |
|------|-----|
| **文件名** | `hiperf` |
| **类型** | ELF 可执行文件 |
| **目标平台** | OpenHarmony ARM64/ARM32 |
| **安装路径** | `/system/bin/hiperf` |
| **所属 Target** | `//developtools/hiperf:hiperf` |

**功能**:
- 性能采样（record）
- 事件列表（list）
- 计数器监控（stat）
- 数据转储（dump）
- 报告生成（report）

**运行时依赖**:
- `libhiperf_client.so`（可选，用于 API 调用）
- `libhilog.so`（日志）
- `libipc_core.so`（IPC）
- `libunwinder.so`（调用链回溯）

#### hiperf_host (Host 端)

| 属性 | Linux | Windows |
|------|-------|---------|
| **文件名** | `hiperf_host` | `hiperf_host.exe` |
| **类型** | ELF 可执行文件 | PE 可执行文件 |
| **目标平台** | x86_64 Linux | x86_64 Windows |
| **安装路径** | Host 工具目录 | Host 工具目录 |

**功能**:
- 数据转储（dump）
- 报告生成（report）
- 不支持采样（record/stat/list）

### 2. 动态库

#### libhiperf_client.so

| 属性 | 值 |
|------|-----|
| **文件名** | `libhiperf_client.so` |
| **类型** | 共享库 |
| **API 级别** | Platform SDK |
| **安装路径** | `/system/lib64/` 或 `/system/lib/` |
| **所属 Target** | `//developtools/hiperf/interfaces/innerkits/native/hiperf_client:hiperf_client` |

**导出符号**:
- `OHOS::Developtools::HiPerf::HiperfClient::Client`
- `OHOS::Developtools::HiPerf::HiperfClient::RecordOption`

**头文件**:
- `interfaces/innerkits/native/hiperf_client/include/hiperf_client.h`

**使用方式**:
```cpp
#include "hiperf_client.h"
```

#### libhiperf_local.so

| 属性 | 值 |
|------|-----|
| **文件名** | `libhiperf_local.so` |
| **类型** | 共享库 |
| **API 级别** | Platform SDK |
| **安装路径** | `/system/lib64/` 或 `/system/lib/` |
| **所属 Target** | `//developtools/hiperf/interfaces/innerkits/native/hiperf_local:hiperf_local` |

**导出符号** (由 `hiperf_local.map` 控制):
- `OHOS::Developtools::HiPerf::HiPerfLocal::Lperf::GetInstance`
- `OHOS::Developtools::HiPerf::HiPerfLocal::Lperf::StartProcessStackSampling`
- `OHOS::Developtools::HiPerf::HiPerfLocal::Lperf::CollectSampleStackByTid`
- `OHOS::Developtools::HiPerf::HiPerfLocal::Lperf::FinishProcessStackSampling`

**头文件**:
- `interfaces/innerkits/native/hiperf_local/include/lperf.h`

#### libhiperf_report.so / libhiperf_report.dll

| 属性 | Linux | Windows |
|------|-------|---------|
| **文件名** | `libhiperf_report.so` | `libhiperf_report.dll` |
| **类型** | 共享库 | DLL |
| **目标平台** | x86_64 Linux | x86_64 Windows |

**功能**:
- Host 端数据解析
- 报告生成
- 供 Python 脚本调用

### 3. 配置文件

#### hiperf.cfg

| 属性 | 值 |
|------|-----|
| **文件名** | `hiperf.cfg` |
| **类型** | JSON 配置文件 |
| **安装路径** | `/system/etc/init/hiperf.cfg` |
| **用途** | init 进程配置 |

**内容**:
```json
{
    "jobs": [{
        "name": "post-fs-data",
        "cmds": [
            "mkdir /data/log/hiperflog 0770 shell log",
            "restorecon /data/log/hiperflog",
            "chmod 0666 /dev/lperf"
        ]
    }]
}
```

#### hiperf.para

| 属性 | 值 |
|------|-----|
| **文件名** | `hiperf.para` |
| **类型** | 参数配置文件 |
| **安装路径** | `/system/etc/param/hiperf.para` |
| **用途** | 系统参数默认值 |

**内容**:
```
hiviewdfx.hiperf.perf_event_max_sample_rate=100000
hiviewdfx.hiperf.perf_cpu_time_max_percent=25
hiviewdfx.hiperf.perf_event_mlock_kb=516
```

### 4. Python 脚本

| 文件名 | 功能 | 运行环境 |
|--------|------|----------|
| `command_script.py` | 采样命令包装 | Host Python 3.7+ |
| `hiperf_utils.py` | 工具函数库 | Host Python 3.7+ |
| `make_report.py` | 生成 HTML 报告 | Host Python 3.7+ |
| `recv_binary_cache.py` | 收集符号表 | Host Python 3.7+ |
| `make_diff.py` | 生成对比报告 | Host Python 3.7+ |
| `report.html` | 报告模板 | - |

## 运行时加载关系

### 设备端运行时

```
hiperf (进程)
├── 动态链接依赖
│   ├── libhilog.so
│   ├── libipc_core.so
│   ├── libunwinder.so
│   ├── libprotobuf_lite.so
│   ├── libcjson.so
│   └── libz.so
│
├── 运行时加载 (dlopen)
│   └── (无)
│
└── 系统调用
    ├── perf_event_open
    ├── mmap/munmap
    ├── fork/exec
    └── /proc 文件系统访问
```

### API 调用时序

```
应用进程
├── 加载 libhiperf_client.so
│   └── 链接 libhilog.so
│
├── Client::Start()
│   ├── fork()
│   └── execv("/system/bin/hiperf")
│
└── 管道通信
    ├── Client -> Hiperf: 控制命令
    └── Hiperf -> Client: 响应
```

### Host 端运行时

```
hiperf_host
├── 动态链接依赖
│   ├── libhiperf_report.so
│   └── (Host 系统库)
│
└── Python 脚本调用
    └── libhiperf_report.so (通过 ctypes/CDLL)
```

## 产物输出路径

### 标准构建输出

```
out/ohos-arm-release/
├── developtools/hiperf/
│   ├── hiperf
│   └── libhiperf_client.so
│
├── clang_x64/
│   └── developtools/hiperf/
│       ├── hiperf_host
│       └── libhiperf_report.so
│
└── mingw_x86_64/
    └── developtools/hiperf/
        ├── hiperf_host.exe
        └── libhiperf_report.dll
```

### 打包输出

运行 `script/package.sh` 后:

```
out/host/developtools/hiperf/
├── bin/
│   ├── linux/x86_64/
│   │   ├── hiperf_host
│   │   └── libhiperf_report.so
│   ├── ohos/arm/
│   │   └── hiperf
│   └── windows/x86_64/
│       ├── hiperf_host.exe
│       └── libhiperf_report.dll
│
├── command_script.py
├── hiperf_utils.py
├── make_report.py
├── recv_binary_cache.py
└── report.html
```

## 调试符号

### 带符号版本

| 产物类型 | 路径模式 |
|----------|----------|
| 可执行文件 | `out/*/exe.unstripped/` |
| 共享库 | `out/*/lib.unstripped/` |

### 示例

```
out/ohos-arm-release/clang_x64/
├── exe.unstripped/clang_x64/developtools/hiperf/hiperf_host
└── lib.unstripped/clang_x64/developtools/hiperf/libhiperf_report.so
```

## 关键结论

1. **设备端为主**: 核心功能在设备端运行
2. **Host 端辅助**: Host 端仅用于数据分析和报告生成
3. **API 库分离**: 客户端库与主程序分离，支持独立使用
4. **配置即代码**: 通过 .cfg 和 .para 文件配置系统行为
5. **符号分离**: 调试符号单独存放，减小发布包体积

## 相关跳转

- [GN Targets](06_GN_Targets.md) - 构建目标详细说明
- [对外 API](04_Public_API.md) - API 使用说明
- [目录结构](02_Directory_Structure.md) - 代码组织

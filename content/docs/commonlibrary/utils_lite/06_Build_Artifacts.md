# 编译产物

> utils_lite 的编译产物清单、安装路径与运行时加载关系。

## 产物清单

### 静态库 (.a)

| 产物 | 模块 | 平台 | 说明 |
|------|------|------|------|
| `libnative_file.a` | file | liteos_m | 文件操作静态库 |
| `libstatic_hal_file.a` | hals/file | 全平台 | HAL 文件静态库 |
| `libace_kit_timer.a` | timer_task | liteos_m | 定时器任务静态库 |
| `libkal_timer.a` | kal/timer | liteos_m | KAL 定时器静态库 |
| `libace_kit_common.a` | js/builtin/common | liteos_m | JS 公共工具静态库 |
| `libace_kit_file.a` | js/builtin/filekit | liteos_m | JS 文件操作静态库 |
| `libace_kit_kvstore.a` | js/builtin/kvstorekit | liteos_m | JS KV 存储静态库 |
| `libace_kit_deviceinfo.a` | js/builtin/deviceinfokit | liteos_m | JS 设备信息静态库 |
| `libace_kit_common_simulator.a` | js/builtin/simulator | 全平台 | SDK 模拟器库 |
| `libace_kit_deviceinfo_simulator.a` | js/builtin/simulator | 全平台 | SDK 模拟器库 |
| `libace_kit_file_simulator.a` | js/builtin/simulator | 全平台 | SDK 模拟器库 |
| `libace_kit_kvstore_simulator.a` | js/builtin/simulator | 全平台 | SDK 模拟器库 |

### 动态库 (.so)

| 产物 | 模块 | 平台 | 说明 |
|------|------|------|------|
| `libnative_api.so` | native_api | liteos_a | NDK API 动态库 |
| `libace_kit_timer.so` | timer_task | liteos_a | 定时器任务动态库 |
| `libkal_timer.so` | kal/timer | liteos_a | KAL 定时器动态库 |
| `libace_kit_common.so` | js/builtin/common | liteos_a | JS 公共工具动态库 |
| `libace_kit_file.so` | js/builtin/filekit | liteos_a | JS 文件操作动态库 |
| `libace_kit_kvstore.so` | js/builtin/kvstorekit | liteos_a | JS KV 存储动态库 |
| `libace_kit_deviceinfo.so` | js/builtin/deviceinfokit | liteos_a | JS 设备信息动态库 |

---

## 产物映射表

### 按 Feature 映射

| Feature | Targets | 产物 |
|---------|---------|------|
| `utils_lite_feature_file` | file:file, file:native_file | libnative_file.a |
| `utils_lite_feature_kal_timer` | kal/timer:kal_timer | libkal_timer.a/.so |
| `utils_lite_feature_timer_task` | timer_task:ace_kit_timer | libace_kit_timer.a/.so |
| `utils_lite_feature_js_builtin` | js/builtin:ace_utils_kits | libace_kit_*.a/.so |

### 按平台映射

| 平台 | 产物类型 | 示例 |
|------|----------|------|
| liteos_m | 静态库 .a | libnative_file.a, libkal_timer.a |
| liteos_a | 动态库 .so | libnative_api.so, libkal_timer.so |

---

## 安装路径

### 典型安装路径

| 产物类型 | 路径模式 | 说明 |
|----------|----------|------|
| NDK 头文件 | `//commonlibrary/utils_lite/include/` | utils_config.h, utils_file.h |
| 系统库 | `//out/{product}/libs/` | libnative_api.so |
| SDK 模拟器 | `//sdk/ets/modules/` | 模拟器库 |

### NDK 头文件清单

| 头文件 | 说明 |
|--------|------|
| `utils_config.h` | 配置宏定义 |
| `utils_file.h` | 文件操作 API |
| `kv_store.h` | KV 存储 API |
| `utils_list.h` | 链表 API |
| `ohos_types.h` | 类型定义 |
| `ohos_errno.h` | 错误码 |
| `ohos_init.h` | 初始化框架 |

---

## 运行时加载关系

### 静态链接场景

```
应用程序
    │
    └── 静态链接 libnative_file.a
            │
            └── 依赖 hmos_spiffs (liteos_m)
            └── 依赖 libsec_shared (liteos_a)
```

### 动态链接场景

```
应用程序 (.hap)
    │
    └── 加载 libnative_api.so
            │
            ├── 依赖 libc.so
            ├── 依赖 libpthread.so
            └── 依赖 libcompiler_rt.so
```

### JS 运行时加载

```
ACE Engine Lite
    │
    ├── 加载 libace_kit_*.so (liteos_a)
    │       │
    │       ├── libace_kit_file.so ────────▶ libnative_api.so
    │       │                                   │
    │       │                                   └── 文件系统
    │       │
    │       ├── libace_kit_kvstore.so ───────▶ KV 存储
    │       │
    │       └── libace_kit_deviceinfo.so ───▶ 系统属性
    │
    └── 加载 libace_kit_timer.so
            │
            └── libkal_timer.so
                    │
                    └── POSIX Timer API
```

---

## SDK 模拟器

SDK 模拟器库仅在 `build_ohos_sdk = true` 时构建：

```gn
# js/builtin/simulator/BUILD.gn
if (build_ohos_sdk) {
  # 构建模拟器库
}
```

**模拟器用途**：
- IDE 预览器使用
- 本地调试时模拟 Native 实现

**模拟器目标**：
| 目标 | 路径 | 行号 |
|------|------|------|
| `ace_kit_common_simulator` | /js/builtin/simulator/BUILD.gn:20 |
| `ace_kit_deviceinfo_simulator` | /js/builtin/simulator/BUILD.gn:37 |
| `ace_kit_file_simulator` | /js/builtin/simulator/BUILD.gn:79 |
| `ace_kit_kvstore_simulator` | /js/builtin/simulator/BUILD.gn:104 |

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [目录结构](01_Directory_Structure.md) - 模块布局
- [GN 构建](05_GN_Build.md) - 构建配置
- [故障排查](08_Troubleshooting.md) - 构建问题

# 目录结构

## 顶层结构

```
battery_statistics/
├── figures/                      # 架构图
├── frameworks/                   # 框架层（API 暴露）
│   ├── napi/                    # JS/TS API（N-API）
│   │   ├── include/             # 头文件
│   │   └── src/                 # 源文件
│   └── ets/taihe/               # ArkTS API（Taihe）
├── interfaces/                   # 接口层
│   └── inner_api/                # 内部 API（C++ Kit）
├── sa_profile/                   # SA 配置
├── services/                     # 服务层（核心逻辑）
│   ├── native/                   # Native 服务
│   │   ├── include/              # 头文件
│   │   │   └── entities/         # 实体类（15 个）
│   │   └── src/                  # 源文件
│   └── profile/                  # 功耗配置
├── utils/                        # 工具类
├── bundle.json                   # 组件配置
├── batterystats.gni              # GN 配置
├── batterystats.yaml             # HiSysEvent 配置
└── README.md / README_zh.md
```

## frameworks/（框架层）

负责向外部暴露 API。

### frameworks/napi/（N-API）

**职责**：提供 JavaScript/TypeScript 接口。

| 文件 | 类型 | 说明 |
|------|------|------|
| `include/battery_stats.h` | .h | NAPI 主类声明 |
| `include/napi_utils.h` | .h | NAPI 工具函数 |
| `include/napi_error.h` | .h | N-API 错误处理 |
| `include/async_callback_info.h` | .h | 异步回调信息 |
| `src/battery_stats.cpp` | .cpp | NAPI 实现 |
| `src/battery_stats_module.cpp` | .cpp | **模块注册入口** |
| `src/napi_utils.cpp` | .cpp | 工具实现 |
| `src/napi_error.cpp` | .cpp | 错误处理 |
| `src/async_callback_info.cpp` | .cpp | 回调实现 |

**证据来源**：`frameworks/napi/BUILD.gn:26-32`

### frameworks/ets/taihe/（ArkTS）

**职责**：提供 ArkTS 接口（生成 ABC 字节码）。

**证据来源**：`bundle.json:57` - `batterystats_taihe`

## interfaces/inner_api/（内部接口）

**职责**：供其他模块调用的 C++ Kit。

| 文件 | 类型 | 说明 |
|------|------|------|
| `include/battery_stats_client.h` | .h | 客户端单例 |
| `include/battery_stats_errors.h` | .h | 错误码定义 |
| `include/battery_stats_info.h` | .h | 统计数据结构 |

**证据来源**：`bundle.json:69-75`

**编译产物**：`libbatterystats_client.z.so`

**证据来源**：`interfaces/inner_api/BUILD.gn:20`

## sa_profile/（SA 配置）

**职责**：定义 System Ability 元信息。

| 文件 | 说明 |
|------|------|
| `3304.json` | **SA ID 3304 配置** |
| `BUILD.gn` | SA Profile 构建配置 |

**证据来源**：`sa_profile/3304.json` + `sa_profile/BUILD.gn:17-20`

## services/native/（服务层）

**职责**：核心耗电统计逻辑。

### 核心服务

| 文件 | 类型 | 说明 |
|------|------|------|
| `include/battery_stats_service.h` | .h | **主服务类** |
| `include/battery_stats_core.h` | .h | 核心统计逻辑 |
| `include/battery_stats_detector.h` | .h | HiSysEvent 检测 |
| `include/battery_stats_parser.h` | .h | 功耗配置解析 |
| `include/battery_stats_listener.h` | .h | HiSysEvent 监听 |
| `include/battery_stats_subscriber.h` | .h | 公共事件订阅 |
| `include/battery_stats_dumper.h` | .h | Debug dump |
| `include/cpu_time_reader.h` | .h | CPU 时间读取 |
| `src/battery_stats_service.cpp` | .cpp | **服务实现** |
| `src/battery_stats_core.cpp` | .cpp | **核心实现** |
| `src/battery_stats_detector.cpp` | .cpp | 检测器实现 |
| `src/battery_stats_parser.cpp` | .cpp | 解析器实现 |
| `src/battery_stats_listener.cpp` | .cpp | 监听器实现 |
| `src/battery_stats_subscriber.cpp` | .cpp | 订阅者实现 |
| `src/battery_stats_dumper.cpp` | .cpp | Dump 实现 |
| `src/cpu_time_reader.cpp` | .cpp | CPU 时间实现 |

### 实体类（entities/）

**职责**：跟踪各硬件/软件组件的耗电。

| 文件 | 实体类型 | 跟踪对象 |
|------|----------|----------|
| `entities/battery_stats_entity.h` | 基类 | 所有实体的基类 |
| `entities/cpu_entity.h/cpp` | CpuEntity | CPU 耗电 |
| `entities/wifi_entity.h/cpp` | WifiEntity | Wi-Fi 耗电 |
| `entities/bluetooth_entity.h/cpp` | BluetoothEntity | 蓝牙耗电 |
| `entities/screen_entity.h/cpp` | ScreenEntity | 屏幕耗电 |
| `entities/phone_entity.h/cpp` | PhoneEntity | 通话/无线耗电 |
| `entities/audio_entity.h/cpp` | AudioEntity | 音频耗电 |
| `entities/camera_entity.h/cpp` | CameraEntity | 相机耗电 |
| `entities/flashlight_entity.h/cpp` | FlashlightEntity | 手电筒耗电 |
| `entities/gnss_entity.h/cpp` | GnssEntity | GNSS 耗电 |
| `entities/sensor_entity.h/cpp` | SensorEntity | 传感器耗电 |
| `entities/wakelock_entity.h/cpp` | WakelockEntity | 唤醒锁耗电 |
| `entities/alarm_entity.h/cpp` | AlarmEntity | 闹钟耗电 |
| `entities/uid_entity.h/cpp` | UidEntity | 按应用聚合 |
| `entities/idle_entity.h/cpp` | IdleEntity | 空闲状态耗电 |
| `entities/user_entity.h/cpp` | UserEntity | 按用户聚合 |

**证据来源**：`services/BUILD.gn:105-130`

## services/profile/（配置）

**职责**：功耗配置 JSON 文件。

| 文件 | 说明 |
|------|------|
| `power_average.json` | 设备平均功耗配置 |

**证据来源**：`services/BUILD.gn:198`

## utils/（工具类）

**职责**：通用工具函数。

| 文件 | 类型 | 说明 |
|------|------|------|
| `include/stats_utils.h` | .h | 核心工具、常量 |
| `include/stats_helper.h` | .h | 计时器、计数器 |
| `include/stats_types.h` | .h | 类型定义 |
| `include/stats_log.h` | .h | 日志宏 |
| `include/stats_errors.h` | .h | 错误定义 |
| `include/stats_common.h` | .h | 公共定义 |
| `include/stats_hisysevent.h` | .h | 系统事件 |
| `include/stats_xcollie.h` | .h | 看门狗 |
| `include/stats_cjson_utils.h` | .h | JSON 校验 |
| `src/*.cpp` | .cpp | 实现文件 |

**证据来源**：`utils/native/include/*.h`

## 构建配置

| 文件 | 说明 |
|------|------|
| `batterystats.gni` | GN 配置（feature flags、路径） |
| `BUILD.gn` | 各层 BUILD.gn 文件 |
| `bundle.json` | 组件配置 |

## 模块职责总结

| 目录 | 职责 | 稳定性 |
|------|------|--------|
| `frameworks/napi` | JS API 暴露 | **公开 API** |
| `frameworks/ets/taihe` | ArkTS API | **公开 API** |
| `interfaces/inner_api` | C++ Kit | **内部 API** |
| `services/native` | 核心逻辑 | **内部实现** |
| `utils` | 通用工具 | **内部实现** |
| `sa_profile` | SA 配置 | **配置** |

## 相关文档

- [架构设计](02_Architecture.md) - 组件关系与数据流
- [N-API 接口](03_NAPI.md) - JS API 清单
- [GN 构建](05_GN_Build.md) - Targets 列表

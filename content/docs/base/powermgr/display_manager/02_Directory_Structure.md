# 目录结构与模块职责 - display_manager

> 本文档说明 display_manager 模块的目录结构组织与各模块职责

---

## 文档目的

本文档提供：
- 完整的目录树（排除测试）
- 各子目录的职责说明
- 关键文件位置

## 适用范围

- **适用对象**：模块开发者、代码审查者、贡献者
- **前置知识**：熟悉 C++ 项目结构、GN 构建系统

---

## 完整目录树

```
base/powermgr/display_manager/
├── brightness_manager/                    # 亮度管理模块（静态库）
│   ├── include/                              # 头文件（23 个）
│   │   ├── brightness_action.h              # 亮度动作执行
│   │   ├── brightness_base.h               # 基础定义与枚举
│   │   ├── brightness_config_parser.h      # 亮度曲线配置解析
│   │   ├── brightness_dimming.h             # 渐变动画回调
│   │   ├── brightness_dimming_callback.h    # 渐变回调接口
│   │   ├── brightness_ffrt.h               # FFRT 任务封装
│   │   ├── brightness_manager.h            # 亮度管理器（对外接口）
│   │   ├── brightness_manager_ext.h        # 亮度扩展接口
│   │   ├── brightness_param_helper.h       # 亮度参数辅助
│   │   ├── brightness_service.h            # 亮度服务（组合层）
│   │   ├── brightness_setting_helper.h      # 设置存储读写
│   │   ├── calculation_config_parser.h     # 计算曲线配置
│   │   ├── calculation_curve.h             # 计算曲线类
│   │   ├── calculation_manager.h           # 计算管理器
│   │   ├── config_parser.h                # 配置解析基类
│   │   ├── config_parser_base.h          # 配置解析基类实现
│   │   ├── light_lux_buffer.h             # 光传感器缓冲
│   │   ├── light_lux_manager.h            # 光传感器管理器
│   │   ├── lux_filter_config_parser.h      # 滤波配置解析
│   │   ├── lux_threshold_config_parser.h   # 阈值配置解析
│   │   └── ilight_lux_manager.h           # 光传感器接口
│   ├── src/                                  # 源文件（20 个）
│   │   ├── brightness_action.cpp            # 亮度动作实现
│   │   ├── brightness_config_parser.cpp     # 配置解析实现
│   │   ├── brightness_dimming.cpp           # 渐变动画实现
│   │   ├── brightness_manager.cpp          # 亮度管理器实现
│   │   ├── brightness_manager_ext.cpp      # 亮度扩展实现
│   │   ├── brightness_param_helper.cpp      # 参数辅助实现
│   │   ├── brightness_service.cpp          # 亮度服务实现
│   │   ├── brightness_setting_helper.cpp   # 设置存储读写
│   │   ├── calculation_config_parser.cpp   # 计算配置解析
│   │   ├── calculation_curve.cpp         # 计算曲线实现
│   │   ├── calculation_manager.cpp       # 计算管理器实现
│   │   ├── config_parser.cpp              # 配置解析实现
│   │   ├── config_parser_base.cpp        # 配置解析基类实现
│   │   ├── light_lux_buffer.cpp           # 光缓冲实现
│   │   ├── light_lux_manager.cpp          # 光传感器管理实现
│   │   ├── lux_filter_config_parser.cpp   # 滤波配置解析
│   │   └── lux_threshold_config_parser.cpp # 阈值配置解析
│   ├── BUILD.gn                               # GN 构建配置
│   └── test/                                 # 测试代码（不作为业务证据）
│       └── unittest/                          # 单元测试
│
├── state_manager/                          # 显示状态管理模块（System Ability）
│   ├── figures/                              # 架构图
│   │   └── power-management-subsystem-architecture.png
│   ├── frameworks/                            # Framework 层
│   │   ├── napi/                        # 传统 N-API JS 绑定
│   │   │   ├── brightness_module.cpp     # N-API 模块注册
│   │   │   └── brightness.cpp              # N-API 方法实现
│   │   └── ets/taihe/                  # 新版 ETS/ANI 绑定
│   │       ├── brightness/                # @ohos.brightness 模块
│   │       │   ├── idl/
│   │       │   │   └── ohos.brightness.taihe    # IDL 定义
│   │       │   ├── src/
│   │       │   │   ├── ani_constructor.cpp    # ANI 构造函数
│   │       │   │   └── ohos.brightness.impl.cpp # ANI 实现
│   │       │   └── BUILD.gn                  # Taihe 构建配置
│   │       └── BUILD.gn                        # Taihe 组目标
│   ├── interfaces/                            # 接口层
│   │   └── inner_api/                   # 内部 API（Client 库）
│   │       ├── native/
│   │       │   ├── include/
│   │       │   │   ├── display_brightness_callback_stub.h      # 回调 Stub
│   │       │   │   ├── display_brightness_callback_ipc_interface_code.h
│   │       │   │   ├── display_brightness_listener_stub.h         # 监听 Stub
│   │       │   │   ├── display_brightness_listener_ipc_interface_code.h
│   │       │   │   ├── display_power_callback_stub.h             # 电源回调 Stub
│   │       │   │   ├── display_power_callback_ipc_interface_code.h
│   │       │   │   ├── display_power_info.h                  # 数据结构定义
│   │       │   │   ├── display_power_mgr_client.h             # Client 类定义
│   │       │   │   ├── display_mgr_errors.h                # 错误码定义
│   │       │   │   ├── idisplay_brightness_callback.h          # 回调接口
│   │       │   │   ├── idisplay_brightness_listener.h         # 监听接口
│   │       │   │   └── idisplay_power_callback.h              # 电源回调接口
│   │       │   └── src/
│   │       │       └── display_power_mgr_client.cpp        # Client 实现
│   │       └── BUILD.gn                        # Client 库构建配置
│   ├── sa_profile/                            # System Ability 配置
│   │   ├── 3308.json                      # SA ID 3308 配置
│   │   └── BUILD.gn                        # SA 配置构建
│   ├── service/                               # 服务层（System Ability 实现）
│   │   ├── native/
│   │   │   ├── include/
│   │   │   │   ├── display_auto_brightness.h      # 自动亮度逻辑
│   │   │   │   ├── display_common_event_mgr.h    # 公共事件管理
│   │   │   │   ├── display_param_helper.h         # 参数辅助
│   │   │   │   ├── display_power_mgr_service.h    # 服务类定义
│   │   │   │   ├── display_system_ability.h       # SA 类定义
│   │   │   │   ├── display_xcollie.h             # XCollie 监控
│   │   │   │   ├── gradual_animator.h            # 渐变动画器
│   │   │   │   ├── miscellaneous_display_power_strategy.h  # 关闭策略（可选）
│   │   │   │   ├── screen_action.h                # 屏幕动作
│   │   │   │   └── screen_controller.h            # 屏幕控制器
│   │   │   └── src/
│   │   │       ├── display_auto_brightness.cpp
│   │   │       ├── display_common_event_mgr.cpp
│   │   │       ├── display_param_helper.cpp
│   │   │       ├── display_power_mgr_service.cpp   # 服务实现
│   │   │       ├── display_system_ability.cpp    # SA 生命周期
│   │   │       ├── display_xcollie.cpp
│   │   │       ├── gradual_animator.cpp
│   │   │       ├── miscellaneous_display_power_strategy.cpp
│   │   │       ├── screen_action.cpp
│   │   │       └── screen_controller.cpp
│   │   ├── zidl/                         # ZIDL 生成代码
│   │   │   ├── include/
│   │   │   │   ├── display_brightness_callback_proxy.h        # 回调 Proxy
│   │   │   │   ├── display_brightness_listener_proxy.h       # 监听 Proxy
│   │   │   │   └── display_power_callback_proxy.h           # 电源 Proxy
│   │   │   └── src/
│   │   │       ├── display_brightness_callback_proxy.cpp
│   │   │       ├── display_brightness_listener_proxy.cpp
│   │   │       └── display_power_callback_proxy.cpp
│   │   ├── BUILD.gn                           # 服务 GN 配置
│   │   ├── IDisplayPowerMgr.idl             # 主服务 IDL
│   │   └── DisplayPowerMgrIdlTypes.idl      # 类型定义 IDL
│   ├── utils/                                 # 工具层
│   │   ├── native/
│   │   │   ├── include/
│   │   │   │   ├── delayed_sp_singleton.h       # 延迟单例模板
│   │   │   │   └── display_xcollie.h           # XCollie 包装
│   │   │   └── src/
│   │   │       └── display_xcollie.cpp
│   │   └── BUILD.gn                              # 工具 GN 配置
│   ├── frameworks/
│   │   └── native/
│   │       └── display_power_mgr_client.cpp  # frameworks 层客户端
│   └── test/                                    # 测试代码（不作为业务证据）
│       ├── fuzztest/
│       ├── systemtest/
│       └── unittest/
│
├── bundle.json                              # 组件元数据
├── displaymgr.gni                           # GN 全局配置
├── displaymanager.yaml                      # 显示管理 HisyEvent 配置
├── powermanager.yaml                       # 电源管理 HisyEvent 配置
├── LICENSE                                 # Apache 2.0 许可证
├── README.md / README_zh.md                # 项目说明
└── figures/                                # 文档插图
```

**代码统计**：
- **总文件数**（非测试）：约 93 个
- **brightness_manager**：23 头文件 + 20 源文件
- **state_manager**：11 服务层源文件 + frameworks 层文件

---

## 模块职责

### brightness_manager（亮度管理模块）

**定位**：核心亮度管理静态库（libbrightness_manager.a）

**职责**：
1. **亮度计算**：根据环境光、场景、用户偏好计算目标亮度
2. **亮度设置**：协调 ScreenController 执行亮度设置
3. **渐变动画**：管理 GradualAnimator 执行平滑过渡
4. **配置管理**：解析亮度曲线、光阈值等 JSON 配置
5. **场景适配**：支持游戏/相机等不同场景的亮度曲线
6. **数据监听**：分发亮度变化事件给注册监听器

**关键类**（证据：`brightness_manager/include/`）：

| 类 | 文件 | 职责 |
|------|------|------|
| BrightnessManager | brightness_manager.h:24 | 对外亮度管理接口 |
| BrightnessService | brightness_service.h:51 | 亮度服务组合层 |
| BrightnessCalculationManager | calculation_manager.h:28 | 亮度曲线插值计算 |
| LightLuxManager | light_lux_manager.h:27 | 光传感器数据管理 |
| GradualAnimator | （通过 ScreenController 使用） | 渐变动画（由 state_manager 提供） |

**依赖**：无（自包含静态库）

**被依赖方**：`state_manager/service`（作为 public_deps 链接）

---

### state_manager（显示状态管理模块）

**定位**：System Ability 服务（SA ID 3308）

**职责**：
1. **屏幕控制**：管理 ScreenController 控制屏幕开关
2. **电源策略**：实现屏幕关闭延迟、Doze 等策略
3. **IPC 服务**：通过 ZIDL 接口对外提供显示管理服务
4. **亮度协调**：调用 brightness_manager 实现亮度逻辑
5. **事件通知**：分发电源状态变化事件

**子模块职责**：

#### frameworks/napi/（传统 N-API）

**职责**：提供 JS 调用接口（@ohos.display.brightness 模块）

**证据**：`state_manager/frameworks/napi/brightness_module.cpp:216`
- 导出 5 个 JS 方法：getValue, setValue, getMode, setMode, setKeepScreenOn

**输出**：`libbrightness.so`（system/lib/module/）

#### frameworks/ets/taihe/（新版 ETS 绑定）

**职责**：提供新版 ArkTS 调用接口（@ohos.brightness 命名空间）

**证据**：`state_manager/frameworks/ets/taihe/brightness/src/ani_constructor.cpp:19`
- 导出 2 个 setValue 方法重载：SetValueContinuous, SetValueInt

**输出**：`libdisplay_manager_brightness_taihe_native.so`（system/lib/）

#### frameworks/native/

**职责**：提供 frameworks 层的客户端实现

**证据**：`state_manager/frameworks/native/display_power_mgr_client.cpp`
- 实现 DisplayPowerMgrClient 延迟单例

#### service/native/（核心服务实现）

**职责**：System Ability 服务实现（libdisplaymgrservice.so）

**关键类**（证据：`state_manager/service/native/include/`）：

| 类 | 文件 | 职责 |
|------|------|------|
| DisplaySystemAbility | display_system_ability.h:29 | SA 生命周期管理 |
| DisplayPowerMgrService | display_power_mgr_service.h:44 | IPC 服务实现 |
| ScreenController | screen_controller.h:30 | 屏幕控制器 |
| GradualAnimator | gradual_animator.h | 渐变动画器 |

#### interfaces/inner_api/（内部 API）

**职责**：提供 C++ 客户端接口（libdisplaymgr.so）

**关键类**（证据：`state_manager/interfaces/inner_api/native/include/`）：

| 类 | 文件 | 职责 |
|------|------|------|
| DisplayPowerMgrClient | display_power_mgr_client.h:31 | 延迟单例客户端 |

**输出**：`libdisplaymgr.so`（system/lib/platformsdk/）

#### service/zidl/（ZIDL 生成代码）

**职责**：Proxy/Stub 实现

**证据**：`state_manager/service/zidl/`
- display_power_callback_proxy.cpp/stub.cpp
- display_brightness_callback_proxy.cpp/stub.cpp
- display_brightness_listener_proxy.cpp/stub.cpp

#### utils/native/（工具类）

**职责**：通用工具函数

**关键类**（证据：`state_manager/utils/native/include/`）：

| 类 | 文件 | 职责 |
|------|------|------|
| DelayedSpSingleton | delayed_sp_singleton.h | 延迟单例模板 |

---

## 配置文件

| 配置文件 | 位置 | 用途 | 格式 |
|---------|------|------|------|
| 3308.json | state_manager/sa_profile/ | System Ability 注册 | JSON |
| display.para | state_manager/service/etc/ | 亮度参数 | 系统 param |
| display.para.dac | state_manager/service/etc/ | 参数 DAC | 系统 param |
| 亮度曲线配置 | brightness_manager/src/ | 亮度-光强映射 | JSON（未看到具体文件名） |
| displaymanager.yaml | 根目录 | HisyEvent 配置 | YAML |

---

## 编译产物映射

| 目标 | 输出文件 | 安装位置 |
|------|----------|---------|
| displaymgrservice | libdisplaymgrservice.z.so | system/lib/ |
| displaymgr | libdisplaymgr.so | system/lib/platformsdk/ |
| brightness | libbrightness.so | system/lib/module/ |
| display_manager_brightness_taihe_native | libdisplay_manager_brightness_taihe_native.so | system/lib/ |

---

## 相关链接

- **架构文档**：[03_Architecture.md](03_Architecture.md)
- **GN 文档**：[06_GN_Targets.md](06_GN_Targets.md)

---

## 文档更新记录

- **2026-02-06**：初始版本 v1.0

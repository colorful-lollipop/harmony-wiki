# 目录结构与模块职责

## 目的

本文档详细说明 ace_engine_lite 项目的目录组织和各模块职责，帮助开发者快速定位代码和理解模块关系。

## 适用范围

- arkui_ace_engine_lite 3.1 版本
- 所有源代码目录（排除 test/）

---

## 完整目录树

```
/Volumes/lexar/code/d/work/oh/foundation/arkui/ace_engine_lite/
├── frameworks/                    # 框架代码核心
│   ├── common/                   # 公共工具和基础类
│   │   ├── log/              # 日志系统 (ace_log.cpp)
│   │   ├── memory/           # 内存管理
│   │   │   ├── cache/        # 缓存管理 (cache_manager.cpp)
│   │   │   └── mem_proc/      # 内存处理 (mem_proc.cpp)
│   │   └── utils/           # 工具类
│   ├── examples/                 # 示例代码
│   │   ├── airquality/       # 空气质量示例
│   │   ├── alarm/            # 闹钟示例
│   │   ├── calculator/       # 计算器示例
│   │   ├── music/            # 音乐播放器示例
│   │   └── testRotation/      # 旋转测试示例
│   ├── include/                  # 公共头文件
│   │   ├── base/             # 基础类接口
│   │   ├── context/          # 上下文接口
│   │   ├── modules/          # 模块接口
│   │   └── resource/         # 资源接口
│   ├── module_manager/           # JS 模块管理器
│   │   ├── module_manager.h
│   │   ├── module_manager.cpp
│   │   └── ohos_module_config.h
│   ├── native_engine/           # JS 引擎适配层
│   │   ├── async/             # 异步操作
│   │   │   ├── js_async_work.cpp
│   │   │   └── message_queue_utils.cpp
│   │   └── jsi/              # JSI 封装层
│   │       ├── jsi.h
│   │       ├── jsi.cpp
│   │       └── internal/
│   ├── packages/                # JS 实现和库
│   ├── src/                    # 核心源代码
│   │   ├── core/             # 核心框架实现
│   │   │   ├── animation/    # 动画系统
│   │   │   │   └── transition_impl.cpp
│   │   │   ├── base/          # 基础类和工具
│   │   │   │   ├── ace_lock.cpp
│   │   │   │   ├── async_task_manager.cpp
│   │   │   │   ├── dft_impl.cpp
│   │   │   │   ├── dfx_assist.cpp
│   │   │   │   ├── event_util.cpp
│   │   │   │   ├── js_debugger_config.cpp
│   │   │   │   ├── js_fwk_common.cpp
│   │   │   │   ├── key_parser.cpp
│   │   │   │   ├── lazy_load_manager.cpp
│   │   │   │   ├── lazy_load_watcher.cpp
│   │   │   │   ├── locale_util.cpp
│   │   │   │   ├── number_parser.cpp
│   │   │   │   ├── product_adapter.cpp
│   │   │   │   ├── string_util.cpp
│   │   │   │   ├── system_info.cpp
│   │   │   │   └── time_util.cpp
│   │   │   ├── components/    # UI 组件系统
│   │   │   │   ├── component.cpp (基类)
│   │   │   │   ├── component_utils.cpp
│   │   │   │   ├── event_listener.h/cpp
│   │   │   │   ├── input_button_component.cpp
│   │   │   │   ├── input_checkbox_component.cpp
│   │   │   │   ├── input_edittext_component.cpp
│   │   │   │   ├── input_radio_component.cpp
│   │   │   │   ├── list_adapter.cpp
│   │   │   │   ├── list_component.cpp
│   │   │   │   ├── marquee_component.cpp
│   │   │   │   ├── scroll_layer.cpp
│   │   │   │   ├── slider_component.cpp
│   │   │   │   ├── stack_component.cpp
│   │   │   │   ├── swiper_component.cpp
│   │   │   │   ├── switch_component.cpp
│   │   │   │   ├── tab_bar_component.cpp
│   │   │   │   ├── tab_content_component.cpp
│   │   │   │   ├── tabs_component.cpp
│   │   │   │   ├── text_component.cpp
│   │   │   │   ├── image_component.cpp
│   │   │   │   ├── chart_component.cpp
│   │   │   │   ├── canvas_component.cpp
│   │   │   │   ├── video_component.cpp
│   │   │   │   ├── analog_clock_component.cpp
│   │   │   │   ├── clock_hand_component.cpp
│   │   │   │   ├── panel_view.cpp
│   │   │   │   ├── picker_view_component.cpp
│   │   │   │   ├── qrcode_component.cpp
│   │   │   │   ├── circle_progress_component.cpp
│   │   │   │   ├── horizon_progress_component.cpp
│   │   │   │   ├── camera_component.cpp
│   │   │   │   └── image_animator_component.cpp
│   │   │   ├── context/       # 上下文和环境
│   │   │   │   ├── ace_ability.cpp
│   │   │   │   ├── js_ability.cpp
│   │   │   │   ├── js_ability_impl.cpp
│   │   │   │   ├── js_app_context.cpp
│   │   │   │   ├── js_app_environment.cpp
│   │   │   │   ├── js_framework_raw.cpp
│   │   │   │   ├── js_profiler.cpp
│   │   │   │   ├── js_timer_list.cpp
│   │   │   │   ├── fatal_handler.cpp
│   │   │   │   ├── ace_event_error_code.cpp
│   │   │   │   └── slite_ace_ability.cpp
│   │   │   ├── dialog/         # 对话框实现
│   │   │   │   └── js_dialog.cpp
│   │   │   ├── directive/      # 指令系统
│   │   │   │   ├── descriptor_utils.cpp
│   │   │   │   └── directive_watcher_callback.cpp
│   │   │   ├── modules/        # JS 模块实现
│   │   │   │   ├── app_module.cpp
│   │   │   │   ├── dialog_module.cpp
│   │   │   │   ├── digital_crown_module.cpp
│   │   │   │   ├── dfx_module.cpp
│   │   │   │   ├── router_module.cpp
│   │   │   │   └── sample_module.cpp
│   │   │   ├── modules/presets/  # 预设模块
│   │   │   │   ├── app_data_module.cpp
│   │   │   │   ├── cjson_parser.cpp
│   │   │   │   ├── console_log_impl.cpp
│   │   │   │   ├── console_module.cpp
│   │   │   │   ├── date_time_format_module.cpp
│   │   │   │   ├── image_module.cpp
│   │   │   │   ├── intl_module.cpp
│   │   │   │   ├── localization_module.cpp
│   │   │   │   ├── number_format_module.cpp
│   │   │   │   ├── preset_module.cpp
│   │   │   │   ├── profiler_module.cpp
│   │   │   │   ├── render_module.cpp
│   │   │   │   ├── require_module.cpp
│   │   │   │   ├── syscap_module.cpp
│   │   │   │   ├── timer_module.cpp
│   │   │   │   ├── version_module.cpp
│   │   │   │   └── feature_ability_module.cpp
│   │   │   ├── router/        # 路由和页面管理
│   │   │   │   ├── js_router.cpp
│   │   │   │   ├── js_page_state.cpp
│   │   │   │   └── js_page_state_machine.cpp
│   │   │   ├── stylemgr/      # 样式管理
│   │   │   │   ├── app_style.cpp
│   │   │   │   ├── app_style_item.cpp
│   │   │   │   ├── app_style_list.cpp
│   │   │   │   ├── app_style_manager.cpp
│   │   │   │   ├── app_style_sheet.cpp
│   │   │   │   ├── condition_arbitrator.cpp
│   │   │   │   ├── link_queue.cpp
│   │   │   │   └── link_stack.cpp
│   │   │   └── wrapper/       # JS 包装工具
│   │   │       └── js.cpp
│   │   ├── resource/        # 资源文件
│   │   │   ├── video_muted_image_res.cpp
│   │   │   └── video_play_image_res.cpp
│   │   └── targets/         # 平台适配
│   │       └── platform_adapter.cpp
│   └── tools/                   # 开发工具
│       ├── profiler/           # 性能分析工具
│       ├── qt/                 # Qt 模拟器
│       │   └── simulator/
│       ├── snapshot/           # 快照工具
│       └── syscap/            # 系统能力工具
├── interfaces/                # 对外接口
│   └── inner_api/          # 内部子系统 API
│       └── builtin/          # JS 模块对外接口
│           ├── async/          # 异步操作接口
│           │   ├── message_queue_utils.h
│           │   └── js_async_work.h
│           ├── base/           # 基础接口
│           │   ├── ace_mem_base.h
│           │   └── memory_heap.h
│           └── jsi/            # JSI 接口
│               ├── jsi.h
│               └── jsi_types.h
├── test/                    # 测试用例（排除）
├── figures/                  # 文档图片
├── ace_lite.gni            # GN 构建配置
├── bundle.json              # Bundle 配置
├── simulator.gni            # 模拟器配置
└── wiki/                    # 本文档
```

---

## 模块职责详解

### 1. frameworks/common/ - 公共基础

| 子目录 | 职责 | 主要文件 |
|--------|------|---------|
| log/ | 日志系统，提供 HILOG_* 宏 | ace_log.cpp |
| memory/cache/ | 缓存管理，LRU 缓存实现 | cache_manager.cpp |
| memory/mem_proc/ | 内存处理和分配 | mem_proc.cpp |
| utils/ | 工具类和辅助函数 | - |

**证据**：`frameworks/common/BUILD.gn:21-54`（ace_common target）

### 2. frameworks/native_engine/ - JS 引擎适配

| 子目录 | 职责 | 主要文件 |
|--------|------|---------|
| async/ | 异步操作和任务队列 | js_async_work.cpp, message_queue_utils.cpp |
| jsi/ | JerryScript 封装层（JSI） | jsi.h, jsi.cpp |

**证据**：`frameworks/native_engine/BUILD.gn:23-72`（ace_native_engine target）

### 3. frameworks/module_manager/ - 模块管理

| 文件 | 职责 |
|------|------|
| module_manager.cpp | 模块加载和初始化 |
| ohos_module_config.h | 模块配置定义（OHOS_MODULES 数组） |

**证据**：`frameworks/module_manager/BUILD.gn:22-96`（ace_module_manager target）

### 4. frameworks/src/core/ - 核心框架

#### animation/ - 动画系统
| 文件 | 职责 |
|------|------|
| transition_impl.cpp | 基础过渡动画实现 |

#### base/ - 基础类和工具
| 文件 | 职责 |
|------|------|
| ace_lock.cpp | 互斥锁 |
| async_task_manager.cpp | 异步任务管理 |
| dft_impl.cpp | DFT（默认值）实现 |
| dfx_assist.cpp | DFX（可观测性）辅助 |
| event_util.cpp | 事件工具 |
| js_debugger_config.cpp | JS 调试器配置 |
| js_fwk_common.cpp | JS 框架通用工具 |
| key_parser.cpp | 键值解析 |
| lazy_load_manager.cpp | 延迟加载管理 |
| lazy_load_watcher.cpp | 延迟加载监听 |
| locale_util.cpp | 本地化工具 |
| number_parser.cpp | 数字解析 |
| product_adapter.cpp | 产品适配层 |
| string_util.cpp | 字符串工具 |
| system_info.cpp | 系统信息 |
| time_util.cpp | 时间工具 |

#### components/ - UI 组件系统

| 分类 | 文件 | 职责 |
|------|------|------|
| 基础 | component.cpp | 组件基类 |
| | component_utils.cpp | 组件工具 |
| | event_listener.h/cpp | 事件监听器 |
| 输入 | input_button_component.cpp | 按钮组件 |
| | input_checkbox_component.cpp | 复选框组件 |
| | input_edittext_component.cpp | 文本输入组件 |
| | input_radio_component.cpp | 单选框组件 |
| 布局 | div_component.cpp | 容器组件 |
| | list_component.cpp | 列表组件 |
| | stack_component.cpp | 堆叠组件 |
| | swiper_component.cpp | 滑动容器组件 |
| | tabs_component.cpp | 标签页容器 |
| | tab_bar_component.cpp | 标签栏 |
| | tab_content_component.cpp | 标签页内容 |
| 显示 | text_component.cpp | 文本显示 |
| | image_component.cpp | 图像显示 |
| | chart_component.cpp | 图表组件 |
| | canvas_component.cpp | 画布组件 |
| | video_component.cpp | 视频播放 |
| 进度 | circle_progress_component.cpp | 圆形进度 |
| | horizon_progress_component.cpp | 水平进度 |
| | slider_component.cpp | 滑块组件 |
| 其他 | picker_view_component.cpp | 选择器视图 |
| | qrcode_component.cpp | 二维码 |
| | marquee_component.cpp | 跑马灯 |
| | scroll_layer.cpp | 滚动层 |
| | panel_view.cpp | 面板视图 |
| | analog_clock_component.cpp | 模拟时钟 |
| | clock_hand_component.cpp | 时钟指针 |
| | image_animator_component.cpp | 图像动画 |
| | list_adapter.cpp | 列表适配器 |
| | camera_component.cpp | 相机组件 |

#### context/ - 上下文管理

| 文件 | 职责 |
|------|------|
| ace_ability.cpp | Ability 上下文（liteos_a/linux） |
| js_ability.cpp | JS Ability 基类 |
| js_ability_impl.cpp | JS Ability 实现 |
| js_app_context.cpp | JS 应用上下文 |
| js_app_environment.cpp | JS 应用环境 |
| js_framework_raw.cpp | 原始框架初始化 |
| js_profiler.cpp | JS 性能分析 |
| js_timer_list.cpp | 定时器列表 |
| fatal_handler.cpp | 致命错误处理 |
| ace_event_error_code.cpp | 事件错误码 |
| slite_ace_ability.cpp | LiteOS-M Ability 实现 |

#### modules/ - JS 模块

| 文件 | 职责 |
|------|------|
| app_module.cpp | 应用信息模块 |
| dialog_module.cpp | 对话框模块 |
| digital_crown_module.cpp | 数码表冠模块 |
| dfx_module.cpp | DFX 模块 |
| router_module.cpp | 路由模块 |
| sample_module.cpp | 示例模块 |

#### modules/presets/ - 预设 JS 模块

| 文件 | 职责 |
|------|------|
| app_data_module.cpp | 应用数据 |
| cjson_parser.cpp | JSON 解析器 |
| console_log_impl.cpp | Console 日志实现 |
| console_module.cpp | Console 模块 |
| date_time_format_module.cpp | 日期时间格式化 |
| image_module.cpp | 图像处理 |
| intl_module.cpp | 国际化（Intl） |
| localization_module.cpp | 本地化 |
| number_format_module.cpp | 数字格式化 |
| preset_module.cpp | 预设模块基类 |
| profiler_module.cpp | 性能分析模块 |
| render_module.cpp | 渲染模块 |
| require_module.cpp | Require 实现 |
| syscap_module.cpp | 系统能力模块 |
| timer_module.cpp | 定时器模块 |
| version_module.cpp | 版本模块 |
| feature_ability_module.cpp | Feature Ability 模块（跨进程通信） |

#### router/ - 路由系统

| 文件 | 职责 |
|------|------|
| js_router.cpp | 路由接口 |
| js_page_state.cpp | 页面状态 |
| js_page_state_machine.cpp | 页面状态机 |

#### stylemgr/ - 样式管理

| 文件 | 职责 |
|------|------|
| app_style.cpp | 样式定义 |
| app_style_item.cpp | 样式项 |
| app_style_list.cpp | 样式列表 |
| app_style_manager.cpp | 样式管理器 |
| app_style_sheet.cpp | 样式表 |
| condition_arbitrator.cpp | 条件仲裁器 |
| link_queue.cpp | 链接队列 |
| link_stack.cpp | 链接栈 |

#### wrapper/ - JS 包装

| 文件 | 职责 |
|------|------|
| js.cpp | JS 工具函数 |

#### dialog/ - 对话框

| 文件 | 职责 |
|------|------|
| js_dialog.cpp | JS 对话框实现 |

#### directive/ - 指令系统

| 文件 | 职责 |
|------|------|
| descriptor_utils.cpp | 描述符工具 |
| directive_watcher_callback.cpp | 指令监听回调 |

#### resource/ - 资源

| 文件 | 职责 |
|------|------|
| video_muted_image_res.cpp | 静音图标资源 |
| video_play_image_res.cpp | 播放图标资源 |

#### targets/ - 平台适配

| 文件 | 职责 |
|------|------|
| platform_adapter.cpp | 平台适配层 |

### 5. frameworks/tools/ - 开发工具

| 目录 | 职责 |
|------|------|
| profiler/ | 性能分析工具 |
| qt/simulator/ | Qt 模拟器（PC 调试） |
| snapshot/ | 快照工具 |
| syscap/ | 系统能力工具 |

### 6. interfaces/inner_api/builtin/ - 对外接口

| 目录 | 职责 |
|------|------|
| async/ | 异步操作接口 |
| base/ | 基础接口 |
| jsi/ | JSI 接口定义 |

---

## 依赖关系图

### 内部依赖

```
ace_lite (主库)
  ├── ace_common (日志、内存)
  ├── ace_native_engine (JS 引擎适配)
  │   └── jerryscript (外部依赖)
  ├── ace_module_manager (模块管理)
  │   ├── ace_common
  │   └── ace_native_engine
  └── ui_lite (图形框架)
```

### 外部依赖

| 依赖 | 用途 |
|------|------|
| jerryscript | JavaScript 引擎 |
| ui_lite | 2D 图形框架 |
| i18n_lite | 国际化 |
| resource_management_lite | 资源管理 |
| kv_store | KV 存储 |
| ability_lite | Ability 框架 |
| timer_task | 定时器任务 |
| bounds_checking_function | 安全 C 函数 |

**证据**：`bundle.json:24-47`（deps 列表）

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 概览
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [04_JS_API.md](04_JS_API.md) - JS 模块 API
- [05_Inner_API.md](05_Inner_API.md) - 内部 API
- [06_GN_Targets.md](06_GN_Targets.md) - 构建目标

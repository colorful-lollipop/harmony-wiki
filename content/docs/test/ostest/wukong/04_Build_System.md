# WuKong 构建系统

## 构建配置概述

WuKong 使用 **GN (Generate Ninja)** 构建系统，配置文件位于根目录 `BUILD.gn`。

> 证据: `BUILD.gn:14` - `import("//build/ohos.gni")`

## 主要 Target

### wukong 可执行程序

```gn
// 证据: BUILD.gn:27-139
ohos_executable("wukong") {
  # 配置
  configs = [ ":wukong_common_config" ]
  subsystem_name = "ostest"
  part_name = "wukong"
  output_name = "wukong"
  install_enable = true

  # 源文件
  sources = [
    # 51 个 .cpp 源文件
  ]

  # 头文件目录
  include_dirs = [
    "./common/include",
    "./component_event/include",
    "./input_factory/include",
    "./report/include",
    "./shell_command/include",
    "./test_flow/include",
  ]

  # 依赖配置
  external_deps = [
    "ability_base:want",
    "ability_runtime:*",
    "accessibility:*",
    "bundle_framework:*",
    "c_utils:utils",
    "graphic_2d:*",
    "hidumper:lib_dump_usage",
    "hilog:libhilog",
    "hisysevent:*",
    "image_framework:image_native",
    "init:libbegetutil",
    "input:libmmi-client",
    "ipc:ipc_core",
    "samgr:samgr_proxy",
    "window_manager:*",
    "libpng:libpng",
  ]

  # 宏定义
  defines = [
    "LOG_TAG=\"WuKong\"",
    "LOG_DOMAIN = 0xD003200",
  ]
}
```

## 配置详情

### wukong_common_config

```gn
// 证据: BUILD.gn:18-21
config("wukong_common_config") {
  cflags = [ "-D__OHOS__" ]
  cflags_cc = [ "-fexceptions" }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `__OHOS__` | 宏定义 | 标记为 OpenHarmony 代码 |
| `-fexceptions` | C++ 标志 | 启用 C++ 异常处理 |

## 源文件清单

### common 模块 (7 个文件)

| 源文件 | 职责 |
|--------|------|
| `common/src/app_manager.cpp` | 应用管理 |
| `common/src/component_manager.cpp` | 组件管理 |
| `common/src/count_down_latch.cpp` | 倒计时锁 |
| `common/src/multimode_manager.cpp` | 多模管理 |
| `common/src/wukong_logger.cpp` | 日志管理 |
| `common/src/wukong_util.cpp` | 工具函数 |

### component_event 模块 (8 个文件)

| 源文件 | 职责 |
|--------|------|
| `component_event/src/ability_tree.cpp` | Ability 树 |
| `component_event/src/component_tree.cpp` | 组件树 |
| `component_event/src/focus_scene_delegate.cpp` | 焦点场景 |
| `component_event/src/normal_scene.cpp` | 普通场景 |
| `component_event/src/page_tree.cpp` | Page 树 |
| `component_event/src/scene_delegate.cpp` | 场景委托 |
| `component_event/src/tree_manager.cpp` | 树管理 |
| `component_event/src/wukong_tree.cpp` | WuKong 树 |

### input_factory 模块 (19 个文件)

| 源文件 | 职责 |
|--------|------|
| `input_factory/src/appswitch_input.cpp` | 应用切换 |
| `input_factory/src/component_input.cpp` | 组件输入 |
| `input_factory/src/hardkey_input.cpp` | 硬键输入 |
| `input_factory/src/input_action.cpp` | 输入动作 |
| `input_factory/src/input_factory.cpp` | 输入工厂 |
| `input_factory/src/keyboard_input.cpp` | 键盘输入 |
| `input_factory/src/mouse_input.cpp` | 鼠标输入 |
| `input_factory/src/record_input.cpp` | 录制回放 |
| `input_factory/src/rotate_input.cpp` | 旋转输入 |
| `input_factory/src/swap_input.cpp` | 滑动输入 |
| `input_factory/src/touch_input.cpp` | 触摸输入 |
| `input_factory/src/knuckle_input.cpp` | 指关节输入 |
| `input_factory/src/pinch_input.cpp` | 捏合输入 |
| `input_factory/src/watch_crown_input.cpp` | 手表表冠 |
| `input_factory/src/watch_gestures_input.cpp` | 手表手势 |
| `input_factory/src/watch_idle_input.cpp` | 手表空闲 |
| `input_factory/src/watch_keypress_input.cpp` | 手表按键 |
| `input_factory/src/float_split_input.cpp` | 浮窗分裂 |
| `input_factory/src/collapse_input.cpp` | 折叠展开 |
| `input_factory/src/browser_input.cpp` | 浏览器操作 |

### report 模块 (14 个文件)

| 源文件 | 职责 |
|--------|------|
| `report/src/data_set.cpp` | 数据集 |
| `report/src/exception_manager.cpp` | 异常管理 |
| `report/src/filter.cpp` | 过滤器 |
| `report/src/filter_category.cpp` | 过滤器分类 |
| `report/src/format.cpp` | 格式基类 |
| `report/src/format_csv.cpp` | CSV 格式 |
| `report/src/format_json.cpp` | JSON 格式 |
| `report/src/report.cpp` | 报告管理 |
| `report/src/statistics.cpp` | 统计基类 |
| `report/src/statistics_ability.cpp` | Ability 统计 |
| `report/src/statistics_componment.cpp` | 组件统计 |
| `report/src/statistics_event.cpp` | 事件统计 |
| `report/src/statistics_exception.cpp` | 异常统计 |
| `report/src/sysevent_listener.cpp` | 系统事件监听 |
| `report/src/table.cpp` | 表格 |

### shell_command 模块 (2 个文件)

| 源文件 | 职责 |
|--------|------|
| `shell_command/src/wukong_main.cpp` | 主入口 |
| `shell_command/src/wukong_shell_command.cpp` | 命令行解析 |

### test_flow 模块 (5 个文件)

| 源文件 | 职责 |
|--------|------|
| `test_flow/src/focus_test_flow.cpp` | 专注测试 |
| `test_flow/src/random_test_flow.cpp` | 随机测试 |
| `test_flow/src/special_test_flow.cpp` | 专项测试 |
| `test_flow/src/test_flow.cpp` | 测试流程基类 |
| `test_flow/src/test_flow_factory.cpp` | 测试流程工厂 |

## 依赖组件详解

### external_deps 列表

| 组件 | GN Target | 用途 |
|------|-----------|------|
| ability_base | `:want` | Want 机制 |
| ability_runtime | `:ability_context_native` | 能力上下文 |
| ability_runtime | `:ability_manager` | 能力管理 |
| ability_runtime | `:abilitykit_native` | AbilityKit 原生 |
| ability_runtime | `:app_manager` | 应用管理 |
| ability_runtime | `:runtime` | 运行时 |
| accessibility | `:accessibility_common` | 无障碍公共 |
| accessibility | `:accessibilityclient` | 无障碍客户端 |
| accessibility | `:accessibleability` | 无障碍能力 |
| bundle_framework | `:appexecfwk_base` | 包框架基础 |
| bundle_framework | `:appexecfwk_core` | 包框架核心 |
| bundle_framework | `:libappexecfwk_common` | 包框架公共库 |
| c_utils | `:utils` | C 工具库 |
| graphic_2d | `:librender_service_base` | 渲染服务基础 |
| hidumper | `:lib_dump_usage` | 转储使用统计 |
| hilog | `:libhilog` | 日志库 |
| hisysevent | `:libhisysevent` | 系统事件库 |
| hisysevent | `:libhisyseventmanager` | 系统事件管理 |
| image_framework | `:image_native` | 图像原生 |
| init | `:libbegetutil` | 初始化工具 |
| input | `:libmmi-client` | MMI 客户端 |
| ipc | `:ipc_core` | IPC 核心 |
| samgr | `:samgr_proxy` | SAMgr 代理 |
| window_manager | `:libdm` | 显示管理 |
| window_manager | `:libwm` | 窗口管理 |
| libpng | `:libpng` | PNG 图像库 |

## 编译产物

### 输出文件

| 产物 | 类型 | 说明 |
|------|------|------|
| `wukong` | 可执行文件 | WuKong 主程序 |

### 安装路径

| 路径 | 说明 |
|------|------|
| `/bin/wukong` | 设备上的可执行文件路径 |

> 证据: `README.md:50` - `hdc_std shell mv /wukong /bin/`

### 构建命令

```bash
# 完整构建
./build.sh --product-name rk3568 --build-target wukong

# 仅构建 wukong
./build.sh --product-name <product> --build-target wukong
```

## 编译配置宏

| 宏定义 | 值 | 说明 |
|--------|-----|------|
| `LOG_TAG` | `"WuKong"` | 日志标签 |
| `LOG_DOMAIN` | `0xD003200` | 日志域 |

> 证据: `BUILD.gn:135-138`

## 头文件目录

| 目录 | 模块 |
|------|------|
| `./common/include` | common |
| `./component_event/include` | component_event |
| `./input_factory/include` | input_factory |
| `./report/include` | report |
| `./shell_command/include` | shell_command |
| `./test_flow/include` | test_flow |

## 产物与源文件映射

```
wukong (可执行文件)
├── main() → shell_command/src/wukong_main.cpp
├── Command Parse → shell_command/src/wukong_shell_command.cpp
├── Test Flow → test_flow/src/*.cpp
├── Input Events → input_factory/src/*.cpp
├── Component Tree → component_event/src/*.cpp
├── Report → report/src/*.cpp
└── Common → common/src/*.cpp
```

## 相关文档

- [模块详情](03_Module_Details.md)
- [故障排查](06_Troubleshooting.md)

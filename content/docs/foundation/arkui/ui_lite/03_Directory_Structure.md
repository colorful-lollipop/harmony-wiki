# ui_lite 目录结构

## 文档信息

- **文档用途**: 描述代码目录组织结构和各模块职责
- **适用范围**: 新加入开发者、代码维护人员
- **相关文档**: [项目概览](01_Overview.md), [架构说明](02_Architecture.md)

## 顶层目录

```
/foundation/arkui/ui_lite
├── frameworks/          # 框架实现代码
├── interfaces/          # API 接口定义
├── ext/                 # 扩展模块
├── tools/               # 开发和调试工具
├── test/                # 测试代码（本文档不详细描述）
├── figures/             # 文档图片
├── BUILD.gn             # 主构建文件
├── ui.gni               # 构建变量定义
└── bundle.json          # 组件配置
```

## frameworks/ 目录

框架核心实现，按功能模块组织。

### 组件系统 (components/)

```
frameworks/components/
├── ui_view.cpp              # 视图基类实现
├── ui_view.h
├── ui_view_group.cpp        # 容器基类实现
├── ui_view_group.h
├── root_view.cpp            # 根视图实现
├── root_view.h
├── ui_tree_manager.cpp      # 视图树管理
├── ui_tree_manager.h
├── ui_label.cpp             # 文本标签
├── ui_label.h
├── ui_button.cpp            # 按钮
├── ui_button.h
├── ui_image_view.cpp        # 图像视图
├── ui_image_view.h
├── ui_scroll_view.cpp       # 滚动视图
├── ui_scroll_view.h
├── ui_list.cpp              # 列表
├── ui_list.h
├── ui_canvas.cpp            # 画布
├── ui_canvas.h
├── ui_dialog.cpp            # 对话框
├── ui_dialog.h
├── ui_chart.cpp             # 图表
├── ui_chart.h
├── ui_qrcode.cpp            # 二维码
├── ui_qrcode.h
├── ui_video.cpp             # 视频（可选）
├── ui_video.h
└── ...                      # 其他 30+ 组件
```

**职责**: 实现所有 UI 组件的渲染、事件处理和状态管理。

**关键文件**:
- `ui_view.h:157` - UIView 基类定义 ([frameworks/components/ui_view.h](frameworks/components/ui_view.h))
- `root_view.h:77` - RootView 单例 ([frameworks/components/root_view.h](frameworks/components/root_view.h))

### 核心管理器 (core/)

```
frameworks/core/
├── task_manager.cpp         # 任务管理器
├── task_manager.h
├── render_manager.cpp       # 渲染管理器
├── render_manager.h
├── input_method_manager.cpp # 输入法管理器
└── input_method_manager.h
```

**职责**: 任务调度、渲染调度和输入法管理。

**关键文件**:
- `task_manager.h:31` - TaskManager 单例 ([frameworks/core/task_manager.h](frameworks/core/task_manager.h))

### 绘制系统 (draw/)

```
frameworks/draw/
├── draw_canvas.cpp          # 画布绘制
├── draw_canvas.h
├── draw_rect.cpp            # 矩形绘制
├── draw_rect.h
├── draw_image.cpp           # 图像绘制
├── draw_image.h
├── draw_label.cpp           # 文字绘制
├── draw_label.h
├── draw_line.cpp            # 线条绘制
├── draw_line.h
├── draw_arc.cpp             # 圆弧绘制
├── draw_arc.h
├── draw_curve.cpp           # 曲线绘制
├── draw_curve.h
├── draw_triangle.cpp        # 三角形绘制
├── draw_triangle.h
├── draw_utils.cpp           # 绘制工具
├── draw_utils.h
└── clip_utils.cpp           # 裁剪工具
```

**职责**: 实现基础图元绘制、图像渲染和裁剪。

### 渲染系统 (render/)

```
frameworks/render/
├── render_base.cpp          # 渲染基类
├── render_base.h
├── render_scanline.cpp      # 扫描线渲染
├── render_scanline.h
└── render_pixfmt_rgba_blend.cpp  # 像素格式混合
```

**职责**: 低层渲染实现，像素级操作。

### 布局系统 (layout/)

```
frameworks/layout/
├── flex_layout.cpp          # Flex 布局
├── flex_layout.h
├── grid_layout.cpp          # Grid 布局
├── grid_layout.h
└── list_layout.cpp          # List 布局
```

**职责**: 实现布局算法，计算子视图位置和大小。

### 动画系统 (animator/)

```
frameworks/animator/
├── animator.cpp             # 动画器
├── animator.h
├── animator_manager.cpp     # 动画管理器
├── animator_manager.h
├── easing_equation.cpp      # 缓动方程
├── easing_equation.h
└── interpolation.cpp        # 插值器
```

**职责**: 动画计算、时间管理和缓动效果。

### 事件系统 (events/)

```
frameworks/events/
└── event.cpp                # 事件基类实现
```

**职责**: 事件对象定义。

### 字体系统 (font/)

```
frameworks/font/
├── base_font.cpp            # 字体基类
├── base_font.h
├── ui_font.cpp              # 字体管理
├── ui_font.h
├── ui_font_vector.cpp       # 矢量字体
├── ui_font_vector.h
├── ui_font_bitmap.cpp       # 位图字体
├── ui_font_bitmap.h
├── ui_font_cache.cpp        # 字体缓存
├── ui_font_cache.h
├── ui_font_cache_manager.cpp # 缓存管理器
├── ui_font_cache_manager.h
├── ui_line_break.cpp        # 断行处理
├── ui_line_break.h
├── ui_text_shaping.cpp      # 文字整形
├── ui_text_shaping.h
├── glyphs_manager.cpp       # 字形管理
├── glyphs_manager.h
└── ...
```

**职责**: 字体加载、字形渲染、缓存管理和文字排版。

### 图像解码 (imgdecode/)

```
frameworks/imgdecode/
├── file_img_decoder.cpp     # 文件图像解码器
├── file_img_decoder.h
├── image_load.cpp           # 图像加载
├── image_load.h
└── cache_manager.cpp        # 缓存管理
```

**职责**: JPEG/PNG 图像文件解码和缓存。

**关键文件**:
- `file_img_decoder.h:34` - FileImgDecoder 单例 ([frameworks/imgdecode/file_img_decoder.h](frameworks/imgdecode/file_img_decoder.h))

### 主题系统 (themes/)

```
frameworks/themes/
├── theme.cpp                # 主题
├── theme.h
├── theme_manager.cpp        # 主题管理器
└── theme_manager.h
```

**职责**: 样式定义和主题切换。

### 适配层 (dock/)

```
frameworks/dock/
├── input_device.cpp         # 输入设备基类
├── input_device.h
├── key_input_device.cpp     # 按键输入
├── key_input_device.h
├── pointer_input_device.cpp # 指针输入（触摸/鼠标）
├── pointer_input_device.h
├── rotate_input_device.cpp  # 旋转输入
├── rotate_input_device.h
├── virtual_input_device.cpp # 虚拟输入
├── virtual_input_device.h
├── focus_manager.cpp        # 焦点管理
├── focus_manager.h
├── rotate_manager.cpp       # 旋转管理
├── rotate_manager.h
├── vibrator_manager.cpp     # 震动管理
├── vibrator_manager.h
├── screen_device_proxy.cpp  # 显示设备代理
├── screen_device_proxy.h
└── ohos/
    ├── ohos_input_device.cpp    # OHOS 平台输入适配
    └── ohos_input_device.h
```

**职责**: 平台抽象，输入设备适配和系统服务交互。

### 绘制引擎 (engines/)

```
frameworks/engines/
└── gfx/
    ├── gfx_engine_manager.cpp   # 引擎管理器
    ├── gfx_engine_manager.h
    ├── soft_engine.cpp          # 软件渲染引擎
    ├── soft_engine.h
    └── hi3516/
        ├── hi3516_engine.cpp    # hi3516 硬件引擎
        └── hi3516_engine.h
```

**职责**: 图形引擎抽象和具体实现。

**关键文件**:
- `gfx_engine_manager.h:103` - BaseGfxEngine 接口 ([frameworks/engines/gfx/gfx_engine_manager.h](frameworks/engines/gfx/gfx_engine_manager.h))

### 窗口管理 (window/)

```
frameworks/window/
├── window.cpp               # 窗口实现
├── window.h
└── window_impl.cpp          # 窗口实现细节
```

**职责**: 窗口生命周期管理和 WMS 交互。

### DFX 调试 (dfx/)

```
frameworks/dfx/
├── event_injector.cpp       # 事件注入（测试）
├── event_injector.h
├── key_event_injector.cpp   # 按键事件注入
├── key_event_injector.h
├── point_event_injector.cpp # 触摸事件注入
├── point_event_injector.h
├── ui_dump_dom_tree.cpp     # DOM 树导出
├── ui_dump_dom_tree.h
├── ui_screenshot.cpp        # 截图
├── ui_screenshot.h
├── ui_view_bounds.cpp       # 视图边界显示
├── ui_view_bounds.h
└── performance_task.cpp     # 性能监控
```

**职责**: 调试工具和测试辅助功能。

**安全注意**: `event_injector.h` 提供模拟输入能力，仅在 `ENABLE_DEBUG` 时可用 ([frameworks/dfx/event_injector.h](frameworks/dfx/event_injector.h))。

### 通用模块 (common/)

```
frameworks/common/
├── graphic_startup.cpp      # 图形启动
├── graphic_startup.h
├── task.cpp                 # 任务基类
├── task.h
├── image.cpp                # 图像处理
├── image.h
├── text.cpp                 # 文字处理
├── text.h
├── screen.cpp               # 屏幕信息
├── screen.h
└── input_device_manager.cpp # 输入设备管理
```

**职责**: 通用工具和系统初始化。

### 默认资源 (default_resource/)

```
frameworks/default_resource/
└── check_box_res.cpp        # 复选框默认资源
```

**职责**: 内置默认图片资源。

### 路径处理 (path/)

```
frameworks/path/
├── path_base.cpp            # 路径基类
└── path_base.h
```

**职责**: 矢量路径操作。

## interfaces/ 目录

API 接口定义，分为对外 API (kits) 和内部 API (innerkits)。

### 对外 API (kits/)

```
interfaces/kits/
├── components/              # 组件 API（40+ 头文件）
│   ├── ui_view.h
│   ├── ui_view_group.h
│   ├── root_view.h
│   ├── ui_label.h
│   ├── ui_button.h
│   └── ...
├── animator/                # 动画 API
│   ├── animator.h
│   ├── easing_equation.h
│   └── interpolation.h
├── events/                  # 事件 API
│   ├── event.h
│   ├── click_event.h
│   ├── drag_event.h
│   └── ...
├── layout/                  # 布局 API
│   ├── layout.h
│   ├── flex_layout.h
│   ├── grid_layout.h
│   └── list_layout.h
├── font/                    # 字体 API
│   ├── ui_font.h
│   └── base_font.h
├── themes/                  # 主题 API
│   ├── theme.h
│   └── theme_manager.h
├── window/                  # 窗口 API
│   └── window.h
├── common/                  # 通用 API
│   ├── task.h
│   ├── image.h
│   └── screen.h
└── dfx/                     # 调试 API
    ├── event_injector.h
    └── ui_screenshot.h
```

**使用方式**: 应用开发者包含这些头文件使用 UI 框架。

### 内部 API (innerkits/)

```
interfaces/innerkits/
├── common/                  # 核心内部 API
│   ├── graphic_startup.h
│   ├── task_manager.h
│   ├── input_device_manager.h
│   └── image_decode_ability.h
├── engines/gfx/             # 引擎内部 API
│   ├── gfx_engine_manager.h
│   └── soft_engine.h
├── dock/                    # 适配层内部 API
│   ├── focus_manager.h
│   ├── rotate_manager.h
│   └── vibrator_manager.h
├── font/                    # 字体内部 API
│   ├── ui_font_vector.h
│   ├── ui_font_bitmap.h
│   └── ui_font_builder.h
└── path/                    # 路径内部 API
    └── path_base.h
```

**使用方式**: 框架内部模块间调用，不建议外部直接使用。

## ext/ 目录

扩展模块，特定场景的布局实现。

```
ext/
├── updater/                 # 升级器布局
│   ├── BUILD.gn
│   └── ...
├── home_host/               # 智能家居主机布局
│   ├── BUILD.gn
│   └── ...
└── ide/                     # IDE 支持
    └── BUILD.gn
```

**职责**: 特定产品的定制化布局，独立编译为库。

## tools/ 目录

开发和调试工具。

```
tools/
├── qt/                      # Qt 模拟器
│   └── simulator/           # 完整的 Qt 项目
│       ├── simulator.pro    # Qt 工程文件
│       ├── drivers/         # 模拟驱动
│       ├── uitest/          # UI 测试
│       └── third_party/     # 依赖库
└── server/                  # 测试服务器
    └── test_case/
```

**职责**: PC 端模拟调试环境。

## 关键文件速查

| 功能 | 文件路径 | 行号 |
|------|----------|------|
| UIView 基类 | `interfaces/kits/components/ui_view.h` | 157 |
| RootView 单例 | `interfaces/kits/components/root_view.h` | 77 |
| Window 创建 | `interfaces/kits/window/window.h` | 132 |
| TaskManager | `interfaces/innerkits/common/task_manager.h` | 31 |
| GraphicStartUp | `interfaces/innerkits/common/graphic_startup.h` | 24 |
| 图形引擎 | `interfaces/innerkits/engines/gfx/gfx_engine_manager.h` | 103 |
| 图像解码 | `frameworks/imgdecode/file_img_decoder.h` | 34 |
| 事件注入 | `interfaces/kits/dfx/event_injector.h` | 64 |
| 构建配置 | `BUILD.gn` | 18 |
| 组件配置 | `bundle.json` | 1 |

## 相关文档

- [项目概览](01_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构
- [对外 API](04_Public_API.md) - API 详细说明
- [GN 构建](06_GN_Targets.md) - 构建配置

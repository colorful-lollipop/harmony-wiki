# 配置宏清单

## 文档信息

- **文档用途**: 汇总 ui_lite 所有编译时配置宏（Feature Flags）
- **适用范围**: 开发者、配置工程师
- **相关文档**: [GN 构建目标](../06_GN_Targets.md)

## 图形功能宏

### 基础图形

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `GRAPHIC_ENABLE_ELLIPSE_FLAG` | 1 | 启用椭圆绘制 | BUILD.gn:29 |
| `GRAPHIC_ENABLE_BEZIER_ARC_FLAG` | 1 | 启用贝塞尔弧线 | BUILD.gn:30 |
| `GRAPHIC_ENABLE_LINECAP_FLAG` | 1 | 启用线帽样式 | BUILD.gn:31 |
| `GRAPHIC_ENABLE_LINEJOIN_FLAG` | 1 | 启用线连接样式 | BUILD.gn:32 |
| `GRAPHIC_ENABLE_ARC_FLAG` | 1 | 启用圆弧绘制 | BUILD.gn:33 |
| `GRAPHIC_ENABLE_ROUNDEDRECT_FLAG` | 1 | 启用圆角矩形 | BUILD.gn:34 |
| `GRAPHIC_ENABLE_GRADIENT_FILL_FLAG` | 1 | 启用渐变填充 | BUILD.gn:35 |
| `GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG` | 1 | 启用阴影效果 | BUILD.gn:36 |
| `GRAPHIC_ENABLE_DRAW_TEXT_FLAG` | 1 | 启用文字绘制 | BUILD.gn:37 |

### 高级图形

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `GRAPHIC_ENABLE_DASH_GENERATE_FLAG` | 1 (非 LiteOS-M) | 启用虚线生成 | BUILD.gn:64 |
| `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 1 (非 LiteOS-M) | 启用模糊效果 | BUILD.gn:65 |
| `GRAPHIC_ENABLE_PATTERN_FILL_FLAG` | 1 (非 LiteOS-M) | 启用图案填充 | BUILD.gn:66 |
| `GRAPHIC_ENABLE_DRAW_IMAGE_FLAG` | 1 (非 LiteOS-M) | 启用图像绘制 | BUILD.gn:67 |

## 字体功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `ENABLE_VECTOR_FONT` | 1 (非 LiteOS-M) | 启用矢量字体 | BUILD.gn:48 |
| `ENABLE_BITMAP_FONT` | 0 | 启用位图字体 | BUILD.gn:49 |
| `ENABLE_SHAPING` | 0 | 启用文字整形 | BUILD.gn:50 |
| `ENABLE_ICU` | 1 (非 LiteOS-M) | 启用 ICU 库 | BUILD.gn:51 |
| `ENABLE_MULTI_FONT` | 1 (非 LiteOS-M) | 启用多字体 | BUILD.gn:52 |
| `ENABLE_CANVAS_EXTEND` | 0/1 (LiteOS-M 可配置) | 扩展画布功能 | BUILD.gn:72 |

## 动画功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `DEFAULT_ANIMATION` | 1 (非 LiteOS-M) | 默认启用动画 | BUILD.gn:53 |

## 窗口功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `ENABLE_WINDOW` | 平台相关 | 启用窗口支持 | window.h:41 |
| `ENABLE_ROTATE_INPUT` | 平台相关 | 启用旋转输入 | ui_view.h:46 |
| `ENABLE_FOCUS_MANAGER` | 平台相关 | 启用焦点管理 | ui_view.h:454 |

## 调试功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `ENABLE_DEBUG` | 0 | 启用调试功能 | graphic_config.h |
| `ENABLE_DEBUG` | 影响 EventInjector | event_injector.h:39 |

## 资源路径宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `RESOURCE_DIR` | "/storage/data/" (Linux) | 资源目录 | BUILD.gn:56 |
| `RESOURCE_DIR` | "/user/data/" (其他) | 资源目录 | BUILD.gn:58 |

## 视频功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `ENABLE_VIDEO_COMPONENT` | 0 (可配置) | 启用视频组件 | BUILD.gn:264 |

## 图像格式宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `ENABLE_GIF` | 0 | 启用 GIF 支持 | BUILD.gn:276 |

## 布局功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `CONFIG_DYNAMIC_LAYOUT` | 0/1 | 动态布局支持 | ui_view.h:57 |

## 渲染功能宏

| 宏 | 默认值 | 说明 | 定义位置 |
|----|--------|------|----------|
| `LOCAL_RENDER` | 平台相关 | 本地渲染模式 | root_view.h:50 |

## 使用示例

### 在 BUILD.gn 中配置

```gn
# 启用视频组件
declare_args() {
  ui_lite_enable_video_component_config = true
}

# 自定义字体配置
declare_args() {
  ui_lite_enable_graphic_font_config = true
}
```

### 在代码中使用

```cpp
#if GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG
    // 使用阴影效果
    style.shadowOpacity = 128;
#endif

#if ENABLE_VECTOR_FONT
    // 使用矢量字体
    UIFontVector::GetInstance()->Open(fontPath);
#endif

#ifdef ENABLE_DEBUG
    // 调试代码
    EventInjector::GetInstance()->SetClickEvent(point);
#endif
```

## 宏依赖关系

```
ENABLE_WINDOW
├── 启用窗口功能
└── 依赖: window_manager_lite

ENABLE_VECTOR_FONT
├── 启用 TTF 字体
└── 依赖: freetype, ENABLE_ICU

ENABLE_VIDEO_COMPONENT (ui_lite_enable_video_component_config)
├── 启用 UISurfaceView
└── 依赖: media_lite

GRAPHIC_ENABLE_*_FLAG
├── 启用对应图形功能
└── 影响: 库大小和性能
```

## 性能影响

| 宏 | 启用影响 | 建议 |
|----|----------|------|
| `GRAPHIC_ENABLE_BLUR_EFFECT_FLAG` | 高（GPU/CPU 消耗） | 低端设备禁用 |
| `GRAPHIC_ENABLE_SHADOW_EFFECT_FLAG` | 中 | 按需启用 |
| `ENABLE_VECTOR_FONT` | 中（内存占用） | 需要中文时启用 |
| `ENABLE_ICU` | 高（库大小 +500KB） | 多语言应用启用 |
| `DEFAULT_ANIMATION` | 低 | 建议启用 |

## 平台默认配置

### LiteOS-M (轻量设备)

```
ENABLE_VECTOR_FONT = 0
ENABLE_WINDOW = 0
ENABLE_VIDEO_COMPONENT = 0
GRAPHIC_ENABLE_BLUR_EFFECT_FLAG = 0
```

### LiteOS-A (小型设备)

```
ENABLE_VECTOR_FONT = 1
ENABLE_WINDOW = 1
ENABLE_VIDEO_COMPONENT = 可选
GRAPHIC_ENABLE_BLUR_EFFECT_FLAG = 1
```

### Linux (标准设备)

```
所有功能启用
```

## 相关文档

- [GN 构建目标](../06_GN_Targets.md) - 构建配置
- [项目概览](../01_Overview.md) - 功能说明

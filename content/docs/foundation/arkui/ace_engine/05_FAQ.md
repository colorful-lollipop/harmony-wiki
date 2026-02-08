# Ace Engine 常见问题

> **文档版本**: v1.0  
> **更新时间**: 2026-02-06  
> **源码版本**: OpenHarmony ace_engine

---

## 📋 目录

1. [构建问题](#构建问题)
2. [运行问题](#运行问题)
3. [调试指南](#调试指南)
4. [代码定位](#代码定位)

---

## 构建问题

### Q1: 构建失败，提示找不到头文件

**问题描述**:
```
fatal error: 'xxx.h' file not found
```

**解决方案**:

1. **检查 GN 配置文件**: 确保 `include_dirs` 包含头文件路径
2. **检查模块依赖**: 确认 `deps` 中包含相关模块
3. **清理构建**: 执行 `gn clean` 后重新构建

**常用路径**:
```bash
# 清理构建产物
rm -rf out/{product}/arkui/ace_engine/*

# 重新生成构建
gn gen out/{product}/arkui/ace_engine
ninja -C out/{product}/arkui/ace_engine
```

---

### Q2: 编译链接失败，符号未定义

**问题描述**:
```
undefined reference to `XXX'
```

**解决方案**:

1. **检查依赖**: 确认 `deps` 包含定义符号的模块
2. **检查可见性**: 确认符号已导出（非 `visibility=hidden`）
3. **检查链接顺序**: 确保静态库在正确的链接位置

**常见符号位置**:

| 符号 | 所在模块 |
|------|----------|
| `FrameNode` | `frameworks/core/components_ng/base/` |
| `Pattern` | `frameworks/core/components_ng/pattern/` |
| `AceType` | `frameworks/core/base/` |

---

### Q3: 组件库编译失败

**问题描述**:
```
error: no member named 'xxx' in 'xxxPattern'
```

**解决方案**:

1. **检查组件模式**: 确认组件已实现必要方法
2. **检查头文件**: 确认已包含正确的头文件
3. **检查 GN 依赖**: 确认组件库依赖正确

**组件必需方法**:
```cpp
// Pattern 基类必需方法
void OnModifyDone() override;
RefPtr<LayoutProperty> CreateLayoutProperty() override;
RefPtr<PaintProperty> CreatePaintProperty() override;
RefPtr<EventHub> CreateEventHub() override;
```

---

## 运行问题

### Q4: 应用启动崩溃

**问题描述**:
```
Fatal signal 6 (SIGABRT) ...
```

**解决方案**:

1. **检查帧节点初始化**: 确认 `FrameNode` 正确创建
2. **检查属性设置**: 确认属性值有效
3. **检查资源加载**: 确认资源路径正确

**常见崩溃原因**:
- 空指针访问 (`nullptr`)
- 非法属性值
- 资源加载失败

---

### Q5: 组件不显示

**问题描述**: 组件创建后未显示

**解决方案**:

1. **检查父节点**: 确认组件已添加到父节点
2. **检查尺寸**: 确认宽高大于 0
3. **检查可见性**: 确认 `visibility` 属性正确
4. **检查布局**: 确认布局约束有效

**调试步骤**:
```cpp
// 检查 FrameNode 是否创建
auto frameNode = ViewStackProcessor::GetInstance()->GetCurrentFrameNode();
if (!frameNode) {
    // FrameNode 未创建
}
```

---

### Q6: 手势不响应

**问题描述**: 手势事件未触发

**解决方案**:

1. **检查手势绑定**: 确认手势已绑定到组件
2. **检查事件穿透**: 确认无上层组件遮挡
3. **检查手势方向**: 确认手势方向与组件匹配

**手势绑定示例**:
```cpp
// 在 Pattern 中绑定手势
auto gestureHub = GetEventHub<EventHub>()->GetOrCreateGestureEventHub();
auto tapGesture = AceType::MakeRefPtr<TapGesture>();
gestureHub->AddGesture(tapGesture);
```

---

## 调试指南

### 7.1 日志输出

**HiLog 日志**:

```cpp
#include "hilog/log.h"

OH_LOG_INFO(LogLabel) << "Custom log message";
```

**日志级别**:

| 级别 | 宏 | 用途 |
|------|-----|------|
| DEBUG | `OH_LOG_DEBUG` | 调试信息 |
| INFO | `OH_LOG_INFO` | 普通信息 |
| WARN | `OH_LOG_WARN` | 警告信息 |
| ERROR | `OH_LOG_ERROR` | 错误信息 |
| FATAL | `OH_LOG_FATAL` | 致命错误 |

---

### 7.2 调试编译

**启用调试符号**:
```bash
# 在 ohos.build 中设置
enable_debug_symbols = true
```

**优化级别调整**:
```gn
# 降低优化级别便于调试
cflags_cc = [
  "-O0",  # 禁用优化
  "-g",   # 调试符号
]
```

---

### 7.3 常用调试工具

| 工具 | 用途 |
|------|------|
| **hdc** | 设备连接和文件传输 |
| **hilog** | 日志查看 |
| **hidumper** | 进程转储 |
| **perf** | 性能分析 |

**日志查看**:
```bash
# 查看 Ace Engine 日志
hilog | grep -i "ace"
```

---

## 代码定位

### 8.1 核心文件定位

| 功能 | 文件路径 | 行号 |
|------|----------|------|
| FrameNode | `frameworks/core/components_ng/base/frame_node.h` | 65 |
| Pattern | `frameworks/core/components_ng/pattern/pattern.h` | - |
| UINode | `frameworks/core/components_ng/base/ui_node.h` | - |
| EventHub | `frameworks/core/components_ng/event/event_hub.h` | - |

### 8.2 组件文件定位

| 组件 | Pattern 路径 | Model 路径 |
|------|--------------|------------|
| Text | `pattern/text/text_pattern.h` | `pattern/text/text_model_ng.h` |
| Button | `pattern/button/button_pattern.h` | `pattern/button/button_model_ng.h` |
| Image | `pattern/image/image_pattern.h` | `pattern/image/image_model_ng.h` |
| List | `pattern/list/list_pattern.h` | `pattern/list/list_model_ng.h` |

### 8.3 N-API 文件定位

| 模块 | 文件路径 |
|------|----------|
| Router | `interfaces/napi/kits/router/js_router.cpp` |
| Animator | `interfaces/napi/kits/animator/js_animator.cpp` |
| Prompt | `interfaces/napi/kits/prompt/js_prompt.cpp` |
| DragController | `interfaces/napi/kits/drag_controller/js_drag_controller.cpp` |

---

## 常用命令速查

| 操作 | 命令 |
|------|------|
| **完整构建** | `./build.sh --product-name rk3568 --build-target ace_engine` |
| **清理构建** | `./build.sh --product-name rk3568 --build-target ace_engine -b` |
| **构建测试** | `./build.sh --product-name rk3568 --build-target unittest` |
| **构建 SDK** | `./build.sh --product-name ohos-sdk --build-target ace_engine` |
| **查看日志** | `hilog | grep -i "ace"` |
| **设备连接** | `hdc list targets` |
| **推送到设备** | `hdc shell mount -o rw,remount /` |
| **安装应用** | `hdc install app.hap` |

---

## 🔗 相关文档

- 项目概览: [00_Overview](00_Overview.md)
- 架构说明: [01_Architecture](01_Architecture.md)
- N-API 接口: [02_NAPI](02_NAPI.md)
- 构建系统: [03_Build](03_Build.md)
- 安全评审: [04_Security](04_Security.md)

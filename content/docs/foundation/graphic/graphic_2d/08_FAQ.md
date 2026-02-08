# 常见问题 (FAQ)

## 构建问题

### Q1: 编译graphic_2d时提示找不到skia依赖

**问题**:
```
error: cannot find 'skia' dependency for target '2d_graphics'
```

**原因**: Skia 是 graphic_2d 的核心依赖，需要在构建环境中正确配置。

**解决方案**:
```bash
# 确保在 OpenHarmony 根目录执行
cd /path/to/openharmony

# 检查 skia 子模块
git submodule update --init --recursive

# 清理后重新构建
rm -rf out/* && ./build.sh --product-name <product>
```

**证据来源**: `bundle.json:132` (third_party/skia)

---

### Q2: 如何只构建特定模块？

**解决方案**:

```bash
# 构建渲染服务
./build.sh --product-name <product> --build-target librender_service

# 构建客户端库
./build.sh --product-name <product> --build-target librender_service_client

# 构建 2D 图形
./build.sh --product-name <product> --build-target 2d_graphics

# 构建 NDK 绘图
./build.sh --product-name <product> --build-target native_drawing_ndk

# 构建特定 N-API
./build.sh --product-name <product> --build-target drawingnapi
```

**证据来源**: `06_Build.md` - 构建命令

---

### Q3: 如何启用/禁用 Vulkan 或 OpenGL 后端？

**解决方案**:

编辑 `graphic_config.gni`:

```gn
# 启用 Vulkan
graphic_2d_feature_enable_vulkan = true

# 启用 OpenGL (默认启用)
graphic_2d_feature_enable_opengl = true

# phone/pc/tablet/wearable 产品自动启用 Vulkan
if (product == "phone" || product == "pc" || ...) {
  graphic_2d_feature_enable_vulkan = true
}
```

**证据来源**: `graphic_config.gni:31,80-82`

---

### Q4: 编译时 sanitizer 相关警告过多

**问题**: CFI、UBSan 等 sanitizer 产生大量警告。

**解决方案**:

```gn
# 在产品配置中禁用 sanitizer
sanitize = {
  cfi = false
  cfi_cross_dso = false
  ubsan = false
}
```

注意：生产版本建议启用 sanitizer 以检测潜在安全问题。

**证据来源**: `06_Build.md` - Sanitizer 配置

---

## 运行时问题

### Q5: 渲染服务启动失败

**问题**: 设备启动时渲染服务崩溃或无法连接。

**日志定位**:
```bash
# 查看渲染服务日志
hilog | grep -E "RS|RenderService"

# 查看系统能力日志
hilog | grep -E "SA|SystemAbility"
```

**常见原因**:
1. SA 注册失败 - 检查 `ENABLE_IPC_SECURITY` 编译开关
2. 权限不足 - 确认应用声明了必要权限
3. 依赖服务未就绪 - 检查 HDI、Surface 服务

**证据来源**: `rs_render_service_connect_hub.cpp` - SA 获取

---

### Q6: 屏幕截图返回空结果

**问题**: 调用 `TakeSurfaceCapture` 返回 null 或空图像。

**原因分析**:
1. 权限不足 - 需要 `CAPTURE_SCREEN` 权限
2. 目标窗口不存在或已销毁
3. GPU 渲染模式下的限制

**解决方案**:
```cpp
// 检查权限
if (!CheckPermission(CAPTURE_SCREEN_PERMISSION)) {
    return nullptr;
}

// 检查窗口有效性
if (!surfaceNode->IsValid()) {
    return nullptr;
}
```

**证据来源**: `04_N-API.md` - TakeSurfaceCapture

---

### Q7: 动画不执行或卡顿

**问题**: RSAnimation 启动后没有效果或掉帧。

**排查步骤**:
1. 检查节点是否已添加到渲染树
2. 确认动画目标节点有效
3. 检查是否有 Modifier 覆盖了动画属性
4. 查看帧率是否正常

```bash
# 查看帧率信息
hilog | grep -E "FPS|HGM|RefreshRate"
```

**证据来源**: `03_Architecture.md` - 动画系统

---

## API 使用问题

### Q8: 如何创建自定义绘图节点？

**解决方案**:

```cpp
// 1. 创建 CanvasNode
auto canvasNode = RSCanvasNode::Create();

// 2. 获取 Canvas
auto canvas = canvasNode->BeginRecording(100, 100);

// 3. 绘制内容
canvas->DrawRect(rect, paint);

// 4. 结束录制
canvasNode->FinishRecording();

// 5. 添加到父节点
parentNode->AddChild(canvasNode);
```

**证据来源**: `05_Inner_API.md` - RSCanvasNode

---

### Q9: 如何实现磨砂玻璃效果？

**问题**: 使用 frostedGlass 效果时无效果或崩溃。

**解决方案**:

```javascript
// 必须是系统应用才能使用
if (!IsSystemApp()) {
    throw new Error("System permission required");
}

const filter = new uieffect.Filter();
filter.frostedGlass(10, 0.5);  // radius, saturation
node.setBackgroundFilter(filter);
```

注意：`frostedGlass` 系列 API 仅限系统应用使用。

**证据来源**: `04_N-API.md` - Filter 类

---

### Q10: 如何正确释放 Graphic 资源？

**问题**: 应用退出后内存未释放或资源泄漏。

**解决方案**:

```cpp
// 1. 移除所有子节点
node->ClearChildren();

// 2. 从父节点移除
node->RemoveFromTree();

// 3. 清理 Modifier
node->RemoveAllModifiers();

// 4. 释放共享指针引用
node.reset();
```

**证据来源**: `05_Inner_API.md` - RSNode API

---

## 调试技巧

### Q11: 如何启用渲染调试日志？

**解决方案**:

```cpp
// 启用 RS 日志
#include "hilog/log.h"
#define RS_LOG_DOMAIN 0xD002900  // Graphic 子系统域

// 日志级别
RS_LOGI("Info message");
RS_LOGW("Warning message");
RS_LOGE("Error message");
```

**证据来源**: `bundle.json:79` (hilog 依赖)

---

### Q12: 如何查看 Drawable 渲染顺序？

**问题**: 需要调试渲染层级问题。

**解决方案**:

```bash
# 启用 Drawable 调试
hilog | grep -E "RSDrawable|Slot"
```

代码中添加：
```cpp
// 继承 RSDrawable 并重写 OnDraw
class DebugDrawable : public RSDrawable {
    void OnDraw(Drawing::Canvas* canvas) override {
        RS_LOGI("Drawing slot: %d", GetSlot());
        RSDrawable::OnDraw(canvas);
    }
};
```

**证据来源**: `03_Architecture.md` - Drawable 系统

---

### Q13: 如何定位 IPC 调用失败？

**问题**: IPC 通信失败，需要定位失败点。

**排查步骤**:

```bash
# 1. 查看 IPC 错误码
hilog | grep -E "IPC|TRANSACTION"

# 2. 检查接口代码是否可访问
hilog | grep -E "IsInterfaceCodeAccessible|ACCESS_VERIFIER"

# 3. 验证权限
hilog | grep -E "CheckPermission|PERMISSION"
```

代码中添加：
```cpp
// 检查 IPC 接口访问
if (!securityManager.IsInterfaceCodeAccessible(code)) {
    RS_LOGE("IPC code %d access denied", code);
    return;
}
```

**证据来源**: `05_Inner_API.md` - IPC 接口

---

## 移植问题

### Q14: 如何移植到新平台？

**问题**: 将 graphic_2d 移植到新的硬件平台。

**步骤**:

1. **配置平台开关** (`graphic_config.gni`)
   ```gn
   is_cross_platform = true
   ```

2. **实现平台适配层** (`rosen/modules/platform/`)
   - PlatformAdapter 接口
   - Window 抽象
   - 输入事件处理

3. **配置 Graphics Backend**
   ```gn
   graphic_2d_feature_enable_opengl = true  // 或 Vulkan
   ```

4. **验证 HDI 集成**
   - Display HDI 1.0-1.4
   - GPU 驱动兼容性

**证据来源**:
- `03_Architecture.md` - 平台层
- `06_Build.md` - 跨平台配置

---

### Q15: 如何禁用测试代码的包含？

**问题**: 生产版本不需要测试代码。

**解决方案**:

测试代码默认不包含在构建产物中，通过 `testonly = true` 标记：

```gn
group("module_test") {
  testonly = true  # 不会包含在发布版本
  public_deps = [ ":test_target" ]
}
```

**证据来源**: `BUILD.gn:30-31` (graphic_common_test)

---

## 相关文档链接

| 问题类型 | 参考文档 |
|---------|----------|
| 架构问题 | [03_Architecture.md](03_Architecture.md) |
| API 使用 | [04_N-API.md](04_N-API.md), [05_Inner_API.md](05_Inner_API.md) |
| 构建配置 | [06_Build.md](06_Build.md) |
| 安全相关 | [07_Security.md](07_Security.md) |
| 源码结构 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 项目概览 | [01_Overview.md](01_Overview.md) |

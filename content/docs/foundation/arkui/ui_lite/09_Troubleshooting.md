# ui_lite 常见问题

## 文档信息

- **文档用途**: 汇总构建、运行、调试中的常见问题及解决方案
- **适用范围**: 开发者、测试人员
- **相关文档**: [GN 构建目标](06_GN_Targets.md), [编译产物](07_Build_Artifacts.md)

## 构建问题

### Q1: 编译报错 "undefined reference to 'OHOS::RootView::GetInstance()'"

**现象**:
```
undefined reference to `OHOS::RootView::GetInstance()'
undefined reference to `OHOS::GraphicStartUp::Init()'
```

**原因**: 未链接 libui.so 或链接顺序错误

**解决**:
```cmake
# BUILD.gn
deps = [
    "//foundation/arkui/ui_lite:ui",
]

# 或 CMake
target_link_libraries(my_app
    ui
    graphic_utils_lite
)
```

**证据**: `BUILD.gn:278`

---

### Q2: LiteOS-M 编译报错 "Window 类未定义"

**现象**:
```
error: 'Window' was not declared in this scope
```

**原因**: LiteOS-M 平台裁剪了 Window 功能

**解决**:
```cpp
// 使用条件编译
#ifdef ENABLE_WINDOW
    Window* window = Window::CreateWindow(config);
#else
    // LiteOS-M 使用 RootView 直接渲染
    RootView* rootView = RootView::GetInstance();
#endif
```

**证据**: `BUILD.gn:231-236`

---

### Q3: 编译报错 "freetype.h not found"

**现象**:
```
fatal error: 'freetype/freetype.h' file not found
```

**原因**: 未添加 freetype 包含路径

**解决**:
```gn
# BUILD.gn
include_dirs = [
    "//third_party/freetype/include",
]

deps = [
    "//third_party/freetype:freetype",
]
```

---

### Q4: 链接报错 "multiple definition of 'xxx'"

**现象**:
```
multiple definition of `OHOS::UIView::SetPosition(short, short)'
```

**原因**: 头文件内联函数重复定义

**解决**:
```cpp
// 头文件中
#ifndef UI_VIEW_H
#define UI_VIEW_H

// 内联函数定义
inline void SetPosition(int16_t x, int16_t y) { ... }

#endif
```

---

## 运行问题

### Q5: 应用启动崩溃，堆栈在 GraphicStartUp::Init()

**现象**: 启动时立即崩溃

**原因**: 字体路径未设置或字体文件不存在

**解决**:
```cpp
// 1. 检查字体路径存在
if (access("/system/data/SourceHanSansSC-Regular.otf", F_OK) != 0) {
    // 字体文件缺失，使用备用方案
}

// 2. 正确初始化
GraphicStartUp::Init();
GraphicStartUp::InitFontEngine(
    cacheMemAddr,      // 字体缓存内存地址
    cacheMemLen,       // 缓存大小
    "/system/data/",   // 字体目录
    "SourceHanSansSC-Regular.otf"  // 字体文件名
);
```

**证据**: `interfaces/innerkits/common/graphic_startup.h:27`

---

### Q6: 中文显示为方块或乱码

**现象**: 文本标签显示方块 □□□

**原因**: 字体未正确加载

**解决**:
```cpp
// 1. 检查字体注册
UIFont::GetInstance()->SetFontPath("/system/data/");
int8_t ret = UIFont::GetInstance()->RegisterFontInfo("SourceHanSansSC-Regular.otf");
if (ret != 0) {
    // 注册失败
}

// 2. 设置标签字体
label->SetFontId(0);  // 使用默认字体
```

---

### Q7: 图像无法显示

**现象**: UIImageView 显示空白

**原因排查**:
```cpp
// 1. 检查图像文件存在
if (access("/system/data/image.png", F_OK) != 0) {
    // 文件不存在
}

// 2. 检查图像格式支持
// 支持的格式: JPEG, PNG
// 不支持的格式: GIF (ENABLE_GIF=0), WebP

// 3. 检查内存是否足够
// 大图像需要足够内存解码
```

**解决**:
```cpp
// 使用绝对路径
imageView->SetSrc("/system/data/image.png");

// 或使用资源 ID（如果支持）
imageView->SetSrc(IMAGE_ID_MYIMAGE);
```

---

### Q8: 触摸事件无响应

**现象**: 点击按钮无反应

**原因排查**:
```cpp
// 1. 检查视图是否可触摸
button->SetTouchable(true);

// 2. 检查视图是否可见
button->SetVisible(true);

// 3. 检查事件监听是否设置
button->SetOnClickListener(listener);

// 4. 检查是否添加到视图树
rootView->Add(button);

// 5. 检查是否调用 Invalidate
rootView->Invalidate();
```

---

### Q9: 动画不生效

**现象**: 设置了动画但无效果

**原因排查**:
```cpp
// 1. 检查任务管理器运行
TaskManager::GetInstance()->SetTaskRun(true);

// 2. 检查动画是否启动
animator->Start();

// 3. 检查是否在 TaskHandler 中调用
while (running) {
    TaskManager::GetInstance()->TaskHandler();
}

// 4. 检查动画回调
class MyCallback : public AnimatorCallback {
    void Callback(UIView* view) override {
        // 更新视图属性
        view->Invalidate();  // 别忘了刷新
    }
};
```

---

### Q10: 窗口创建失败

**现象**: `Window::CreateWindow()` 返回 nullptr

**原因排查**:
```cpp
// 1. 检查 WMS 服务是否运行
// Window Manager Service 必须先启动

// 2. 检查配置参数
WindowConfig config;
config.rect = {0, 0, 480, 800};  // 合理的尺寸
config.opacity = OPA_OPAQUE;
config.pixelFormat = WINDOW_PIXEL_FORMAT_ARGB8888;

// 3. 检查内存是否足够
// 窗口需要分配 framebuffer
```

**解决**:
```cpp
// 确保 WMS 已初始化
// 在系统启动脚本中先启动 WMS
```

---

## 调试问题

### Q11: 如何查看视图树结构

**解决**:
```cpp
// 使用 DFX 功能（DEBUG 模式）
#include "dfx/ui_dump_dom_tree.h"

// 导出视图树
UIDumpDomTree::GetInstance()->Dump(rootView, "/data/ui_tree.txt");
```

**证据**: `interfaces/kits/dfx/ui_dump_dom_tree.h`

---

### Q12: 如何截取屏幕

**解决**:
```cpp
// 使用 DFX 功能（DEBUG 模式）
#include "dfx/ui_screenshot.h"

// 截图
UIScreenshot::GetInstance()->Screenshot("/data/screenshot.png");
```

**证据**: `interfaces/kits/dfx/ui_screenshot.h`

---

### Q13: 如何查看视图边界

**解决**:
```cpp
// 使用 DFX 功能（DEBUG 模式）
#include "dfx/ui_view_bounds.h"

// 显示边界
UIViewBounds::GetInstance()->Show();
```

**证据**: `interfaces/kits/dfx/ui_view_bounds.h`

---

### Q14: 如何模拟输入事件

**解决**:
```cpp
// 使用 DFX 功能（DEBUG 模式）
#include "dfx/event_injector.h"

// 模拟点击
EventInjector::GetInstance()->SetClickEvent({100, 100});

// 模拟拖拽
EventInjector::GetInstance()->SetDragEvent(
    {100, 100}, {200, 200}, 500);
```

**注意**: 仅在 `ENABLE_DEBUG` 时可用，生产环境应禁用

**证据**: `interfaces/kits/dfx/event_injector.h`

---

## 性能问题

### Q15: 界面卡顿

**原因排查**:
```cpp
// 1. 检查渲染耗时
// 使用 performance_task 监控

// 2. 减少过度绘制
// 避免重叠视图的重复绘制

// 3. 使用脏矩形优化
view->InvalidateRect(dirtyRect);  // 只刷新变化区域

// 4. 减少动画复杂度
animator->SetDuration(200);  // 缩短动画时间
```

---

### Q16: 内存占用过高

**优化建议**:
```cpp
// 1. 及时释放不用的视图
rootView->Remove(unusedView);
delete unusedView;

// 2. 使用图像缓存限制
// 调整 cache_manager 配置

// 3. 字体缓存优化
// 限制字体缓存大小

// 4. 避免创建过多视图
// 使用 List + Adapter 模式复用视图
```

---

## 平台适配问题

### Q17: Qt 模拟器编译失败

**解决**:
```bash
# 1. 确保 Qt 环境安装
qmake --version

# 2. 进入 Qt 项目目录
cd tools/qt/simulator

# 3. 生成 Makefile
qmake simulator.pro

# 4. 编译
make
```

---

### Q18: 不同平台行为不一致

**注意点**:
```cpp
// 1. LiteOS-M 不支持的功能
#ifdef ENABLE_WINDOW
    // 窗口相关代码
#endif

// 2. 不同内核的资源路径
#if defined(__LITEOS__)
    #define RESOURCE_DIR "/user/data/"
#else
    #define RESOURCE_DIR "/storage/data/"
#endif

// 3. 字体支持差异
#if ohos_kernel_type == "liteos_m"
    // 可能不支持某些字体特性
#endif
```

**证据**: `BUILD.gn:56-59`

---

## 其他问题

### Q19: 如何自定义主题

**解决**:
```cpp
// 1. 创建主题
Theme* theme = new Theme();
theme->SetThemeId(1);

// 2. 设置样式
Style* buttonStyle = theme->GetButtonStyle();
buttonStyle->SetStyle(STYLE_BG_COLOR, Color::Red().full);

// 3. 应用主题
ThemeManager::GetInstance()->SetCurrentTheme(theme);
```

---

### Q20: 如何处理按键事件

**解决**:
```cpp
// 1. 设置按键监听
class MyKeyListener : public RootView::OnKeyActListener {
    bool OnKeyAct(UIView& view, const KeyEvent& event) override {
        if (event.GetKeyId() == KEY_BACK) {
            // 处理返回键
            return true;  // 消费事件
        }
        return false;  // 继续传递
    }
};

RootView::GetInstance()->SetOnKeyActListener(new MyKeyListener());
```

---

## 调试技巧

### 日志输出

```cpp
#include "gfx_utils/graphic_log.h"

// 使用日志宏
GRAPHIC_LOGI("Info message: %d", value);
GRAPHIC_LOGW("Warning message");
GRAPHIC_LOGE("Error message");
```

### 断言检查

```cpp
#include "gfx_utils/graphic_assert.h"

GRAPHIC_ASSERT(view != nullptr);
GRAPHIC_ASSERT_MSG(size > 0, "Size must be positive");
```

### 性能分析

```cpp
#include "dfx/performance_task.h"

// 添加性能监控点
PerformanceTask::GetInstance()->StartRecord("render");
// ... 执行渲染
PerformanceTask::GetInstance()->EndRecord("render");
```

## 获取帮助

### 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [graphic_utils_lite](https://gitee.com/openharmony/graphic_utils)
- [window_manager_lite](https://gitee.com/openharmony/graphic_wms)

### 调试信息收集

遇到问题时，请收集以下信息：

1. **构建日志**: `hb build` 完整输出
2. **运行日志**: 串口/ADB 日志
3. **系统信息**: 产品型号、内核类型、系统版本
4. **最小复现**: 能复现问题的最小代码

## 相关文档

- [GN 构建目标](06_GN_Targets.md) - 构建配置
- [编译产物](07_Build_Artifacts.md) - 输出文件
- [对外 API](04_Public_API.md) - API 参考
- [安全风险](08_Security.md) - 安全注意事项

# 05. API/接口差异

> **文档版本**: 1.0
> **最后更新**: 2026-02-08
> **阅读时间**: 25 分钟

---

## 目录

- [1. OH 新增 API](#1-oh-新增-api)
- [2. 行为变更的 API](#2-行为变更的-api)
- [3. 废弃或禁用的功能](#3-废弃或禁用的功能)
- [4. 使用示例](#4-使用示例)
- [5. API 迁移指南](#5-api-迁移指南)

---

## 1. OH 新增 API

### 1.1 字体管理 API

#### SkFontMgr_OHOS

**文件**: `src/ports/skia_ohos/SkFontMgr_ohos.h`

**描述**：OHOS 专用的字体管理器，使用 JSON 配置文件管理字体系统。

**类定义**：
```cpp
class SkFontMgr_OHOS : public SkFontMgr {
public:
    // 构造函数
    SkFontMgr_OHOS(const char* path);

    // 字体家族数量
    int onCountFamilies() const override;

    // 获取字体家族名称
    void onGetFamilyName(int index, SkString* familyName) const override;

    // 创建字体样式集合
    sk_sp<SkFontStyleSet> onCreateStyleSet(int index) const override;

    // 匹配字体家族
    sk_sp<SkFontStyleSet> onMatchFamily(const char familyName[]) const override;

    // 匹配字体家族样式
    sk_sp<SkTypeface> onMatchFamilyStyle(
        const char familyName[],
        const SkFontStyle& style) const override;

    // 创建默认字型
    sk_sp<SkTypeface> onMakeFromData(
        sk_sp<SkData>,
        int ttcIndex) const override;

    // 创建默认字型（流）
    sk_sp<SkTypeface> onMakeFromStreamIndex(
        std::unique_ptr<SkStreamAsset>,
        int ttcIndex) const override;

private:
    std::shared_ptr<FontConfig_OHOS> fFontConfig;
    FontScanner fFontScanner;
    int fFamilyCount;
};
```

**使用示例**：
```cpp
#include "src/ports/skia_ohos/SkFontMgr_ohos.h"

// 创建 OHOS 字体管理器
const char* fontConfigPath = "/etc/fonts/fontconfig_ohos.json";
auto fontMgr = sk_sp<SkFontMgr_OHOS>(new SkFontMgr_OHOS(fontConfigPath));

// 获取字体家族数量
int familyCount = fontMgr->countFamilies();
for (int i = 0; i < familyCount; i++) {
    SkString familyName;
    fontMgr->getFamilyName(i, &familyName);

    // 创建字体样式集合
    auto styleSet = fontMgr->createStyleSet(i);

    // 获取样式数量
    int styleCount = styleSet->countStyles();
    for (int j = 0; j < styleCount; j++) {
        SkFontStyle style;
        SkString styleName;
        styleSet->getStyle(j, &style, &styleName);

        // 创建字型
        auto typeface = styleSet->createTypeface(j);
        if (typeface) {
            // 使用字型
            SkFont font(typeface, 12);
            canvas->drawString("Hello", 0, 20, font, paint);
        }
    }
}
```

---

#### SkTypeface_OHOS

**文件**: `src/ports/skia_ohos/SkTypeface_ohos.h`

**描述**：OHOS 专用的字型实现，支持 OHOS 字体系统特性。

**类定义**：
```cpp
class SkTypeface_OHOS : public SkTypeface_FreeType {
public:
    // 构造函数
    SkTypeface_OHOS(
        const SkFontStyle& style,
        const SkString& familyName,
        const std::shared_ptr<FontInfo_OHOS>& fontInfo);

    // 获取字体信息
    const std::shared_ptr<FontInfo_OHOS>& getFontInfo() const;

    // 获取字体文件路径
    const std::string& getFontPath() const;

protected:
    void onGetFontDescriptor(
        SkFontDescriptor*,
        void* context) const override;

    std::unique_ptr<SkStreamAsset> onOpenStream(int* ttcIndex) const override;

    sk_sp<SkTypeface> onMakeClone(
        const SkFontStyle& style) const override;

private:
    std::shared_ptr<FontInfo_OHOS> fFontInfo;
    std::string fFontPath;
};
```

---

#### SkFontStyleSet_OHOS

**文件**: `src/ports/skia_ohos/SkFontStyleSet_ohos.h`

**描述**：OHOS 字体样式集合，管理同一字体家族的不同样式。

**类定义**：
```cpp
class SkFontStyleSet_OHOS : public SkFontStyleSet {
public:
    // 构造函数
    SkFontStyleSet_OHOS(
        std::shared_ptr<FontConfig_OHOS> fontConfig,
        size_t index,
        bool isFallback = false);

    // 样式数量
    int onCountStyles() const override;

    // 获取样式
    void onGetStyle(
        int index,
        SkFontStyle* style,
        SkString* name) const override;

    // 创建字型
    sk_sp<SkTypeface> onCreateTypeface(int index) const override;

    // 匹配样式
    sk_sp<SkTypeface> onMatchStyle(const SkFontStyle& pattern) const override;

private:
    std::shared_ptr<FontConfig_OHOS> fFontConfig;
    size_t fFamilyIndex;
    bool fIsFallback;
};
```

---

### 1.2 日志 API

#### SkDebugf（OHOS 版本）

**文件**: `src/ports/SkDebug_ohos.cpp`

**描述**：OHOS 特定的日志输出函数，集成 HiLog 日志系统。

**函数声明**：
```cpp
#include "include/private/base/SkDebug.h"

// Skia 日志函数
void SkDebugf(const char format[], ...);
```

**实现**：
```cpp
void SkDebugf(const char format[], ...)
{
    va_list args1, args2;
    va_start(args1, format);

    // 输出到标准输出（可选）
    if (gSkDebugToStdOut) {
        va_copy(args2, args1);
        vprintf(format, args2);
        va_end(args2);
    }

    // 输出到 HiLog
    HiLogPrintArgs(
        LOG_CORE,
        LogLevel::LOG_DEBUG,
        0xD001406,      // 域 ID
        "skia",         // TAG
        format,
        args1
    );

    va_end(args1);
}
```

**使用示例**：
```cpp
#include "include/private/base/SkDebug.h"

// 基础日志
SkDebugf("Skia initialization started\n");

// 格式化日志
int width = 800;
int height = 600;
SkDebugf("Canvas size: %dx%d\n", width, height);
```

---

#### SkShaderReduceProperty（OHOS 扩展）

**文件**: `src/ports/SkDebug_ohos.cpp`

**描述**：检查是否启用 Shader 优化缩减。

**函数声明**：
```cpp
#ifdef SKIA_OHOS_SHADER_REDUCE
bool SkShaderReduceProperty();
#endif
```

**实现**：
```cpp
#ifdef SKIA_OHOS_SHADER_REDUCE
bool SkShaderReduceProperty()
{
    static bool debugProp = std::atoi(
        OHOS::system::GetParameter("persist.sys.skia.shader.reduce", "1").c_str()
    ) != 0;
    return debugProp;
}
#endif
```

**使用示例**：
```cpp
#ifdef SKIA_OHOS_SHADER_REDUCE
if (SkShaderReduceProperty()) {
    // 使用简化 Shader
    paint.setShader(CreateSimpleShader());
} else {
    // 使用完整 Shader
    paint.setShader(CreateComplexShader());
}
#endif
```

---

### 1.3 调试 API

#### IsRenderService

**文件**: `src/ports/SkDebug_ohos.cpp`

**描述**：判断当前进程是否为 Render Service。

**函数声明**：
```cpp
#if defined(SKIA_OHOS_SINGLE_OWNER) || defined(SKIA_DFX_FOR_RECORD_VKIMAGE)
bool IsRenderService();
#endif
```

**实现**：
```cpp
#if defined(SKIA_OHOS_SINGLE_OWNER) || defined(SKIA_DFX_FOR_RECORD_VKIMAGE)
bool IsRenderService()
{
    std::ifstream procfile("/proc/self/cmdline");
    if (!procfile.is_open()) {
        SK_LOGE("IsRenderService open failed");
        return false;
    }
    std::string processName;
    std::getline(procfile, processName);
    procfile.close();

    static const std::string target = "/system/bin/render_service";
    bool result = processName.compare(0, target.size(), target) == 0;
    return result;
}
#endif
```

---

#### GetEnableSkiaSingleOwner

**文件**: `src/ports/SkDebug_ohos.cpp`

**描述**：检查是否启用 Skia 单所有者模式（Render Service 专用）。

**函数声明**：
```cpp
#ifdef SKIA_OHOS_SINGLE_OWNER
bool GetEnableSkiaSingleOwner();
#endif
```

**实现**：
```cpp
#ifdef SKIA_OHOS_SINGLE_OWNER
bool GetEnableSkiaSingleOwner()
{
    static const bool gIsEnableSingleOwner = IsRenderService() && IsBeta();
    return gIsEnableSingleOwner;
}
#endif
```

---

#### PrintBackTrace

**文件**: `src/ports/SkDebug_ohos.cpp`

**描述**：打印线程调用栈。

**函数声明**：
```cpp
#ifdef SKIA_OHOS_SINGLE_OWNER
void PrintBackTrace(uint32_t tid);
#endif
```

**实现**：
```cpp
#ifdef SKIA_OHOS_SINGLE_OWNER
void PrintBackTrace(uint32_t tid)
{
    std::string msg = "";
    OHOS::HiviewDFX::GetBacktraceStringByTid(msg, tid, 0, false);
    if (!msg.empty()) {
        std::vector<std::string> out;
        std::stringstream ss(msg);
        std::string s;
        while (std::getline(ss, s, '\n')) {
            out.push_back(s);
        }
        SK_LOGE(" ======== tid:%{public}d", tid);
        for (auto const& line : out) {
            SK_LOGE(" callstack %{public}s", line.c_str());
        }
    }
}
#endif
```

---

## 2. 行为变更的 API

### 2.1 字体管理器创建

**上游默认行为**：
```cpp
// 使用默认字体管理器
auto fontMgr = SkFontMgr::RefDefault();
```

**OH 特定行为**：
```cpp
// 使用 OHOS 字体管理器
const char* fontConfigPath = "/etc/fonts/fontconfig_ohos.json";
auto fontMgr = sk_sp<SkFontMgr_OHOS>(new SkFontMgr_OHOS(fontConfigPath));

// 或者通过工厂函数
auto fontMgr = SkFontMgr_New_OHOS();
```

**差异**：
- 上游：使用系统默认字体管理器
- OH：使用 JSON 配置驱动的字体管理器

---

### 2.2 日志输出

**上游默认行为**：
```cpp
// 输出到标准错误
SkDebugf("Debug message\n");
```

**OH 特定行为**：
```cpp
// 输出到 HiLog 系统日志
SkDebugf("Debug message\n");  // 自动路由到 HiLog
```

**差异**：
- 上游：输出到 stderr
- OH：输出到 HiLog（同时可选输出到 stdout）

---

### 2.3 文本断行

**上游默认行为**：
```cpp
// 标准 ICU 断行规则
auto paragraph = builder.Build();
paragraph->layout(width);
```

**OH 特定行为**：
```cpp
// 使用 OH 特定的断行规则（wordbrk.patch）
auto paragraph = builder.Build();
paragraph->layout(width);

// 在 @ 和 . 处断开（Chromium 特定行为）
```

**差异**：
- 上游：标准 ICU 断行规则
- OH：在 @ 和 . 处强制断行

---

## 3. 废弃或禁用的功能

### 3.1 GPU 后端限制

**上游支持**：
- OpenGL ES
- Vulkan
- Metal
- Direct3D
- Dawn（WebGPU）

**OH 支持限制**：
```gn
# 仅支持 OpenGL 和 Vulkan
if (is_ohos) {
  deps += [ "${skia_root_dir}:gpu" ]

  if (skia_feature_use_vulkan && !is_arkui_x) {
    public_external_deps += [ "vulkan-headers:vulkan_headers" ]
  }
}

# Metal 和 Direct3D 在 OH 中不可用
if (is_mac || is_ios) {
  # Metal 仅在 macOS/iOS 上可用
}
if (is_win) {
  # Direct3D 仅在 Windows 上可用
}
```

---

### 3.2 字体管理器限制

**上游支持**：
- FontConfig（Linux）
- CoreText（macOS/iOS）
- DirectWrite（Windows）
- Android（NDK）
- Custom Directory
- Empty

**OH 支持限制**：
```gn
# 仅启用 fontmgr_ohos
deps = [
  "${skia_root_dir}:fontmgr_ohos",
]

# 禁用其他字体管理器
if (is_arkui_x) {
  deps -= [ "${skia_root_dir}:fontmgr_ohos" ]
}
if (!use_oh_skia) {
  deps -= [ "${skia_root_dir}:fontmgr_ohos" ]
}
```

---

### 3.3 编解码器限制

**上游支持**：
- JPEG、PNG、WebP、HEIF、BMP、WBMP、GIF（单帧）

**OH 编解码器**：
```gn
# 启用特定编解码器
defines = [
  "SK_CODEC_DECODES_JPEG",
  "SK_CODEC_DECODES_PNG",
  "SK_CODEC_DECODES_WEBP",
  "SK_HAS_ANDROID_CODEC",      # Android 编解码器
  "SK_ENABLE_OHOS_CODEC",      # OHOS 特定编解码器
]

# 性能优化
defines = [
  "TURBO_JPEG_HUFF_DECODE_OPT",   # JPEG Huffman 优化
  "TURBO_PNG_MULTY_LINE_OPT",    # PNG 多行优化
]
```

---

## 4. 使用示例

### 4.1 完整的 OHOS 字体管理示例

```cpp
#include "src/ports/skia_ohos/SkFontMgr_ohos.h"
#include "src/ports/skia_ohos/FontConfig_ohos.h"

class OHOSFontManager {
public:
    OHOSFontManager(const char* configPath)
        : fontMgr_(new SkFontMgr_OHOS(configPath))
    {}

    // 绘制文本
    void DrawText(
        SkCanvas* canvas,
        const std::string& text,
        const std::string& familyName,
        float fontSize)
    {
        // 获取字体家族
        auto styleSet = fontMgr_->matchFamily(familyName.c_str());
        if (!styleSet) {
            styleSet = fontMgr_->matchFamily(nullptr);  // 默认字体
        }

        // 匹配样式（常规）
        SkFontStyle style(SkFontStyle::kNormal_Weight,
                         SkFontStyle::kNormal_Width,
                         SkFontStyle::kUpright_Slant);

        auto typeface = styleSet->matchStyle(style);
        if (!typeface) {
            return;
        }

        // 创建字体
        SkFont font(typeface, fontSize);
        font.setSubpixel(true);

        // 创建 Paint
        SkPaint paint;
        paint.setAntiAlias(true);
        paint.setColor(SK_ColorBLACK);

        // 绘制文本
        canvas->drawString(text.c_str(), 0, fontSize, font, paint);
    }

private:
    sk_sp<SkFontMgr_OHOS> fontMgr_;
};

// 使用
OHOSFontManager fontManager("/etc/fonts/fontconfig_ohos.json");
fontManager.DrawText(canvas, "Hello, OpenHarmony!", "HarmonyOS Sans", 24);
```

---

### 4.2 OHOS 日志集成示例

```cpp
#include "include/private/base/SkDebug.h"
#include "src/ports/SkDebug_ohos.h"

class SkiaLogger {
public:
    static void Init() {
        SkDebugf("Skia Logger initialized\n");

#ifdef SKIA_OHOS_SINGLE_OWNER
        if (GetEnableSkiaSingleOwner()) {
            SkDebugf("Skia single owner mode enabled\n");
        }
#endif

#ifdef SKIA_OHOS_SHADER_REDUCE
        if (SkShaderReduceProperty()) {
            SkDebugf("Shader reduce optimization enabled\n");
        }
#endif
    }

    static void LogError(const std::string& message) {
        SK_LOGE("%s", message.c_str());
    }

    static void LogInfo(const std::string& message) {
        SK_LOGI("%s", message.c_str());
    }
};

// 使用
SkiaLogger::Init();
SkiaLogger::LogInfo("Canvas created successfully");
SkiaLogger::LogError("Failed to load image");
```

---

### 4.3 HarmonyOS Symbol 渲染示例

```cpp
#include "src/ports/skia_ohos/HmSymbolConfig_ohos.h"

class HmSymbolRenderer {
public:
    HmSymbolRenderer()
        : symbolConfig_(new HmSymbolConfig_OHOS())
    {}

    // 渲染 HarmonyOS Symbol
    bool DrawSymbol(
        SkCanvas* canvas,
        const std::string& symbolName,
        const SkRect& rect,
        SkColor color)
    {
        // 获取 Symbol 路径
        SkPath symbolPath;
        if (!symbolConfig_->GetSymbolPath(symbolName, symbolPath)) {
            SK_LOGE("Symbol not found: %s", symbolName.c_str());
            return false;
        }

        // 创建 Paint
        SkPaint paint;
        paint.setColor(color);
        paint.setAntiAlias(true);
        paint.setStyle(SkPaint::kFill_Style);

        // 变换路径到目标矩形
        SkMatrix matrix;
        SkRect symbolBounds = symbolPath.getBounds();
        matrix.setRectToRect(symbolBounds, rect, SkMatrix::kFill_ScaleToFit);

        SkPath transformedPath;
        symbolPath.transform(matrix, &transformedPath);

        // 绘制
        canvas->drawPath(transformedPath, paint);
        return true;
    }

private:
    std::unique_ptr<HmSymbolConfig_OHOS> symbolConfig_;
};

// 使用
HmSymbolRenderer renderer;
renderer.DrawSymbol(canvas, "star", SkRect::MakeWH(32, 32), SK_ColorYELLOW);
renderer.DrawSymbol(canvas, "heart", SkRect::MakeWH(24, 24), SK_ColorRED);
```

---

## 5. API 迁移指南

### 5.1 从上游 Skia 迁移到 OH Skia

#### 字体管理器迁移

**上游代码**：
```cpp
auto fontMgr = SkFontMgr::RefDefault();
auto typeface = fontMgr->matchFamilyStyle(
    "Roboto",
    SkFontStyle()
);
```

**OH 代码**：
```cpp
auto fontMgr = sk_sp<SkFontMgr_OHOS>(
    new SkFontMgr_OHOS("/etc/fonts/fontconfig_ohos.json")
);
auto typeface = fontMgr->matchFamilyStyle(
    "HarmonyOS Sans",  // 改为 OH 字体
    SkFontStyle()
);
```

---

#### 日志迁移

**上游代码**：
```cpp
// 自定义日志函数
void MySkiaDebugf(const char format[], ...) {
    va_list args;
    va_start(args, format);
    vfprintf(stderr, format, args);
    va_end(args);
}

// 替换 SkDebugf
#define SkDebugf MySkiaDebugf
```

**OH 代码**：
```cpp
// 直接使用 OH 版本的 SkDebugf
SkDebugf("Debug message: %s\n", message);
```

---

### 5.2 从旧版本 Skia 迁移到 M133

#### 版本检查

```cpp
#if defined(USE_M133_SKIA)
    // M133 特定代码
    #include "src/ports/skia_ohos/SkFontMgr_ohos.h"
#else
    // 旧版本代码
    #include "include/ports/SkFontMgr.h"
#endif
```

---

#### 条件编译

```cpp
// 使用 M133 的 OHOS 功能
#ifdef SK_ENABLE_OHOS_CODEC
    auto codec = SkOHOSCodec::MakeFromData(data);
#else
    auto codec = SkCodec::MakeFromData(data);
#endif
```

---

## 附录

### A. OHOS API 完整列表

| API | 文件 | 用途 |
|-----|------|------|
| `SkFontMgr_OHOS` | `src/ports/skia_ohos/SkFontMgr_ohos.h` | OHOS 字体管理器 |
| `SkTypeface_OHOS` | `src/ports/skia_ohos/SkTypeface_ohos.h` | OHOS 字型 |
| `SkFontStyleSet_OHOS` | `src/ports/skia_ohos/SkFontStyleSet_ohos.h` | OHOS 字体样式集合 |
| `SkDebugf` | `src/ports/SkDebug_ohos.cpp` | OHOS 日志函数 |
| `SkShaderReduceProperty` | `src/ports/SkDebug_ohos.cpp` | Shader 优化检查 |
| `IsRenderService` | `src/ports/SkDebug_ohos.cpp` | Render Service 检测 |
| `GetEnableSkiaSingleOwner` | `src/ports/SkDebug_ohos.cpp` | 单所有者模式检查 |
| `PrintBackTrace` | `src/ports/SkDebug_ohos.cpp` | 调用栈打印 |
| `HmSymbolConfig_OHOS` | `src/ports/skia_ohos/HmSymbolConfig_ohos.h` | HarmonyOS Symbol 配置 |

---

**下一节**: [06. 安全风险分析](06_Security.md)

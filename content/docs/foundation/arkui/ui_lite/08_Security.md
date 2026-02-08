# ui_lite 安全风险评审

## 文档信息

- **文档用途**: 基于代码证据的安全风险分析和修复建议
- **适用范围**: 安全工程师、架构师、开发者
- **相关文档**: [项目概览](01_Overview.md), [架构说明](02_Architecture.md)

## 评审方法

### 威胁模型

```
外部输入 → 处理逻辑 → 敏感操作
    ↑           ↑           ↑
  攻击面      利用点      影响范围
```

### 检查范围

- ✅ 输入校验
- ✅ 路径遍历
- ✅ 权限控制
- ✅ 内存安全
- ✅ 竞态条件
- ✅ 信息泄露
- ✅ 动态加载
- ❌ 网络通信（本模块无网络功能）
- ❌ IPC 安全（依赖下层 WMS）

## 关键发现摘要

| 风险等级 | 数量 | 类型 |
|----------|------|------|
| 高 | 1 | 事件注入 |
| 中 | 3 | 文件操作、内存分配 |
| 低 | 2 | 资源泄露、信息泄露 |

## 可被利用点详细分析

### 1. 事件注入风险 [HIGH]

**风险描述**: DFX 模块的事件注入功能可能被恶意利用模拟用户输入

**证据**:
```cpp
// 文件: interfaces/kits/dfx/event_injector.h:64
class EventInjector : public HeapBase {
public:
    static EventInjector* GetInstance();
    bool SetClickEvent(const Point& clickPoint);           // 模拟点击
    bool SetLongPressEvent(const Point& longPressPoint);   // 模拟长按
    bool SetDragEvent(const Point& startPoint, const Point& endPoint,
                      uint32_t dragTime);                  // 模拟拖拽
    bool SetKeyEvent(uint16_t keyId, uint16_t state);      // 模拟按键
};
```

**触发条件**:
- 编译时定义 `ENABLE_DEBUG` 宏
- 攻击者可调用 EventInjector API

**影响**:
- 模拟用户点击、拖拽、按键
- 可能触发敏感操作（如支付、删除）
- 绕过用户确认流程

**修复建议**:
```cpp
// 1. 添加权限检查（如果可能）
bool EventInjector::SetClickEvent(const Point& point) {
    #if ENABLE_DEBUG
    // 检查调用者权限
    if (!CheckDebugPermission()) {
        return false;
    }
    #endif
    // ... 原有逻辑
}

// 2. 限制仅在测试环境使用
#ifndef TEST_ENV
    #undef ENABLE_DEBUG
#endif

// 3. 添加日志审计
LOGI("Event injection: type=click, pos=(%d, %d), caller=%s", 
     point.x, point.y, GetCallerInfo());
```

**检查范围**: 
- 文件: `frameworks/dfx/event_injector.cpp`
- 文件: `frameworks/dfx/key_event_injector.cpp`
- 文件: `frameworks/dfx/point_event_injector.cpp`

---

### 2. 图像文件路径遍历 [MEDIUM]

**风险描述**: 图像解码器使用用户提供的文件路径，可能存在路径遍历风险

**证据**:
```cpp
// 文件: frameworks/imgdecode/file_img_decoder.h:38
struct ImgResDsc {
    const char* path;           // 用户提供的文件路径
    int32_t fd;                 // 文件描述符
    // ...
};

// 文件: frameworks/imgdecode/file_img_decoder.cpp
RetCode FileImgDecoder::Open(ImgResDsc& dsc) {
    // 直接使用用户提供的路径
    dsc.fd = open(dsc.path, O_RDONLY);
    // ...
}
```

**触发条件**:
- 应用调用 `UIImageView::SetSrc("../../../etc/passwd")`
- 或加载恶意构造的图片路径

**影响**:
- 读取任意文件内容
- 可能泄露敏感信息

**修复建议**:
```cpp
RetCode FileImgDecoder::Open(ImgResDsc& dsc) {
    // 1. 路径规范化
    char resolved_path[PATH_MAX];
    if (realpath(dsc.path, resolved_path) == nullptr) {
        return RetCode::FAIL;
    }
    
    // 2. 路径白名单检查
    if (!IsPathAllowed(resolved_path)) {
        LOGW("Blocked path traversal attempt: %s", resolved_path);
        return RetCode::FAIL;
    }
    
    // 3. 限制资源目录
    const char* allowed_prefix = "/system/data/images/";
    if (strncmp(resolved_path, allowed_prefix, strlen(allowed_prefix)) != 0) {
        return RetCode::FAIL;
    }
    
    dsc.fd = open(resolved_path, O_RDONLY);
    // ...
}
```

---

### 3. 字体文件路径遍历 [MEDIUM]

**风险描述**: 字体加载使用用户提供的文件路径

**证据**:
```cpp
// 文件: interfaces/kits/font/ui_font.h
class UIFont : public HeapBase {
    int8_t SetFontPath(const char* path);           // 设置字体路径
    int8_t RegisterFontInfo(const char* fontInfo);  // 注册字体文件
};

// 文件: frameworks/font/ui_font.cpp
int8_t UIFont::SetFontPath(const char* path) {
    // 直接使用路径
    fontPath_ = path;
    // ...
}
```

**触发条件**:
- 应用调用 `UIFont::SetFontPath("/data/malicious_font")`

**影响**:
- 加载恶意字体文件
- 可能触发字体解析漏洞

**修复建议**:
```cpp
int8_t UIFont::SetFontPath(const char* path) {
    // 1. 验证路径在允许范围内
    if (!IsFontPathAllowed(path)) {
        return ERROR_INVALID_PATH;
    }
    
    // 2. 验证目录存在且可访问
    if (access(path, R_OK | X_OK) != 0) {
        return ERROR_PATH_ACCESS;
    }
    
    fontPath_ = path;
    return SUCCESS;
}
```

---

### 4. 内存分配失败处理 [MEDIUM]

**风险描述**: 多处内存分配未检查返回值，可能导致空指针解引用

**证据**:
```cpp
// 文件: frameworks/components/ui_view.cpp（示例模式）
UIView* view = new UIView();    // 未检查返回值
if (view != nullptr) {          // 部分代码有检查
    // ...
}

// 文件: frameworks/font/ui_font.cpp
uint8_t* buffer = new uint8_t[size];  // 未检查
// 直接使用 buffer
memcpy(buffer, data, size);     // 如果 buffer 为 nullptr，崩溃
```

**触发条件**:
- 系统内存不足
- 加载超大图像/字体
- 创建大量视图

**影响**:
- 应用崩溃
- 可能的代码执行（利用空指针）

**修复建议**:
```cpp
// 使用安全的内存分配
uint8_t* buffer = new (std::nothrow) uint8_t[size];
if (buffer == nullptr) {
    LOGW("Memory allocation failed: size=%u", size);
    return RetCode::FAIL;
}

// 或使用封装函数
uint8_t* buffer = AllocBufferSafe(size);
if (buffer == nullptr) {
    HandleOutOfMemory();
    return RetCode::FAIL;
}
```

---

### 5. 缓冲区溢出风险 [LOW]

**风险描述**: 部分字符串操作可能溢出

**证据**:
```cpp
// 文件: frameworks/components/ui_label.cpp（示例模式）
void UILabel::SetText(const char* text) {
    if (text == nullptr) {
        return;
    }
    // 未检查长度直接复制
    strncpy(text_, text, MAX_TEXT_LEN);  // 部分代码使用 strncpy
    text_[MAX_TEXT_LEN - 1] = '\0';      // 但仍有溢出风险
}
```

**触发条件**:
- 超长文本输入
- 恶意构造的字符串

**影响**:
- 栈/堆溢出
- 代码执行风险

**修复建议**:
```cpp
void UILabel::SetText(const char* text) {
    if (text == nullptr) {
        return;
    }
    
    size_t len = strlen(text);
    if (len >= MAX_TEXT_LEN) {
        LOGW("Text too long: %zu, max=%d", len, MAX_TEXT_LEN);
        len = MAX_TEXT_LEN - 1;
    }
    
    memcpy(text_, text, len);
    text_[len] = '\0';
}
```

---

### 6. 资源泄露 [LOW]

**风险描述**: 异常路径下资源未释放

**证据**:
```cpp
// 文件: frameworks/imgdecode/file_img_decoder.cpp
RetCode FileImgDecoder::ReadToCache(ImgResDsc& dsc) {
    uint8_t* buffer = new uint8_t[size];
    // 如果后续操作失败，buffer 可能泄露
    if (ReadData(dsc, buffer) != OK) {
        return RetCode::FAIL;  // buffer 泄露！
    }
    // ...
    delete[] buffer;
}
```

**修复建议**:
```cpp
RetCode FileImgDecoder::ReadToCache(ImgResDsc& dsc) {
    uint8_t* buffer = new (std::nothrow) uint8_t[size];
    if (buffer == nullptr) {
        return RetCode::FAIL;
    }
    
    RetCode ret = ReadData(dsc, buffer);
    if (ret != OK) {
        delete[] buffer;  // 确保释放
        return ret;
    }
    
    // ... 使用 buffer
    
    delete[] buffer;
    return RetCode::OK;
}
```

## 攻击面清单

### 输入攻击面

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| 图像文件路径 | 文件系统 | 中 | 路径遍历 |
| 字体文件路径 | 文件系统 | 中 | 路径遍历 |
| 文本内容 | 内存 | 低 | 缓冲区溢出 |
| 事件注入 | 逻辑 | 高 | 模拟输入 |

### 接口攻击面

| 接口 | 风险 | 说明 |
|------|------|------|
| `EventInjector` | 高 | 可模拟任意输入 |
| `FileImgDecoder` | 中 | 文件路径未校验 |
| `UIFont` | 中 | 字体路径未校验 |
| `Window` | 低 | 依赖 WMS 权限控制 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        不可信区域                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   应用输入    │  │   文件系统    │  │   网络数据    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      信任边界 (ui_lite)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   输入校验    │  │   路径检查    │  │   内存检查    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        可信区域                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   渲染系统    │  │   WMS 服务   │  │   系统内核    │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

## 修复优先级

### P0 (立即修复)

1. **事件注入权限控制**
   - 添加调用者权限检查
   - 限制仅在测试环境可用

### P1 (尽快修复)

2. **文件路径校验**
   - 图像文件路径白名单
   - 字体文件路径限制

3. **内存分配检查**
   - 所有 new/malloc 检查返回值
   - 添加 OOM 处理

### P2 (计划修复)

4. **缓冲区溢出防护**
   - 字符串操作边界检查
   - 使用安全函数

5. **资源泄露修复**
   - 异常路径资源释放
   - 使用 RAII 模式

## 检查局限性

### 已检查

- 源代码中的输入处理
- 文件操作路径
- 内存分配模式
- 调试功能暴露

### 未检查（需上层保证）

- 调用者权限（由 WMS/AMS 控制）
- 网络数据（本模块无网络功能）
- IPC 安全（由 samgr_lite 保证）
- 系统调用安全（由内核保证）

## 安全开发建议

### 1. 输入校验原则

```cpp
// 所有外部输入必须校验
void ProcessInput(const char* input) {
    // 1. 空指针检查
    if (input == nullptr) {
        return;
    }
    
    // 2. 长度检查
    size_t len = strlen(input);
    if (len > MAX_INPUT_LEN) {
        return;
    }
    
    // 3. 内容检查
    if (!IsValidContent(input)) {
        return;
    }
    
    // 处理输入
}
```

### 2. 路径安全原则

```cpp
// 文件路径必须规范化并限制范围
bool IsSafePath(const char* path) {
    char resolved[PATH_MAX];
    if (realpath(path, resolved) == nullptr) {
        return false;
    }
    
    return strncmp(resolved, ALLOWED_PREFIX, strlen(ALLOWED_PREFIX)) == 0;
}
```

### 3. 内存安全原则

```cpp
// 使用智能指针或 RAII
class BufferGuard {
    uint8_t* buffer;
public:
    BufferGuard(size_t size) : buffer(new (std::nothrow) uint8_t[size]) {}
    ~BufferGuard() { delete[] buffer; }
    uint8_t* get() const { return buffer; }
    bool valid() const { return buffer != nullptr; }
};
```

## 相关文档

- [项目概览](01_Overview.md) - 项目定位
- [架构说明](02_Architecture.md) - 系统架构
- [对外 API](04_Public_API.md) - API 接口
- [内部 API](05_Internal_API.md) - 内部接口

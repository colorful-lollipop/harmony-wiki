# 安全风险评审

## 概述

本章节对 `media_utils_lite` 组件进行安全风险评审，识别潜在攻击面、信任边界，并提供风险点分析及修复建议。

**评审范围**: 本组件所有对外接口、构建配置、数据结构

**评审方法**: 基于代码静态分析，识别安全敏感代码路径

## 信任边界

### 信任边界图

```
┌─────────────────────────────────────────────────────────────────────┐
│                          信任边界                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  media_utils_lite (本组件 - 受信任区域)                      │   │
│  │  • interfaces/kits/ - 公共 API (半信任)                       │   │
│  │  • src/ - 内部实现 (完全信任)                                │   │
│  │  • hals/ - HAL 接口定义 (半信任)                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│  ┌──────────────────────────▼──────────────────────────────────┐  │
│  │  硬件/驱动层 (不可信，需验证输入)                             │  │
│  │  • HAL 实现                                                 │  │
│  │  • 设备驱动                                                 │  │
│  │  • 内核接口                                                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────┐                                     │
│  │  上层应用/子系统 (半信任)                                     │
│  │  • camera_lite, audio_lite, media_lite                       │  │
│  │  • 用户应用                                                 │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 边界说明

| 区域 | 信任级别 | 说明 |
|------|----------|------|
| `src/` | 高 | 内部实现，代码可控 |
| `interfaces/kits/` | 中 | 公共 API，需校验输入 |
| `hals/` | 中 | HAL 接口定义，需验证实现 |
| `third_party/` | 低 | 第三方依赖 |
| 硬件/驱动 | 低 | 不可信外部实现 |

## 攻击面分析

### 攻击面清单

| 攻击面 | 类型 | 说明 | 风险等级 |
|--------|------|------|----------|
| Source URI 解析 | 输入验证 | 外部传入的 URI 字符串 | 中 |
| Format 参数设置 | 输入验证 | 动态键值对设置 | 中 |
| HAL 接口调用 | 系统调用 | 硬件抽象层函数 | 中 |
| 缓冲区操作 | 内存安全 | DataBuffer 读写 | 高 |
| 字符串处理 | 内存安全 | std::string 操作 | 中 |
| 文件描述符 | 资源安全 | SOURCE_TYPE_FD 类型 | 中 |

### 数据流图

```
外部输入                    内部处理                     硬件
   │                          │                          │
   ▼                          ▼                          ▼
┌─────────┐              ┌──────────┐              ┌──────────┐
│  URI    │ ───────►     │  Source  │ ───────►    │  HAL     │
│  参数   │              │  解析    │              │  调用    │
└─────────┘              └──────────┘              └──────────┘
                              │                          │
                              ▼                          ▼
                        ┌──────────┐              ┌──────────┐
                        │  Format  │              │ 硬件驱动 │
                        │  键值对  │              │ (不可信) │
                        └──────────┘              └──────────┘
```

## 安全风险点

### 风险 1: Format 字符串指针悬空 (高风险)

**位置**: `format.h:206-212`, `format.cpp:129-136`

**代码证据**:
```cpp
// format.h:206-212
union {
    int32_t int32Val;
    int64_t int64Val;
    float floatVal;
    double doubleVal;
    std::string *stringVal;  // 指针类型，可能悬空
} val_;

// format.cpp:129-136
bool FormatData::SetValue(const std::string &val)
{
    if (type_ != FORMAT_TYPE_STRING) {
        return false;
    }
    val_.stringVal = new (std::nothrow) std::string();
    if (val_.stringVal == nullptr) {
        type_ = FORMAT_TYPE_NONE;  // 状态改变，但外部可能不知情
        return false;
    }
    *(val_.stringVal) = val;
    return true;
}
```

**风险描述**:
- `FormatData` 使用 union 存储字符串指针
- `new` 失败时仅设置 `type_` 为 `FORMAT_TYPE_NONE`
- 后续如果其他代码依赖 `type_` 状态判断，可能访问已释放内存

**触发条件**:
1. 内存耗尽时调用 `SetValue(std::string)`
2. `FormatData` 拷贝构造时源对象已被修改

**影响**: 内存访问错误，程序崩溃，可能被利用

**修复建议**:
```cpp
bool FormatData::SetValue(const std::string &val)
{
    if (type_ != FORMAT_TYPE_STRING) {
        return false;
    }
    // 先清理旧值
    if (val_.stringVal != nullptr) {
        delete val_.stringVal;
        val_.stringVal = nullptr;
    }
    val_.stringVal = new (std::nothrow) std::string(val);
    if (val_.stringVal == nullptr) {
        return false;
    }
    return true;
}
```

---

### 风险 2: Format 字符串双重释放 (中风险)

**位置**: `format.cpp:74-80`

**代码证据**:
```cpp
FormatData::~FormatData()
{
    if (type_ == FORMAT_TYPE_STRING) {
        if (val_.stringVal != nullptr) {
            delete val_.stringVal;  // 删除指针
        }
    }
}
```

**风险描述**:
- 析构函数删除 `stringVal` 指针
- 但未设置指针为 `nullptr`
- 如果对象被拷贝后析构，可能导致双重释放

**触发条件**:
1. `FormatData` 被拷贝构造
2. 两个对象析构时都尝试删除同一指针

**影响**: 内存损坏，程序崩溃

**修复建议**:
```cpp
FormatData::~FormatData()
{
    if (type_ == FORMAT_TYPE_STRING) {
        if (val_.stringVal != nullptr) {
            delete val_.stringVal;
            val_.stringVal = nullptr;  // 设置为 nullptr 防止双重释放
        }
    }
}

// 建议添加拷贝构造函数和赋值运算符
FormatData(const FormatData&) = delete;
FormatData& operator=(const FormatData&) = delete;
```

---

### 风险 3: Source URI 缺少路径验证 (中风险)

**位置**: `source.h:204-205`, `source.cpp:21-24`

**代码证据**:
```cpp
// source.h:204-205
explicit Source(const std::string& uri);  // 直接接受字符串

// source.cpp:21-24
Source::Source(const std::string &uri)
    : uri_(uri),
      sourceType_(SourceType::SOURCE_TYPE_URI)
{}
```

**风险描述**:
- `Source` 构造函数直接接受 URI 字符串
- 未验证 URI 格式、长度、是否包含特殊字符
- 可能导致路径遍历攻击（如果是文件路径）

**触发条件**:
1. 传入恶意构造的 URI，如 `../../../etc/passwd`
2. 传入超长 URI 导致缓冲区溢出（虽然 std::string 可变长）

**影响**: 越权文件访问，路径遍历

**修复建议**:
```cpp
Source::Source(const std::string &uri) : sourceType_(SourceType::SOURCE_TYPE_URI)
{
    // URI 长度限制
    constexpr size_t MAX_URI_LENGTH = 4096;
    if (uri.length() > MAX_URI_LENGTH) {
        MEDIA_ERR_LOG("URI too long: %zu", uri.length());
        throw std::invalid_argument("URI too long");
    }
    
    // 对于文件路径，检查路径遍历
    if (uri.compare(0, 8, "file:///") == 0) {
        std::string path = uri.substr(8);
        // 检查路径遍历模式
        if (path.find("..") != std::string::npos) {
            MEDIA_ERR_LOG("Potential path traversal in URI");
            throw std::invalid_argument("Invalid path");
        }
    }
    
    uri_ = uri;
}
```

---

### 风险 4: HAL 返回值未充分校验 (中风险)

**位置**: `hal_camera.h:213-237` 等 HAL 接口

**代码证据**:
```c
// hal_camera.h:213-237
int32_t HalCameraInit(void);
int32_t HalCameraDeinit(void);
int32_t HalCameraGetDeviceNum(uint8_t *num);  // 输出参数
int32_t HalCameraDeviceOpen(uint32_t cameraId);
// ... 更多返回 int32_t 的函数

// 调用示例（假设）
int32_t ret = HalCameraDeviceOpen(cameraId);
// if (ret != 0) { /* 错误处理 */ }  // 调用方可能未检查
```

**风险描述**:
- HAL 接口返回 `int32_t` 错误码
- `HalCameraGetDeviceNum` 等函数通过指针输出参数返回数据
- 如果调用方未检查返回值，可能使用未初始化的数据

**触发条件**:
1. HAL 函数返回错误但调用方未检查
2. 使用了未初始化的输出指针

**影响**: 使用未初始化内存，数据不一致

**修复建议**:
```cpp
// HAL 接口应明确文档说明错误处理方式
// 调用方应始终检查返回值

int32_t HalCameraGetDeviceNum(uint8_t *num)
{
    if (num == nullptr) {
        return HAL_MEDIA_ERR;  // 参数校验
    }
    // ... 实现
    *num = deviceCount;
    return HAL_MEDIA_OK;
}

// 调用方代码
uint8_t deviceNum = 0;
int32_t ret = HalCameraGetDeviceNum(&deviceNum);
if (ret != HAL_MEDIA_OK) {
    MEDIA_ERR_LOG("Failed to get device num: %d", ret);
    return ret;
}
// 安全使用 deviceNum
```

---

### 风险 5: StreamSource 空指针解引用 (中风险)

**位置**: `source.cpp:103-120`

**代码证据**:
```cpp
uint8_t* StreamSource::GetSharedBuffer(size_t& size)
{
#ifndef SURFACE_DISABLED
    if ((surface_ == nullptr) || (curBuffer_ != nullptr)) {
        return nullptr;  // 返回 nullptr
    }
    SurfaceBuffer* surfaceBuffer = surface_->RequestBuffer();
    if (surfaceBuffer != nullptr) {
        curBuffer_ = surfaceBuffer;
        size = surface_->GetSize();
        return static_cast<uint8_t*>(surfaceBuffer->GetVirAddr());
    } else {
        return nullptr;  // 返回 nullptr
    }
#else
    return nullptr;
#endif
}
```

**风险描述**:
- 函数可能返回 `nullptr`
- 调用方如果未检查直接使用，可能导致空指针解引用

**触发条件**:
1. `surface_` 为空
2. `RequestBuffer()` 返回空
3. 调用方未检查返回值

**影响**: 程序崩溃

**修复建议**:
```cpp
// 调用方代码示例
size_t bufferSize = 0;
uint8_t* buffer = streamSource->GetSharedBuffer(bufferSize);
if (buffer == nullptr) {
    MEDIA_ERR_LOG("Failed to get shared buffer");
    return ERR_INVALID_OPERATION;
}
// 安全使用 buffer
```

---

### 风险 6: 整数溢出潜力 (低风险)

**位置**: `hal_camera.h:34-38`

**代码证据**:
```c
typedef struct {
    uint32_t width;
    uint32_t height;
    uint32_t fps;
} HalVideoProcessorAttr;

// 计算缓冲区大小时可能溢出
size_t bufferSize = attr->width * attr->height * 4;  // ARGB8888
```

**风险描述**:
- `width * height * bytesPerPixel` 可能导致整数溢出
- 溢出后分配的缓冲区过小，导致写越界

**触发条件**:
1. 传入异常的 `width` 和 `height` 值
2. 乘法运算溢出

**影响**: 内存损坏，潜在代码执行

**修复建议**:
```cpp
#include <cstdint>
#include <cstdlib>

bool CalculateBufferSize(uint32_t width, uint32_t height, size_t& outSize)
{
    constexpr size_t MAX_BUFFER_SIZE = 1024 * 1024 * 4; // 4MB 限制
    
    // 检查宽度和高度的上限
    if (width == 0 || height == 0) {
        return false;
    }
    
    // 使用 64 位计算避免溢出
    uint64_t total = static_cast<uint64_t>(width) * height * 4;
    
    if (total > MAX_BUFFER_SIZE) {
        return false;  // 超出允许范围
    }
    
    outSize = static_cast<size_t>(total);
    return true;
}
```

---

## 安全最佳实践

### 输入验证

| 检查项 | 方法 | 位置示例 |
|--------|------|----------|
| 字符串长度 | `uri.length() <= MAX_LENGTH` | Source 构造函数 |
| 数值范围 | 范围检查 (min <= x <= max) | Format::SetValue |
| 指针有效性 | `nullptr` 检查 | StreamSource |
| 缓冲区大小 | 溢出检查 | HAL 接口 |

### 内存安全

| 实践 | 说明 |
|------|------|
| RAII | 使用智能指针管理内存 |
| 避免裸指针 | 优先使用 `std::shared_ptr` |
| 禁用拷贝 | 必要时删除拷贝构造函数 |
| 双重释放防护 | 释放后设置 `nullptr` |

### 错误处理

| 实践 | 说明 |
|------|------|
| 始终检查返回值 | HAL 函数返回值 |
| 传播错误码 | 使用统一的错误码体系 |
| 记录错误日志 | 使用 `MEDIA_ERR_LOG` |

## 安全加固建议

### 短期建议 (高优先级)

1. **修复 Format 字符串管理**
   - 添加拷贝构造函数/赋值运算符删除
   - 析构函数中设置 `stringVal = nullptr`
   - 改进 `SetValue` 错误处理

2. **强化输入验证**
   - Source URI 长度限制
   - 路径遍历检查
   - HAL 参数校验

### 中期建议

3. **代码审计**
   - 审查所有指针操作
   - 添加 AddressSanitizer 测试

4. **安全测试**
   - 模糊测试 (Fuzz Testing)
   - 边界值测试

### 长期建议

5. **架构改进**
   - 考虑使用更安全的字符串类型
   - 添加内存安全断言
   - 引入形式化验证

## 结论

| 类别 | 评估 |
|------|------|
| 整体风险等级 | **中** |
| 高风险点 | 1 处 (Format 字符串管理) |
| 中风险点 | 4 处 (URI 验证、HAL 校验、空指针、整数溢出) |
| 低风险点 | 1 处 |
| 建议 | 优先修复 Format 字符串相关问题 |

**备注**: 本评审基于静态代码分析，建议在实际部署前进行动态安全测试。

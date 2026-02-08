# 安全风险评审

## 评审概述

本文档对 ImageEffect 模块进行安全风险评估，基于代码审计和架构分析，识别潜在的安全漏洞、攻击面和信任边界，并提供修复建议。

**评审范围**：
- C API 接口层（frameworks/native/capi/）
- 核心引擎层（frameworks/native/effect/）
- 滤镜实现层（frameworks/native/efilter/）
- 内存管理模块（frameworks/native/effect/manager/memory_manager/）
- 外部扩展加载（frameworks/native/effect/base/external_loader.cpp）

**评审方法**：
- 静态代码审计
- 威胁模型分析
- 输入验证检查
- 内存安全检查

**评审时间**：2024-02-06

## 攻击面分析

### 输入入口

| 入口点 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| `OH_ImageEffect_Create()` | 无参数 | 低 | 无直接输入风险 |
| `OH_ImageEffect_AddFilter()` | 字符串 | 中 | 滤镜名称字符串 |
| `OH_ImageEffect_SetInputPixelmap()` | 指针 | 高 | OH_PixelmapNative 对象 |
| `OH_ImageEffect_SetInputNativeBuffer()` | 指针 | 高 | NativeBuffer 对象 |
| `OH_ImageEffect_SetInputUri()` | 字符串 | 高 | URI 字符串 |
| `OH_ImageEffect_SetInputTexture()` | 结构体 | 中 | TextureInfo 结构体 |
| `OH_EffectFilter_SetValue()` | 联合体 | 高 | ImageEffect_Any 联合体 |
| `OH_EffectFilter_Register()` | 回调结构体 | 高 | 自定义回调函数 |

### 系统依赖攻击面

| 依赖组件 | 潜在风险 | 说明 |
|----------|----------|------|
| `libimage_effect_ext.so` | 动态库加载 | dlopen 加载外部库 |
| OpenGL/EGL | 图形驱动 | GPU 渲染上下文 |
| image_framework | 图像解析 | PixelMap 数据处理 |
| file system | URI 解析 | 外部资源访问 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           信任边界                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌────────────────────────────────────────────────────────────────┐     │
│   │                    可信区域（可信代码）                          │     │
│   │   ├── frameworks/native/capi/  (C API 实现)                    │     │
│   │   ├── frameworks/native/effect/ (核心引擎)                      │     │
│   │   ├── frameworks/native/efilter/ (滤镜实现)                     │     │
│   │   └── interfaces/inner_api/ (内部 API)                           │     │
│   └────────────────────────────────────────────────────────────────┘     │
│                              │                                           │
│                    边界：EFFECT_EXPORT 导出符号                          │
│                              │                                           │
│   ┌────────────────────────────────────────────────────────────────┐     │
│   │                   半可信区域（第三方依赖）                        │     │
│   │   ├── skia (Skia 图形库)                                        │     │
│   │   ├── EGL/GLES (OpenGL 驱动)                                    │     │
│   │   └── libexif (EXIF 解析)                                        │     │
│   └────────────────────────────────────────────────────────────────┘     │
│                              │                                           │
│                    边界：系统调用接口                                      │
│                              │                                           │
│   ┌────────────────────────────────────────────────────────────────┐     │
│   │                   不可信区域（外部输入）                          │     │
│   │   ├── 应用层 JS 参数                                            │     │
│   │   ├── URI 指向的外部资源                                        │     │
│   │   └── 自定义滤镜回调                                            │     │
│   └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## 可被利用点

### 风险 1：动态库加载路径验证不足

**严重程度**：高

**证据文件**：`frameworks/native/effect/base/external_loader.cpp:39`

**证据代码**：
```cpp
void* handle = dlopen("libimage_effect_ext.so", RTLD_NOW);
if (handle == nullptr) {
    // 仅有错误日志，无路径验证
    IMG_LOGE("dlopen libimage_effect_ext.so failed, error: %{public}s", dlerror());
}
```

**问题描述**：
- 动态库路径为硬编码 `libimage_effect_ext.so`
- 未验证库文件签名或来源可信度
- 攻击者可替换系统库路径中的同名文件

**触发条件**：
1. 攻击者获得 system 分区的写权限
2. 替换 `/system/lib64/libimage_effect_ext.so` 为恶意库
3. 调用 `OH_ImageEffect_Start()` 触发加载

**影响**：
- 恶意代码以 root 权限执行
- 提权攻击
- 数据窃取

**修复建议**：
1. 添加库文件签名验证机制
2. 使用安全路径白名单
3. 加载前验证文件完整性（hash/MD5）

### 风险 2：URI 路径遍历未防护

**严重程度**：高

**证据文件**：`frameworks/native/capi/image_effect.cpp:455`

**证据代码**：
```cpp
ImageEffect_ErrorCode OH_ImageEffect_SetInputUri(OH_ImageEffect* effect, const char* uri) {
    CHECK_AND_RETURN_RET_LOG(effect != nullptr && uri != nullptr, 
                              ERR_INPUT_NULL, "uri is null");
    // 无路径规范化检查
    // 无恶意路径检测
    return SetInputUriInner(effect, uri);
}
```

**问题描述**：
- URI 参数仅有空值检查
- 未对路径遍历攻击（`../`）进行过滤
- 未限制可访问的目录范围

**触发条件**：
1. 应用传入恶意构造的 URI：`file:///../../etc/passwd`
2. 调用 `OH_ImageEffect_Start()` 处理该 URI
3. 访问系统敏感文件

**影响**：
- 读取系统敏感文件
- 绕过沙箱限制
- 信息泄露

**修复建议**：
1. 在 `SetInputUriInner()` 中添加路径规范化（realpath）
2. 检查规范化后的路径是否在允许目录内
3. 拒绝包含 `..` 的路径

### 风险 3：ImageEffect_Any 联合体类型混淆

**严重程度**：中

**证据文件**：`interfaces/kits/native/image_effect_filter.h:156`

**证据代码**：
```c
typedef struct {
    ImageEffect_DataType type;
    union {
        int32_t int32Value;
        float floatValue;
        double doubleValue;
        bool boolValue;
        void* ptrValue;
    } value;
} ImageEffect_Any;
```

**问题描述**：
- 联合体支持 `void* ptrValue` 类型
- JS 层传入的指针未验证有效性
- 类型字段可被篡改

**触发条件**：
1. JS 层构造恶意的 ImageEffect_Any 结构体
2. type 设置为 `EFFECT_DATA_TYPE_POINTER`
3. value.ptrValue 指向任意内存地址
4. C++ 层直接解引用该指针

**影响**：
- 内存越界访问
- 程序崩溃
- 潜在的代码执行

**修复建议**：
1. 对 ptrValue 类型添加有效性验证
2. 仅允许特定类型的指针（如 ImageEffect_Buffer*）
3. 指针解引用前验证地址空间范围

### 风险 4：滤镜名称缓冲区溢出风险

**严重程度**：中

**证据文件**：`frameworks/native/capi/image_effect_filter.cpp:89`

**证据代码**：
```cpp
ImageEffect_ErrorCode OH_EffectFilter_Create(const char* filterName) {
    CHECK_AND_RETURN_RET_LOG(filterName != nullptr, 
                              ERR_INPUT_NULL, "filterName is null");
    CHECK_AND_RETURN_RET_LOG(strlen(filterName) <= MAX_CHAR_LEN,
                              ERR_PARAM_INVALID, "filterName too long");
    // 后续使用 strcpy 或类似函数复制
}
```

**问题描述**：
- 仅有长度检查，无格式验证
- 滤镜名称用于后续动态库符号查找
- 可能存在格式化字符串漏洞

**触发条件**：
1. 构造超长滤镜名称（接近 MAX_CHAR_LEN）
2. 名称包含特殊字符（如 `%s`, `%n`）
3. 触发潜在的格式化字符串处理

**影响**：
- 缓冲区溢出（如果内部使用 sprintf 等）
- 拒绝服务
- 信息泄露

**修复建议**：
1. 明确允许的字符集（字母、数字、下划线）
2. 使用安全字符串函数（strlcpy/snprintf）
3. 拒绝包含特殊字符的名称

### 风险 5：内存分配缺乏上限保护

**严重程度**：低

**证据文件**：`frameworks/native/effect/manager/memory_manager/effect_memory.cpp:52`

**证据代码**：
```cpp
ImageEffect_ErrorCode Allocate(size_t size, EffectMemory** memory) {
    CHECK_AND_RETURN_RET_LOG(size > 0 && size <= MAX_RAM_SIZE,
                              ERR_PARAM_INVALID, "size invalid");
    void* ptr = malloc(size);
    if (ptr == nullptr) {
        return ERR_ALLOC_MEMORY_FAIL;
    }
    // ...
}
```

**问题描述**：
- 单次分配有大小限制
- 未限制总内存使用量
- 恶意应用可多次触发大内存分配

**触发条件**：
1. 多次调用 `OH_ImageEffect_SetInputPixelmap()` 设置大图像
2. 每次触发新 buffer 分配
3. 总内存使用超过系统限制

**影响**：
- 内存耗尽
- 拒绝服务
- 系统不稳定

**修复建议**：
1. 添加进程级内存使用计数器
2. 设置总内存配额
3. 超限后拒绝新分配请求

### 风险 6：回调函数未验证调用来源

**严重程度**：中

**证据文件**：`frameworks/native/efilter/custom/filter_delegate.cpp`

**证据代码**：
```cpp
ImageEffect_ErrorCode FilterDelegate::InvokeRender(EffectBuffer* buffer) {
    if (delegate_->render != nullptr) {
        // 直接调用用户提供的回调，无来源验证
        return delegate_->render(filter_, buffer);
    }
}
```

**问题描述**：
- 自定义滤镜回调直接执行
- 未验证回调函数的来源可信度
- 恶意回调可访问内部数据结构

**触发条件**：
1. 应用注册恶意自定义滤镜
2. 回调中访问 EffectBuffer 内部指针
3. 修改或泄露处理中的图像数据

**影响**：
- 数据篡改
- 信息泄露
- 绕过安全检查

**修复建议**：
1. 添加回调函数来源验证（如签名验证）
2. 在回调执行前进行参数校验
3. 限制回调可访问的内存范围

## 安全机制评估

### 已实现的安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| 输入空值检查 | CHECK_AND_RETURN_RET_LOG | 有效 |
| 字符串长度限制 | MAX_CHAR_LEN | 有效 |
| 内存大小限制 | MAX_RAM_SIZE | 有效 |
| 安全内存拷贝 | memcpy_s | 有效 |
| 错误码体系 | ErrorCode | 有效 |
| 动态库错误处理 | dlopen/dlsym 错误检查 | 部分有效 |
| 整数溢出检测 | sanitize config | 有效（编译时） |

### 未实现的安全机制

| 机制 | 风险等级 | 建议实现 |
|------|----------|----------|
| 路径规范化 | 高 | realpath + 白名单 |
| 库文件签名验证 | 高 | hash/MD5 校验 |
| 回调来源验证 | 中 | 签名/权限检查 |
| 内存配额限制 | 中 | 进程级计数器 |
| 类型混淆防护 | 中 | 指针类型标记 |
| 格式化字符串过滤 | 低 | 字符白名单 |

## 安全最佳实践

### 对于应用开发者

1. **验证输入源**：确保传入 ImageEffect 的 URI 来源可信
2. **限制图像尺寸**：避免处理超大分辨率图像
3. **复用实例**：避免频繁创建销毁 ImageEffect 实例
4. **注册前验证**：自定义回调中执行参数校验

### 对于模块开发者

1. **防御性编程**：所有外部输入必须经过验证
2. **最小权限**：仅申请必要的系统能力
3. **安全编码**：使用安全版本的内存函数（memcpy_s）
4. **错误处理**：失败时安全回滚，不泄露敏感信息

## 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview.md](./00_Overview.md) | 项目概览 |
| [01_N-API_Reference.md](./01_N-API_Reference.md) | N-API 接口 |
| [02_Architecture.md](./02_Architecture.md) | 内部架构 |
| [03_Build_and_Targets.md](./03_Build_and_Targets.md) | 构建配置 |

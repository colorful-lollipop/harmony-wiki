# C NDK API 文档

## 目的

本文档详细说明 ability_base 组件提供的 C NDK API，这是面向 NDK 开发者的 C 语言接口，主要用于 Want 的创建和操作。C API 在 C++ API 之上封装，提供与 JavaScript 层兼容的接口。

**适用范围**:
- OpenHarmony 标准系统（standard system type）
- API 版本：15+
- 组件版本：3.1
- NDK 接口标签：`ndk`

---

## 1. C API 概览

### 1.1 模块信息

| 属性 | 值 |
|------|-----|
| **头文件** | `interfaces/kits/c/cwant/include/want.h` |
| **实现文件** | `interfaces/kits/c/cwant/src/want.cpp` |
| **所属库** | `libability_base_want.so` |
| **输出路径** | `/system/lib/libability_base_want.so` |
| **API 前缀** | `OH_AbilityBase_*` |

**证据位置**: `interfaces/kits/c/cwant/include/want.h`

### 1.2 核心数据结构

#### AbilityBase_Want

**定义**:
```c
/**
 * @brief Want 结构体，用于 Ability 间通信
 * @since 15
 */
typedef struct AbilityBase_Want AbilityBase_Want;
```

**说明**:
- `AbilityBase_Want` 是 Want 的 C 语言封装
- 内部包含 `AAFwk::Want` C++ 对象
- 所有 Want 操作通过该结构体的指针进行

**证据位置**: `interfaces/kits/c/cwant/include/want.h:35`

### 1.3 内存管理

| 操作 | 函数 | 说明 |
|------|------|------|
| 创建 | `OH_AbilityBase_CreateWant()` | 分配并初始化 Want |
| 销毁 | `OH_AbilityBase_DestroyWant()` | 释放 Want 内存 |

**重要**: C API 不自动管理内存，调用者必须显式调用 `Destroy` 函数释放资源。

---

## 2. Want 生命周期 API

### 2.1 创建 Want

#### OH_AbilityBase_CreateWant

```c
/**
 * @brief 创建 Want 对象
 * @return 成功返回 Want 指针，失败返回 nullptr
 * @since 15
 */
AbilityBase_Want* OH_AbilityBase_CreateWant(void);
```

**使用示例**:
```c
#include "want.h"

AbilityBase_Want* want = OH_AbilityBase_CreateWant();
if (want == nullptr) {
    // 处理内存分配失败
    return;
}
// 使用 want ...

// 使用后销毁
OH_AbilityBase_DestroyWant(want);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:44`

### 2.2 销毁 Want

#### OH_AbilityBase_DestroyWant

```c
/**
 * @brief 销毁 Want 对象，释放内存
 * @param want 要销毁的 Want 指针
 * @since 15
 */
void OH_AbilityBase_DestroyWant(AbilityBase_Want* want);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `want` | `AbilityBase_Want*` | 要销毁的 Want 指针，可为 nullptr |

**注意**:
- 如果 `want` 为 `nullptr`，函数不执行任何操作
- 销毁后指针变为悬空指针，不应再使用

**证据位置**: `interfaces/kits/c/cwant/include/want.h:52`

---

## 3. Element 操作 API

### 3.1 设置组件名

#### OH_AbilityBase_SetWantElement

```c
/**
 * @brief 设置 Want 的组件名（ElementName）
 * @param want Want 对象指针
 * @param bundleName 应用包名
 * @param abilityName Ability 名称
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantElement(
    AbilityBase_Want* want,
    const char* bundleName,
    const char* abilityName
);
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| `want` | `AbilityBase_Want*` | Want 对象 |
| `bundleName` | `const char*` | 应用包名（如 "com.example.app"） |
| `abilityName` | `const char*` | Ability 名称（如 "MainAbility"） |

**返回值**:
| 值 | 说明 |
|-----|------|
| `0` | 成功 |
| `-1` | 无效参数（want/bundleName/abilityName 为 nullptr） |

**使用示例**:
```c
AbilityBase_Want* want = OH_AbilityBase_CreateWant();
int32_t ret = OH_AbilityBase_SetWantElement(want, "com.example.app", "MainAbility");
if (ret != 0) {
    // 处理错误
}
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:68`

### 3.2 设置模块名

#### OH_AbilityBase_SetWantModuleName

```c
/**
 * @brief 设置 Want 的模块名
 * @param want Want 对象指针
 * @param moduleName 模块名称
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantModuleName(
    AbilityBase_Want* want,
    const char* moduleName
);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:79`

---

## 4. 参数操作 API

### 4.1 布尔参数

#### OH_AbilityBase_SetWantBoolParam

```c
/**
 * @brief 设置布尔类型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param value 参数值
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantBoolParam(
    AbilityBase_Want* want,
    const char* key,
    bool value
);
```

#### OH_AbilityBase_GetWantBoolParam

```c
/**
 * @brief 获取布尔类型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param defaultValue 默认值
 * @return 参数值，如果 key 不存在返回 defaultValue
 * @since 15
 */
bool OH_AbilityBase_GetWantBoolParam(
    AbilityBase_Want* want,
    const char* key,
    bool defaultValue
);
```

**证据位置**:
- Set: `interfaces/kits/c/cwant/include/want.h:94`
- Get: `interfaces/kits/c/cwant/include/want.h:105`

### 4.2 整型参数（32位）

#### OH_AbilityBase_SetWantInt32Param

```c
/**
 * @brief 设置 32 位整型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param value 参数值
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantInt32Param(
    AbilityBase_Want* want,
    const char* key,
    int32_t value
);
```

#### OH_AbilityBase_GetWantInt32Param

```c
/**
 * @brief 获取 32 位整型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param defaultValue 默认值
 * @return 参数值，如果 key 不存在返回 defaultValue
 * @since 15
 */
int32_t OH_AbilityBase_GetWantInt32Param(
    AbilityBase_Want* want,
    const char* key,
    int32_t defaultValue
);
```

**证据位置**:
- Set: `interfaces/kits/c/cwant/include/want.h:118`
- Get: `interfaces/kits/c/cwant/include/want.h:129`

### 4.3 整型参数（64位）

#### OH_AbilityBase_SetWantInt64Param

```c
/**
 * @brief 设置 64 位整型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param value 参数值
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantInt64Param(
    AbilityBase_Want* want,
    const char* key,
    int64_t value
);
```

#### OH_AbilityBase_GetWantInt64Param

```c
/**
 * @brief 获取 64 位整型参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param defaultValue 默认值
 * @return 参数值，如果 key 不存在返回 defaultValue
 * @since 15
 */
int64_t OH_AbilityBase_GetWantInt64Param(
    AbilityBase_Want* want,
    const char* key,
    int64_t defaultValue
);
```

**证据位置**:
- Set: `interfaces/kits/c/cwant/include/want.h:142`
- Get: `interfaces/kits/c/cwant/include/want.h:153`

### 4.4 双精度浮点参数

#### OH_AbilityBase_SetWantDoubleParam

```c
/**
 * @brief 设置双精度浮点参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param value 参数值
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantDoubleParam(
    AbilityBase_Want* want,
    const char* key,
    double value
);
```

#### OH_AbilityBase_GetWantDoubleParam

```c
/**
 * @brief 获取双精度浮点参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param defaultValue 默认值
 * @return 参数值，如果 key 不存在返回 defaultValue
 * @since 15
 */
double OH_AbilityBase_GetWantDoubleParam(
    AbilityBase_Want* want,
    const char* key,
    double defaultValue
);
```

**证据位置**:
- Set: `interfaces/kits/c/cwant/include/want.h:166`
- Get: `interfaces/kits/c/cwant/include/want.h:177`

### 4.5 字符串参数

#### OH_AbilityBase_SetWantCharParam

```c
/**
 * @brief 设置字符串参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param value 参数值（UTF-8 编码）
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantCharParam(
    AbilityBase_Want* want,
    const char* key,
    const char* value
);
```

**注意**: 字符串使用 UTF-8 编码，内部会转换为 C++ std::string。

#### OH_AbilityBase_GetWantCharParam

```c
/**
 * @brief 获取字符串参数
 * @param want Want 对象指针
 * @param key 参数键名
 * @param defaultValue 默认值
 * @return 参数值（UTF-8 编码），如果 key 不存在返回 defaultValue
 *         返回值指向内部缓冲区，不需要释放
 * @since 15
 */
const char* OH_AbilityBase_GetWantCharParam(
    AbilityBase_Want* want,
    const char* key,
    const char* defaultValue
);
```

**重要**:
- 返回的字符串指针指向 Want 内部的缓冲区
- 不需要调用者释放
- 在 Want 销毁后指针变为无效

**证据位置**:
- Set: `interfaces/kits/c/cwant/include/want.h:191`
- Get: `interfaces/kits/c/cwant/include/want.h:204`

---

## 5. URI 操作 API

### 5.1 设置 URI

#### OH_AbilityBase_SetWantUri

```c
/**
 * @brief 设置 Want 的 URI
 * @param want Want 对象指针
 * @param uri URI 字符串（UTF-8 编码）
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantUri(
    AbilityBase_Want* want,
    const char* uri
);
```

**使用示例**:
```c
AbilityBase_Want* want = OH_AbilityBase_CreateWant();
OH_AbilityBase_SetWantUri(want, "https://example.com/resource");
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:218`

### 5.2 获取 URI

#### OH_AbilityBase_GetWantUri

```c
/**
 * @brief 获取 Want 的 URI
 * @param want Want 对象指针
 * @return URI 字符串，如果未设置返回空字符串
 *         返回值指向内部缓冲区，不需要释放
 * @since 15
 */
const char* OH_AbilityBase_GetWantUri(AbilityBase_Want* want);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:228`

---

## 6. 标志和实体 API

### 6.1 设置 Action

#### OH_AbilityBase_SetWantAction

```c
/**
 * @brief 设置 Want 的 Action
 * @param want Want 对象指针
 * @param action Action 字符串（如 "ohos.action.home"）
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_SetWantAction(
    AbilityBase_Want* want,
    const char* action
);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:241`

### 6.2 添加 Entity

#### OH_AbilityBase_AddWantEntity

```c
/**
 * @brief 添加 Want 的 Entity
 * @param want Want 对象指针
 * @param entity Entity 字符串（如 "entity.system.home"）
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_AddWantEntity(
    AbilityBase_Want* want,
    const char* entity
);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:252`

### 6.3 添加 Flags

#### OH_AbilityBase_AddWantFlags

```c
/**
 * @brief 添加 Want 的 Flags
 * @param want Want 对象指针
 * @param flags 标志位（如 ABILITY_FLAG_ABILITY_FORWARD_RESULT）
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_AddWantFlags(
    AbilityBase_Want* want,
    uint32_t flags
);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:263`

---

## 7. 文件描述符 API

### 7.1 添加文件描述符

#### OH_AbilityBase_AddWantFd

```c
/**
 * @brief 向 Want 添加文件描述符
 * @param want Want 对象指针
 * @param fd 文件描述符
 * @return 成功返回 0，失败返回错误码
 * @since 15
 */
int32_t OH_AbilityBase_AddWantFd(
    AbilityBase_Want* want,
    int32_t fd
);
```

**使用场景**:
- 跨进程传递文件访问权限
- 传递大文件数据（避免内存拷贝）

**安全注意**:
- FD 权限在系统服务端验证
- 应用不应依赖 FD 的持久性

**证据位置**: `interfaces/kits/c/cwant/include/want.h:277`

### 7.2 获取文件描述符

#### OH_AbilityBase_GetWantFd

```c
/**
 * @brief 从 Want 获取文件描述符
 * @param want Want 对象指针
 * @param fdName 文件描述符名称（key）
 * @return 文件描述符，如果不存在返回 -1
 * @since 15
 */
int32_t OH_AbilityBase_GetWantFd(
    AbilityBase_Want* want,
    const char* fdName
);
```

**证据位置**: `interfaces/kits/c/cwant/include/want.h:288`

---

## 8. 完整使用示例

### 8.1 显式启动 Ability

```c
#include "want.h"
#include <stdio.h>

void startExplicitAbility() {
    // 创建 Want
    AbilityBase_Want* want = OH_AbilityBase_CreateWant();
    if (want == nullptr) {
        printf("Failed to create want\n");
        return;
    }

    // 设置目标组件（显式启动）
    int32_t ret = OH_AbilityBase_SetWantElement(want, "com.example.app", "MainAbility");
    if (ret != 0) {
        printf("Failed to set element\n");
        OH_AbilityBase_DestroyWant(want);
        return;
    }

    // 设置模块名（可选）
    OH_AbilityBase_SetWantModuleName(want, "entry");

    // 添加参数
    OH_AbilityBase_SetWantInt32Param(want, "userId", 12345);
    OH_AbilityBase_SetWantCharParam(want, "userName", "John");
    OH_AbilityBase_SetWantBoolParam(want, "isGuest", false);

    // 启动 Ability（调用系统 API）
    // ...

    // 清理
    OH_AbilityBase_DestroyWant(want);
}
```

### 8.2 隐式启动 Ability

```c
#include "want.h"
#include <stdio.h>

void startImplicitAbility() {
    AbilityBase_Want* want = OH_AbilityBase_CreateWant();
    if (want == nullptr) {
        return;
    }

    // 设置 Action（隐式启动）
    OH_AbilityBase_SetWantAction(want, "ohos.action.view");

    // 添加 Entity
    OH_AbilityBase_AddWantEntity(want, "entity.video");

    // 设置 URI
    OH_AbilityBase_SetWantUri(want, "https://example.com/video.mp4");

    // 添加 Flags
    OH_AbilityBase_AddWantFlags(want, ABILITY_FLAG_ABILITY_NEW_MISSION);

    // 启动 Ability（调用系统 API）
    // ...

    // 清理
    OH_AbilityBase_DestroyWant(want);
}
```

### 8.3 带文件描述符的 Want

```c
#include "want.h"
#include <fcntl.h>
#include <stdio.h>

void startAbilityWithFd() {
    AbilityBase_Want* want = OH_AbilityBase_CreateWant();
    if (want == nullptr) {
        return;
    }

    // 打开文件
    int fd = open("/path/to/file.txt", O_RDONLY);
    if (fd < 0) {
        printf("Failed to open file\n");
        OH_AbilityBase_DestroyWant(want);
        return;
    }

    // 设置目标组件
    OH_AbilityBase_SetWantElement(want, "com.example.app", "FileViewerAbility");

    // 添加文件描述符到 Want
    int32_t ret = OH_AbilityBase_AddWantFd(want, fd);
    if (ret != 0) {
        printf("Failed to add fd\n");
        close(fd);
        OH_AbilityBase_DestroyWant(want);
        return;
    }

    // 添加其他参数
    OH_AbilityBase_SetWantCharParam(want, "fileName", "file.txt");

    // 启动 Ability（调用系统 API）
    // 目标 Ability 可以通过 OH_AbilityBase_GetWantFd 获取 FD

    // 清理
    close(fd);  // 应用仍需关闭自己的 FD 副本
    OH_AbilityBase_DestroyWant(want);
}
```

---

## 9. 错误处理

### 9.1 错误码定义

**头文件**: `interfaces/kits/c/common/ability_base_common.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ABILITY_BASE_ERROR_CODE_NO_ERROR` | 0 | 成功 |
| `ABILITY_BASE_ERROR_CODE_PARAM_INVALID` | -1 | 无效参数 |
| `ABILITY_BASE_ERROR_CODE_PARAM_NULL` | -2 | 空指针参数 |
| `ABILITY_BASE_ERROR_CODE_MEMORY_OPERATION_FAILED` | -3 | 内存操作失败 |

### 9.2 错误处理模式

```c
AbilityBase_Want* want = OH_AbilityBase_CreateWant();
if (want == nullptr) {
    // 内存分配失败
    return;
}

int32_t ret = OH_AbilityBase_SetWantElement(want, bundleName, abilityName);
switch (ret) {
    case ABILITY_BASE_ERROR_CODE_NO_ERROR:
        // 成功
        break;
    case ABILITY_BASE_ERROR_CODE_PARAM_INVALID:
        // 参数无效（如空字符串）
        break;
    case ABILITY_BASE_ERROR_CODE_PARAM_NULL:
        // 空指针
        break;
    default:
        // 其他错误
        break;
}
```

---

## 10. 与 C++ API 的映射

| C API | C++ API | 说明 |
|-------|---------|------|
| `OH_AbilityBase_CreateWant()` | `new Want()` | 创建对象 |
| `OH_AbilityBase_DestroyWant()` | `delete want` | 销毁对象 |
| `OH_AbilityBase_SetWantElement()` | `want.SetElementName()` | 设置组件名 |
| `OH_AbilityBase_SetWantInt32Param()` | `want.SetParam(key, int)` | 设置整型参数 |
| `OH_AbilityBase_SetWantCharParam()` | `want.SetParam(key, string)` | 设置字符串参数 |
| `OH_AbilityBase_SetWantUri()` | `want.SetUri()` | 设置 URI |

**证据位置**: `interfaces/kits/c/cwant/src/want.cpp` - C API 实现调用 C++ API

---

## 11. 线程安全

| 函数 | 线程安全 | 说明 |
|------|----------|------|
| `OH_AbilityBase_CreateWant()` | ✅ 是 | 可并发创建 |
| `OH_AbilityBase_DestroyWant()` | ❌ 否 | 同一 Want 不能并发销毁 |
| `OH_AbilityBase_Set*Param()` | ❌ 否 | 同一 Want 不能并发修改 |
| `OH_AbilityBase_Get*Param()` | ✅ 是 | 可并发读取（无修改时） |

**建议**: 每个线程使用独立的 Want 对象，避免共享。

---

## 12. 性能考虑

### 12.1 内存分配

- C API 内部使用 C++ 容器（std::string, std::map）
- 字符串参数会复制到内部缓冲区
- 建议在设置大量参数前预分配 Want

### 12.2 字符串编码

- 输入字符串必须是 UTF-8 编码
- 内部会自动转换编码
- 避免频繁设置/获取字符串参数

### 12.3 FD 传递

- FD 传递涉及内核操作，有一定开销
- 适合传递大文件或需要共享访问的场景
- 小数据建议直接使用字符串或二进制参数

---

## 13. 安全注意事项

### 13.1 输入验证

- 所有字符串参数都会被内部验证
- 过长的字符串可能被截断
- 特殊字符会被转义

### 13.2 内存安全

- 必须调用 `OH_AbilityBase_DestroyWant()` 释放资源
- 使用 valgrind 或 AddressSanitizer 检测内存泄漏
- 避免 Use-After-Free

### 13.3 FD 安全

- FD 在 IPC 传输后可能失效
- 系统服务端会验证 FD 权限
- 不要依赖 FD 的持久性

---

## 14. 构建配置

### 14.1 链接库

**GN 依赖**:
```gn
external_deps += [
    "ability_base:ability_base_want",
]
```

**链接器参数**:
```
-lability_base_want
```

### 14.2 头文件路径

```
interfaces/kits/c/cwant/include/
interfaces/kits/c/common/
```

### 14.3 Sanitize 选项

`ability_base_want` target 启用了严格的安全检查:

```gn
sanitize = {
    integer_overflow = true
    ubsan = true
    boundary_sanitize = true
    cfi = true
    cfi_cross_dso = true
    cfi_vcall_icall_only = true
}
```

**证据位置**: `BUILD.gn:424-433`

---

## 相关跳转

- 🏠 **项目概览**：[index.md](index.md)
- 📁 **目录结构**：[01_Directory_Structure.md](01_Directory_Structure.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- 🔌 **Native C++ API**：[03_Native_CPP_API.md](03_Native_CPP_API.md)
- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)
- 🔒 **安全评审**：[06_Security_Review.md](06_Security_Review.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)

# bounds_checking_function API 差异说明

## 1. 概述

### 1.1 说明

**bounds_checking_function 与上游 libboundscheck 在 API 层面完全一致**，本库**没有**添加、修改或废弃任何 API。

这是"零 Patch"策略的延续 - 所有 OpenHarmony 特有的适配都通过**构建配置**（BUILD.gn）而非**源码修改**实现。

### 1.2 文档目的

本文档旨在：
1. 明确说明 bounds_checking_function **无 API 差异**
2. 介绍 C11 Annex K 标准 API 的使用方式
3. 对比安全函数与标准 C 库函数的差异
4. 提供 OH 推荐的编码规范

---

## 2. API 清单（C11 Annex K 标准）

### 2.1 内存操作类

| 安全函数 | 对应标准函数 | 新增参数 |
|----------|-------------|----------|
| `memcpy_s` | `memcpy` | `destsz` (目标缓冲区大小) |
| `wmemcpy_s` | `wmemcpy` | `destsz` |
| `memmove_s` | `memmove` | `destsz` |
| `wmemmove_s` | `wmemmove` | `destsz` |
| `memset_s` | `memset` | `destsz` |

**函数签名对比**:
```c
// 标准函数
void *memcpy(void *dest, const void *src, size_t n);

// 安全函数
errno_t memcpy_s(void *dest, rsize_t destsz, const void *src, rsize_t n);
```

### 2.2 字符串操作类

| 安全函数 | 对应标准函数 | 新增参数 |
|----------|-------------|----------|
| `strcpy_s` | `strcpy` | `destsz` |
| `wcscpy_s` | `wcscpy` | `destsz` |
| `strncpy_s` | `strncpy` | `destsz` |
| `wcsncpy_s` | `wcsncpy` | `destsz` |
| `strcat_s` | `strcat` | `destsz` |
| `wcscat_s` | `wcscat` | `destsz` |
| `strncat_s` | `strncat` | `destsz` |
| `wcsncat_s` | `wcsncat` | `destsz` |
| `strtok_s` | `strtok` | `destsz`, `context` |
| `wcstok_s` | `wcstok` | `destsz`, `context` |

**关键改进 - strtok_s**:
```c
// 标准 strtok - 非线程安全，使用静态状态
char *strtok(char *str, const char *delim);

// 安全 strtok_s - 线程安全，显式状态
char *strtok_s(char *str, rsize_t *strsz, const char *delim, char **context);
```

### 2.3 格式化输出类

| 安全函数 | 对应标准函数 | 新增参数 |
|----------|-------------|----------|
| `sprintf_s` | `sprintf` | `destsz` |
| `swprintf_s` | `swprintf` | `destsz` |
| `snprintf_s` | `snprintf` | `destsz` |
| `vsprintf_s` | `vsprintf` | `destsz` |
| `vsnprintf_s` | `vsnprintf` | `destsz` |
| `vswprintf_s` | `vswprintf` | `destsz` |

**格式化输入类的特殊参数**:
```c
// 标准函数
int sscanf(const char *str, const char *format, ...);

// 安全函数 - 每个 %s %c %[ 都需要对应的大小参数
int sscanf_s(const char *str, const char *format, ...);
// 示例: sscanf_s(input, "%s", buffer, sizeof(buffer));
```

### 2.4 格式化输入类

| 安全函数 | 对应标准函数 | 特殊说明 |
|----------|-------------|----------|
| `scanf_s` | `scanf` | 字符串参数需带大小 |
| `wscanf_s` | `wscanf` | 字符串参数需带大小 |
| `vscanf_s` | `vscanf` | 字符串参数需带大小 |
| `vwscanf_s` | `vwscanf` | 字符串参数需带大小 |
| `sscanf_s` | `sscanf` | 字符串参数需带大小 |
| `swscanf_s` | `swscanf` | 字符串参数需带大小 |
| `vsscanf_s` | `vsscanf` | 字符串参数需带大小 |
| `vswscanf_s` | `vswscanf` | 字符串参数需带大小 |
| `fscanf_s` | `fscanf` | 字符串参数需带大小 |
| `fwscanf_s` | `fwscanf` | 字符串参数需带大小 |
| `vfscanf_s` | `vfscanf` | 字符串参数需带大小 |
| `vfwscanf_s` | `vfwscanf` | 字符串参数需带大小 |

### 2.5 输入类

| 安全函数 | 对应标准函数 | 新增参数 |
|----------|-------------|----------|
| `gets_s` | `gets` | `destsz` |

**注意**: C11 已废弃 `gets()`，`gets_s()` 是其安全替代方案。

---

## 3. 错误码定义

### 3.1 标准错误码

| 错误码 | 值 | 说明 |
|--------|----|------|
| `EOK` | 0 | 操作成功 |
| `EINVAL` | 22 | 无效参数（空指针等） |
| `ERANGE` | 34 | 范围错误（缓冲区太小） |

### 3.2 扩展错误码

bounds_checking_function 增加了额外的错误码，用于更精确的错误诊断：

| 错误码 | 值 | 说明 |
|--------|----|------|
| `EINVAL_AND_RESET` | 150 | 无效参数，目标缓冲区已重置 |
| `ERANGE_AND_RESET` | 162 | 范围错误，目标缓冲区已重置 |
| `EOVERLAP_AND_RESET` | 182 | 检测到缓冲区重叠，目标缓冲区已重置 |

**注意**: 这些扩展错误码是华为实现特有的，严格来说**不是** C11 Annex K 标准的一部分，但对调试非常有用。

---

## 4. 安全函数与标准函数的关键差异

### 4.1 参数检查

```c
// 标准函数 - 无边界检查，可能导致溢出
char dest[10];
char src[] = "This is a very long string";
strcpy(dest, src);  // 溢出！未定义行为

// 安全函数 - 边界检查，返回错误码
char dest[10];
char src[] = "This is a very long string";
errno_t ret = strcpy_s(dest, sizeof(dest), src);
// ret == ERANGE (34) - 目标缓冲区太小
```

### 4.2 返回值

| 类型 | 标准函数 | 安全函数 |
|------|----------|----------|
| 成功 | 返回结果指针 | 返回 EOK (0) |
| 失败 | 未定义/返回 NULL | 返回 errno_t 错误码 |

### 4.3 空终止保证

```c
// strncpy - 不保证空终止
char dest[5];
strncpy(dest, "hello", 5);
// dest = "hello" (无空终止符！)

// strncpy_s - 保证空终止
char dest[5];
strncpy_s(dest, sizeof(dest), "hello", 5);
// dest = "hell\0" (强制空终止)
```

### 4.4 重叠处理

```c
char buffer[] = "abcdef";

// memcpy - 重叠区域行为未定义
memcpy(buffer + 2, buffer, 4);  // 危险！

// memmove - 正确处理重叠
memmove(buffer + 2, buffer, 4);  // 安全

// memcpy_s - 检测到重叠时返回错误
errno_t ret = memcpy_s(buffer + 2, 4, buffer, 4);
// ret == EOVERLAP_AND_RESET (182)
```

---

## 5. OpenHarmony 编码规范

### 5.1 强制规则

#### ✅ 必须遵循

1. **所有新代码必须使用安全函数**
2. **禁止使用以下不安全函数**:
   - `strcpy`, `strcat`, `sprintf`, `vsprintf`, `gets`
   - `wcscpy`, `wcscat`, `swprintf`, `vswprintf`
   - `memcpy`, `memmove`, `memset`（除非性能关键路径且已验证安全）

3. **必须检查返回值**:
   ```cpp
   // ✅ 正确
   if (strcpy_s(dest, sizeof(dest), src) != EOK) {
       // 错误处理
   }
   
   // ❌ 错误 - 忽略返回值
   strcpy_s(dest, sizeof(dest), src);
   ```

### 5.2 推荐模式

#### 字符串操作

```cpp
#include "securec.h"
#include "hilog/log.h"

bool SafeStringCopy(char *dest, size_t destSize, const char *src) {
    if (dest == nullptr || src == nullptr) {
        HILOG_ERROR("Null pointer");
        return false;
    }
    
    errno_t ret = strcpy_s(dest, destSize, src);
    if (ret != EOK) {
        HILOG_ERROR("String copy failed: %d", ret);
        return false;
    }
    
    return true;
}
```

#### 格式化输出

```cpp
bool SafeFormat(char *dest, size_t destSize, const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    int ret = vsnprintf_s(dest, destSize, destSize - 1, format, args);
    va_end(args);
    
    if (ret < 0) {
        HILOG_ERROR("Format failed");
        return false;
    }
    
    return true;
}
```

#### 字符串分割

```cpp
void ParseTokens(char *str, size_t strSize, const char *delim) {
    char *context = nullptr;
    char *token = strtok_s(str, &strSize, delim, &context);
    
    while (token != nullptr) {
        // 处理 token
        ProcessToken(token);
        
        token = strtok_s(nullptr, &strSize, delim, &context);
    }
}
```

---

## 6. 类型定义

### 6.1 securectype.h

bounds_checking_function 提供了 `securectype.h` 头文件，定义了以下类型：

```c
// 错误码类型
#ifndef errno_t
typedef int errno_t;
#endif

// 大小类型 (限制最大值为 RSIZE_MAX)
typedef size_t rsize_t;

// 约束处理回调函数类型
typedef void (*constraint_handler_t)(const char *msg, void *ptr, errno_t error);
```

### 6.2 约束处理程序

C11 Annex K 定义了约束处理机制：

```c
// 设置自定义约束处理程序
constraint_handler_t set_constraint_handler_s(constraint_handler_t handler);

// 默认处理程序 - 调用 abort()
void abort_handler_s(const char *msg, void *ptr, errno_t error);

// 忽略处理程序 - 仅返回错误码
void ignore_handler_s(const char *msg, void *ptr, errno_t error);
```

**OpenHarmony 默认行为**: 返回错误码，不调用 abort()

---

## 7. 与上游 libboundscheck 的对比

### 7.1 功能一致性

| 项目 | 上游 libboundscheck | bounds_checking_function | 差异 |
|------|---------------------|-------------------------|------|
| API 数量 | 40 | 40 | 无 |
| 函数签名 | 一致 | 一致 | 无 |
| 错误码 | 一致 | 一致 | 无 |
| 行为 | 一致 | 一致 | 无 |

### 7.2 实现细节

bounds_checking_function 在实现层面与上游保持同步：
- 源码文件一一对应
- 函数实现完全相同
- 版权头文件一致

**版本对应关系**:
```
upstream: libboundscheck v1.1.16
     |
     | 直接同步（无修改）
     v
OH: bounds_checking_function 3.1
```

---

## 8. 常见问题

### Q1: 可以直接替换标准 C 库函数吗？

**A**: 不可以直接替换，因为参数不同。需要修改调用代码：

```cpp
// 旧代码
strcpy(dest, src);

// 新代码
strcpy_s(dest, sizeof(dest), src);
```

### Q2: sizeof(dest) 和 strlen(dest) 用哪个？

**A**: 使用 `sizeof(dest)`（数组）或明确的大小值。不要使用 `strlen()`，因为它返回的是当前字符串长度，不是缓冲区大小。

```cpp
char buffer[100] = "hello";

// ✅ 正确 - 使用缓冲区大小
strcpy_s(buffer, sizeof(buffer), newString);

// ❌ 错误 - 使用当前字符串长度
strcpy_s(buffer, strlen(buffer), newString);  // 缓冲区溢出风险！
```

### Q3: 动态分配的内存怎么用？

**A**: 使用分配时记录的大小：

```cpp
char *buffer = new char[size];
// ...
strcpy_s(buffer, size, source);  // 使用 size，不是 sizeof(buffer)
delete[] buffer;
```

### Q4: 宽字符和多字节字符转换？

**A**: bounds_checking_function 同时提供窄字符和宽字符版本：

```cpp
// 窄字符
char str[100];
strcpy_s(str, sizeof(str), "hello");

// 宽字符
wchar_t wstr[100];
wcscpy_s(wstr, sizeof(wstr) / sizeof(wchar_t), L"hello");
```

---

## 9. 总结

### 9.1 API 一致性确认

bounds_checking_function **与上游 libboundscheck API 完全一致**：
- ✅ 无新增 API
- ✅ 无修改 API
- ✅ 无废弃 API
- ✅ 函数签名相同
- ✅ 行为一致

### 9.2 使用建议

1. **始终使用安全函数** 替代标准 C 库函数
2. **始终检查返回值** 处理可能的错误
3. **使用正确的缓冲区大小** 避免计算错误
4. **参考本文档的编码规范** 编写安全代码

### 9.3 相关文档

- [01_Overview.md](./01_Overview.md) - 库概述
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统使用
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系和使用场景
- [06_Security.md](./06_Security.md) - 安全风险分析

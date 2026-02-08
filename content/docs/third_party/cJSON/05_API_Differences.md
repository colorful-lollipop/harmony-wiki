# API/接口差异

## 概述

本库在 OpenHarmony 中**未对上游 API 进行任何修改**。所有 API 接口与上游版本保持完全一致。

## API 差异总结

| 差异类型 | 状态 | 说明 |
|---------|------|------|
| 新增 API | 无 | OH 未添加任何新 API |
| 修改 API | 无 | OH 未修改任何现有 API |
| 废弃 API | 无 | OH 未废弃任何 API |
| 条件编译 | 无 | 无平台特定的 API 条件编译 |

## 上游 API 兼容性

### 核心 API（完全兼容）

| API | 说明 | OH 状态 |
|-----|------|--------|
| `cJSON_Parse` | JSON 解析 | ✓ 兼容 |
| `cJSON_Print` | JSON 打印 | ✓ 兼容 |
| `cJSON_Create*` | 创建函数 | ✓ 兼容 |
| `cJSON_Get*` | 获取函数 | ✓ 兼容 |
| `cJSON_Is*` | 类型检查 | ✓ 兼容 |
| `cJSON_Delete` | 删除 | ✓ 兼容 |

### 工具 API（LiteOS 模式）

| API | 说明 | OH 状态 |
|-----|------|--------|
| `cJSON_Duplicate` | 复制 | ✓ 兼容 |
| `cJSON_Compare` | 比较 | ✓ 兼容 |
| `cJSON_Minify` | 压缩 | ✓ 兼容 |
| `cJSON_PrintUnformatted` | 无格式打印 | ✓ 兼容 |

## 条件编译说明

### 平台检测宏

cJSON 使用以下宏检测平台：

```c
#if !defined(__WINDOWS__) && (defined(WIN32) || defined(WIN64) || defined(_MSC_VER) || defined(_WIN32))
#define __WINDOWS__
#endif
```

### 符号可见性

```c
#ifdef __WINDOWS__
#define CJSON_PUBLIC(type) type CJSON_STDCALL
#else
#if (defined(__GNUC__) || defined(__SUNPRO_CC) || defined (__SUNPRO_C)) && defined(CJSON_API_VISIBILITY)
#define CJSON_PUBLIC(type) __attribute__((visibility("default"))) type
#else
#define CJSON_PUBLIC(type) type
#endif
#endif
```

**说明**：
- Windows：支持 DLL 导入/导出
- Linux/OH：支持 GCC 可见性属性

## 内存管理 API

### 钩子函数接口

```c
typedef struct cJSON_Hooks {
    void *(CJSON_CDECL *malloc_fn)(size_t sz);
    void (CJSON_CDECL *free_fn)(void *ptr);
} cJSON_Hooks;

CJSON_PUBLIC(void) cJSON_InitHooks(cJSON_Hooks* hooks);
```

**状态**：完全兼容上游接口

### 内部内存函数

```c
CJSON_PUBLIC(void *) cJSON_malloc(size_t size);
CJSON_PUBLIC(void) cJSON_free(void *object);
```

**状态**：完全兼容上游接口

## 版本宏

```c
#define CJSON_VERSION_MAJOR 1
#define CJSON_VERSION_MINOR 7
#define CJSON_VERSION_PATCH 19

CJSON_PUBLIC(const char*) cJSON_Version(void);
```

**注意**：OH 版本号（3.1）与上游版本宏（1.7.19）不一致，这是版本管理差异，不是 API 差异。

## 限制配置

### 嵌套深度限制

```c
#ifndef CJSON_NESTING_LIMIT
#define CJSON_NESTING_LIMIT 1000
#endif
```

**OH 配置**：`CJSON_NESTING_LIMIT=128`

**影响**：
- 解析超过 128 层嵌套的 JSON 会失败
- 这是编译时配置，不影响 API

### 循环引用限制

```c
#ifndef CJSON_CIRCULAR_LIMIT
#define CJSON_CIRCULAR_LIMIT 10000
#endif
```

**状态**：上游配置，未修改

## 头文件引用方式

### OH 标准引用

```c
#include <cjson/cJSON.h>
```

**说明**：
- OH 使用 `<cjson/cJSON.h>` 格式
- 上游默认使用 `"cJSON.h"` 或 `"cjson/cJSON.h"`

### 引用路径

| 环境 | 引用方式 |
|-----|---------|
| OH | `<cjson/cJSON.h>` |
| 上游 CMake | `<cjson/cJSON.h>` |
| 上游单文件 | `"cJSON.h"` |

## 构建配置差异

### 上游 CMake 配置

```cmake
# 可选配置
enable_testing()
option(ENABLE_CJSON_UTILS "Enable cJSON_Utils" OFF)
option(ENABLE_LOCALES "Enable localeconv" ON)
```

### OH GN 配置

| 配置 | OH | 上游 |
|-----|-----|------|
| cJSON_Utils | 仅 lite 模式 | 可选 |
| 嵌套限制 | 128 | 1000 |
| PAC 保护 | 标准模式启用 | 无 |

## 兼容性声明

### 完全兼容的 API 列表

以下 API 在 OH 中与上游 100% 兼容：

**解析和错误处理**：
- `cJSON_Parse`
- `cJSON_ParseWithLength`
- `cJSON_ParseWithOpts`
- `cJSON_ParseWithLengthOpts`
- `cJSON_GetErrorPtr`

**打印和生成**：
- `cJSON_Print`
- `cJSON_PrintUnformatted`
- `cJSON_PrintBuffered`
- `cJSON_PrintPreallocated`
- `cJSON_Minify`

**创建函数**：
- `cJSON_CreateNull`
- `cJSON_CreateTrue` / `cJSON_CreateFalse`
- `cJSON_CreateBool`
- `cJSON_CreateNumber`
- `cJSON_CreateString` / `cJSON_CreateStringReference`
- `cJSON_CreateRaw`
- `cJSON_CreateArray` / `cJSON_CreateArrayReference`
- `cJSON_CreateObject` / `cJSON_CreateObjectReference`

**数组操作**：
- `cJSON_AddItemToArray`
- `cJSON_AddItemReferenceToArray`
- `cJSON_InsertItemInArray`
- `cJSON_DetachItemFromArray`
- `cJSON_DeleteItemFromArray`
- `cJSON_GetArraySize`
- `cJSON_GetArrayItem`

**对象操作**：
- `cJSON_AddItemToObject`
- `cJSON_AddItemToObjectCS`
- `cJSON_AddItemReferenceToObject`
- `cJSON_DetachItemFromObject`
- `cJSON_DetachItemFromObjectCaseSensitive`
- `cJSON_DeleteItemFromObject`
- `cJSON_DeleteItemFromObjectCaseSensitive`
- `cJSON_GetObjectItem`
- `cJSON_GetObjectItemCaseSensitive`
- `cJSON_HasObjectItem`

**修改函数**：
- `cJSON_ReplaceItemViaPointer`
- `cJSON_ReplaceItemInArray`
- `cJSON_ReplaceItemInObject`
- `cJSON_ReplaceItemInObjectCaseSensitive`
- `cJSON_SetNumberHelper`
- `cJSON_SetValuestring`

**类型检查**：
- `cJSON_IsInvalid`
- `cJSON_IsFalse`
- `cJSON_IsTrue`
- `cJSON_IsNull`
- `cJSON_IsNumber`
- `cJSON_IsString`
- `cJSON_IsArray`
- `cJSON_IsObject`
- `cJSON_IsRaw`
- `cJSON_IsBool`

**值获取**：
- `cJSON_GetStringValue`
- `cJSON_GetNumberValue`

**工具函数**：
- `cJSON_Duplicate`
- `cJSON_Compare`
- `cJSON_Version`
- `cJSON_InitHooks`
- `cJSON_Delete`

## 迁移建议

### 从上游迁移到 OH

如果代码基于上游 cJSON 编写，迁移到 OH 时：

1. **无需修改 API 调用**：所有 API 完全兼容
2. **更新头文件路径**：确保头文件搜索路径正确
3. **检查嵌套深度**：如需更深嵌套，配置 CJSON_NESTING_LIMIT

### 从 OH 迁移到上游

如果需要将 OH 中的 cJSON 代码迁移到其他平台：

1. **API 无需修改**：所有 API 与上游兼容
2. **移除 BUILD.gn 配置**：使用 CMake 或 Makefile
3. **检查嵌套限制**：根据需要调整 CJSON_NESTING_LIMIT

## 相关文档

- [01_Overview.md](01_Overview.md)：原始库功能介绍
- [03_Build_Integration.md](03_Build_Integration.md)：构建适配详解
- [04_Usage_in_OH.md](04_Usage_in_OH.md)：使用方式详解

# 04_Inner_API - 内部 API 说明

## 目的

本文档详细介绍 `syscap_codec` 提供的内部 C/C++ API，包括模块接口、依赖方向和稳定性说明。

## 适用范围

- 需要调用内部接口的 Native 开发者
- 需要理解模块依赖关系的架构师

## 接口概览

内部 API 定义在 `interfaces/inner_api/` 目录，编译为动态库 `libsyscap_interface_shared.so`。

### 头文件位置

**主头文件**: `interfaces/inner_api/syscap_interface.h`

### 导出符号

完整导出符号列表见: `libsyscap_interface_shared.versionscript`

```
global:
  ComparePcidString;
  DecodeOsSyscap;
  DecodePrivateSyscap;
  EncodeOsSyscap;
  EncodePrivateSyscap;
  DecodeRpcidToStringFormat;
  FreeCompareError;
```

## API 清单

### 1. 编码接口

#### EncodeOsSyscap

**功能**: 从 `/system/etc/pcid.sc` 读取并编码 OS 系统能力

**声明**: `interfaces/inner_api/syscap_interface.h:46`
```c
bool EncodeOsSyscap(char *output, int len);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| output | char* | 输出缓冲区，必须 >= 128 字节 |
| len | int | 缓冲区长度，必须等于 128 |

**返回值**:
- `true` - 成功
- `false` - 失败

**实现位置**: `interfaces/inner_api/syscap_interface.c:81-108`

**实现逻辑**:
1. 打开 `/system/etc/pcid.sc`
2. 读取前 128 字节（PCIDMain结构）
3. 复制到输出缓冲区

**稳定性**: ⭐⭐⭐ 稳定

---

#### EncodePrivateSyscap

**功能**: 从 `/system/etc/pcid.sc` 读取并编码私有系统能力

**声明**: `interfaces/inner_api/syscap_interface.h:48`
```c
bool EncodePrivateSyscap(char **output, int *outputLen);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| output | char** | 输出字符串指针（动态分配） |
| outputLen | int* | 输出字符串长度 |

**返回值**:
- `true` - 成功
- `false` - 失败

**实现位置**: `interfaces/inner_api/syscap_interface.c:110-155`

**实现逻辑**:
1. 打开 `/system/etc/pcid.sc`
2. 读取文件总长度
3. 计算私有 syscap 长度（总长度 - 128 - 1）
4. 动态分配内存并复制

**注意**: 调用者需要释放 `*output` 内存

**稳定性**: ⭐⭐⭐ 稳定

### 2. 解码接口

#### DecodeOsSyscap

**功能**: 将 OS 系统能力位图解码为字符串数组

**声明**: `interfaces/inner_api/syscap_interface.h:47`
```c
bool DecodeOsSyscap(const char input[PCID_MAIN_BYTES], char (**output)[SINGLE_SYSCAP_LEN], int *outputCnt);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| input | const char[128] | 输入的位图数据（PCIDMain前128字节） |
| output | char(**)[SINGLE_SYSCAP_LEN] | 输出字符串数组（动态分配） |
| outputCnt | int* | 输出数组元素个数 |

**返回值**:
- `true` - 成功
- `false` - 失败

**实现位置**: `interfaces/inner_api/syscap_interface.c:157-204`

**实现逻辑**:
1. 解析位图中的每个 bit
2. 根据 `g_arraySyscap` 映射表查找对应的字符串
3. 动态分配字符串数组
4. 填充字符串

**注意**: 调用者需要释放 `*output` 内存

**稳定性**: ⭐⭐⭐ 稳定

---

#### DecodePrivateSyscap

**功能**: 将私有系统能力字符串解码为数组

**声明**: `interfaces/inner_api/syscap_interface.h:49`
```c
bool DecodePrivateSyscap(char *input, char (**output)[SINGLE_SYSCAP_LEN], int *outputCnt);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| input | char* | 输入的逗号分隔字符串 |
| output | char(**)[SINGLE_SYSCAP_LEN] | 输出字符串数组（动态分配） |
| outputCnt | int* | 输出数组元素个数 |

**返回值**:
- `true` - 成功
- `false` - 失败

**实现位置**: `interfaces/inner_api/syscap_interface.c:221-264`

**实现逻辑**:
1. 统计逗号数量确定元素个数
2. 动态分配数组内存
3. 使用 `strtok` 分割字符串
4. 添加 "SystemCapability." 前缀

**注意**: 调用者需要释放 `*output` 内存

**稳定性**: ⭐⭐⭐ 稳定

### 3. 比较接口

#### ComparePcidString

**功能**: 比较 PCID 字符串和 RPCID 字符串的兼容性

**声明**: `interfaces/inner_api/syscap_interface.h:70`
```c
int32_t ComparePcidString(const char *pcidString, const char *rpcidString, CompareError *result);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| pcidString | const char* | PCID字符串格式 |
| rpcidString | const char* | RPCID字符串格式 |
| result | CompareError* | 比较结果（输出） |

**返回值**:

| 返回值 | 宏定义 | 含义 |
|--------|--------|------|
| 0 | `E_OK` | 比较成功且满足要求 |
| 1 | `E_APIVERSION` | API版本不满足 |
| 2 | `E_SYSCAP` | 缺少系统能力 |
| 3 | - | API版本和系统能力都不满足 |
| -1 | `E_ERROR` | 比较失败 |

**CompareError结构**: `interfaces/inner_api/syscap_interface.h:40-44`
```c
typedef struct CompareErrorMessage {
    char *syscap[MAX_MISS_SYSCAP];  // 缺失的syscap数组（512个）
    uint16_t missSyscapNum;          // 缺失的syscap数量
    uint16_t targetApiVersion;       // 目标API版本（当版本不满足时）
} CompareError;
```

**实现位置**: `interfaces/inner_api/syscap_interface.c:649-689`

**实现逻辑**:
1. 解析 PCID 字符串（32个uint32 + 私有syscap）
2. 解析 RPCID 字符串（32个uint32 + 私有syscap）
3. 比较 API 版本
4. 比较 OS 系统能力位图
5. 比较私有系统能力

**注意**: 调用者需要使用 `FreeCompareError()` 释放结果内存

**稳定性**: ⭐⭐⭐ 稳定

---

#### FreeCompareError

**功能**: 释放 ComparePcidString 返回的结果内存

**声明**: `interfaces/inner_api/syscap_interface.h:71`
```c
int32_t FreeCompareError(CompareError *result);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| result | CompareError* | ComparePcidString的输出结果 |

**返回值**:
- 0 - 成功

**实现位置**: `interfaces/inner_api/syscap_interface.c:691-703`

**实现逻辑**:
1. 遍历 `syscap` 数组，释放每个元素
2. 清零计数器

**稳定性**: ⭐⭐⭐ 稳定

### 4. 工具接口

#### DecodeRpcidToStringFormat

**功能**: 将 RPCID 文件解码为字符串格式

**声明**: `interfaces/inner_api/syscap_interface.h:50`
```c
char *DecodeRpcidToStringFormat(const char *inputFile);
```

**参数**:

| 参数名 | 类型 | 说明 |
|--------|------|------|
| inputFile | const char* | RPCID文件路径 |

**返回值**:
- 成功 - 返回动态分配的字符串
- 失败 - 返回 NULL

**实现位置**: `interfaces/inner_api/syscap_interface.c:478-528`

**实现逻辑**:
1. 检查 RPCID 文件格式
2. 解析为 JSON
3. 转换为字符串格式

**注意**: 调用者需要释放返回的内存

**稳定性**: ⭐⭐ 较稳定（较少使用）

## 模块依赖关系

### 依赖图

```
┌─────────────────────────────────────────────────────────────────┐
│                    内部API层                                     │
│              syscap_interface.c                                  │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  EncodeOsSyscap                                           │ │
│  │  EncodePrivateSyscap                                      │ │
│  │  DecodeOsSyscap                                           │ │
│  │  DecodePrivateSyscap                                      │ │
│  │  ComparePcidString                                        │ │
│  │  DecodeRpcidToStringFormat                                │ │
│  └────────────────────┬──────────────────────────────────────┘ │
└───────────────────────┼─────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ 核心编解码    │ │ 文件操作      │ │ JSON处理     │
│              │ │              │ │              │
│ create_pcid.c│ │ context_tool.│ │ cJSON库      │
│ syscap_tool.c│ │ c            │ │              │
│ (部分函数)    │ │              │ │              │
└──────────────┘ └──────────────┘ └──────────────┘
                        │
                        ▼
               ┌──────────────┐
               │ 系统调用      │
               │              │
               │ open/read/   │
               │ close/malloc │
               └──────────────┘
```

### 依赖说明

| 被依赖模块 | 用途 | 稳定性 |
|------------|------|--------|
| `src/context_tool.c` | 文件读写 | ⭐⭐⭐ 稳定 |
| `src/syscap_tool.c` | RPCID解析 | ⭐⭐⭐ 稳定 |
| `src/create_pcid.c` | PCID结构 | ⭐⭐⭐ 稳定 |
| `cJSON` | JSON解析 | ⭐⭐⭐ 稳定（外部库）|
| `securec` | 安全函数 | ⭐⭐⭐ 稳定（外部库）|

## 接口稳定性说明

### 稳定性等级定义

| 等级 | 符号 | 说明 |
|------|------|------|
| 稳定 | ⭐⭐⭐ | 接口已固化，不会变更 |
| 较稳定 | ⭐⭐ | 接口可能扩展，但保持兼容 |
| 实验性 | ⭐ | 接口可能变更，不建议生产使用 |

### 各接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `EncodeOsSyscap` | ⭐⭐⭐ | 核心接口，已稳定 |
| `EncodePrivateSyscap` | ⭐⭐⭐ | 核心接口，已稳定 |
| `DecodeOsSyscap` | ⭐⭐⭐ | 核心接口，已稳定 |
| `DecodePrivateSyscap` | ⭐⭐⭐ | 核心接口，已稳定 |
| `ComparePcidString` | ⭐⭐⭐ | 核心接口，已稳定 |
| `FreeCompareError` | ⭐⭐⭐ | 核心接口，已稳定 |
| `DecodeRpcidToStringFormat` | ⭐⭐ | 工具接口，较少使用 |

## 使用示例

### 示例1: 查询系统能力

```c
#include "syscap_interface.h"
#include <stdio.h>
#include <stdlib.h>

int main() {
    // 编码OS Syscap
    char osOutput[128];
    if (!EncodeOsSyscap(osOutput, sizeof(osOutput))) {
        printf("Failed to encode OS syscap\n");
        return -1;
    }
    
    // 编码Private Syscap
    char *priOutput = NULL;
    int priLen = 0;
    if (!EncodePrivateSyscap(&priOutput, &priLen)) {
        printf("Failed to encode private syscap\n");
        return -1;
    }
    
    // 解码OS Syscap
    char (*osSyscap)[SINGLE_SYSCAP_LEN] = NULL;
    int osCnt = 0;
    if (!DecodeOsSyscap(osOutput, &osSyscap, &osCnt)) {
        printf("Failed to decode OS syscap\n");
        free(priOutput);
        return -1;
    }
    
    printf("OS Syscap count: %d\n", osCnt);
    for (int i = 0; i < osCnt; i++) {
        printf("  %s\n", osSyscap[i]);
    }
    
    // 解码Private Syscap
    char (*priSyscap)[SINGLE_SYSCAP_LEN] = NULL;
    int priCnt = 0;
    if (!DecodePrivateSyscap(priOutput, &priSyscap, &priCnt)) {
        printf("Failed to decode private syscap\n");
        free(osSyscap);
        free(priOutput);
        return -1;
    }
    
    printf("Private Syscap count: %d\n", priCnt);
    for (int i = 0; i < priCnt; i++) {
        printf("  %s\n", priSyscap[i]);
    }
    
    // 释放资源
    free(osSyscap);
    free(priSyscap);
    free(priOutput);
    
    return 0;
}
```

### 示例2: 比较兼容性

```c
#include "syscap_interface.h"
#include <stdio.h>

int main() {
    // PCID字符串（设备能力）
    const char *pcidStr = "0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0";
    
    // RPCID字符串（应用需求）
    const char *rpcidStr = "0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,SystemCapability.Customization.ConfigPolicy";
    
    CompareError result = {0};
    int32_t ret = ComparePcidString(pcidStr, rpcidStr, &result);
    
    switch (ret) {
        case E_OK:
            printf("Compatible!\n");
            break;
        case E_APIVERSION:
            printf("API version too low, need: %u\n", result.targetApiVersion);
            break;
        case E_SYSCAP:
            printf("Missing syscaps (%u):\n", result.missSyscapNum);
            for (int i = 0; i < result.missSyscapNum; i++) {
                printf("  %s\n", result.syscap[i]);
            }
            break;
        case E_ERROR:
            printf("Compare failed\n");
            break;
        default:
            printf("API version and syscap both not met\n");
            printf("Missing syscaps (%u):\n", result.missSyscapNum);
            for (int i = 0; i < result.missSyscapNum; i++) {
                printf("  %s\n", result.syscap[i]);
            }
            break;
    }
    
    // 释放结果内存
    FreeCompareError(&result);
    
    return ret;
}
```

## 相关跳转

- [项目概览](00_Overview.md) - 系统能力概念
- [架构说明](02_Architecture.md) - 模块依赖关系
- [N-API接口](03_NAPI_Interface.md) - JS层接口
- [GN构建目标](05_GN_Targets.md) - 动态库构建
- [调用链附录](appendix/Callgraphs.md) - 详细调用链

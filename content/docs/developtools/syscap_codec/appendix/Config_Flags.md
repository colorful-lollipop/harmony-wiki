# Appendix B - 配置标志

## 目的

本文档汇总 `syscap_codec` 中使用的关键宏、配置参数和 feature flags。

## 编译时宏定义

### 1. 功能开关宏

#### SYSCAP_DEFINE_EXTERN_ENABLE

**定义位置**: 通过 `cflags` 在 BUILD.gn 中定义

**相关代码**:
```gn
# BUILD.gn:21-25
if (syscap_codec_config_extern_path != "") {
  include_dirs += [ "${root_build_dir}" ]
  cflags = [ "-DSYSCAP_DEFINE_EXTERN_ENABLE" ]
}
```

**用途**: 启用外部 syscap 定义文件支持

**影响代码**:
```c
// src/syscap_tool.c:32-36
#ifdef SYSCAP_DEFINE_EXTERN_ENABLE
#include "syscap_define_custom.h"
#else
#include "syscap_define.h"
#endif
```

**使用场景**: 当需要扩展系统能力定义而不修改主仓库时

---

#### _POSIX_

**定义位置**: `BUILD.gn:47-49`

**相关代码**:
```gn
if (is_mingw) {
  defines += [ "_POSIX_" ]
}
```

**用途**: Windows (MinGW) 环境下使用 POSIX 兼容代码路径

**影响代码**:
```c
// src/context_tool.c:48-58
#ifdef _POSIX_
  if (strlen(inputFile) > PATH_MAX || strncpy_s(path, PATH_MAX, inputFile, strlen(inputFile)) != EOK) {
    // Windows路径处理
  }
#else
  if (strlen(inputFile) > PATH_MAX || realpath(inputFile, path) == NULL) {
    // Unix路径处理
  }
#endif
```

---

### 2. 大小限制宏

#### SINGLE_SYSCAP_LEN

**定义位置**: `include/codec_config/syscap_define.h:21`

```c
#define SINGLE_SYSCAP_LEN (256 + 17)  // 273 bytes
```

**用途**: 单个系统能力字符串的最大长度

**分解**:
- 256: 特征名最大长度
- 17: "SystemCapability." 前缀长度

**使用位置**:
- `SyscapWithNum` 结构体
- 各种字符串缓冲区定义

---

#### OS_SYSCAP_BYTES

**定义位置**: `include/create_pcid.h:21`

```c
#define OS_SYSCAP_BYTES 120
```

**用途**: OS 系统能力位图的字节数

**计算**: 120 bytes = 960 bits，最多支持 960 个 OS syscap

**使用位置**:
- `PCIDMain` 结构体的 `osSyscap` 字段
- 位图操作代码

---

#### PCID_MAIN_BYTES

**定义位置**: `interfaces/inner_api/syscap_interface.h:26`

```c
#define PCID_MAIN_BYTES 128
```

**用途**: PCID 主结构体的字节数

**计算**: sizeof(PCIDMain) = 8 (header) + 120 (osSyscap) = 128

**使用位置**:
- `EncodeOsSyscap()` 的缓冲区大小
- 文件读取偏移量

---

#### MAX_MISS_SYSCAP

**定义位置**: `interfaces/inner_api/syscap_interface.h:25`

```c
#define MAX_MISS_SYSCAP 512
```

**用途**: 比较结果中最多可报告的缺失 syscap 数量

**使用位置**:
- `CompareError` 结构体的 `syscap` 数组大小

---

#### RPCID_OUT_BUFFER / PCID_OUT_BUFFER

**定义位置**: `src/syscap_tool.c:44`

```c
#define RPCID_OUT_BUFFER 32
#define PCID_OUT_BUFFER RPCID_OUT_BUFFER
```

**用途**: 字符串格式转换时的 uint32 缓冲区大小

**说明**: 32个uint32 = 128字节 = PCID主结构大小

---

### 3. 错误码宏

#### E_OK / E_ERROR

**定义位置**: `interfaces/inner_api/syscap_interface.h:29-30`

```c
#define E_ERROR (-1)
#define E_OK 0
```

**用途**: 基础错误码

---

#### E_APIVERSION / E_SYSCAP

**定义位置**: `interfaces/inner_api/syscap_interface.h:31-32`

```c
#define E_APIVERSION 1
#define E_SYSCAP 2
```

**用途**: `ComparePcidString()` 的返回码

| 返回值 | 含义 |
|--------|------|
| 0 (E_OK) | 兼容 |
| 1 (E_APIVERSION) | API版本不满足 |
| 2 (E_SYSCAP) | 缺少系统能力 |
| 3 | 版本和系统能力都不满足 |
| -1 (E_ERROR) | 比较失败 |

---

### 4. 类型定义宏

#### TYPE_FILE / TYPE_STRING

**定义位置**: `include/syscap_tool.h:21-22`

```c
#define TYPE_FILE (0U)
#define TYPE_STRING (1U)
```

**用途**: `ComparePcidWithRpcidString()` 的输入类型

---

### 5. 内部计算宏

#### SYSCAP_PREFIX_LEN

**定义位置**: `src/syscap_tool.c:38`

```c
#define SYSCAP_PREFIX_LEN 17  // "SystemCapability."
```

**用途**: 计算 syscap 字符串前缀长度

---

#### SINGLE_FEAT_LEN

**定义位置**: `src/syscap_tool.c:39`

```c
#define SINGLE_FEAT_LEN (SINGLE_SYSCAP_LEN - SYSCAP_PREFIX_LEN)  // 256
```

**用途**: RPCID 中存储的单个特征名长度（不含前缀）

---

#### UINT8_BIT / INT_BIT

**定义位置**: `src/syscap_tool.c:41-42`

```c
#define UINT8_BIT 8
#define INT_BIT 32
```

**用途**: 位图计算

---

#### U32_TO_STR_MAX_LEN

**定义位置**: `src/syscap_tool.c:46`

```c
#define U32_TO_STR_MAX_LEN 11  // "4294967295" + null
```

**用途**: uint32 转换为字符串的最大长度

---

## 构建配置参数

### config.gni 参数

#### syscap_codec_config_path

**默认值**: `//developtools/syscap_codec/include/codec_config`

**用途**: 指定 syscap_define.h 所在目录

**使用位置**:
```gn
# BUILD.gn:19-20
config("internal") {
  include_dirs = [ "include" ]
  include_dirs += [ syscap_codec_config_path ]
}
```

---

#### syscap_codec_config_extern_path

**默认值**: `""` (空字符串)

**用途**: 指定外部 syscap 定义文件路径

**使用位置**:
```gn
# BUILD.gn:171-189
if (syscap_codec_config_extern_path != "") {
  action("gen_syscap_define_custom") {
    script = "./tools/syscap_config_merge.py"
    args = [
      "--base",
      rebase_path("include/codec_config/syscap_define.h"),
      "--extern",
      rebase_path(syscap_codec_config_extern_path),
      # ...
    ]
  }
}
```

---

## 版本信息

### SYSCAP_VERSION

**定义位置**: `src/main.c:27`

```c
#define SYSCAP_VERSION "2.0.1"
```

**用途**: 命令行工具的版本号

**显示**:
```bash
$ ./syscap_tool --version
syscap_tool v2.0.1
```

---

## 调试宏

### PRINT_ERR

**定义位置**: `include/context_tool.h:24-28`

```c
#define PRINT_ERR(...) \
    do { \
        printf("ERROR: [%s: %d] -> ", __FILE__, __LINE__); \
        printf(__VA_ARGS__); \
    } while (0)
```

**用途**: 统一错误输出格式

**输出示例**:
```
ERROR: [src/context_tool.c: 55] -> get file(/path/to/file) real path failed
```

---

### SafeFree

**定义位置**: `src/common_method.h`

```c
void SafeFree(char *pointer);
```

**实现**: `src/common_method.c:19-24`

```c
void SafeFree(char *pointer)
{
    if ((pointer) != NULL) {
        free(pointer);
    }
}
```

**用途**: 安全的 free 宏，检查 NULL 指针

---

## 配置使用示例

### 扩展系统能力定义

1. 创建外部 syscap 定义文件 `vendor/my_vendor/syscap_extern.h`:

```c
// 在 SYSCAP_BASIC_END = 500 之后添加
VENDOR_CUSTOM_FEATURE1,  // 500
VENDOR_CUSTOM_FEATURE2,  // 501

// 在 g_arraySyscap 中添加对应条目
{"SystemCapability.Vendor.Custom.Feature1", VENDOR_CUSTOM_FEATURE1},
{"SystemCapability.Vendor.Custom.Feature2", VENDOR_CUSTOM_FEATURE2},
```

2. 在产品的 `config.gni` 中配置:

```gn
declare_args() {
  syscap_codec_config_extern_path = "//vendor/my_vendor/syscap_extern.h"
}
```

3. 重新编译

---

## 宏定义汇总表

| 宏名 | 定义位置 | 值 | 用途 |
|------|----------|-----|------|
| SINGLE_SYSCAP_LEN | syscap_define.h:21 | 273 | 最大syscap字符串长度 |
| OS_SYSCAP_BYTES | create_pcid.h:21 | 120 | OS syscap位图字节数 |
| PCID_MAIN_BYTES | syscap_interface.h:26 | 128 | PCID主结构大小 |
| MAX_MISS_SYSCAP | syscap_interface.h:25 | 512 | 最大缺失syscap数 |
| RPCID_OUT_BUFFER | syscap_tool.c:44 | 32 | uint32缓冲区大小 |
| SYSCAP_PREFIX_LEN | syscap_tool.c:38 | 17 | 前缀长度 |
| SINGLE_FEAT_LEN | syscap_tool.c:39 | 256 | 特征名长度 |
| UINT8_BIT | syscap_tool.c:41 | 8 | 字节位数 |
| INT_BIT | syscap_tool.c:42 | 32 | 整数位数 |
| U32_TO_STR_MAX_LEN | syscap_tool.c:46 | 11 | uint32字符串长度 |
| E_ERROR | syscap_interface.h:29 | -1 | 错误码 |
| E_OK | syscap_interface.h:30 | 0 | 成功码 |
| E_APIVERSION | syscap_interface.h:31 | 1 | API版本错误 |
| E_SYSCAP | syscap_interface.h:32 | 2 | Syscap缺失错误 |
| TYPE_FILE | syscap_tool.h:21 | 0 | 文件类型 |
| TYPE_STRING | syscap_tool.h:22 | 1 | 字符串类型 |

## 相关跳转

- [GN构建目标](../05_GN_Targets.md) - 构建配置
- [安全风险分析](../06_Security_Analysis.md) - 配置相关风险
- [常见问题](../07_Troubleshooting.md) - 配置问题排查

# NDK 接口

## 7.1 接口概述

NDK（Native Development Kit）接口为 C/C++ 开发者提供原生编程能力。应用文件服务提供两个 NDK 模块：ohfileshare（文件分享）和 ohfileuri（URI 管理）。NDK 接口编译为动态库，可被 Native 应用直接链接使用。

### 7.1.2 模块列表

| 模块 | 头文件 | 库文件 |
|------|--------|--------|
| FileShare | `oh_file_share.h` | `libohfileshare.so` |
| FileURI | `oh_file_uri.h` | `libohfileuri.so` |

### 7.1.3 系统能力依赖

```c
// FileShare 需要
syscap SystemCapability.FileManagement.AppFileService.FolderAuthorization

// FileURI 需要
syscap SystemCapability.FileManagement.AppFileService
```

## 7.2 FileURI 模块

### 7.2.1 头文件

**文件**：`interfaces/kits/ndk/fileuri/include/oh_file_uri.h`

### 7.2.2 API 列表

**OH_FileUri_GetUriFromPath**：

```c
FileManagement_ErrCode OH_FileUri_GetUriFromPath(
    const char *path,
    unsigned int length,
    char **result
);
```

**功能**：从文件路径生成 URI 字符串。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `path` | const char* | 是 | 文件路径字符串 |
| `length` | unsigned int | 是 | 路径字符串长度 |
| `result` | char** | 是 | 输出参数，返回 URI 字符串指针 |

**返回值**：`FileManagement_ErrCode` 错误码

**内存管理**：调用方使用 `free()` 释放 `result` 指向的内存。

**示例**：

```c
#include "oh_file_uri.h"

int main() {
    const char *path = "/data/storage/el2/base/files/test.txt";
    char *uri = NULL;
    
    FileManagement_ErrCode ret = OH_FileUri_GetUriFromPath(
        path,
        strlen(path),
        &uri
    );
    
    if (ret == FILE_MANAGEMENT_SUCCESS) {
        printf("URI: %s\n", uri);  // 输出: file:///data/storage/el2/base/files/test.txt
        free(uri);
    }
    
    return ret;
}
```

**OH_FileUri_GetPathFromUri**：

```c
FileManagement_ErrCode OH_FileUri_GetPathFromUri(
    const char *uri,
    unsigned int length,
    char **result
);
```

**功能**：从 URI 字符串解析文件路径。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | const char* | 是 | URI 字符串 |
| `length` | unsigned int | 是 | URI 字符串长度 |
| `result` | char** | 是 | 输出参数，返回路径字符串指针 |

**返回值**：`FileManagement_ErrCode` 错误码

**示例**：

```c
const char *uri = "file:///data/storage/el2/base/files/test.txt";
char *path = NULL;

FileManagement_ErrCode ret = OH_FileUri_GetPathFromUri(
    uri,
    strlen(uri),
    &path
);

if (ret == FILE_MANAGEMENT_SUCCESS) {
    printf("Path: %s\n", path);  // 输出: /data/storage/el2/base/files/test.txt
    free(path);
}
```

**OH_FileUri_GetFullDirectoryUri**：

```c
FileManagement_ErrCode OH_FileUri_GetFullDirectoryUri(
    const char *uri,
    unsigned int length,
    char **result
);
```

**功能**：获取 URI 所在目录的 URI。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | const char* | 是 | 文件 URI |
| `length` | unsigned int | 是 | URI 字符串长度 |
| `result` | char** | 是 | 输出参数，返回目录 URI |

**返回值**：`FileManagement_ErrCode` 错误码

**示例**：

```c
const char *uri = "file:///data/storage/el2/base/files/test.txt";
char *dirUri = NULL;

FileManagement_ErrCode ret = OH_FileUri_GetFullDirectoryUri(
    uri,
    strlen(uri),
    &dirUri
);

if (ret == FILE_MANAGEMENT_SUCCESS) {
    printf("Dir URI: %s\n", dirUri);  // 输出: file:///data/storage/el2/base/files/
    free(dirUri);
}
```

**OH_FileUri_IsValidUri**：

```c
bool OH_FileUri_IsValidUri(
    const char *uri,
    unsigned int length
);
```

**功能**：验证 URI 格式是否有效。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | const char* | 是 | URI 字符串 |
| `length` | unsigned int | 是 | URI 字符串长度 |

**返回值**：`true` 表示有效，`false` 表示无效。

**示例**：

```c
const char *validUri = "file:///data/storage/el2/base/files/test.txt";
const char *invalidUri = "not-a-valid-uri";

printf("Valid: %d\n", OH_FileUri_IsValidUri(validUri, strlen(validUri)));  // 1
printf("Invalid: %d\n", OH_FileUri_IsValidUri(invalidUri, strlen(invalidUri)));  // 0
```

**OH_FileUri_GetFileName**：

```c
FileManagement_ErrCode OH_FileUri_GetFileName(
    const char *uri,
    unsigned int length,
    char **result
);
```

**功能**：从 URI 获取文件名或目录名。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | const char* | 是 | URI 字符串 |
| `length` | unsigned int | 是 | URI 字符串长度 |
| `result` | char** | 是 | 输出参数，返回文件名 |

**返回值**：`FileManagement_ErrCode` 错误码

**示例**：

```c
const char *uri = "file:///data/storage/el2/base/files/test.txt";
char *filename = NULL;

FileManagement_ErrCode ret = OH_FileUri_GetFileName(
    uri,
    strlen(uri),
    &filename
);

if (ret == FILE_MANAGEMENT_SUCCESS) {
    printf("FileName: %s\n", filename);  // 输出: test.txt
    free(filename);
}
```

## 7.3 FileShare 模块

### 7.3.1 头文件

**文件**：`interfaces/kits/ndk/fileshare/include/oh_file_share.h`

### 7.3.2 枚举定义

**FileShare_OperationMode**：

```c
typedef enum FileShare_OperationMode {
    READ_MODE = 1 << 0,       // 读模式
    WRITE_MODE = 1 << 1,      // 写模式
} FileShare_OperationMode;
```

**FileShare_PolicyErrorCode**：

```c
typedef enum FileShare_PolicyErrorCode {
    PERSISTENCE_FORBIDDEN = 1,    // 不允许持久化
    INVALID_MODE = 2,             // 无效模式
    INVALID_PATH = 3,             // 无效路径
    PERMISSION_NOT_PERSISTED = 4,  // 权限未持久化
} FileShare_PolicyErrorCode;
```

### 7.3.3 结构体定义

**FileShare_PolicyErrorResult**：

```c
typedef struct FileShare_PolicyErrorResult {
    char *uri;              // 失败的 URI
    int32_t code;           // 错误码
    char *message;          // 错误消息
} FileShare_PolicyErrorResult;
```

### 7.3.4 API 列表

**OH_FileShare_GrantUriPermission**：

```c
FileManagement_ErrCode OH_FileShare_GrantUriPermission(
    const char *uri,
    unsigned int uriLen,
    const char *bundleName,
    unsigned int bundleNameLen,
    int32_t mode,
    FileShare_PolicyErrorResult *results,
    unsigned int resultsLen
);
```

**功能**：授予其他应用对指定 URI 的访问权限。

**参数说明**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `uri` | const char* | 是 | 目标 URI |
| `uriLen` | unsigned int | 是 | URI 长度 |
| `bundleName` | const char* | 是 | 被授权应用包名 |
| `bundleNameLen` | unsigned int | 是 | 包名长度 |
| `mode` | int32_t | 是 | 操作模式 |
| `results` | FileShare_PolicyErrorResult* | 是 | 错误结果数组 |
| `resultsLen` | unsigned int | 是 | 结果数组长度 |

**返回值**：`FileManagement_ErrCode` 错误码

**示例**：

```c
#include "oh_file_share.h"

int main() {
    const char *uri = "file:///data/storage/el2/base/files/test.txt";
    const char *bundleName = "com.example.receiver";
    int32_t mode = READ_MODE;
    
    FileShare_PolicyErrorResult results[1] = {0};
    
    FileManagement_ErrCode ret = OH_FileShare_GrantUriPermission(
        uri,
        strlen(uri),
        bundleName,
        strlen(bundleName),
        mode,
        results,
        1
    );
    
    if (ret == FILE_MANAGEMENT_SUCCESS) {
        printf("权限授予成功\n");
    } else if (ret == FILE_MANAGEMENT_OPERATION_FAILED) {
        printf("授权失败: %s (code: %d)\n", 
               results[0].message, results[0].code);
    }
    
    // 释放错误消息内存
    if (results[0].uri) free(results[0].uri);
    if (results[0].message) free(results[0].message);
    
    return ret;
}
```

## 7.4 错误码定义

### 7.4.1 FileManagement_ErrCode

```c
typedef enum FileManagement_ErrCode {
    FILE_MANAGEMENT_SUCCESS = 0,                    // 成功
    FILE_MANAGEMENT_ERROR_BASE = 13900000,          // 错误基准值
    FILE_MANAGEMENT_INVALID_PARAMETER = 401,        // 参数错误
    FILE_MANAGEMENT_OPERATION_FAILED = 14300000,    // 操作失败
} FileManagement_ErrCode;
```

### 7.4.2 常用错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 13900001 | 操作不允许（EPERM） |
| 13900002 | 文件不存在（ENOENT） |
| 13900005 | 内存不足（ENOMEM） |
| 13900013 | 权限不足（EACCES） |
| 14300001 | 权限被拒绝 |

## 7.5 构建配置

### 7.5.1 CMakeLists.txt

```cmake
# 链接 FileURI
target_link_libraries(app PUBLIC
    libhilog_ndk.z.so
    libohfileuri.so
)

# 链接 FileShare
target_link_libraries(app PUBLIC
    libhilog_ndk.z.so
    libohfileshare.so
)
```

### 7.5.2 头文件包含

```c
// FileURI
#include "oh_file_uri.h"

// FileShare
#include "oh_file_share.h"
```

## 7.6 相关文档

| 文档 | 说明 |
|------|------|
| [JS N-API 接口](10_NAPI_JS.md) | JS 编程接口 |
| [内部 API](20_Inner_API.md) | 内部编程接口 |
| [系统架构](01_Architecture.md) | NDK 接口在架构中的位置 |
| [项目概览](00_Overview.md) | 接口层概述 |
| [安全评审](05_Security_Review.md) | API 安全风险 |

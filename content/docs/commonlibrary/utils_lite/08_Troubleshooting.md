# 故障排查

> utils_lite 常见构建/运行/调试问题与定位路径。

## 构建问题

### 问题 1：Feature 未启用

**现象**：`utils_lite` 模块未参与构建

**错误日志**：
```
error: 'utils_lite_feature_file' is not defined
```

**原因**：
- Feature flag 未在 GN args 中启用
- `bundle.json` 未正确注册

**解决方案**：

1. 检查 GN 配置：
```bash
# 查看当前配置
gn args out/product --list | grep utils_lite

# 设置配置
gn args out/product
# 添加：
#   utils_lite_feature_file = true
#   utils_lite_feature_js_builtin = true
```

2. 验证 `bundle.json`：
```json
"features": [
    "utils_lite_feature_file",
    "utils_lite_feature_js_builtin"
]
```

**证据**：`BUILD.gn:16-21`，`bundle.json:19-24`

---

### 问题 2：liteos_m/liteos_a 产物类型错误

**现象**：
- 期望静态库但生成动态库
- 期望动态库但生成静态库

**原因**：未正确设置 `ohos_kernel_type`

**解决方案**：

```bash
# 检查内核类型
gn args out/product --list | grep ohos_kernel_type

# 确认构建目标
hb set --product <product_name>
```

**证据**：`BUILD.gn:25-27`，`BUILD.gn:31-33`

---

### 问题 3：SDK 模拟器未构建

**现象**：
- 预览器报错：找不到模拟器模块
- `build_ohos_sdk` 未生效

**解决方案**：

```bash
# 启用 SDK 构建
gn args out/sdk
# 添加：
#   build_ohos_sdk = true
```

**证据**：`js/builtin/simulator/BUILD.gn:19`

---

## 运行时问题

### 问题 1：文件操作返回 -1

**现象**：`UtilsFileOpen` 返回 -1

**排查步骤**：

1. 检查路径是否存在
2. 检查文件权限
3. 检查 SPIFFS 限制

**代码证据**：`file/src/file_impl_hal/file.c:26-28`

```c
int UtilsFileOpen(const char* path, int oflag, int mode)
{
    return HalFileOpen(path, oflag, mode);
}
```

**常见原因**：

| 返回值 | 说明 |
|--------|------|
| -1 | 文件不存在或权限错误 |
| SPIFFS 限制 | 文件名超过 32 字节 |

**SPIFFS 限制**：
- 文件名最大 32 字节
- 最多 32 个打开文件
- 不支持多级目录

---

### 问题 2：KV Store 读写失败

**现象**：
- `UtilsGetValue` 返回 -1 或 -9
- `UtilsSetValue` 返回 -1

**排查步骤**：

1. 检查 Key 格式（仅小写字母、数字、下划线、点）
2. 检查 Key 长度（1-32 字节）
3. 检查 Value 长度（1-128 字节）
4. 检查 `FEATURE_KV_CACHE` 是否启用

**代码证据**：`include/kv_store.h:55-80`

```c
/**
 * @param key Indicates the key to be indexed. It allows only lowercase letters,
 * digits, underscores (_), and dots (.). Its length cannot exceed 32 bytes.
 */
int UtilsGetValue(const char* key, char* value, unsigned int len);
```

---

### 问题 3：定时器不触发

**现象**：
- `StartTimerTask` 成功但回调不执行
- `KalTimerStart` 返回错误

**排查步骤**：

1. 检查 `KalErrCode` 返回值
2. 验证回调函数签名
3. 检查定时器是否已删除

**代码证据**：`kal/timer/include/kal.h:33-37`

```c
typedef enum {
    KAL_OK = 0,
    KAL_ERR_PARA = 1,           // 参数错误
    KAL_ERR_INNER = 2,          // 内部错误
    KAL_ERR_TIMER_STATE = 0x100  // 定时器状态错误
} KalErrCode;
```

---

### 问题 4：JS API 返回错误

**现象**：
- Promise 被 reject
- 回调返回错误码

**排查步骤**：

1. 检查 JS 参数类型
2. 检查参数长度限制
3. 查看 N-API 返回值

**示例**：

```javascript
// ❌ 错误：Key 包含大写字母
kvstore.set("KEY", "value");  // 失败

// ✅ 正确：Key 仅小写字母
kvstore.set("key", "value");  // 成功
```

**代码证据**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp:30-43`

```cpp
static bool IsValidKey(const char* key)
{
    // 检查长度 1-32
    // 检查字符集：小写字母、数字、下划线、点
}
```

---

## 调试技巧

### 日志打印

```c
// C 层添加调试日志
#include "hilog/log.h"
OH_LOG_INFO(LOG_CORE, "UtilsFileOpen: path=%s", path);
```

### 符号断点

| 断点 | 作用 |
|------|------|
| `UtilsFileOpen` | 文件操作入口 |
| `HalFileOpen` | HAL 层入口 |
| `KalTimerCreate` | 定时器创建 |
| `UtilsGetValue` | KV 获取 |

### GDB 调试

```bash
# 附加到进程
gdb ./your_app

# 设置断点
(gdb) break UtilsFileOpen

# 运行
(gdb) run

# 查看变量
(gdb) print path
```

---

## 常见错误码

### 文件操作错误码

| 错误码 | 说明 |
|--------|------|
| -1 | 操作失败 |
| 0 | 成功 |

### KV Store 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 操作失败 |
| -9 | 参数错误 |

### KAL Timer 错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 1 | 参数错误 |
| 2 | 内部错误 |
| 0x100+ | 定时器状态错误 |

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [N-API 参考](03_NAPI_Reference.md) - JS API
- [GN 构建](05_GN_Build.md) - 构建配置
- [安全评审](07_Security_Review.md) - 安全考量

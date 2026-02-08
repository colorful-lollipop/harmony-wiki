# CUPS Patch 详细分析

> 本文档详细记录 OpenHarmony 对 CUPS v2.4.14 的所有 Patch。

---

## Patch 清单总表

### 按功能分类

| 分类 | 数量 | 描述 |
|-----|------|------|
| **OHOS 特有适配** | 22 个 | USB、日志、网络、安全适配 |
| **CUPS 功能增强** | 7 个 | 作业监控、超时处理等增强 |
| **CVE 安全修复** | 9 个 | 安全漏洞后向移植 |

---

## 第一部分：OHOS 特有适配 Patch

这些 Patch 为 OpenHarmony 特有，用于适配 OH 系统特性和服务。

### 1.1 USB 打印支持

#### Patch: `ohos-usb-print.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 54 KB |
| **新建文件** | `backend/usb-oh.c` (1845 行) |
| **修改文件** | 无 |
| **关联需求** | OH USB 服务打印支持 |

**功能概述**：

实现完整的 USB 打印后端，替代传统 Linux 的 `usblp` 内核模块方案。

**关键实现**：

```c
// usb-oh.c 核心功能

// 1. 设备发现和枚举
static const char *find_device(void);

// 2. 打印机连接管理
static int open_device(const char *device_uri);
static void close_device(int fd);

// 3. 打印作业处理
static int print_fd(int fd, int copies, const char *title);

// 4. IEEE-1284 设备 ID 获取
static int get_device_id(int fd, char *id, size_t idsiz);

// 5. USB 传输操作
static ssize_t oh_bulk_read(int fd, unsigned char *data, size_t datalen);
static ssize_t oh_bulk_write(int fd, const unsigned char *data, size_t datalen);
```

**OH 特有功能**：

| 功能 | 实现方式 |
|-----|---------|
| USB 设备发现 | 调用 `OH_GetDevices()` |
| 设备打开 | 调用 `OH_OpenDevice()` |
| 设备关闭 | 调用 `OH_CloseDevice()` |
| 数据传输 | 调用 `OH_BulkTransferRead/Write()` |
| 接口声明 | 调用 `OH_ClaimInterface()` |

**设备 URI 格式**：

```
usb://MANUFACTURER/MODEL?serial=序列号
示例：usb://HP/LaserJet_Pro?serial=CND12345
```

**打印机quirks支持**：
- 支持标准 USB 打印机
- 支持双向/单向打印机
- 支持非标准打印机quirks配置

**升级建议**：
此 Patch 完全依赖 OH USB 服务 API，与上游实现完全不同。如 OH USB API 发生变化，需重新适配。

---

#### Patch: `ohos-usb-manager.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 43 KB |
| **新建文件** | `backend/usb_manager.cxx`, `backend/usb_manager.h` |
| **修改文件** | 无 |

**功能概述**：

实现 OH USB 服务 C++ API 到 C 函数的封装桥接层。

**核心数据结构**：

```cpp
// usb_manager.h

// OH USB 设备描述符包装
struct OHUSB_DEVICE_DESCRIPTOR {
    uint8_t  bLength;              // 描述符长度
    uint8_t  bDescriptorType;      // 描述符类型
    uint16_t bcdUSB;               // USB 版本
    uint8_t  bDeviceClass;         // 设备类
    // ...
};

// OH USB 传输描述符
struct OHUSB_TRANSFER {
    OHUSB_PIPE pipe;               // 管道
    unsigned char *buffer;         // 数据缓冲区
    size_t buflen;                // 缓冲区长度
    int timeout;                  // 超时时间
};
```

**关键 API 封装**：

| C++ 原生 API | C 封装函数 | 用途 |
|-------------|-----------|------|
| `UsbSrvClient::GetInstance()` | `OH_GetInstance()` | 获取 USB 服务实例 |
| `GetDevices()` | `OH_GetDevices()` | 获取设备列表 |
| `OpenDevice()` | `OH_OpenDevice()` | 打开设备 |
| `ClaimInterface()` | `OH_ClaimInterface()` | 声明接口 |
| `BulkTransfer()` | `OH_BulkTransferRead()` | 批量读取 |
| `BulkTransfer()` | `OH_BulkTransferWrite()` | 批量写入 |
| `ControlTransfer()` | `OH_ControlTransferRead()` | 控制读取 |
| `ControlTransfer()` | `OH_ControlTransferWrite()` | 控制写入 |

**重试机制**：

```cpp
static ssize_t OH_BulkTransferRead(OHUSB_PIPE pipe,
                                   unsigned char *data,
                                   size_t datalen,
                                   int timeout) {
    int retries = 3;
    while (retries-- > 0) {
        int ret = pipe->BulkTransfer(data, datalen, timeout);
        if (ret == HDF_SUCCESS) {
            return ret;
        }
        // 超时重试
        usleep(10000);  // 10ms
    }
    return ret;
}
```

**升级建议**：
此封装层与 OH USB API 紧密绑定，是 OH 特有实现。如 USB 服务 API 升级，需同步更新。

---

### 1.2 日志系统集成

#### Patch: `ohos-hilog-print.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 2.5 KB |
| **新建文件** | `scheduler/hilog-helper.c`, `scheduler/hilog-helper.h` |
| **修改文件** | `scheduler/log.c` |

**功能概述**：

将 CUPS 日志系统集成到 OpenHarmony HiLog 子系统。

**新建文件**：

```c
// hilog-helper.h
#ifndef _HILOG_HELPER_H
#define _HILOG_HELPER_H

#include <hilog/log_c.h>

int hilogPrint(int level, const char *format, ...);

#endif /* _HILOG_HELPER_H */
```

```c
// hilog-helper.c
#include "hilog-helper.h"

__attribute__((format(printf, 2, 3)))
int hilogPrint(int level, const char *format, ...) {
    va_list args;
    int ret;

    va_start(args, format);
    ret = HiLogPrint(LOG_CORE, DLOG_DEBUG, LOG_DOMAIN, "cupslog", format, args);
    va_end(args);

    return ret;
}
```

**HiLog 配置**：

| 参数 | 值 | 说明 |
|-----|-----|------|
| `LOG_CORE` | 日志类型 | 核心日志 |
| `DLOG_DEBUG` | 日志级别 | 调试级别 |
| `LOG_DOMAIN` | 0xD0050B0 | CUPS 专属域 |
| Tag | "cupslog" | CUPS 日志标签 |

**与 CUPS 日志集成**：

在 `scheduler/log.c` 中修改 `cupsdWriteErrorLog()` 函数：

```c
void cupsdWriteErrorLog(int level, const char *message) {
    // 原有 syslog 输出...

    // 新增 HiLog 输出
    hilogPrint(level, "%s", message);

    // 敏感数据过滤
    if (strstr(message, "auth-info") ||
        strstr(message, "password")) {
        // 跳过日志输出
        return;
    }
}
```

**升级建议**：
此 Patch 实现了 CUPS 到 HiLog 的桥接，属于 OH 特有适配。上游不支持 HiLog，需保留此修改。

---

#### Patch: `cups-log-datamasking.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 21 KB |
| **新建文件** | `scheduler/datamasking.c`, `scheduler/datamasking.h` |
| **修改文件** | 5 个文件 |

**功能概述**：

实现日志数据脱敏，保护用户隐私信息。

**新建文件**：

```c
// datamasking.h

#ifndef _DATAMASKING_H
#define _DATAMASKING_H

// 获取安全的参数值
const char *getSafeArgument(const char *arg, int arg_index, int argc, char **argv);

// 脱敏作业名称
char *maskJobName(const char *job_name);

// 脱敏用户名
const char *maskUserName(const char *user_name);

#endif /* _DATAMASKING_H */
```

```c
// datamasking.c

#include "datamasking.h"

const char *getSafeArgument(const char *arg, int arg_index, int argc, char **argv) {
    if (arg_index < 0 || arg_index >= argc) {
        return NULL;
    }

    const char *value = argv[arg_index];

    // 检查敏感关键字
    if (strstr(value, "auth-info") ||
        strstr(value, "password") ||
        strstr(value, "secret")) {
        return "<masked>";
    }

    return value;
}

char *maskJobName(const char *job_name) {
    if (!job_name) return NULL;

    char *masked = strdup(job_name);
    if (!masked) return NULL;

    // 保留首字符，其余用 * 替换
    size_t len = strlen(masked);
    for (size_t i = 1; i < len && i < 8; i++) {
        masked[i] = '*';
    }

    return masked;
}
```

**修改的文件**：

| 文件 | 修改内容 |
|-----|---------|
| `backend/ipp.c` | 脱敏打印作业参数 |
| `scheduler/auth.c` | 脱敏认证信息 |
| `scheduler/ipp.c` | 脱敏 IPP 请求中的敏感字段 |
| `scheduler/job.c` | 脱敏作业名称 |
| `scheduler/process.c` | 脱敏进程参数 |

**脱敏规则**：

| 字段 | 脱敏方式 |
|-----|---------|
| `job-name` | 首字符保留，其余用 * 替换 |
| `document-name` | 首字符保留，其余用 * 替换 |
| `requesting-user-name` | 替换为 `<anonymous>` |
| 密码相关参数 | 完全替换为 `<masked>` |
| 设备 URI 中的密码 | 替换为 `***` |

**升级建议**：
此安全功能符合 OHOS 隐私合规要求，建议推向上游。属于安全增强功能，与具体系统无关。

---

### 1.3 网络适配

#### Patch: `ohos_ip_conflict.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 14 KB |
| **修改文件** | 5 个 |
| **关联需求** | 多网络接口设备打印支持 |

**功能概述**：

添加网络接口绑定功能，防止多网络环境下的 IP 冲突问题。

**修改的文件**：

| 文件 | 修改内容 |
|-----|---------|
| `cups/http.h` | 添加 `httpConnect3()` 声明 |
| `cups/http.c` | 实现 `httpConnect3()` |
| `cups/http-addrlist.c` | 添加 `httpAddrConnect2()` |
| `backend/ipp.c` | 支持 `nic=` 参数 |
| `tools/ippeveprinter.c` | 支持 NIC 选项 |

**新增 API**：

```c
// http.h

// 新增：支持网络接口绑定的连接函数
http_t *httpConnect3(const char *hostname, int port,
                     const char *nic,    // 网络接口名称
                     int *msec);
int httpReconnect3(http_t *http, int timeout, const char *nic);

// 原有函数保留，但内部调用新的函数
http_t *httpConnect2(const char *hostname, int port, int *msec);
```

**实现原理**：

```c
// http.c

http_t *httpConnect3(const char *hostname, int port,
                     const char *nic, int *msec) {
    http_t *http = httpConnect2(hostname, port, msec);
    if (http && nic) {
        // 绑定到指定网络接口
        setsockopt(http->fd, SOL_SOCKET, SO_BINDTODEVICE,
                   nic, strlen(nic));
    }
    return http;
}
```

**使用方法**：

```
# IPP 打印机指定网络接口
ipp://192.168.1.100/ipp/print?nic=eth0

# WiFi 网络
ipp://192.168.1.100/ipp/print?nic=wlan0
```

**升级建议**：
此功能在上游已有类似实现 (`httpConnect2`)，`httpConnect3` 可考虑推向上游。`nic=` 参数格式可标准化后提交上游。

---

#### Patch: `ohos-ipp-authenticate.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 3.8 KB |
| **修改文件** | `scheduler/ipp.c` |
| **关联需求** | OHOS 安全模型适配 |

**功能概述**：

适配 OHOS 安全模型，移除文件凭证存储，改用环境变量传递。

**主要修改**：

| 修改前 | 修改后 |
|-------|-------|
| 凭证写入文件 `/var/spool/cups/aXXXXX` | 凭证仅存内存 `job->auth_env[]` |
| 凭证以 root 权限存储 | 无文件操作 |
| 支持持久化凭证 | 一次性凭证 |

**关键代码变更**：

```c
// scheduler/ipp.c - save_auth_info()

// 修改前：写入文件
void save_auth_info(job_t *job, const char *filename) {
    FILE *fp = fopen(filename, "w");
    fprintf(fp, "%s\n", auth_info);
    fclose(fp);
}

// 修改后：仅存内存
void save_auth_info(job_t *job, const char *auth_info) {
    // 仅保存到环境变量数组
    snprintf(job->auth_env[JOB_AUTH_INDEX],
             sizeof(job->auth_env[JOB_AUTH_INDEX]),
             "CUPS_AUTH_INFO=%s", auth_info);
}
```

**安全优势**：
- 无持久化凭证文件，防止凭证泄露
- 无 root 权限凭证文件
- 容器环境友好

**升级建议**：
此修改适配 OHOS 无 root 的安全模型。上游版本保留文件存储方式，如需统一需与上游协商。

---

### 1.4 其他 OH 适配 Patch

#### Patch: `ohos-filetypes-crash.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 2.6 KB |
| **修改文件** | 2 个 |
| **功能** | 修复特定文件类型导致的崩溃 |

#### Patch: `ohos-cups-badfd.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 3.2 KB |
| **修改文件** | 3 个 |
| **功能** | 修复错误文件描述符处理 |

#### Patch: `ohos-verify-backend.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 447 字节 |
| **修改文件** | 1 个 |
| **功能** | 后端验证增强 |

#### Patch: `ohos-uni-print-driver-path.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 967 字节 |
| **修改文件** | 2 个 |
| **功能** | 统一打印驱动路径配置 |

#### Patch: `ohos-ppdfile-not-generated.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 717 字节 |
| **修改文件** | 1 个 |
| **功能** | PPD 文件生成修复 |

#### Patch: `ohos-multi-file-print.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 432 字节 |
| **修改文件** | 1 个 |
| **功能** | 多文件打印支持 |

#### Patch: `ohos-modify-pthread.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 332 字节 |
| **修改文件** | 1 个 |
| **功能** | 线程库适配 |

#### Patch: `ohos-default-paper-size.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 369 字节 |
| **修改文件** | 1 个 |
| **功能** | 默认纸张大小配置 |

#### Patch: `ohos-cloud-pagesize-error-fix.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 447 字节 |
| **修改文件** | 1 个 |
| **功能** | 云打印页面尺寸错误修复 |

#### Patch: `ohos-ipp-everywhere-color-fix.patch`

| 属性 | 值 |
|------|-----|
| **文件大小** | 370 字节 |
| **修改文件** | 1 个 |
| **功能** | IPP Everywhere 颜色模式修复 |

---

## 第二部分：CUPS 功能增强 Patch

这些 Patch 增强 CUPS 功能，可能适用于上游。

### 2.1 作业状态监控

#### Patch: `cups-usb-job-state-monitor.patch`

| 属性 | 值 |
|------|-----|
| **功能** | USB 打印作业状态监控增强 |

#### Patch: `cups-usb-paperout.patch`

| 属性 | 值 |
|------|-----|
| **功能** | USB 打印机缺纸状态检测 |

### 2.2 超时处理

#### Patch: `cups-web-devices-timeout.patch`

| 属性 | 值 |
|------|-----|
| **功能** | Web 设备发现超时处理 |

#### Patch: `cups-driverd-timeout.patch`

| 属性 | 值 |
|------|-----|
| **功能** | 驱动加载超时处理 |

### 2.3 其他增强

| Patch | 功能 |
|-------|------|
| `cups-system-auth` | 系统认证增强 |
| `cups-freebind` | 地址绑定增强 |
| `cups-ipp-multifile` | IPP 多文件支持 |
| `cups-banners` | 横幅打印支持 |
| `cups-multilib` | 多库支持 |
| `cups-direct-usb` | 直接 USB 访问 |
| `cups-uri-compat` | URI 兼容性增强 |

---

## 第三部分：CVE 安全修复 Patch

详见 [06_Security.md](06_Security.md)

---

## Patch 升级建议矩阵

| Patch | 可推向上游 | 需保留 OH 特有 | 优先级 |
|-------|-----------|--------------|-------|
| `ohos-usb-print.patch` | ❌ | ✅ | 高 |
| `ohos-usb-manager.patch` | ❌ | ✅ | 高 |
| `ohos-hilog-print.patch` | ❌ | ✅ | 中 |
| `cups-log-datamasking.patch` | ✅ | - | 高 |
| `ohos_ip_conflict.patch` | ⚠️ 部分 | ✅ | 中 |
| `ohos-ipp-authenticate.patch` | ❌ | ✅ | 中 |
| `ohos-filetypes-crash.patch` | ✅ | - | 高 |
| `ohos-cups-badfd.patch` | ✅ | - | 高 |
| `cups-usb-*.patch` | ✅ | - | 中 |
| `cups-web-*.patch` | ✅ | - | 中 |
| **所有 CVE Patch** | ✅ | - | **最高** |

---

## 注意事项

1. **USB 相关 Patch** 完全依赖 OH USB API，必须保留
2. **日志相关 Patch** 需要与 OHOS 日志系统保持一致
3. **安全 CVE Patch** 需要及时同步上游新版本
4. **认证相关 Patch** 需要符合 OHOS 安全模型

---

## 相关文档

- [01_Overview.md](01_Overview.md) - 库概览
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 使用情况
- [06_Security.md](06_Security.md) - 安全分析

# HDF Core 安全风险评审

## 1. 概述

本文档基于 HDF Core 代码分析，识别潜在安全风险，提供攻击面分析和修复建议。

**分析范围**:
- 代码路径: `drivers/hdf_core/adapter/uhdf2/`, `drivers/hdf_core/framework/`
- 排除: 测试代码 (`test/`, `unittest/`, `fuzztest/`)
- 版本: 4.0

## 2. 攻击面清单

### 2.1 外部输入入口

| 入口类型 | 位置 | 说明 |
|----------|------|------|
| IPC 接口 | `devsvc_manager_stub.c` | 服务管理 IPC 调用 |
| HDI 接口 | `servmgr_client.c` | HDI 客户端接口 |
| 配置文件 | HCS 文件 | 设备配置解析 |
| 模块加载 | `devmgr_service_stub.c:221-238` | .ko 模块加载 |
| Netlink | `devmgr_uevent.c` | 内核事件接收 |
| IOCTL | `hdf_security.c` | 安全设备 IOCTL |

### 2.2 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  Untrusted Domain                                           │
│  - 第三方应用                                                │
│  - 非特权进程                                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ HDI Interface
┌─────────────────────────────────────────────────────────────┐
│  Semi-Trusted Domain                                        │
│  - System Services                                           │
│  - 有 SELinux 约束的进程                                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ IPC (Binder)
┌─────────────────────────────────────────────────────────────┐
│  Trusted Domain                                             │
│  - DevMgr Process (hdf_devmgr)                              │
│  - DevHost Process(es)                                      │
│  - SELinux 强制访问控制                                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ System Call
┌─────────────────────────────────────────────────────────────┐
│  Kernel Domain                                              │
│  - KHDF 内核模块                                             │
│  - 直接硬件访问                                              │
└─────────────────────────────────────────────────────────────┘
```

## 3. 可被利用点分析

### 3.1 风险 #1: 模块加载路径验证

**位置**: `adapter/uhdf2/manager/src/devmgr_service_stub.c:240-256`

**代码**:
```c
static int32_t MakeModulePath(char *buffer, const char *moduleName)
{
    char temp[PATH_MAX] = {0};
    if (sprintf_s(temp, PATH_MAX, "%s/%s.ko", HDF_MODULE_DIR, moduleName) <= 0) {
        return HDF_FAILURE;
    }
    
    char *path = realpath(temp, buffer);
    if (path == NULL || strncmp(path, HDF_MODULE_DIR, strlen(HDF_MODULE_DIR)) != 0) {
        HDF_LOGE("driver module file is invalid: %{public}s", temp);
        return HDF_ERR_INVALID_PARAM;
    }
    return HDF_SUCCESS;
}
```

**分析**:
- ✅ 使用 `realpath()` 解析绝对路径
- ✅ 使用 `strncmp()` 验证路径前缀
- ⚠️ `moduleName` 未经验证直接拼接到路径
- ⚠️ `HDF_MODULE_DIR` 长度计算在每次调用时重复

**风险等级**: 中

**影响**: 如果路径验证被绕过，可能加载任意内核模块

**修复建议**:
```c
// 建议: 验证 moduleName 格式
static bool IsValidModuleName(const char *name) {
    if (name == NULL || strlen(name) == 0 || strlen(name) > MAX_MODULE_NAME_LEN) {
        return false;
    }
    // 只允许字母、数字、下划线、连字符
    for (const char *p = name; *p; p++) {
        if (!isalnum(*p) && *p != '_' && *p != '-') {
            return false;
        }
    }
    return true;
}
```

### 3.2 风险 #2: IPC 接口权限检查

**位置**: `adapter/uhdf2/manager/src/devsvc_manager_stub.c:36-95`

**代码**:
```c
static int32_t AddServicePermCheck(const char *servName)
{
#ifdef WITH_SELINUX
    pid_t callingPid = HdfRemoteGetCallingPid();
    char *callingSid = HdfRemoteGetCallingSid();
    if (callingSid == NULL) {
        return HDF_ERR_NOPERM;
    }
    if (HdfAddServiceCheck(callingSid, servName) != 0) {
        free(callingSid);
        return HDF_ERR_NOPERM;
    }
    free(callingSid);
#endif
    return HDF_SUCCESS;
}
```

**分析**:
- ✅ 获取调用者 PID/SID
- ✅ SELinux 权限检查
- ⚠️ `WITH_SELINUX` 宏控制，非 SELinux 系统无检查
- ⚠️ `servName` 未在权限检查前验证有效性

**风险等级**: 中

**影响**: 非 SELinux 系统可能绕过服务注册权限控制

**修复建议**:
1. 在非 SELinux 系统添加基于 UID/GID 的备选检查
2. 验证 servName 格式后再进行权限检查

### 3.3 风险 #3: 字符串长度验证

**位置**: `adapter/uhdf2/security/src/hdf_security.c:123-135`

**代码**:
```c
static int32_t HdfSecCheckParameters(const char *id)
{
    if (id == NULL) {
        return HDF_FAILURE;
    }
    uint32_t len = (uint32_t)strlen(id);
    if (len >= (ID_MAX_SIZE - 1)) {
        return HDF_FAILURE;
    }
    return HDF_SUCCESS;
}
```

**分析**:
- ✅ 检查 NULL 指针
- ✅ 检查长度上限
- ⚠️ 未检查最小长度（空字符串）
- ⚠️ 未检查字符有效性

**风险等级**: 低

**影响**: 可能使用空字符串或无效字符作为安全 ID

**修复建议**:
```c
static int32_t HdfSecCheckParameters(const char *id)
{
    if (id == NULL) {
        return HDF_FAILURE;
    }
    uint32_t len = (uint32_t)strlen(id);
    if (len == 0 || len >= ID_MAX_SIZE) {
        return HDF_FAILURE;
    }
    // 验证字符有效性
    for (uint32_t i = 0; i < len; i++) {
        if (!isprint(id[i]) || isspace(id[i])) {
            return HDF_FAILURE;
        }
    }
    return HDF_SUCCESS;
}
```

### 3.4 风险 #4: HdfSBuf 反序列化

**位置**: `adapter/uhdf2/manager/src/devsvc_manager_stub.c:220-252`

**代码**:
```c
static int32_t DevSvcMgrStubGetPara(
    struct HdfSBuf *data, struct HdfServiceInfo *info, struct HdfRemoteService **service)
{
    info->servName = HdfSbufReadString(data);
    if (info->servName == NULL) {
        return HDF_FAILURE;
    }
    // ...
    *service = HdfSbufReadRemoteService(data);
    if (*service == NULL) {
        return HDF_FAILURE;
    }
    info->servInfo = HdfSbufReadString(data);
    info->interfaceDesc = HdfSbufReadString(data);
    return HDF_SUCCESS;
}
```

**分析**:
- ✅ 检查 NULL 返回值
- ⚠️ 未验证字符串长度
- ⚠️ 未验证 `devClass` 枚举值范围
- ⚠️ `servInfo` 和 `interfaceDesc` 为 NULL 时继续执行

**风险等级**: 中

**影响**: 可能导致服务信息不完整或无效

**修复建议**:
```c
static int32_t DevSvcMgrStubGetPara(
    struct HdfSBuf *data, struct HdfServiceInfo *info, struct HdfRemoteService **service)
{
    info->servName = HdfSbufReadString(data);
    if (info->servName == NULL || strlen(info->servName) > MAX_SERVICE_NAME_LEN) {
        return HDF_ERR_INVALID_PARAM;
    }
    
    if (!HdfSbufReadUint16(data, &info->devClass)) {
        return HDF_ERR_INVALID_PARAM;
    }
    // 验证枚举值
    if (info->devClass >= DEVICE_CLASS_MAX) {
        return HDF_ERR_INVALID_PARAM;
    }
    // ...
}
```

### 3.5 风险 #5: 动态内存分配检查

**位置**: 多处使用 `OsalMemCalloc` 后未立即检查

**代码示例**:
```c
// devsvc_manager_stub.c:165-194
static struct HdfDeviceObject *ObtainServiceObject(...)
{
    struct HdfDeviceObjectHolder *serviceObjectHolder = OsalMemCalloc(sizeof(*serviceObjectHolder));
    // 未立即检查 NULL
    serviceObjectHolder->remoteSvcAddr = (uintptr_t)service;
    // ...
}
```

**分析**:
- ⚠️ 部分路径缺少 NULL 检查
- ⚠️ 内存分配失败可能导致空指针解引用

**风险等级**: 中

**影响**: 内存不足时可能导致崩溃

**修复建议**:
```c
static struct HdfDeviceObject *ObtainServiceObject(...)
{
    struct HdfDeviceObjectHolder *holder = OsalMemCalloc(sizeof(*holder));
    if (holder == NULL) {
        HDF_LOGE("Failed to allocate memory for service object holder");
        return NULL;
    }
    // ...
}
```

## 4. 安全机制评估

### 4.1 SELinux 集成

**状态**: ✅ 良好

**实现**:
- `AddServicePermCheck()` - 服务注册权限
- `GetServicePermCheck()` - 服务获取权限
- `ListServicePermCheck()` - 服务列表权限

**建议**:
- 在非 SELinux 系统添加备选检查机制
- 考虑添加更细粒度的权限控制（按设备类）

### 4.2 进程隔离

**状态**: ✅ 良好

**实现**:
- DevMgr 独立进程
- 每个 Host 配置独立 DevHost 进程
- 进程崩溃不影响其他驱动

### 4.3 路径安全

**状态**: ⚠️ 需改进

**实现**:
- `realpath()` 解析绝对路径
- 路径前缀验证

**建议**:
- 添加模块名格式验证
- 统一路径验证逻辑

### 4.4 权限位图

**状态**: ✅ 良好

**实现**:
- `hdf_security.c` 实现硬件访问权限位图
- 支持 I2C、SPI、GPIO 等细粒度权限

## 5. 信息泄露风险

### 5.1 日志信息

**位置**: 多处使用 `%{public}s` 输出敏感信息

**风险**: 日志可能包含服务名、模块路径等信息

**建议**:
- 审查日志输出，避免敏感信息泄露
- 使用分级日志控制

### 5.2 接口描述符

**位置**: `devsvc_manager_stub.c:646`

```c
HdfRemoteServiceSetInterfaceDesc(inst->remote, "HDI.IServiceManager.V1_0");
```

**风险**: 接口版本信息暴露

**影响**: 低，版本信息通常公开

## 6. 拒绝服务风险

### 6.1 资源耗尽

**风险点**:
- 服务注册数量无上限
- 设备加载数量无上限
- IPC 调用无速率限制

**建议**:
- 添加服务数量限制
- 添加设备数量限制
- 考虑 IPC 调用频率限制

### 6.2 死锁风险

**位置**: 多处使用互斥锁

**分析**:
- 锁的获取顺序是否一致
- 是否存在循环等待

**建议**:
- 统一锁的获取顺序
- 使用超时机制

## 7. 修复建议汇总

| 优先级 | 风险 | 建议 |
|--------|------|------|
| 高 | 模块加载验证 | 添加模块名格式验证 |
| 高 | 内存分配检查 | 统一添加 NULL 检查 |
| 中 | IPC 权限 | 非 SELinux 备选检查 |
| 中 | SBuf 验证 | 添加枚举值和长度验证 |
| 低 | 字符串验证 | 添加字符有效性检查 |
| 低 | 资源限制 | 添加服务和设备数量限制 |

## 8. 检查局限性

本安全评审存在以下局限：

1. **静态分析**: 仅基于代码静态分析，未进行动态测试
2. **范围限制**: 未包含具体驱动模型（audio、camera 等）的详细分析
3. **依赖组件**: 未深入分析依赖组件（ipc、samgr 等）的安全问题
4. **内核态**: KHDF 内核态代码分析有限

## 9. 相关文档

- [项目概览](./01_Overview.md)
- [架构说明](./03_Architecture.md)
- [HDI 接口](./04_HDI_API.md)
- [内部 API](./05_Inner_API.md)

## 10. 参考

- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/en/device-dev/security/)
- [SELinux 策略文档](https://gitee.com/openharmony/selinux_adapter)

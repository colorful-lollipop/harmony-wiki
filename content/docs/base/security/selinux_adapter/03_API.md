# Inner API 接口

> **说明**: 本组件为系统级安全组件，不提供 N-API (JS API)，仅提供 **C/C++ Inner API** 接口。

## 1. policycoreutils.h - 策略加载与文件恢复

**头文件**: `interfaces/policycoreutils/include/policycoreutils.h`

**产物**: `libload_policy.so`, `librestorecon.so`

### 1.1 API 清单

| 函数名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `LoadPolicy` | void | int | 加载 SELinux 二进制策略 |
| `Restorecon` | const char *path | int | 恢复单个文件标签 |
| `RestoreconRecurse` | const char *path | int | 递归恢复目录标签 |
| `RestoreconRecurseParallel` | const char *path, unsigned int nthreads | int | 并行递归恢复 |
| `RestoreconRecurseForce` | const char *path | int | 强制恢复（跳过 skipacl 检查） |
| `RestoreconFromParentDir` | const char *path | int | 从父目录继承标签 |
| `RestoreconCommon` | const char *path, unsigned int flag, unsigned int nthreads | int | 通用恢复接口 |

### 1.2 详细接口

#### LoadPolicy

```c
int LoadPolicy(void);
```

| 项目 | 说明 |
|------|------|
| **功能** | 加载编译后的 SELinux 二进制策略文件 |
| **实现** | `framework/policycoreutils/src/load_policy.cpp` |
| **内部调用** | `selinuxfs` -> `write("/sys/fs/selinux/policy")` |
| **线程安全** | 应该在 init 阶段单线程调用 |
| **错误码** | 负值表示失败（具体定义见 libselinux） |
| **前置条件** | `/etc/selinux/targeted/policy/policy.31` 存在 |

#### RestoreconCommon

```c
int RestoreconCommon(const char *path, unsigned int flag, unsigned int nthreads);
```

| 项目 | 说明 |
|------|------|
| **功能** | 通用文件标签恢复接口 |
| **参数** | `path`: 文件/目录路径<br>`flag`: 控制标志<br>`nthreads`: 并行线程数 |
| **flag 取值** | 参考 SELINUX_RESTORECON_* 常量 |
| **线程安全** | 内部使用线程池，可并发调用 |
| **返回值** | 0: 成功<br>负值: 失败 |

## 2. selinux_parameter.h - 参数管理

**头文件**: `interfaces/policycoreutils/include/selinux_parameter.h`

**产物**: `libselinux_parameter_static` (静态库)

### 2.1 数据结构

```c
typedef struct ParameterNode {
    const char *paraName;      // 参数名
    const char *paraContext;   // 安全上下文
    int index;                 // 索引
} ParameterNode;

typedef struct ParamContextsList {
    struct ParameterNode info;
    struct ParamContextsList *next;
} ParamContextsList;

typedef struct SrcInfo {
    int sockFd;                // socket 文件描述符
    struct ucred uc;           // 用户凭证 (pid, uid, gid)
} SrcInfo;
```

### 2.2 API 清单

| 函数名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `InitParamSelinux` | int isInit | int | 初始化参数 SELinux |
| `GetParamList` | void | ParamContextsList* | 获取参数上下文列表 |
| `DestroyParamList` | ParamContextsList **list | void | 释放参数列表 |
| `GetParamLabel` | const char *paraName | const char* | 获取参数的安全上下文 |
| `GetParamLabelIndex` | const char *paraName | int | 获取参数上下文索引 |

### 2.3 详细接口

#### InitParamSelinux

```c
int InitParamSelinux(int isInit);
```

| 项目 | 说明 |
|------|------|
| **功能** | 初始化参数 SELinux 上下文 |
| **参数** | `isInit`: 是否为 init 进程调用 |
| **实现** | `framework/policycoreutils/src/selinux_parameter.c` |
| **前置条件** | `parameter_contexts` 文件已加载 |

#### GetParamLabel

```c
const char *GetParamLabel(const char *paraName);
```

| 项目 | 说明 |
|------|------|
| **功能** | 根据参数名查找对应的 Security Context |
| **参数** | `paraName`: 要查询的参数名 |
| **返回值** | 成功: Security Context 字符串指针<br>失败: NULL |
| **线程安全** | 读安全，需确保初始化完成 |
| **内存说明** | 返回值指向内部共享内存，调用者不应释放 |

## 3. param_checker.h - 参数权限检查

**头文件**: `interfaces/policycoreutils/include/param_checker.h`

**产物**: `libparaperm_checker.so`

### 3.1 API 清单

| 函数名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `SetInitSelinuxLog` | void | void | 初始化 SELinux 日志 |
| `SetParamCheck` | const char *paraName, const char *destContext, const SrcInfo *info | int | 检查参数写入权限 |

### 3.2 详细接口

#### SetParamCheck

```c
int SetParamCheck(const char *paraName, const char *destContext, const SrcInfo *info);
```

| 项目 | 说明 |
|------|------|
| **功能** | 检查进程是否有权限写入指定参数 |
| **参数** | `paraName`: 参数名<br>`destContext`: 目标上下文<br>`info`: 来源进程凭证 |
| **info 结构** | `{sockFd, uc.pid, uc.uid, uc.gid}` |
| **实现** | `framework/policycoreutils/src/param_checker.c` |
| **内部调用** | `GetParamLabel()` -> `selinux_check_access()` |
| **返回值** | 0: 允许<br>负值: 拒绝 (AVC 拒绝) |
| **调用场景** | 参数服务收到 SET 请求时调用 |

**调用示例**:

```c
SrcInfo info = {0};
info.sockFd = clientFd;
info.uc.pid = getpid();
info.uc.uid = getuid();
info.uc.gid = getgid();

int ret = SetParamCheck("sys.selinux", "u:object_r:system_prop:s0", &info);
if (ret != 0) {
    // 权限拒绝
}
```

## 4. service_checker.h - SA 服务检查

**头文件**: `interfaces/policycoreutils/include/service_checker.h`

**产物**: `libservice_checker.so`

### 4.1 数据结构

```cpp
struct ServiceInfo {
    std::string serviceName;    // 服务名
    std::string serviceContext; // 服务的安全上下文
};

class ServiceChecker {
public:
    ServiceChecker(bool isHdf);  // isHdf: true=HDF, false=SA
    int ListServiceCheck(const std::string& callingSid);
    int GetServiceCheck(const std::string& callingSid, const std::string& serviceName);
    int AddServiceCheck(const std::string& callingSid, const std::string& serviceName);
    static ServiceChecker& GetInstance();
};
```

### 4.2 API 清单

| 函数名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `ListServiceCheck` | const std::string& callingSid | int | 列出服务权限检查 |
| `GetServiceCheck` | callingSid, serviceName | int | 获取服务权限检查 |
| `AddServiceCheck` | callingSid, serviceName | int | 添加服务权限检查 |

### 4.3 详细接口

#### ServiceChecker::GetServiceCheck

```cpp
int GetServiceCheck(const std::string& callingSid, const std::string& serviceName);
```

| 项目 | 说明 |
|------|------|
| **功能** | 检查调用者获取指定服务的权限 |
| **参数** | `callingSid`: 调用者的 Security Context<br>`serviceName`: 目标服务名 |
| **实现** | `framework/policycoreutils/src/service_checker.cpp` |
| **内部流程** | 1. 获取服务的 Security Context<br>2. 调用 `selinux_check_access()` |
| **返回值** | 0: 允许<br>负值: 拒绝 |
| **线程安全** | 单例模式，需外部同步 |

## 5. hdf_service_checker.h - HDF 服务检查

**头文件**: `interfaces/policycoreutils/include/hdf_service_checker.h`

**产物**: `libservice_checker.so`

### 5.1 API 清单

| 函数名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `HdfListServiceCheck` | const char *callingSid | int | HDF 服务列表权限检查 |
| `HdfGetServiceCheck` | const char *callingSid, const char *serviceName | int | HDF 服务获取权限检查 |
| `HdfAddServiceCheck` | const char *callingSid, const char *serviceName | int | HDF 服务添加权限检查 |

### 5.2 详细接口

#### HdfGetServiceCheck

```c
int HdfGetServiceCheck(const char *callingSid, const char *serviceName);
```

| 项目 | 说明 |
|------|------|
| **功能** | 检查调用者获取 HDF 服务的权限 |
| **参数** | `callingSid`: 调用者的 Security Context<br>`serviceName`: HDF 服务名 |
| **实现** | `framework/policycoreutils/src/service_checker.cpp` |
| **与 SA 区别** | HDF 服务使用独立的服务上下文文件 |

## 6. hap_restorecon.h - HAP 上下文恢复

**头文件**: `interfaces/policycoreutils/include/hap_restorecon.h`

**产物**: `libhap_restorecon.so`

### 6.1 数据结构

```cpp
struct SehapInfo {
    std::string apl;        // 应用权限级别
    std::string name;       // 应用名
    std::string domain;     // 进程域
    std::string type;       // 文件类型
    std::string extension;  // 扩展类型
    bool debuggable;        // 是否可调试
    uint64_t extra;         // 额外标志
};

struct HapFileInfo {
    std::vector<std::string> pathNameOrig;  // 原始路径
    std::string apl;
    std::string packageName;
    unsigned int flags;
    uint64_t hapFlags;
    uint32_t uid;
};

struct HapDomainInfo {
    std::string apl;
    std::string packageName;
    std::string extensionType;
    uint64_t hapFlags;
    uint32_t uid;
};

struct ResultInfo {
    uint32_t currentCount;  // 当前处理数
    uint32_t totalCount;    // 总数
};
```

### 6.2 API 清单

| 类/函数 | 功能 | 说明 |
|---------|------|------|
| `HapContext::HapFileRestorecon` | 文件标签恢复 | 恢复单个文件的安全上下文 |
| `HapContext::HapDomainSetcontext` | 域设置 | 设置应用进程的安全域 |
| `HapFileRestoreContext::GetInstance` | 获取单例 | 获取 HAP 文件恢复上下文 |
| `HapFileRestoreContext::SetFileConForce` | 强制设置 | 强制设置文件安全上下文 |
| `HapContextLoadConfig` | 加载配置 | 加载 sehap_contexts 配置 |

### 6.3 详细接口

#### HapFileRestoreContext::SetFileConForce

```cpp
int SetFileConForce(const HapFileInfo& hapFileInfo, 
                    const uint32_t remainingNum, 
                    ResultInfo& resultInfo);
```

| 项目 | 说明 |
|------|------|
| **功能** | 设置 HAP 文件的安全上下文（带进度反馈） |
| **参数** | `hapFileInfo`: HAP 文件信息<br>`remainingNum`: 剩余重试次数<br>`resultInfo`: 进度信息 |
| **实现** | `framework/policycoreutils/src/hap_restorecon.cpp` |
| **内部线程** | 创建独立的 RestoreTask 进行文件处理 |
| **返回值** | 0: 成功<br>负值: 失败 |
| **阻塞时间** | 大目录可能需要较长时间 |

## 7. 错误码说明

### 7.1 通用错误码

| 错误码 | 定义位置 | 说明 |
|--------|----------|------|
| 0 | - | 成功 |
| -1 | libselinux | 通用错误 |
| -2 | libselinux | 无效参数 |
| -3 | libselinux | 权限拒绝 (AVC) |

### 7.2 SELinux AVC 拒绝

当访问被 SELinux 拒绝时：
1. 日志输出: `audit: type=1400 ... avc: denied ...`
2. 返回值: `selinux_check_access()` 返回负值
3. 可通过 `dmesg` 或 `hilog` 查看详细拒绝信息

## 8. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概述 | [01_Overview.md](01_Overview.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 构建系统 | [04_Build.md](04_Build.md) |
| 安全评审 | [05_Security.md](05_Security.md) |
| 故障排查 | [06_Troubleshooting.md](06_Troubleshooting.md) |

# 安全风险评审

## 文档信息

| 项目 | 内容 |
|------|------|
| 目标读者 | 安全工程师、架构师 |
| 目的 | 识别和分析分布式文件服务的安全风险 |
| 评审范围 | interfaces/、frameworks/、services/（排除测试） |
| 代码证据 | `utils/system/src/dfsu_access_token_helper.cpp` |

## 威胁模型概述

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  信任边界内（高信任）                                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  • 系统服务进程（SA）                                   │  │
│  │  • Inner API 调用                                      │  │
│  │  • 框架层代码                                          │  │
│  └───────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  信任边界外（低信任）                                         │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  • 第三方应用（JS/NAPI 调用）                          │  │
│  │  • 跨设备请求                                          │  │
│  │  • 云端存储                                            │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 数据流风险等级

| 流向 | 风险等级 | 说明 |
|------|----------|------|
| 应用 → N-API | 中 | 需参数校验 |
| N-API → 框架层 | 低 | 内部调用 |
| 框架层 → SA | 低 | IPC 通信 |
| SA → 云端 | 高 | 需加密传输 |
| 设备间传输 | 高 | 需身份认证 |

---

## 攻击面清单

### 1. N-API 入口

| 入口 | 文件 | 风险点 |
|------|------|--------|
| file.cloudSync | `interfaces/kits/js/cloudfilesync/cloud_sync_napi.cpp` | 参数校验、权限检查 |
| cloudSyncManager | `interfaces/kits/js/cloudsyncmanager/cloud_sync_manager_napi.cpp` | 参数校验、权限检查 |
| NDK CloudDisk | `interfaces/kits/ndk/clouddiskmanager/src/oh_cloud_disk_manager.cpp` | C 接口安全 |

### 2. IPC 通信

| 服务 | SA ID | 风险点 |
|------|-------|--------|
| DistributedFileDaemon | 5201 | 远程设备请求认证 |
| CloudSyncService | 5204 | 跨账户数据泄露 |
| CloudDaemon | 5205 | FUSE 挂载点安全 |
| CloudDiskService | 5207 | 同步文件夹越权访问 |

### 3. 文件系统操作

| 操作 | 文件 | 风险点 |
|------|------|--------|
| 挂载点创建 | `services/distributedfiledaemon/src/mountpoint/mount_manager.cpp` | 路径遍历 |
| 文件读写 | `services/cloudfiledaemon/src/cloud_disk/` | 权限绕过 |
| 远程设备访问 | `services/distributedfiledaemon/src/network/softbus/` | 中间人攻击 |

### 4. 网络通信

| 组件 | 文件 | 风险点 |
|------|------|--------|
| SoftBus | `services/distributedfiledaemon/src/network/softbus/softbus_permission_check.cpp` | 设备认证 |
| 云端传输 | `services/cloudsyncservice/src/transport/` | TLS/加密 |

---

## 权限校验机制

### 统一权限检查类

**文件**：`utils/system/src/dfsu_access_token_helper.cpp`

**关键方法**：

| 方法 | 行号 | 说明 |
|------|------|------|
| `CheckCallerPermission` | 34-52 | 检查调用者权限 |
| `CheckPermission` | 54-62 | 验证权限 |
| `GetBundleNameByToken` | 70-108 | 获取包名 |
| `IsSystemApp` | 109-118 | 判断是否系统应用 |
| `GetUserId` | 120-124 | 获取用户 ID |
| `IsUserVerifyed` | 126-134 | 验证用户 |

**代码证据**：

```cpp
// utils/system/src/dfsu_access_token_helper.cpp:34-52
bool DfsuAccessTokenHelper::CheckCallerPermission(const std::string &permissionName)
{
    auto tokenId = IPCSkeleton::GetCallingTokenID();
    auto uid = IPCSkeleton::GetCallingUid();
    auto tokenType = Security::AccessToken::AccessTokenKit::GetTokenTypeFlag(tokenId);
    if (tokenType == TOKEN_HAP || tokenType == TOKEN_NATIVE) {
        bool isGranted = CheckPermission(tokenId, permissionName);
        if (!isGranted) {
            LOGE("Token Type is %{public}d", tokenType);
        }
        return isGranted;
    } else if ((tokenType == TOKEN_SHELL) && (uid == ROOT_UID)) {
        LOGI("Token type is shell");
        return false;
    } else {
        LOGE("Unsupported token type:%{public}d", tokenType);
        return false;
    }
}
```

### Token 类型支持

| Token 类型 | 处理方式 | 安全级别 |
|------------|----------|----------|
| `TOKEN_HAP` | 验证应用权限 | 中 |
| `TOKEN_NATIVE` | 验证原生权限 | 高 |
| `TOKEN_SHELL` | 拒绝（ROOT UID 除外） | 高 |

### 权限常量

| 权限名 | 定义位置 | 用途 |
|--------|----------|------|
| `ohos.permission.CLOUDFILE_SYNC` | N-API 层 | 云文件同步 |
| `ohos.permission.CLOUDFILE_SYNC_MANAGER` | N-API 层 | 云同步管理 |
| `ohos.permission.ACCESS_CLOUD_DISK_INFO` | NDK 层 | 访问云盘信息 |

---

## 已识别的安全风险

### 风险 1：路径遍历风险

**风险等级**：高

**证据**：`services/distributedfiledaemon/src/mountpoint/mount_manager.cpp`

**说明**：在挂载点创建过程中，路径参数未经过严格校验，可能导致路径遍历攻击。

**触发条件**：
1. 攻击者控制挂载路径参数
2. 路径包含 `../` 遍历序列
3. 挂载到非预期目录

**影响**：
- 任意目录挂载
- 权限提升
- 数据泄露

**修复建议**：
- 对路径参数进行规范化处理
- 使用白名单验证挂载点
- 限制挂载点目录

---

### 风险 2：跨设备认证缺失

**风险等级**：高

**证据**：`services/distributedfiledaemon/src/network/softbus/softbus_permission_check.cpp`

**代码证据**：

```cpp
// services/distributedfiledaemon/src/network/softbus/softbus_permission_check.cpp:42
bool SoftBusPermissionCheck::CheckSrcPermission(const std::string &sinkNetworkId, int32_t userId)
{
    AccountInfo localAccountInfo;
    if (!GetLocalAccountInfo(localAccountInfo, userId)) {
        LOGE("Get os account data failed");
        return false;
    }
    // ...
}
```

**说明**：设备间通信认证依赖本地账户验证，未验证远程设备的合法性。

**触发条件**：
1. 伪造网络 ID
2. 中间人攻击
3. 设备伪装

**影响**：
- 跨设备数据泄露
- 未授权文件访问
- 中间人攻击

**修复建议**：
- 增强设备证书验证
- 使用双向 TLS 认证
- 增加设备绑定校验

---

### 风险 3：IPC 消息验证不严格

**风险等级**：中

**证据**：`services/distributedfiledaemon/src/ipc/daemon_stub.cpp`

**代码证据**：

```cpp
// services/distributedfiledaemon/src/ipc/daemon_stub.cpp:127
if (!DfsuAccessTokenHelper::CheckCallerPermission(PERM_DISTRIBUTED_DATASYNC)) {
    LOGE("DATASYNC permission denied");
    return E_PERMISSION_DENIED;
}
```

**说明**：部分 IPC 消息仅验证权限，未验证消息内容的合法性。

**触发条件**：
1. 构造恶意 IPC 消息
2. 消息参数越界
3. 序列化/反序列化攻击

**影响**：
- 拒绝服务
- 内存损坏
- 权限绕过

**修复建议**：
- 对所有 IPC 参数进行严格校验
- 使用参数白名单
- 增加消息签名验证

---

### 风险 4：N-API 参数校验不完整

**风险等级**：中

**证据**：`interfaces/kits/js/cloudfilesync/cloud_sync_napi.cpp`

**代码证据**：

```cpp
// interfaces/kits/js/cloudfilesync/cloud_sync_napi.cpp:1112-1131
static int32_t CheckPermissions(const string &permission, bool isSystemApp)
{
    if (!permission.empty() && !DfsuAccessTokenHelper::CheckCallerPermission(permission)) {
        LOGE("permission denied");
        return E_PERMISSION_DENIED;
    }
    if (isSystemApp && !DfsuAccessTokenHelper::IsSystemApp()) {
        // ...
    }
}
```

**说明**：部分 N-API 接口参数校验不够全面。

**触发条件**：
1. 传入异常参数
2. 空指针解引用
3. 缓冲区溢出

**影响**：
- 拒绝服务
- 内存损坏
- 信息泄露

**修复建议**：
- 增加参数边界检查
- 使用安全字符串处理函数
- 增强异常处理

---

### 风险 5：FUSE 挂载点暴露

**风险等级**：中

**证据**：`services/cloudfiledaemon/src/fuse_manager/`

**说明**：FUSE 挂载点可能暴露给非预期进程。

**触发条件**：
1. 挂载点权限配置不当
2. 非授权进程访问挂载点

**影响**：
- 未授权文件访问
- 数据篡改

**修复建议**：
- 限制挂载点访问权限
- 使用 SELinux 策略控制
- 增加访问审计

---

## 安全最佳实践

### 1. 输入验证

- 所有 N-API 参数必须校验
- IPC 消息需验证来源和完整性
- 文件路径需规范化处理
- 使用白名单验证输入

### 2. 访问控制

- 使用 `DfsuAccessTokenHelper` 统一校验
- 遵循最小权限原则
- 区分系统应用和普通应用
- 支持 Token 类型验证

### 3. 传输安全

- 设备间通信使用 SoftBus 加密
- 云端传输使用 TLS
- 敏感数据加密存储

### 4. 日志安全

- 避免日志敏感信息
- 使用分级日志策略
- 审计关键操作

---

## 安全配置

### SELinux 上下文

| 服务 | 上下文 | 说明 |
|------|--------|------|
| distributedfiledaemon | `u:r:distributedfiledaemon:s0` | 分布式文件守护 |
| cloudfiledaemon | `u:r:cloudfiledaemon:s0` | 云文件守护 |
| clouddiskservice | `u:r:clouddiskservice:s0` | 云盘服务 |

### 能力配置

| 服务 | 能力 | 说明 |
|------|------|------|
| distributedfiledaemon | `DAC_READ_SEARCH, CHOWN, NET_RAW` | 文件和网络能力 |
| cloudfiledaemon | `CHOWN` | 文件所属权修改 |
| clouddiskservice | `CAP_SYS_ADMIN, CAP_DAC_READ_SEARCH` | 管理员能力 |

### 配置文件权限

| 配置文件 | UID:GID | 权限 | 说明 |
|----------|----------|------|------|
| distributedfiledaemon.cfg | dfs:dfs | 0711 | 服务配置 |
| cloudfiledaemon.cfg | 1009:dfs | 0711 | 云文件配置 |

---

## 相关跳转

- 架构设计：[01_Architecture.md](./01_Architecture.md)
- 对外接口：[02_N-API.md](./02_N-API.md)
- 配置说明：[appendix/Config.md](./appendix/Config.md)

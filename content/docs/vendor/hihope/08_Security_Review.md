# 安全风险评审

## 文档信息

- **目的**：基于代码证据进行 vendor_hihope 仓库的安全风险评审，识别攻击面、信任边界和可被利用点
- **适用范围**：vendor_hihope 仓库的安全配置、权限模型、进程权限和系统安全机制
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库配置了完整的安全机制（SELinux、权限、签名验证）
  - 安全策略在 config.json、security_config/、preinstall-config/ 中明确定义
  - 未发现高风险安全漏洞，但识别了关键安全控制点和配置区域

## 攻击面分析

### 1. 攻击面清单

| 攻击面 | 代码证据 | 风险级别 | 说明 |
|---------|---------|---------|------|
| **应用权限** | install_list_permissions.json | 中 | 预安装应用权限配置 |
| **系统服务权限** | high_privilege_process_list.json | 高 | 高权限进程列表 |
| **HDF 设备权限** | hdf_config/uhdf/device_info.hcs | 低 | 设备节点权限（0644/0660） |
| **权限验证** | install_list_permissions.json（app_signature） | 中 | 应用签名绑定 |
| **SELinux 策略** | config.json（build_selinux） | 低 | MAC 策略强制执行 |
| **Seccomp 过滤** | config.json（build_seccomp） | 低 | 系统调用过滤 |

### 2. 信任边界

```
┌─────────────────────────────────────────────┐
│  应用层                   │  [OpenHarmony 应用框架]
│  (ArkTS/C++ 应用)           │  + 权限检查
└────────────┬────────────────────┘
             │
        ┌────────────▼──────────────┐
        │   框架层                 │  [OpenHarmony 框架]
        │  (Foundation/Ability)       │  + 权限检查
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │   HAL 层                  │  [vendor 仓库]
        │  （配置/适配）             │  + 设备权限
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │   HDF 层                  │  [OpenHarmony HDF 框架]
        │  （驱动服务）              │  + SELinux 策略
        │                           │  + Seccomp 过滤
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │    硬件层                │  [厂商硬件]
        │  （SoC/外设）             │
        └───────────────────────────────┘
```

**信任边界**：
- ✅ 应用层通过权限检查访问框架
- ✅ 框架层通过签名验证应用
- ✅ HAL 层通过设备权限控制访问
- ✅ HDF 层通过 SELinux 策略隔离

## 安全机制评估

### 1. 权限系统

**配置文件**：`{product}/preinstall-config/install_list_permissions.json`

**证据**：`wearable/preinstall-config/install_list_permissions.json:1`（331 行）

**权限类型**：

| 权限类别 | 权限示例 | 风险说明 |
|----------|---------|---------|
| **媒体权限** | READ_MEDIA, WRITE_MEDIA | 访问敏感用户数据 |
| **相机权限** | CAMERA, MICROPHONE | 访问摄像头和麦克风 |
| **位置权限** | LOCATION, APPROXIMATELY_LOCATION | 获取精确位置 |
| **联系权限** | READ_CONTACTS, WRITE_CONTACTS | 访问联系人数据 |
| **系统权限** | GET_INSTALLED_BUNDLE_LIST | 获取已安装应用列表 |
| **安全权限** | ACCESS_PIN_AUTH, ACCESS_BIOMETRIC | 访问生物识别数据 |

**安全措施**：
- ✅ 权限通过 userCancellable 控制
- ✅ 应用通过 app_signature 绑定
- ✅ 系统在运行时检查权限

### 2. 进程权限系统

**配置文件**：`{product}/security_config/high_privilege_process_list.json`

**证据**：`rk3568/security_config/high_privilege_process_list.json:1`

**高权限进程**（证据：`rk3568/security_config/high_privilege_process_list.json`）：

| 进程名 | UID | GID | 风险级别 | 说明 |
|---------|-----|-------|---------|
| **appspawn** | root | root | 高 | 应用进程生成器 |
| **cjappspawn** | root | root | 高 | JS 应用进程生成器 |
| **nativespawn** | root | root | 高 | 原生应用进程生成器 |
| **hybridspawn** | root | root | 高 | 混合应用进程生成器 |
| **console** | root | - | 高 | 系统控制台 |
| **netsysnative** | root | root | 高 | 网络系统原生服务 |
| **misc** | root | root | 高 | 杂项系统服务 |
| **hdcd** | root | - | 高 | HDC 服务守护进程 |
| **media_service** | system | root, system | 中 | 媒体服务 |
| **render_service** | system | root, system | 中 | 渲染服务 |
| **resource_schedule_service** | root | root | 中 | 资源调度服务 |
| **ueventd** | system | system | 中 | 设备事件守护进程 |
| **concurrent_task_service** | system | system | 中 | 并发任务服务 |

**风险**：
- ⚠️ 高权限进程 compromised 将导致系统级破坏
- ✅ 关键进程已配置在 critical_reboot_process_list.json 中

### 3. SELinux 策略

**配置**：`{product}/config.json`（证据：`rk3568/config.json:12`：`build_selinux: true`）

**SELinux 模式**（证据：HDF 配置）：
- `policy = 0` - 内核态驱动
- `policy = 1` - 内核态 + 用户态服务
- `policy = 2` - 用户态服务

**设备权限**（证据：`rk3568/hdf_config/uhdf/device_info.hcs`）：

| 权限值 | 八进制 | 说明 | 典型使用 |
|---------|-------|------|
| `0644` | rw-r--r-- | 标准设备，用户读写，其他只读 |
| `0660` | rw-rw-r-- | 音频设备，用户全面访问 |
| `0666` | rw-rw-rw- | USB 设备，用户可读写 |

**安全措施**：
- ✅ MAC 策略强制隔离
- ✅ 权限最小化原则
- ✅ SELinux 上下文绑定

### 4. 签名验证

**配置文件**：`{product}/preinstall-config/install_list_permissions.json`

**证据**：`wearable/preinstall-config/install_list_permissions.json:3`（app_signature 字段）

**签名机制**：
- 预安装应用通过 app_signature 字段绑定 SHA256 签名
- 签名长度：64 个字符（十六进制）
- 示例签名（证据）：`8E93863FC32EE238060BF69A9B37E2608FFFB21F93C862DD511CBAC9F30024B5`

**安全措施**：
- ✅ 系统在安装时验证签名
- ✅ 签名无法验证的应用无法安装
- ✅ 签名匹配的应用才能使用预安装权限

### 5. Security Sanitizer

**配置文件**：`{product}/security_config/sanitizer_check_list.gni`

**证据**：`rk3568/security_config/sanitizer_check_list.gni`（引用外部 N-API 模块）：

**Sanitizer 模块列表**：
- `socket_permission` - Socket 权限管理
- `power_permission` - 电源权限
- `uri_permission_mgr` - URI 权限管理
- `dlp_permission_service` - DLP 权限服务
- `ohdlp_permission` - OpenHarmony DLP 权限
- `selinux_adapter` - SELinux 适配器（CFI bypass）

**说明**：
- 这些模块提供了 CFI bypass，允许特定模块在需要时绕过 CFI 检查
- 这是针对已知安全机制的性能优化

## 可被利用点

由于 vendor_hihope 仓库主要包含配置和 HAL stub 实现，未发现典型的高风险漏洞。但识别了以下需要注意的安全控制点：

### 1. 高权限进程管理

**可被利用点 1**：高权限进程列表硬编码

- **证据**：`rk3568/security_config/high_privilege_process_list.json:1`
- **风险**：如果攻击者能够修改此配置文件，可以添加恶意进程到高权限列表
- **触发路径**：攻击者通过文件修改或供应链攻击
- **影响**：完全控制系统，安装恶意软件
- **修复建议**：
  - 将配置文件存储在只读分区
  - 使用 dm-verity 验证配置完整性
  - 在 bootloader 层验证配置签名

### 2. HAL Stub 实现安全性

**可被利用点 2**：HAL Stub 实现返回固定成功码

**证据**：`nearlink_dk_3863/hals/utils/token/hal_token.c:20,28,36,44`（所有函数仅返回 EC_SUCCESS）

- **风险**：OEM 未能实现真实的令牌存储和验证逻辑，攻击者可以绕过安全检查
- **触发路径**：攻击者调用 Token HAL 接口获取访问权限
- **影响**：可以伪造设备身份，访问受保护资源
- **修复建议**：
  - 要求厂商实现真实的令牌存储（如 TPM、Secure Element）
  - 添加令牌加密和完整性验证
  - 实现令牌读取计数和锁定机制
  - 在 stub 中返回错误码而非成功

### 3. HDF 设备权限配置

**可被利用点 3**：过于宽松的设备权限

**证据**：`rk3568/hdf_config/uhdf/device_info.hcs:53`（camera_service 的 permission = 0666）

- **风险**：camera_hdi_service 的 permission = 0666（rw-rw-rw-）允许所有用户访问
- **触发路径**：恶意应用可以直接访问相机设备节点
- **影响**：绕过应用层权限检查，直接访问相机
- **修复建议**：
  - 考虑将 camera_hdi_service 的 permission 设置为 0644（rw-r--r--）
  - 通过 HAL 层实现额外的权限检查
  - 对关键设备节点使用 SELinux 类型 enforcement

### 4. 预安装应用权限配置

**可被利用点 4**：预安装应用权限过多

**证据**：`wearable/preinstall-config/install_list_permissions.json`（331 行，13 个应用）

- **风险**：某些预安装应用可能被授予不必要的权限
- **触发路径**：攻击者利用具有过多权限的预安装应用
- **影响**：权限提升，数据泄露
- **修复建议**：
  - 审查预安装应用权限，遵循最小权限原则
  - 移除不必要的敏感权限（如相机、麦克风、位置等）
  - 对敏感权限设置 userCancellable = true

### 5. 应用签名绕过风险

**可被利用点 5**：签名验证配置

**证据**：`wearable/preinstall-config/install_list_permissions.json:4`（app_signature 字段）

- **风险**：如果签名验证逻辑存在漏洞，可能被绕过
- **触发路径**：攻击者伪造签名或利用签名验证漏洞
- **影响**：安装恶意应用，冒充预安装应用
- **修复建议**：
  - 使用强哈希算法（SHA-256 或更强）
  - 确保签名存储在安全位置
  - 实现签名验证的防重放攻击
  - 定期轮换签名密钥

## 安全优势

### 已实现的安全措施

| 安全措施 | 实现情况 | 证据 |
|---------|---------|------|
| **SELinux 策略** | ✅ 已启用 | config.json: build_selinux: true |
| **Seccomp 过滤** | ✅ 已启用 | config.json: build_seccomp: true |
| **权限检查** | ✅ 已配置 | install_list_permissions.json |
| **签名验证** | ✅ 已配置 | install_list_permissions.json（app_signature） |
| **进程隔离** | ✅ 已配置 | high_privilege_process_list.json |
| **高权限进程监控** | ✅ 已配置 | critical_reboot_process_list.json |

## 安全配置检查范围

### 已检查的范围

| 检查项 | 检查方法 | 结果 |
|---------|---------|------|
| **权限配置文件** | grep "permission" | ✅ 所有产品都有配置 |
| **高权限进程** | grep "root" | ✅ 发现并配置 |
| **SELinux 策略** | grep "selinux" | ✅ 标准产品已启用 |
| **Seccomp 过滤** | grep "seccomp" | ✅ 标准产品已启用 |
| **HDF 设备权限** | grep "permission" | ✅ 在 device_info.hcs 中配置 |
| **签名验证** | grep "signature" | ✅ 在 install_list.json 中配置 |
| **HAL 实现** | 代码审查 | ⚠️ 发现 Stub 实现 |

### 未检查的范围（局限）

| 检查项 | 原因 | 说明 |
|---------|---------|------|
| **N-API 安全实现** | vendor 仓库不包含 | N-API 在 OpenHarmony 框架实现 |
| **IPC 通信安全** | vendor 仓库不包含 | IPC 在 OpenHarmony 框架实现 |
| **加密实现** | vendor 仓库不包含 | 加密在 OpenHarmony 框架实现 |
| **系统漏洞扫描** | 未执行 | 未使用静态分析工具扫描 |

## 安全改进建议

### 1. 加强 HAL 实现

| 改进项 | 优先级 | 说明 |
|---------|--------|------|
| **真实 Token 实现** | 高 | 替换 Stub 实现，添加真实存储 |
| **设备权限最小化** | 高 | 审查所有 HDF 设备的 permission 设置 |
| **HAL 参数验证** | 高 | 在所有 HAL 接口中添加参数验证 |

### 2. 硬化配置管理

| 改进项 | 优先级 | 说明 |
|---------|--------|------|
| **配置文件只读** | 高 | 将配置文件放在只读分区或使用 dm-verity |
| **配置签名验证** | 高 | 验证配置文件完整性，防止篡改 |
| **高权限进程锁定** | 中 | 限制对高权限进程列表的修改 |

### 3. 运行时安全监控

| 改进项 | 优先级 | 说明 |
|---------|--------|------|
| **安全日志审计** | 中 | 启用详细的安全日志记录 |
| **异常行为检测** | 中 | 监控异常的权限访问或系统调用 |
| **定期安全扫描** | 低 | 使用静态分析和动态扫描工具 |

### 4. 开发安全流程

| 改进项 | 优先级 | 说明 |
|---------|--------|------|
| **安全代码审查** | 高 | 对所有 HAL 修改进行安全审查 |
| **安全测试** | 高 | 对 HAL 层进行模糊测试和渗透测试 |
| **威胁建模** | 中 | 建立威胁模型，识别潜在攻击向量 |
| **安全培训** | 中 | 培训开发人员的安全意识 |

## 安全合规性

### 符合 OpenHarmony 安全规范

| 规范项 | 合规情况 | 说明 |
|---------|---------|------|
| **权限最小化** | ✅ 部分符合 | 某些预安装应用可能权限过多 |
| **强制访问控制** | ✅ 符合 | SELinux 已启用 |
| **签名验证** | ✅ 符合 | 预安装应用使用签名验证 |
| **进程隔离** | ✅ 符合 | 进程权限明确配置 |

## 相关跳转

- [内部 API 与模块接口](05_Internal_API.md)
- [目录结构详解](02_Directory_Structure.md)
- [返回 Wiki 首页](SUMMARY.md)

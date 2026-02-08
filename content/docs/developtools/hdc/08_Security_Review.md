# 安全风险评审

## 目的

描述 hdc 项目的安全威胁模型、攻击面、信任边界和可被利用点，提供修复建议。

## 适用范围

本文档适用于：
- 安全审计
- 了解安全风险
- 实施安全加固措施
- 合规性评估

## 相关跳转

- [项目概览](./00_Overview.md) - 项目定位和认证机制
- [内部 API](./05_Internal_API.md) - 模块接口和权限
- [常见问题](./09_FAQ.md) - 安全相关问题

---

## 攻击面分析

### 1. 网络攻击面

#### 1.1 USB 攻击面

**范围**：USB 通信接口

**潜在攻击**：
- 恶意 USB 设备伪装（使用有效厂商 ID）
- USB 协议栈溢出攻击
- USB 拒绝服务（DoS）

**证据**：
- USB 设备发现：`src/common/usb.cpp`
- USB 协议头：`src/common/define_plus.h:78-83`

**当前防护**：
- ✅ 使用 libusb 库（已知的 USB 安全实现）
- ✅ USB 协议头大小固定（`USBHead` 结构，pack(1)）
- ⚠️ 缺少 USB 设备认证（仅依赖操作系统）

#### 1.2 TCP 攻击面

**范围**：TCP 网络通信

**潜在攻击**：
- 中间人攻击（MITM）
- TCP 连接劫持
- 端口扫描
- TCP RST 攻击

**证据**：
- TCP 监听：`src/daemon/daemon_tcp.cpp:135-136`
- TCP 连接：`src/host/host_tcp.cpp`

**当前防护**：
- ✅ RSA 公钥认证（`src/common/auth.cpp`）
- ✅ TLS 1.3 PSK 加密（`src/common/hdc_ssl.cpp`）
- ⚠️ 默认未启用 TLS（需手动配置）

#### 1.3 Unix Domain Socket 攻击面

**范围**：UDS 通信

**潜在攻击**：
- Socket 文件路径注入
- 权限提升（通过符号链接）
- 本地拒绝服务

**证据**：
- UDS 路径：`src/common/define.h:151` `/data/hdc/hdc_debug/hdc_server`
- UDS 创建：`src/host/server_for_client.cpp:201-218`

**当前防护**：
- ✅ 固定 UDS 路径（不可变）
- ⚠️ 依赖文件系统权限（DAC）

### 2. 文件操作攻击面

#### 2.1 文件路径遍历

**范围**：文件传输命令

**潜在攻击**：
- 路径遍历攻击（`../`）
- 符号链接攻击
- 路径规范化攻击

**证据**：
- Bundle 路径验证：`src/common/base.cpp:2730-2756` - `CheckBundleName()` 使用正则表达式验证
- Bundle 路径验证：`src/daemon/daemon_unity.cpp:77-94` - `CheckbundlePath()` 检查文件访问

**当前防护**：
- ✅ Bundle 名称验证：`^[0-9a-zA-Z_\.]+$`（仅字母数字、下划线、点）
- ⚠️ 路径验证可能不完整（缺少 Unicode 规范化检查）

#### 2.2 文件权限提升

**潜在攻击**：
- 通过文件传输覆盖系统文件
- 通过符号链接访问受限文件
- 修改文件权限

**证据**：
- 文件模式验证：`src/common/define_enum.h`
- Tar 头解析：`src/common/header.cpp:64-112` - `Header` 类处理权限

**当前防护**：
- ⚠️ 依赖 DAC（Discretionary Access Control）
- ⚠️ Root 操作需要 sudo 工具

### 3. 命令执行攻击面

#### 3.1 Shell 注入

**范围**：Shell 命令执行

**潜在攻击**：
- 命令注入攻击
- 环境变量注入
- Shell 元字符注入

**证据**：
- Shell 执行：`src/daemon/shell.cpp`
- 命令数据：`src/common/define_enum.h:2000-2099`

**当前防护**：
- ⚠️ 缺少命令过滤和参数验证（TODO: 需确认）

### 4. 认证与授权攻击面

#### 4.1 认证绕过

**范围**：认证机制

**潜在攻击**：
- 认证绕过攻击
- 已知主机文件篡改
- 降级攻击

**证据**：
- 认证类型：`src/common/session.h:27` - `AUTH_NONE`, `AUTH_TOKEN`, `AUTH_SIGNATURE`, `AUTH_PUBLICKEY`, `AUTH_OK`, `AUTH_FAIL`
- 认证配置检查：`src/daemon/daemon.cpp:120-166` - `GetAuthByPassValue()`

**配置参数**：
- `const.hdc.secure` - 安全模式
- `const.boot.oemmode` - OEM 锁机模式
- `persist.hdc.daemon.auth_result` - 认证结果
- `persist.hdc.daemon.auth_msg` - 认证消息

**当前防护**：
- ✅ RSA 签名验证（SHA512）
- ✅ 已知主机检查（`AlreadyInKnownHosts()`）
- ✅ 连接验证支持（`connect_validation.cpp`）
- ⚠️ 认证绕过检测依赖于系统参数（可能被篡改）

#### 4.2 权限提升

**范围**：sudo 和权限管理

**潜在攻击**：
- PIN 暴力破解
- 权限提升攻击
- Token 篡改

**证据**：
- sudo 实现：`src/sudo/main.cpp`
- PIN 认证：`src/sudo/sudo_iam.cpp`
- 用户权限助手：`src/hdcd_user_permit/connection.cpp`

**当前防护**：
- ✅ PIN 认证使用 `AuthTrustLevel::ATL3`（最高信任级别）
- ✅ SELinux 上下文切换（`u:r:sudo_execv_label:s0`）
- ✅ 进程安全级别设置（`SetProcessLevelByCommand()`）

---

## 可被利用点

### 风险点 1：TLS 加密未默认启用

**风险等级**：⚠️ 中等

**证据**：
- 代码位置：`src/common/hdc_ssl.cpp`
- TLS PSK 加密实现：`HdcSSLBase::InputPsk()`, `HdcSSLBase::GenPsk()`
- 默认配置：TLS 仅在显式配置时启用（非默认）

**触发路径**：
1. 攻击者使用网络嗅探工具（如 Wireshark）捕获 TCP 流量
2. 如果 TLS 未启用，攻击者可以直接读取明文数据
3. 包括文件传输内容、Shell 命令等敏感信息

**影响**：
- 敏感数据泄露（文件内容、命令输出）
- 中间人攻击
- 会话劫持

**修复建议**：
- **短期**：在文档中明确 TLS 配置要求
- **长期**：考虑默认启用 TLS（需要性能和安全平衡）

---

### 风险点 2：USB 设备认证缺失

**风险等级**：⚠️ 中等

**证据**：
- 代码位置：`src/common/usb.cpp`, `src/host/host_usb.cpp`, `src/daemon/daemon_usb.cpp`
- USB 设备发现：`AdminUsbSession()`, `EnumUSBDeviceRegister()`
- 缺少：USB 设备级别的认证机制

**触发路径**：
1. 攻击者连接伪造的 USB 设备（使用有效厂商 ID 和产品 ID）
2. 由于没有设备认证，操作系统直接接受设备
3. HDC 与伪造设备建立连接
4. 攻击者可以控制设备端的 HDC Daemon

**影响**：
- 设备完全被控制
- 任意代码执行
- 数据泄露

**修复建议**：
- 实现 USB 设备认证（使用设备证书或挑战-响应机制）
- 在 OS 层添加 USB 设备白名单机制
- 记录可疑 USB 设备连接事件

---

### 风险点 3：Bundle 路径验证不完整

**风险等级**：⚠️ 中等

**证据**：
- 代码位置：`src/common/base.cpp:2730-2756` - `CheckBundleName()`
- 正则表达式：`^[0-9a-zA-Z_\.]+$`
- Bundle 路径验证：`src/daemon/daemon_unity.cpp:77-94` - `CheckbundlePath()`

**问题**：
- 未验证路径中的 `..`（路径遍历）
- 未验证路径中的 Unicode 规范化字符
- 未验证绝对路径与相对路径

**触发路径**：
1. 攻击者通过 `file send` 命令发送包含 `..` 的路径
2. 例如：`/data/hdc/../sensitive_file`
3. 系统规范化后，路径指向预期外的文件
4. 攻击者可以读取或覆盖敏感文件

**影响**：
- 信息泄露（读取敏感文件）
- 数据篡改（覆盖关键文件）
- 拒绝服务（覆盖配置文件）

**修复建议**：
- 实现路径规范化函数（如 `realpath()`, `canonicalize_path()`）
- 检查路径中包含 `..` 或绝对路径
- 使用安全的路径拼接函数
- 限制文件操作到特定目录白名单

---

### 风险点 4：Shell 命令注入

**风险等级**：⚠️ 中等

**证据**：
- 代码位置：`src/daemon/shell.cpp`
- Shell 执行实现：TODO(需确认) - 应使用 `exec()` 系列函数而非 `system()`
- 命令数据：`src/common/define_enum.h:2000-2099`

**问题**：
- 缺少输入验证和过滤
- 可能使用不安全的 `system()` 而非 `exec()`
- 缺少命令白名单机制

**触发路径**：
1. 攻击者通过 `shell` 命令发送恶意输入
2. 例如：`shell cmd1; rm -rf /`（命令链注入）
3. 恶意命令直接在设备端执行

**影响**：
- 任意命令执行
- 设备完全控制
- 数据破坏

**修复建议**：
- 实现 Shell 命令白名单（仅允许特定命令）
- 使用安全的 `exec()` 替代 `system()`
- 对输入进行严格的参数验证和转义
- 限制 Shell 模式为非交互模式（如适用）

---

### 风险点 5：认证参数可被系统级篡改

**风险等级**：🔴 高

**证据**：
- 代码位置：`src/daemon/daemon.cpp:120-166`
- 参数读取：`SystemDepend::GetDevItem("const.hdc.secure", secure)`
- 参数读取：`SystemDepend::GetDevItem("const.boot.oemmode", oemmode)`
- 参数读取：`SystemDepend::GetDevItem("persist.hdc.daemon.auth_result", auth_result)`

**问题**：
- 认证参数存储在系统参数中（`param` 框架）
- 具有系统权限的进程可以修改这些参数
- 参数修改后无需重启 hdcd 即可生效（如果读取方式不当）

**触发路径**：
1. 攻击者获取 root 权限
2. 修改 `const.hdc.secure` 为 "0"（禁用认证）
3. 修改 `persist.hdc.daemon.auth_result` 为允许结果
4. 攻击者可以直接连接设备（无需认证）

**影响**：
- 完全绕过认证机制
- 未授权设备访问
- 设备完全控制

**修复建议**：
- **关键**：将认证参数移到安全存储（如 HUKS 密钥库）
- 使用 HUKS 或类似安全存储保护关键参数
- 在参数读取后立即验证其完整性（签名/哈希）
- 考虑添加认证参数变更审计日志
- 限制参数修改为设备启动前或特定维护窗口

---

## 信任边界

### 信任域 1：PC 端（Client/Server）

**边界**：
- 可信基座：开发机 PC
- 不可信输入：网络数据、文件数据

**信任假设**：
- ✅ 假设 PC 端环境安全（开发者控制）
- ✅ 假设文件系统受 DAC 保护
- ⚠️ 不假设网络链路安全（需 TLS）

### 信任域 2：设备端（Daemon）

**边界**：
- 可信基座：OpenHarmony 系统环境
- 不可信输入：PC 端通过网络发送的数据

**信任假设**：
- ✅ 假设 OpenHarmony 系统完整性
- ✅ 假设 SELinux 保护生效
- ✅ 假设 DAC 权限有效
- ⚠️ 不假设 USB 设备可信（可能被伪造）

### 信任域 3：认证通道

**边界**：
- 认证通道：USB/TCP 网络连接
- 信任决策：RSA 公钥验证、TLS 握手

**信任假设**：
- ✅ 假设 RSA 密钥长度足够（2048 位）
- ✅ 假设 TLS 1.3 加密强度（AES-128-GCM）
- ⚠️ 不假设参数框架安全（见风险点 5）

---

## 安全控制总结

### 已实施的安全机制

| 机制 | 状态 | 说明 |
|--------|------|------|
| RSA 公钥认证 | ✅ 已实施 | 2048 位密钥，SHA512 签名验证 |
| TLS 1.3 PSK 加密 | ✅ 已实施 | AES-128-GCM，可选启用 |
| 已知主机检查 | ✅ 已实施 | `/data/service/el1/public/hdc/hdc_keys` |
| 连接验证 | ✅ 已实施 | 基于系统参数的验证 |
| Bundle 名称验证 | ✅ 已实施 | 正则表达式检查 |
| PIN 认证 | ✅ 已实施 | ATL3 信任级别 |
| SELinux 保护 | ✅ 已实施 | 策略配置 |
| DAC 权限 | ✅ 已实施 | `.dac` 文件 |

### 安全弱点

| 弱点 | 风险等级 | 状态 |
|--------|----------|------|
| TLS 未默认启用 | ⚠️ 中等 | 配置问题 |
| USB 设备认证缺失 | ⚠️ 中等 | 依赖 OS 层 |
| 路径遍历防护不完整 | ⚠️ 中等 | 缺少规范化 |
| Shell 命令注入防护 | ⚠️ 中等 | 缺少验证和过滤 |
| 认证参数可篡改 | 🔴 高 | 存储在参数框架中 |

---

## 安全加固建议

### 短期（1-3 个月）

1. **完善路径验证**
   - 实现 `realpath()` 规范化
   - 检查 `..` 遍历
   - 限制操作目录白名单

2. **强化 Shell 命令执行**
   - 使用 `exec()` 替代 `system()`
   - 实现命令白名单
   - 输入转义和验证

3. **启用 TLS 日志**
   - 记录 TLS 握手失败
   - 记录明文通信警告
   - 监控加密通道使用情况

### 中期（3-6 个月）

1. **实现 USB 设备认证**
   - 设备证书验证
   - 设备白名单机制
   - 设备指纹校验

2. **迁移认证参数到安全存储**
   - 使用 HUKS 存储认证参数
   - 添加完整性保护（签名）
   - 实现参数变更审计

3. **增强网络层安全**
   - 考虑默认启用 TLS（评估性能影响）
   - 实现证书固定（Certificate Pinning）
   - 添加网络流量监控

### 长期（6-12 个月）

1. **实现审计日志**
   - 记录所有认证事件
   - 记录敏感操作（文件传输、命令执行）
   - 异常行为检测和告警

2. **入侵检测系统**
   - 检测异常连接模式
   - 检测异常文件操作
   - 与 OpenHarmony 安全中心集成

---

## 合规性评估

### 遵循的安全标准

✅ **Apache License 2.0**：开源许可
✅ **OpenHarmony 安全规范**：遵循系统安全要求
✅ **SELinux 策略**：实现访问控制
✅ **DAC 权限**：实现文件权限保护

---

## 关键结论

1. **攻击面清晰**：网络（USB/TCP）、文件操作、命令执行、认证
2. **主要风险**：
   - 🔴 认证参数可被篡改（高危）
   - ⚠️ TLS 加密未默认启用（中等）
   - ⚠️ 路径遍历防护不完整（中等）
   - ⚠️ Shell 命令注入防护不足（中等）
   - ⚠️ USB 设备认证缺失（中等）

3. **安全机制已实施**：
   - RSA 公钥认证（2048 位）
   - TLS 1.3 PSK 加密（AES-128-GCM）
   - 已知主机检查
   - SELinux 保护
   - DAC 权限

4. **优先修复项**：
   - 立即：迁移认证参数到 HUKS 安全存储
   - 短期：完善路径验证和 Shell 安全
   - 中期：实现 USB 设备认证
   - 长期：默认启用 TLS（评估后）

---

## 待确认事项

**TODO(需确认)**：
1. Shell 命令执行的具体实现细节（system vs exec）
2. 认证参数的完整性保护机制（是否有签名）
3. 网络嗅探防护措施（如是否有混淆）
4. 异常行为检测规则的定义

# OpenHarmony Print Scan Framework - 安全风险评审

**目的**: 基于代码证据识别潜在安全风险、攻击面和信任边界

**适用范围**: OpenHarmony Print Scan Framework 3.1

**检查范围**: 框架核心代码（不包括测试和第三方库）

---

## 目录

- [威胁模型](#威胁模型)
- [攻击面清单](#攻击面清单)
- [信任边界](#信任边界)
- [可被利用点](#可被利用点)
- [输入验证检查](#输入验证检查)
- [权限检查](#权限检查)

---

## 威胁模型

### 外部输入 → 敏感操作

```
JS 应用（第三方应用）
    ↓
N-API 接口（print_napi, scan_napi）
    ↓
参数解析和验证
    ↓
IPC 调用（PrintServiceAbility, ScanServiceAbility）
    ↓
权限检查
    ↓
核心业务逻辑
    ↓
文件操作 / CUPS 调用 / 设备通信
```

### 风险分类

| 风险类别 | 说明 | 严重程度 |
|-----------|------|---------|
| **输入验证** | 未验证或弱验证的用户输入 | 高 |
| **权限绕过** | 权限检查不充分或可被绕过 | 高 |
| **路径遍历** | 文件路径处理不当 | 中 |
| **注入攻击** | IPC 或配置注入 | 高 |
| **信息泄露** | 敏感信息暴露给未授权方 | 中 |
| **拒绝服务** | 资源耗尽导致拒绝服务 | 中 |
| **权限提升** | 某些操作可能导致权限提升 | 低 |

---

## 攻击面清单

### 1. N-API 接口攻击面

**证据**: `interfaces/kits/napi/print_napi/src/`, `interfaces/kits/napi/scan_napi/src/`

| 接口 | 攻击向量 | 证据 |
|--------|---------|------|
| `print()`, `startPrintJob()`, `cancelPrintJob()` | **参数注入**: 打印任务参数未充分验证 | 用户可构造恶意参数 |
| | | **资源耗尽**: 提交大量打印任务 | 恶意应用可导致服务过载 |
| `connectPrinter()`, `disconnectPrinter()` | **状态篡改**: 连接/断开未授权打印机 | 可能劫持其他用户的打印机 |
| `startDiscoverPrinter()`, `addPrinters()` | **信息泄露**: 枚举系统打印机列表 | 获取敏感设备信息 |
| `queryAllPrintJobs()`, `queryPrintJobById()` | **信息泄露**: 查询其他用户打印任务 | 获取敏感任务信息 |
| `init()`, `startScan()`, `openScanner()` | **参数注入**: 扫描参数未充分验证 | 恶意扫描参数可能导致漏洞 |

### 2. IPC 通信攻击面

**证据**: `frameworks/innerkitsimpl/print_impl/src/print_service_proxy.cpp`, `frameworks/innerkitsimpl/scan_impl/src/scan_service_proxy.cpp`

| 接口 | 攻击向量 | 证据 |
|--------|---------|------|
| **PrintServiceProxy** | **IPC 劫持**: 如果攻击者可以控制 IPC 通道 | 可伪造服务端响应 |
| | | **权限提升**: 通过 IPC 调用未授权的系统方法 | 某些 IPC 方法权限检查不足 |
| **ScanServiceProxy** | **IPC 劫持**: 扫描服务 IPC 通信 | 可伪造扫描设备信息 |

### 3. 文件操作攻击面

**证据**: `services/print_service/src/print_user_data.cpp`, `services/print_service/src/print_system_data.cpp`

| 操作 | 攻击向量 | 证据 |
|--------|---------|------|
| **打印机列表存储** | **路径遍历**: `PRINTER_SERVICE_FILE_PATH = "/data/service/el2/public/print_service/printers"` | 路径未验证，可能访问其他目录 |
| | | **符号链接攻击**: 写入恶意链接 | 可能指向敏感文件 |
| **打印任务数据** | **信息泄露**: 任务信息明文存储 | 包含文件路径等敏感信息 |
| **SANE 临时目录** | **路径遍历**: `PRINTER_SERVICE_SANE_TEMPORARY_PATH = "/data/service/el2/public/print_service/sane/tmp"` | 临时文件可能残留敏感信息 |

### 4. 网络通信攻击面

**证据**: `services/print_service/src/vendor_ipp_everywhere.cpp`, `services/scan_service/src/scan_mdns_service.cpp`

| 操作 | 攻击向量 | 证据 |
|--------|---------|------|
| **IPP 打印** | **MITM 攻击**: 未验证的 IPP URL | 可能被劫持，访问本地网络服务 |
| | | **SSL/TLS 验证**: 证书验证不足 | 可能遭受中间人攻击 |
| **mDNS 发现** | **信息泄露**: 广播发现信息 | 可被被动监听获取网络拓扑 |
| **SMB 打印** | **凭据泄露**: SMB 认证凭据处理不当 | 可能暴露用户凭据 |

### 5. 设备通信攻击面

**证据**: `services/scan_service/src/scan_usb_manager.cpp`, `services/sane_service/src/sane_service_ability.cpp`

| 操作 | 攻击向量 | 证据 |
|--------|---------|------|
| **USB 扫描器** | **设备劫持**: USB 设备未验证 | 恶意 USB 设备可能伪装为合法扫描器 |
| | | **拒绝服务**: 大量 USB 设备操作 | 可能导致扫描服务过载 |
| **SANE 扫描** | **缓冲区溢出**: SANE 后端数据未充分验证 | 扫描参数过大时可能溢出 |

### 6. CUPS 通信攻击面

**证据**: `services/print_service/src/print_cups_client.cpp`

| 操作 | 攻击向量 | 证据 |
|--------|---------|------|
| **CUPS IPC** | **命令注入**: CUPS 命令未验证 | PPD 文件路径等可能注入恶意内容 |
| | | **文件描述符泄露**: 文件描述符传递不当 | 可能导致信息泄露 |

---

## 信任边界

### 1. 进程边界

```
系统应用（system_app）
    ↓ [权限检查]
打印框架服务（print_service, scan_service, sane_service）
    ↓ [IPC 边界]
第三方应用（third_party_app）
```

### 2. 权限边界

| 权限 | 信任范围 | 证据 |
|--------|---------|------|
| `ohos.permission.PRINT` | **普通应用**: 需要权限 | `utils/include/print_constant.h:291` |
| `ohos.permission.MANAGE_PRINT_JOB` | **普通应用**: 需要权限 | `utils/include/print_constant.h:292` |
| `ohos.permission.MANAGE_USB_CONFIG` | **系统应用**: 仅扫描服务使用 | `utils/include/scan_constant.h` |
| **System API** | **系统应用**: 调用内部 API | `services/print_service/include/print_service_ability.h` |

### 3. 数据边界

| 数据类型 | 存储位置 | 保护级别 | 证据 |
|---------|---------|----------|--------|
| **打印机列表** | `/data/service/el2/public/print_service/printers` | 应用级 | `utils/include/print_constant.h:293-294` |
| **打印任务数据** | `/data/service/el2/public/print_service/` | 应用级 | `services/print_service/include/print_user_data.h` |
| **系统配置** | `/etc/cups/` | 系统级 | `etc/init/BUILD.gn` |
| **SANE 临时文件** | `/data/service/el2/public/print_service/sane/tmp` | 服务级 | `utils/include/scan_constant.h:124` |

---

## 可被利用点

### 利用点 1: 权限检查不充分

**证据**: `services/print_service/src/print_service_ability.cpp`

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| 权限检查位置不明确 | **权限检查分散在各方法中**，可能遗漏检查 | 权限绕过 | 在服务入口统一权限检查，所有方法调用前验证 |
| `ohos.permission.PRINT` | 打印服务权限 | 可能被绕过 | 统一通过 Access Token 验证调用者身份 |
| MANAGE_PRINT_JOB 权限 | 任务管理权限 | 某些操作未检查 | 加强对敏感操作（取消/重启任务）的权限验证 |

### 利用点 2: 输入验证不足

**证据**: `interfaces/kits/napi/print_napi/src/`，`interfaces/kits/napi/scan_napi/src/`

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| **打印任务参数** | `PrintJob` 对象未充分验证 | 注入攻击 | 在 N-API 层添加严格的参数验证：
<br>• 文件路径长度和格式检查
<br>• 打印机 ID 格式验证
<br>• 页面大小范围检查 |
| **扫描参数** | `ScanParameters` 对象未充分验证 | 溢出攻击 | 在 N-API 层添加严格的参数验证：
<br>• 分辨率范围检查
<br>• 文件大小限制
<br>• 防止整数溢出 |
| **文件描述符** | 传递的 fd 未验证 | 信息泄露 | 验证文件描述符的合法性和范围 |

### 利用点 3: 路径遍历风险

**证据**: `utils/include/print_constant.h:293-296`

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| **打印机列表路径** | 路径硬编码，未验证符号链接 | 路径遍历 | 1. 使用路径规范化函数
<br>2. 检查路径中包含 `..` 或符号链接
<br>3. 限制访问路径在 `/data/service/el2/public/print_service/` 下 |
| **SANE 临时路径** | 临时文件可能被访问 | 信息泄露 | 1. 设置临时目录权限为 0700（仅服务可访问）
<br>2. 定期清理临时文件
<br>3. 避免在临时文件中存储敏感信息 |

### 利用点 4: 信息泄露

**证据**: 查询方法返回完整对象信息

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| **打印机信息** | `queryAllPrintJobs()` 返回所有任务 | 隐私泄露 | 1. 实现基于权限的数据过滤
<br>2. 只返回调用者自己的打印任务
<br>3. 添加数据访问审计日志 |
| **扫描器列表** | `getScannerList()` 返回所有设备 | 设备信息泄露 | 1. 只返回调用者有权限访问的设备
<br>2. 脱敏设备序列号等标识 |
| **扩展信息** | `queryAllExtension()` 返回扩展信息 | 内部信息泄露 | 1. 检查调用者是否为系统应用
<br>2. 限制对系统应用的扩展信息访问 |

### 利用点 5: 资源耗尽

**证据**: 无限制的任务创建和设备发现

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| **打印任务数量** | 无限制提交打印任务 | DoS 攻击 | 1. 每个应用限制同时打印任务数量
<br>2. 实现任务速率限制
<br>3. 添加资源使用监控 |
| **扫描操作** | 无限制扫描操作 | DoS 攻击 | 1. 限制同时进行的扫描操作数
<br>2. 添加扫描队列管理 |
| **打印机发现** | 无限制发现操作 | 资源耗尽 | 1. 添加发现频率限制
<br>2. 使用缓存避免重复发现 |

### 利用点 6: 时序竞态

**证据**: 异步操作缺乏适当的同步

| 细节 | 问题 | 影响 | 修复建议 |
|--------|------|------|----------|
| **打印机连接** | `connectPrinter()` 和 `disconnectPrinter()` 异步 | 状态不一致 | 1. 添加打印机状态锁机制
<br>2. 实现状态检查前的原子性验证 |
| **打印任务取消** | `cancelPrintJob()` 和任务执行异步 | 任务残留 | 1. 添加任务状态转换验证
<br>2. 实现优雅的取消机制 |
| **扫描器打开** | `openScanner()` 和扫描操作异步 | 设备冲突 | 1. 添加扫描器打开锁
<br>2. 实现设备状态互斥 |

---

## 输入验证检查

### 参数验证现状

| 参数类型 | 验证状态 | 证据 |
|---------|---------|------|
| **字符串参数** | ⚠️ 部分验证 | 长度检查存在，但缺少内容验证 |
| **路径参数** | ⚠️ 路径遍历保护不足 | 存在硬编码路径，未充分验证 |
| **数值参数** | ⚠️ 范围检查存在 | 有宏定义，但应用不一致 |
| **文件描述符** | ⚠️ 未充分验证 | 传递给服务的 fd 未验证 |
| **对象参数** | ⚠️ 深度验证不足 | 复杂对象可能包含恶意嵌套 |

### 验证宏定义

| 宏 | 位置 | 说明 |
|-----|------|------|
| `PRINT_ASSERT_BASE` | `print_constant.h:31-37` | 基础参数断言 |
| `CHECK_IS_EXCEED_PRINT_RANGE_BASE` | `print_constant.h:54-60` | 范围检查宏 |
| `PRINT_MAX_PRINT_COUNT` | `print_constant.h:25` | 最大打印数量限制（1000） |
| `PRINT_MAX_PPD_COUNT` | `print_constant.h:26` | 最大 PPD 文件数限制（4096） |
| `SCAN_MAX_COUNT` | `scan_constant.h:24` | 最大扫描数量限制（1000） |

---

## 权限检查

### 权限检查实现

| 检查位置 | 权限类型 | 状态 |
|----------|---------|--------|
| `PrintServiceAbility` | `ohos.permission.PRINT` | ✅ 已实现 |
| `ScanServiceAbility` | `ohos.permission.PRINT` | ✅ 已实现 |
| **N-API 层** | `ohos.permission.PRINT` | ⚠️ 部分方法未检查 |
| **系统扩展方法** | `ohos.permission.MANAGE_PRINT_JOB` | ⚠️ 某些方法未检查 |

### 权限增强建议

1. **统一权限检查入口**: 在服务入口统一验证权限，避免分散检查遗漏
2. **实施最小权限原则**: 每个方法只检查必需的最小权限集
3. **添加权限审计日志**: 记录权限检查失败和授权情况
4. **实现权限动态更新**: 支持运行时权限授予和撤销

---

## 安全改进建议

### 高优先级

1. **加强输入验证**
   - 在 N-API 层添加所有参数的严格类型和范围验证
   - 实现路径规范化，防止路径遍历攻击
   - 对文件描述符和路径参数进行完整验证

2. **实现数据隔离**
   - 为每个用户存储独立的打印任务和扫描数据
   - 实现基于 Access Token 的数据访问控制

3. **添加速率限制**
   - 限制每个应用的打印任务提交速率
   - 限制设备发现和扫描操作频率
   - 实现操作队列和调度机制

### 中优先级

4. **改进错误处理**
   - 避免在错误消息中暴露敏感信息
   - 实现统一的错误码和消息处理
   - 添加详细的错误日志（不包括敏感数据）

5. **加强 IPC 通信安全**
   - 验证 IPC 调用者的身份和权限
   - 实现进程间通信的加密（如需要）
   - 添加 IPC 调用的审计日志

### 低优先级

6. **代码安全加固**
   - 启用所有编译时安全选项（CFI、整数溢出检查等）
   - 定期进行安全审计和代码审查
   - 使用安全编码最佳实践

---

## 安全测试建议

### 测试范围

1. **输入验证测试**
   - 测试边界值和异常输入
   - 测试路径遍历攻击
   - 测试注入攻击（参数、路径、配置）

2. **权限测试**
   - 测试权限绕过场景
   - 测试跨用户数据访问
   - 测试权限提升可能性

3. **拒绝服务测试**
   - 资源耗尽攻击
   - 并发压力测试
   - 长时间运行稳定性测试

4. **模糊测试**
   - 对 N-API 接口进行参数模糊测试
   - 对 IPC 接口进行消息模糊测试

### 测试工具

- **单元测试**: 已存在 `test/unittest/` 目录
- **Fuzz 测试**: 已存在 `test/fuzztest/` 目录
- **建议添加**: 安全回归测试套件

---

## 依赖安全

### 外部依赖安全评估

| 依赖 | 评估 | 说明 |
|------|------|------|
| **CUPS** | ⚠️ 信任外部依赖 | 需要确保 CUPS 版本安全，及时更新 |
| **SANE Backends** | ⚠️ 信任外部依赖 | 需要验证 SANE 后端来源，避免恶意后端 |
| **libcURL / OpenSSL** | ⚠️ 网络库 | 确保使用最新版本，启用安全选项 |

---

## 合规性考虑

### OpenHarmony 安全要求

- ✅ **权限模型**: 已实现基于 Access Token 的权限检查
- ✅ **沙箱隔离**: 应用运行在沙箱中
- ⚠️ **数据加密**: 需要评估敏感数据的加密需求
- ⚠️ **审计日志**: 需要添加安全事件审计

### 行业标准

- **安全编码**: OWASP Top 10 安全实践
- **信息保护**: 数据最小化原则
- **安全测试**: 左移测试、威胁建模

---

**检查局限性**:
- 本评审基于代码静态分析，未包含运行时安全测试
- 未深入分析第三方库（CUPS, SANE）的内部实现
- 建议进行专业的渗透测试和代码审计

---

**相关链接**:
- [项目概览](00_Overview.md)
- [对外 API](04_External_API.md)
- [内部架构](05_Internal_API.md)

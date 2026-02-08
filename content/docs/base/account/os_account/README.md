# OpenHarmony os_account 子系统 Wiki

> 本 Wiki 由代码自动生成，最后更新时间：2026-02-06

## 目录

1. [概述与架构](./01_Overview.md)
2. [目录结构与模块职责](./02_Directory_Structure.md)
3. [N-API 对外接口](./03_NAPI_Interfaces.md)
4. [Inner API 内部接口](./04_Inner_API.md)
5. [系统服务与 IPC 通信](./05_Service_IPC.md)
6. [GN 构建系统与编译产物](./06_Build_System.md)
7. [权限与安全机制](./07_Security.md)
8. [配置与 Feature Flags](./08_Config_Flags.md)
9. [常见问题与调试指南](./09_FAQ_Debug.md)

---

## 项目概述

### 定位

os_account 是 OpenHarmony 的**账号子系统**，提供系统账号生命周期管理、分布式账号状态管理、应用账号管理、身份认证与访问控制等核心能力。

### 核心功能模块

| 模块 | 路径 | 功能描述 |
|------|------|----------|
| **osaccount** | `frameworks/osaccount/` | 系统账号生命周期管理 |
| **appaccount** | `frameworks/appaccount/` | 应用级别账号管理、OAuth 授权 |
| **domain_account** | `frameworks/domain_account/` | 企业域账号管理 |
| **ohosaccount** | `frameworks/ohosaccount/` | 分布式账号同步与管理 |
| **account_iam** | `frameworks/account_iam/` | 用户身份认证与访问控制 |
| **authorization** | `frameworks/authorization/` | 授权管理框架 |

### 组件架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层 (Apps)                            │
├─────────────────────────────────────────────────────────────────┤
│   N-API (JS/TS)        │     C API      │     CJ (FFI)         │
│  ─────────────────────────────────────────────────────────────  │
│  @ohos.account.osAccount│                 │                      │
│  @ohos.account.appAccount│                 │                      │
│  @ohos.account.distributedAccount│          │                      │
├─────────────────────────────────────────────────────────────────┤
│                     框架层 (Frameworks)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  osaccount  │  │  appaccount │  │  ohosaccount│            │
│  │  framework  │  │  framework  │  │  framework  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │domain_account│  │ account_iam │  │authorization│            │
│  │  framework  │  │  framework  │  │  framework  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                     服务层 (Services)                            │
│                    AccountMgrService (SA 200)                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  OsAccountManagerService │ AppAccountManagerService     │   │
│  │  DomainAccountManager    │ AccountIAMService            │   │
│  │  AuthorizationManager    │                              │   │
│  └──────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                     系统能力 (System Abilities)                  │
│                    IPC Binder 通信                              │
│              SAMgr │ AbilityMgr │ BundleMgr │ etc.             │
└─────────────────────────────────────────────────────────────────┘
```

### 主服务信息

- **服务进程**: `accountmgr`
- **SA ID**: `200`
- **库文件**: `libaccountmgr.z.so`
- **启动方式**: `run-on-create: true`（设备启动时自动拉起）
- **日志域**: `0xD001B00`

---

## 覆盖范围

### 已文档化

✅ **完整覆盖**:
- 目录结构与模块边界
- N-API 接口清单与调用链
- GN 构建配置与编译产物
- Feature Flags 配置项
- 权限校验机制
- 目录结构与职责划分

### 未包含内容

❌ **测试相关**: 本 Wiki 不包含任何测试代码、测试用例、模糊测试等内容（按要求忽略）。

❌ **完整 IPC 消息码表**: 详细的消息码枚举需要进一步代码分析。

❌ **运行时行为细节**: 某些边界条件、同步语义等细节待补充。

---

## 如何更新本 Wiki

### 触发条件

当发生以下变更时，建议更新本 Wiki：

1. 新增/删除/重命名模块
2. N-API 接口变更（新增、删除、参数变化）
3. GN 构建配置变更
4. 新增 Feature Flag
5. 安全机制重大变更

### 更新步骤

1. 修改对应模块的源码或配置文件
2. 运行 Wiki 生成工具（或手动更新）
3. 验证文档一致性
4. 提交变更（`git add wiki/ && git commit`）

### 质量检查清单

- [ ] 所有 API 有对应代码证据（文件路径+符号）
- [ ] 无测试代码引用
- [ ] 术语统一
- [ ] 链接可用（SUMMARY.md）
- [ ] 关键结论有证据支撑

---

## 相关资源

- **代码仓库**: `//base/account/os_account`
- **官方文档**: [Account 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/Account.md)
- **API 参考**: [N-API 接口定义](https://gitee.com/openharmony/interface_sdk-js/blob/master/api/@ohos.account.osAccount.d.ts)
- **构建脚本**: `build.sh --product-name <product> --build-target os_account`

---

## 贡献指南

1. 所有文档修改必须可追溯到代码变更
2. 避免主观臆测，结论需有证据支撑
3. 术语使用需与代码保持一致
4. 发现错误请提 Issue 或直接修复

# 项目概览

## 1. 项目定位

**统一用户认证 (userauth)** 是 OpenHarmony [User Identity & Access Management (IAM)](https://gitee.com/openharmony/useriam) 子系统的核心组件，负责:

1. **统一认证入口**: 为第三方应用提供统一的认证 API
2. **生物特征认证**: 支持人脸、指纹、PIN 等认证方式
3. **凭证管理**: 管理用户认证凭证的增删改查
4. **执行器框架**: 集成和管理各种认证执行器 (Driver)

**在 IAM 子系统中的位置**:

```
┌─────────────────────────────────────────────────────────────┐
│                     User IAM Subsystem                       │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ user_auth   │  │ user_idm    │  │ user_auth_framework │ │
│  │ (本项目)    │  │ (凭证管理)  │  │ (执行器框架)        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│         │               │                   │              │
│         └───────────────┼───────────────────┘              │
│                         ▼                                   │
│              ┌─────────────────────┐                       │
│              │  drivers_interface  │ ← TEE/安全实现        │
│              └─────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 核心能力

### 2.1 认证方式 (UserAuthType)

| 类型 | 枚举值 | 说明 |
|------|--------|------|
| PIN | 1 | PIN 码认证 |
| FACE | 2 | 人脸认证 |
| FINGERPRINT | 4 | 指纹认证 |
| PRIVATE_PIN | 8 | 私有 PIN |

### 2.2 认证信任等级 (AuthTrustLevel)

| 等级 | 枚举值 | 安全级别 |
|------|--------|----------|
| ATL1 | 1 | 最低信任等级 |
| ATL2 | 2 | 中等信任等级 |
| ATL3 | 3 | 较高信任等级 |
| ATL4 | 4 | 最高信任等级 |

> **证据**: `interfaces/inner_api/iam_common_defines.h` (AuthTrustLevel 枚举定义)

### 2.3 凭证生命周期

- **AddCredential**: 添加凭证
- **UpdateCredential**: 更新凭证
- **DelCredential**: 删除凭证
- **DelUser**: 删除用户及所有凭证

### 2.4 远程认证

- 通过 SoftBus 实现跨设备认证
- 支持设备间认证结果互认

---

## 3. 运行环境

### 3.1 系统要求

- **OpenHarmony 版本**: 4.0+
- **系统类型**: standard (标准系统)
- **最低内存**: 待确认 (依赖 TEE 环境)

### 3.2 依赖子系统

| 子系统 | 用途 |
|--------|------|
| ability_runtime | Ability 上下文和生命周期 |
| bundle_framework | Bundle 信息获取 |
| ipc | 跨进程通信 |
| safwk | System Ability 框架 |
| samgr | 服务管理 |
| access_token | 权限管理 |
| os_account | 系统账户管理 |
| drivers_interface | HDI 接口 (TEE 认证实现) |

> **证据**: `bundle.json` dependencies

### 3.3 硬件要求

- TEE (Trusted Execution Environment) 环境
- 生物特征传感器 (人脸/指纹)
- 安全存储 (用于凭证)

---

## 4. 关键概念

### 4.1 Challenge (挑战值)

- 每次认证请求的随机挑战值
- 用于防止重放攻击
- 类型: `BigInt` (64位整数)

### 4.2 ScheduleId (调度 ID)

- 认证过程的调度标识
- 用于追踪整个认证流程

### 4.3 Executor (执行器)

- 认证能力的抽象 (如人脸采集器、指纹传感器)
- 通过 HDI 接口与框架交互
- 具有安全等级 (ExecutorSecureLevel)

### 4.4 Context (上下文)

- 认证过程的上下文信息
- 管理认证状态机
- 支持取消、超时等控制

---

## 5. 模块职责

| 模块 | 路径 | 职责 |
|------|------|------|
| **common** | `common/` | 公共工具 (日志、计时器、系统参数) |
| **frameworks/js/napi** | `frameworks/js/napi/` | JS N-API 绑定 |
| **frameworks/ets/ani** | `frameworks/ets/ani/` | ArkTS/ETS ANI 绑定 |
| **frameworks/native** | `frameworks/native/` | Native 框架核心 |
| **interfaces** | `interfaces/` | 对外头文件 |
| **services** | `services/` | SA 服务实现 |
| **sa_profile** | `sa_profile/` | SA 配置 |

---

## 6. 相关仓库

| 仓库 | 用途 |
|------|------|
| [useriam_user_auth_framework](https://gitee.com/openharmony/useriam_user_auth_framework) | 本仓库 |
| [useriam_pin_auth](https://gitee.com/openharmony/useriam_pin_auth) | PIN 认证执行器 |
| [useriam_face_auth](https://gitee.com/openharmony/useriam_face_auth) | 人脸认证执行器 |
| [drivers_interface](https://gitee.com/openharmony/drivers_interface) | 认证 HDI 接口定义 |
| [drivers_peripheral](https://gitee.com/openharmony/drivers_peripheral) | 外设驱动 |

---

## 7. 快速开始

### 7.1 获取代码

```bash
# 克隆仓库
git clone https://gitee.com/openharmony/useriam_user_auth_framework.git
cd useriam_user_auth_framework
```

### 7.2 构建

```bash
# 在 OpenHarmony 构建系统中
hb build -f
```

### 7.3 运行测试

```bash
# 运行单元测试 (需在测试环境下)
./build.sh --test
```

> **注意**: 具体构建命令请参考 OpenHarmony 官方文档

---

## 8. 演进历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 4.0 | 2024 | 支持 ATL4，新增 V10 API |
| 3.2 | 2023 | 新增远程认证 |
| 1.0 | 2022 | 初始版本 |

---

## 9. 参考链接

- **官方指南**: https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/security/userauth-guidelines.md
- **API 参考**: 见 [02_NAPI.md](02_NAPI.md)
- **架构说明**: 见 [01_Architecture.md](01_Architecture.md)

# 项目概览

> 目的：帮助开发者快速理解设备互信认证模块的定位、核心功能、目录结构和运行环境。
>
> 适用范围：所有需要了解或使用本模块的开发者。

---

## 1. 项目定位

### 1.1 基本信息

| 项目 | 值 |
|------|-----|
| **项目名称** | 设备互信认证 (device_auth) |
| **所属子系统** | 安全子系统 (Security Subsystem) |
| **模块名** | `@ohos/device_auth` |
| **版本** | 4.0.2 |
| **许可证** | Apache License 2.0 |
| **仓库路径** | `base/security/device_auth` |

### 1.2 核心职责

设备互信认证模块是 OpenHarmony 安全子系统的核心组件，负责**设备间可信关系的全生命周期管理**：

1. **可信关系建立**：支持基于共享秘密、P2P 绑定、群组创建等方式建立设备间信任
2. **可信关系维护**：管理已建立的信任关系，支持查询、验证、更新
3. **可信关系撤销**：提供删除设备、解绑群组等撤销信任的能力
4. **安全会话协商**：在可信设备间协商会话密钥，支持 DSoftBus 组网

### 1.3 业务价值

```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenHarmony 设备互联生态                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐      设备互信       ┌─────────┐                   │
│   │ 设备 A  │ ◄─────────────────► │ 设备 B  │                   │
│   └────┬────┘                     └────┬────┘                   │
│        │                               │                          │
│        │         DSoftBus 安全会话      │                          │
│        └────────────◇────────────────┘                          │
│                     │                                           │
│              ┌──────┴──────┐                                    │
│              │ device_auth │                                    │
│              │   模块      │                                    │
│              └─────────────┘                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 核心功能模块

### 2.1 功能模块概览

| 模块 | 路径 | 职责 | 对外接口 |
|------|------|------|----------|
| **设备群组管理** | `services/legacy/group_manager/` | 群组创建/删除、成员管理、群组查询 | `DeviceGroupManager` |
| **设备群组认证** | `services/legacy/group_auth/` | 群组内设备认证、会话密钥协商 | `GroupAuthManager` |
| **身份认证服务** | `services/identity_service/` | 凭证管理、凭证协商、变更通知 | `CredManager` |
| **会话管理** | `services/session_manager/` | 认证会话生命周期管理 (v1/v2/mini) | - |
| **认证协议库** | `services/protocol/` | ISO、PAKE、EC-SPEKE、DL-SPEKE 协议实现 | - |
| **主密钥协商** | `services/mk_agree/` | 设备级主密钥建立与密钥材料生成 | - |

### 2.2 核心功能说明

#### 2.2.1 设备群组管理

**功能**：
- 创建可信设备群组（支持 P2P、同账号、跨账号三种类型）
- 删除群组及群组内设备
- 添加/删除群组成员
- 查询群组信息、成员列表、信任设备
- 注册数据变更监听器

**群组类型**：
| 类型 | 说明 | 使用场景 |
|------|------|----------|
| **P2P 群组** | 点对点信任关系 | 两设备间直接绑定 |
| **同账号群组** | 同一账号下的设备 | 账号登录的多设备 |
| **跨账号群组** | 不同账号设备间 | 跨账号设备共享 |

**证据**：`services/legacy/group_manager/inc/group_manager.h`

#### 2.2.2 设备群组认证

**功能**：
- 基于已建立信任关系的群组进行设备认证
- 协商会话密钥（支持短期密钥）
- 支持认证回调（用户确认、错误处理）
- 支持取消认证请求

**证据**：`services/legacy/group_auth/inc/group_auth_manager.h`

#### 2.2.3 身份认证服务（Identity Service）

**功能**：
- 凭证添加、导出、查询、更新、删除
- 凭证协商（基于认证凭据的 P2P 认证）
- 凭证变更监听（onCredAdd/onCredDelete/onCredUpdate）
- 批量凭证更新

**凭证类型**：
- **本地凭据**：设备本地存储的身份信息
- **导入凭据**：从外部导入的身份信息
- **协商凭据**：认证过程中协商生成的密钥材料

**证据**：`services/identity_service/inc/identity_service.h`

#### 2.2.4 认证协议支持

| 协议 | 类型 | 说明 | 适用场景 |
|------|------|------|----------|
| **ISO** | 轻量协议 | ISO/IEC 11770-3 密钥建立 | 资源受限设备 |
| **PAKE v1/v2** | 密码认证 | 基于共享密码的密钥交换 | PIN 码绑定 |
| **EC-SPEKE** | 椭圆曲线 | Elliptic Curve SPEKE | 标准设备 |
| **DL-SPEKE** | 离散对数 | Discrete Logarithm SPEKE | 轻量设备 |

**共享秘密约束**：
| 协议 | PIN 码最小长度 |
|------|----------------|
| EC-SPEKE | >= 6 bit |
| DL-SPEKE | >= 6 bit |
| ISO | >= 128 bit |

**证据**：`services/protocol/inc/` 协议头文件

---

## 3. 目录结构

### 3.1 顶层结构

```
/base/security/device_auth/
├── default_config/              # 默认编译配置
├── frameworks/                   # IPC 框架层代码
├── interfaces/                  # 对外接口
│   ├── kits/napi/              # N-API 实现
│   └── inner_api/              # Inner API 头文件
├── common_lib/                  # C 语言公共基础库
├── deps_adapter/                # 依赖适配层
│   ├── key_management_adapter/ # 密钥管理适配 (HUKS/mbedtls)
│   └── os_adapter/             # 系统能力适配 (Linux/LiteOS)
├── services/                    # 服务层代码 [核心]
│   ├── sa/                     # System Ability
│   ├── common/                 # 公共服务
│   ├── frameworks/             # 框架层
│   ├── data_manager/           # 数据持久化
│   ├── session_manager/        # 会话管理
│   ├── protocol/               # 认证协议库
│   ├── mk_agree/               # 主密钥协商
│   ├── identity_service/       # 身份认证服务
│   ├── ext_plugin_manager/     # 扩展插件管理
│   ├── privacy_enhancement/    # 隐私增强
│   └── legacy/                 # 遗留模块
│       ├── group_manager/      # 群组管理
│       ├── group_auth/         # 群组认证
│       └── authenticators/     # 认证执行器
├── test/                        # 测试代码 [忽略]
├── BUILD.gn                     # 根构建文件
├── bundle.json                  # 模块配置
└── hisysevent.yaml             # 安全事件配置
```

### 3.2 服务层详细结构

```
services/
├── sa/                           # System Ability
│   ├── src/                      # SA 实现
│   └── sa_profile/               # SA 配置 (4701.json)
├── common/                        # 公共服务
│   └── inc/
├── frameworks/                    # 框架层
│   ├── inc/
│   │   ├── task_manager/         # 任务管理
│   │   ├── os_account_adapter/   # 账号适配
│   │   ├── hiview_adapter/       # HiView 适配
│   │   └── permission_adapter/   # 权限适配
│   └── src/
├── data_manager/                  # 数据管理
│   ├── group_data_manager/        # 群组数据
│   ├── cred_data_manager/        # 凭证数据
│   └── operation_data_manager/    # 操作数据
├── session_manager/               # 会话管理
│   ├── inc/
│   │   └── session/
│   │       ├── dev_session_fwk.h # 会话框架
│   │       ├── v1/              # 会话 v1
│   │       └── v2/              # 会话 v2
│   └── src/
├── protocol/                      # 协议库
│   └── inc/
│       ├── iso_protocol/         # ISO 协议
│       └── pake_protocol/        # PAKE 协议
├── mk_agree/                      # 主密钥协商
│   └── inc/
├── identity_service/              # 身份认证服务
│   ├── inc/
│   ├── listener/                 # 监听器
│   └── session/                  # 会话
├── legacy/                        # 遗留模块
│   ├── group_manager/            # 群组管理
│   ├── group_auth/               # 群组认证
│   ├── authenticators/           # 认证执行器
│   ├── identity_manager/         # 身份管理
│   └── creds_manager/           # 凭证管理
├── key_agree_sdk/                 # 密钥协商 SDK
├── ext_plugin_manager/            # 扩展插件
└── privacy_enhancement/          # 隐私增强
```

**证据**：`README_zh.md` 目录结构说明

---

## 4. 运行环境

### 4.1 支持的系统类型

| 系统类型 | 说明 | 内核 |
|----------|------|------|
| **standard** | 标准系统 | Linux |
| **small** | 小型系统 | LiteOS-A / Linux |
| **mini** | 轻量系统 | LiteOS-M |

### 4.2 系统依赖

**核心依赖**：
- `ipc` - IPC 通信
- `samgr` - 系统能力管理
- `hilog` - 日志
- `hisysevent` - 安全事件
- `huks` - 密钥管理
- `os_account` - 账号管理
- `access_token` - 访问控制

**条件依赖**：
- `dsoftbus` - 软总线通信（可选）
- `mbedtls` / `openssl` - 加密库

**证据**：`bundle.json` dependencies 配置

### 4.3 系统能力 (SysCap)

本模块不声明特定 SysCap，依赖系统基础能力。

---

## 5. 关键概念

### 5.1 信任关系

**定义**：两设备间建立的互信状态，基于共享秘密或认证凭据确认对方身份。

**建立方式**：
1. **P2P 绑定**：基于共享 PIN 码建立一对一信任
2. **群组创建**：通过业务创建群组，自动添加信任设备
3. **凭据导入**：从外部导入身份凭据

### 5.2 凭证 (Credential)

**定义**：设备身份的唯一标识，包含公钥、私钥材料或共享秘密。

**类型**：
- **长期凭证**：设备身份公私钥对
- **会话凭证**：认证过程中生成的短期密钥
- **共享秘密**：P2P 绑定的 PIN 码派生密钥

### 5.3 会话 (Session)

**定义**：设备间一次完整的认证或密钥协商过程。

**状态**：
- `INIT` - 初始化
- `PROCESSING` - 处理中
- `FINISHED` - 完成
- `ERROR` - 错误

---

## 6. 相关跳转

| 内容 | 文档 |
|------|------|
| 架构设计 | [02_Architecture.md](./02_Architecture.md) |
| API 接口 | [03_API_Reference.md](./03_API_Reference.md) |
| 内部 API | [04_Inner_API.md](./04_Inner_API.md) |
| 构建配置 | [05_Build_Config.md](./05_Build_Config.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |
| 常见问题 | [07_Troubleshooting.md](./07_Troubleshooting.md) |
| 调用链图 | [appendix/Callgraphs.md](./appendix/Callgraphs.md) |

---

*本文档最后更新：2026-02-06*

# 应用域名校验部件概览

## 1. 项目定位

### 1.1 部件名称与归属

| 属性 | 值 |
|-----|------|
| **部件名称** | `app_domain_verify` |
| **子系统** | `bundlemanager` (包管理子系统) |
| **SysCapability** | `SystemCapability.BundleManager.AppDomainVerify` |
| **代码路径** | `/foundation/bundlemanager/app_domain_verify/` |

### 1.2 核心功能

应用域名校验部件是包管理子系统中的核心部件，与**包管理基础框架**和**元能力管理服务**协作，共同完成 **`Applinking`** 功能：

1. **安装时校验** - 在应用安装阶段，与域名服务器通信，校验应用与域名的双向关联关系
2. **运行时过滤** - 打开链接时，根据保存的关联关系过滤出域名关联应用的 ability
3. **周期刷新** - 设备开机后及固定周期内主动刷新校验失败的应用

### 1.3 Applinking 简介

> Applinking 是一种安全的链接跳转技术，通过 HTTPS 链接直接将用户带到应用特定内容。相比 DeepLink，Applinking 更安全可靠。

**安全优势**：
- 双向认证：服务器验证应用 + 应用验证服务器
- 防劫持：基于数字签名验证
- 无需用户确认：自动跳转，体验更佳

## 2. 目录结构

```
/foundation/bundlemanager/app_domain_verify/
├── etc/                          # 进程配置与权限配置
│   ├── BUILD.gn
│   ├── api_report.conf           # API 上报配置
│   └── init/
│       └── app_domain_verify_agent.cfg  # Agent 服务配置
├── figures/                       # 架构图与文档图片
├── interfaces/                    # 接口代码
│   ├── inner_api/                 # Inner API（系统内部调用）
│   │   ├── client/               # 客户端接口
│   │   └── common/               # 公共数据结构
│   └── kits/                     # 外部接口
│       └── js/                   # N-API 与 ANI
├── frameworks/                   # 框架实现
│   ├── common/                   # 公共框架（HTTP/权限/工具）
│   ├── extension/               # 扩展框架
│   ├── verifier/                 # 校验器实现
│   └── app_details_rdb/          # RDB 数据管理
├── profile/                      # SA 配置文件
│   ├── 6200.json                # Manager Service SA 配置
│   └── 6201.json                # Agent Service SA 配置
├── services/                     # 系统服务实现
│   ├── app_domain_verify_mgr/   # Manager Service
│   └── app_domain_verify_agent/ # Agent Service
├── BUILD.gn                      # 根构建配置
├── bundle.json                   # 部件配置
├── README.md                     # 英文说明
└── README_ZH.md                  # 中文说明
```

## 3. 技术栈与依赖

### 3.1 核心依赖

| 依赖项 | 用途 |
|-------|------|
| **ability_base** | 基础能力框架 |
| **ability_runtime** | 运行时能力 |
| **bundle_framework** | 包管理框架 |
| **ipc** | 进程间通信 (Binder) |
| **relational_store** | RDB 关系数据库 |
| **napi** | Node.js API |
| **safwk** | System Ability Framework |
| **samgr** | Samgr 服务管理 |
| **netstack** | 网络栈 (HTTP 通信) |
| **curl** | HTTP 客户端库 |
| **openssl** | 加密与签名验证 |
| **access_token** | 权限管理 |
| **hilog** | 日志系统 |
| **hisysevent** | 事件上报 |

### 3.2 系统要求

- **系统类型**: standard (标准系统)
- **ROM 占用**: ~300KB
- **RAM 占用**: ~1024KB

## 4. 运行环境

### 4.1 进程模型

部件包含**两个系统服务**：

| 服务 | 进程名 | SA ID | 启动方式 | 说明 |
|-----|-------|-------|---------|------|
| **Manager Service** | `foundation` | 6200 | 常驻 (`run-on-create: true`) | 域名前管理、查询、过滤 |
| **Agent Service** | `app_domain_verify_agent` | 6201 | 按需 (`run-on-create: false`) | 实际校验任务执行 |

### 4.2 启动条件

Agent Service 在以下条件触发启动：

1. **开机完成** - `usual.event.BOOT_COMPLETED`
2. **定时任务** - 每 86400 秒（24小时）刷新失败校验
3. **按需请求** - 收到 Manager Service 的校验请求

## 5. 关键概念

### 5.1 SkillUri

应用声明的 URL 能力配置，包含：

| 字段 | 类型 | 描述 |
|-----|------|------|
| `scheme` | string | URI 协议 (如 `https`) |
| `host` | string | 主机地址 |
| `port` | string | 端口 |
| `path` | string | 路径 (path/pathStartWith/pathRegex 三选一) |
| `pathStartWith` | string | 路径前缀 |
| `pathRegex` | string | 路径正则 |
| `type` | string | MIME 类型 |

### 5.2 域名校验状态

| 状态 | 描述 |
|-----|------|
| **VERIFY_SUCCESS** | 校验成功 |
| **VERIFY_FAILED** | 校验失败 |
| **NOT_VERIFIED** | 未校验 |
| **EXPIRED** | 已过期 |

### 5.3 双向关联

```
应用 ←→ 域名

应用安装时：
1. 应用声明支持的 HTTPS 域名
2. 向域名服务器请求 assetlinks.json
3. 服务器返回应用签名列表
4. 校验应用签名是否在列表中

打开链接时：
1. 收到 HTTPS 链接
2. 查询域名关联的应用
3. 过滤出校验通过的 ability
4. 启动目标应用
```

## 6. 相关文档

| 文档 | 链接 |
|-----|------|
| 架构说明 | [01_Architecture.md](./01_Architecture.md) |
| Inner API | [02_Inner_API.md](./02_Inner_API.md) |
| N-API | [03_N_API.md](./03_N_API.md) |
| 安全评估 | [06_Security.md](./06_Security.md) |

# 项目概览

## 项目定位

**Permission Lite** 是 OpenHarmony 安全子系统的核心组件，负责应用权限的**定义、授予、校验和管理**。

### 核心职责

| 职责 | 描述 |
|------|------|
| 权限定义 | 提供敏感 API 的权限声明机制 |
| 权限授予 | 管理用户/系统对权限的授予状态 |
| 权限校验 | 在 API 调用时验证权限合法性 |
| 运行时管理 | 支持运行时动态授予/回收权限 |
| IPC 认证 | 校验跨进程通信的访问策略 |

### 适用场景

```
┌─────────────────────────────────────────────────────────────┐
│                    Permission Lite                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │  权限定义   │ →  │  权限授予   │ →  │  权限校验   │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                 │           │
│                                                 ▼           │
│                                        ┌─────────────┐     │
│                                        │  IPC 认证   │     │
│                                        └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

## 系统定位

| 属性 | Mini System | Small System |
|------|-------------|--------------|
| RAM 要求 | ≥ 128 KiB | ≥ 1 MiB |
| 处理器 | ARM Cortex-M, RISC-V | ARM Cortex-A |
| 产物形态 | 静态库 (.a) | 动态库 (.so) |
| 功能完整度 | 基础权限管理 | 完整功能（含 IPC） |

**代码证据**：`bundle.json` - `adapted_system_type: ["small", "mini"]`

## 核心能力

### 1. 权限管理

- **权限声明**：定义敏感 API 所需的权限
- **权限授予**：安装时授予 / 运行时授予
- **权限校验**：API 访问前的权限验证
- **权限回收**：动态回收已授予的权限

### 2. IPC 认证

- **策略配置**：支持 RANGE / FIXED / BUNDLENAME 三种策略
- **通信校验**：跨进程访问的权限验证
- **SAMGR 集成**：作为 SAMGR 的权限策略提供者

### 3. 运行时能力

- **动态授予**：应用运行期间授予权限
- **动态回收**：运行时撤销权限
- **标志更新**：修改权限的扩展属性

## 目录结构

```
/base/security/permission_lite
├── interfaces                         # API 暴露层
│   ├── innerkits                      # 内部 API（系统服务调用）
│   │   ├── ipc_auth_interface.h    # IPC 认证内部接口
│   │   └── pms_interface_inner.h     # PMS 内部接口
│   └── kits                           # 外部 API（NDK）
│       ├── pms_interface.h           # PMS 对外 C 接口
│       └── pms_types.h                # 公共类型定义
├── services                           # 核心服务实现
│   ├── ipc_auth                       # IPC 通信认证
│   ├── js_api                         # JS API 封装（ACE Lite JSI）
│   ├── pms                            # 权限管理服务端
│   ├── pms_base                       # SAMGR 服务注册
│   └── pms_client                     # PMS 客户端
└── build                              # 构建配置
```

## 术语表

| 术语 | 定义 |
|------|------|
| UID | User ID，用户标识符 |
| PID | Process ID，进程标识符 |
| Permission | 权限，访问敏感 API 的凭证 |
| Grant | 授予，允许某权限 |
| Revoke | 回收，撤销某权限 |
| IPC | Inter-Process Communication，进程间通信 |
| SAMGR | System Ability Manager，系统能力管理器 |
| NDK | Native Development Kit，原生开发套件 |
| JSI | JavaScript Interface，JS 接口 |

## 依赖关系

### 外部依赖

| 组件 | 用途 |
|------|------|
| hilog_lite | 日志输出 |
| samgr_lite | 系统能力管理 |
| ipc | 进程间通信 |
| cJSON | JSON 解析 |
| bounds_checking_function | 字符串安全函数 |

### 被依赖关系

| 调用方 | 调用场景 |
|--------|----------|
| Bundle Manager | 安装时权限校验 |
| App Framework | 运行时权限检查 |
| SAMGR | IPC 通信策略查询 |

## 版本信息

| 属性 | 值 |
|------|------|
| 当前版本 | 3.1.0 |
| ROM 占用 | ~150 KB |
| RAM 占用 | ~500 KB |
| 许可证 | Apache License 2.0 |

---

## 相关文档

- 架构说明 → `02_Architecture.md`
- API 接口 → `03_APIs.md`
- 构建配置 → `04_Build.md`
- 安全评审 → `05_Security.md`

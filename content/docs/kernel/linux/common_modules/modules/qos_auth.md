# qos_auth - QoS 认证

## 1. 概述

### 1.1 模块定位

qos_auth 为 FFRT（并发编程框架）提供底层调度能力，支持根据业务逻辑在线程间分配调度资源，保障关键任务时延。

**证据来源**: `qos_auth/README_zh.md:1-9`

```
qos_auth/README_zh.md:1-9
为支持并发编程框架FFRT的底层调度能力而设计，允许app侧根据业务逻辑在线程间分配调度资源。
涉及：
1. 高效的能感知状态的app权限管控机制(轻量化权限管控)
2. 动态的调度资源分配机制(动态多级qos)
```

### 1.2 子模块

| 子模块 | 功能 |
|--------|------|
| auth_ctrl | 轻量化权限管控，uid 粒度的权限管控 |
| qos_ctrl | 动态多级 QoS，提供多种 policy |

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/qos_auth/
├── auth_ctl/
│   ├── auth_ctrl.c/h          # 权限管控核心 (662行)
│   ├── qos_ctrl.c/h           # QoS 控制 (762行)
│   ├── auth_qos_debug.c       # 调试接口
│   ├── Kconfig                # 配置选项
│   └── Makefile               # 构建 auth_qos_ctrl.o
├── include/
│   ├── auth_ctrl.h            # 权限管控 API
│   ├── qos_auth.h             # QoS 认证标志
│   ├── qos_ctrl.h             # QoS 控制接口
│   └── rtg_auth.h             # RTG 认证
├── figures/                   # 架构图
├── README_zh.md              # 文档
└── apply_qos_auth.sh         # 部署脚本
```

**证据来源**: `qos_auth/README_zh.md:24-41`

---

## 3. auth_ctrl - 权限管控

### 3.1 功能说明

uid 粒度的权限管控，根据应用前后台状态动态管控对内核 feature 接口的访问权限。

**证据来源**: `qos_auth/README_zh.md:12-14`

### 3.2 主要 API

| 函数 | 功能 |
|------|------|
| `check_authorized()` | 检查是否授权 |
| `get_authority()` | 获取权限结构 |
| `get_auth_struct()` | 增加引用计数 |
| `put_auth_struct()` | 减少引用计数 |
| `do_auth_ctrl_ioctl()` | 处理 auth_ctrl ioctl |

### 3.3 数据结构

| 结构 | 说明 |
|------|------|
| `struct auth_struct` | 权限结构 |
| `struct idr *ua_idr` | uid-based 权限 IDR 表 |

---

## 4. qos_ctrl - QoS 控制

### 4.1 功能说明

动态多级 QoS 模块，提供多种 policy（前台/后台/system 等），每个 policy 包含 6 个 QoS 等级。

**证据来源**: `qos_auth/README_zh.md:16-20`

### 4.2 主要 API

| 函数 | 功能 |
|------|------|
| `qos_apply()` | 应用 QoS |
| `qos_leave()` | 离开 QoS |
| `qos_get()` | 获取 QoS |
| `qos_switch()` | 切换 QoS |
| `init_task_qos()` | 初始化任务 QoS |
| `sched_exit_qos_list()` | 退出时清理 |

### 4.3 QoS 标志

| 标志 | 值 | 说明 |
|------|-------|------|
| `AF_QOS_ALL` | 0x0003 | 全部 QoS |
| `AF_QOS_DELEGATED` | 0x0001 | 委托 QoS |

---

## 5. 构建配置

### 5.1 Kconfig 选项

| 配置项 | 说明 |
|--------|------|
| `CONFIG_AUTHORITY_CTRL` | 权限管控使能 |
| `CONFIG_QOS_CTRL` | 多级 QoS 使能 |
| `CONFIG_RTG_AUTHORITY` | RTG 鉴权使能 |
| `CONFIG_QOS_AUTHORITY` | QoS 鉴权使能 |
| `CONFIG_AUTH_QOS_DEBUG` | Debug 节点使能 |
| `CONFIG_QOS_POLICY_MAX_NR` | QoS 策略数量限制（默认: 5） |

**证据来源**: `qos_auth/README_zh.md:45-50`

### 5.2 依赖配置

```
qos_auth/README_zh.md:52-62
# 时延控制
CONFIG_SCHED_LATENCY_NICE=y

# 供给
CONFIG_UCLAMP_TASK=y
CONFIG_UCLAMP_BUCKETS_COUNT=20
CONFIG_UCLAMP_TASK_GROUP=y
```

---

## 6. 相关文档

| 文档 | 说明 |
|------|------|
| [02_Architecture.md](../02_Architecture.md) | 整体架构 |
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |

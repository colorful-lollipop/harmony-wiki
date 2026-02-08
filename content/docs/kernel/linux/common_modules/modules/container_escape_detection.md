# container_escape_detection - 容器逃逸检测

## 1. 概述

### 1.1 模块定位

container_escape_detection 是容器安全检测机制，用于检测和防止容器逃逸尝试。通过 HCK（Hook Control Kit）hooks 监控进程属性、凭证和命名空间变更。

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/container_escape_detection/
├── include/
│   ├── ced_detection.h         # 核心检测 API
│   ├── ced_detection_points.h  # 检测点结构
│   ├── ced_permission.h        # 权限检查
│   └── ced_log.h              # 日志工具
├── core/
│   ├── ced_module.c            # 模块初始化 (32行)
│   ├── ced_detection.c         # 主检测逻辑 (349行)
│   └── ced_permission.c       # 权限实现
├── Kconfig                     # 配置: CONFIG_SECURITY_CONTAINER_ESCAPE_DETECTION
├── Makefile                    # 构建3个核心对象
└── apply_ced.sh               # 部署脚本
```

---

## 3. 核心功能

### 3.1 检测事件类型

| 事件类型 | 说明 |
|----------|------|
| `CRED_CHANGED` | 凭证变更 |
| `NSPROXY_CHANGED` | 命名空间变更 |
| `ATTRIBUTE_CHANGED` | 属性变更 |
| `TREE_CHANGED` | 进程树变更 |

### 3.2 主要 API

| 函数 | 功能 |
|------|------|
| `ced_initialize()` | 初始化检测 |
| `ced_register_ced_hooks()` | 注册安全 hooks |
| `ced_has_check_perm()` | 检查 SELinux 权限 |
| `detection_hook()` | 进程检测 hook |
| `setattr_insert_hook()` | 属性变更 hook |
| `commit_creds_hook()` | 凭证变更 hook |
| `kernel_clone_hook()` | 进程克隆 hook |
| `exit_hook()` | 进程退出 hook |

### 3.3 跟踪机制

使用 **RB-tree** 进行进程跟踪和快速查找。

---

## 4. 相关文档

| 文档 | 说明 |
|------|------|
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |

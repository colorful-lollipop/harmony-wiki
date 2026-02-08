# SELinux 库概览

## 原始库简介

### 基本信息

| 属性 | 值 |
|-----|-----|
| 名称 | SELinux (Security-Enhanced Linux) Userspace |
| 版本 | 3.7 (README.OpenSource) / 3.1 (bundle.json) |
| 许可证 | GPL-2.0-only; Public Domain; LGPL-2.1-only; BSD-2-Clause-Views |
| 上游地址 | https://github.com/SELinuxProject/selinux |
| 维护者 | maliang34@huawei.com |

### 功能概述

SELinux 是 Linux 内核的安全增强模块，提供**强制访问控制 (MAC)** 机制。用户空间库提供：

1. **libsepol** - SELinux 策略编译库
   - 策略二进制文件编译和链接
   - 策略模块管理
   - CIL (Common Intermediate Language) 支持

2. **libselinux** - SELinux 核心库
   - 安全上下文管理
   - 标签查询和设置
   - 策略状态查询
   - AVC (Access Vector Cache)

3. **工具集**
   - checkpolicy/secilc - 策略编译器
   - getenforce/setenforce - SELinux 状态管理
   - getfilecon/setfilecon - 文件上下文管理
   - restorecon - 恢复文件默认标签

### 在 Linux 生态中的位置

```
┌─────────────────────────────────────────┐
│           应用程序 (App)                 │
├─────────────────────────────────────────┤
│       libselinux (用户空间库)            │
├─────────────────────────────────────────┤
│           SELinux LSM (内核)             │
├─────────────────────────────────────────┤
│         Linux 内核安全框架               │
└─────────────────────────────────────────┘
```

---

## 在 OpenHarmony 中的作用和定位

### 安全架构中的位置

在 OpenHarmony 安全体系中，SELinux 提供系统级的强制访问控制：

```
┌─────────────────────────────────────────────┐
│              应用沙箱 (App Sandbox)          │
│         (由 appspawn + SELinux 实现)         │
├─────────────────────────────────────────────┤
│           权限管理 (AccessToken)             │
├─────────────────────────────────────────────┤
│          SELinux 强制访问控制               │
│    (限制进程/文件访问，即使 root 也受限)      │
├─────────────────────────────────────────────┤
│              Linux DAC                      │
│         (传统用户/组权限)                    │
└─────────────────────────────────────────────┘
```

### 核心使用场景

#### 1. 系统初始化 (init)
- 早期启动时加载 SELinux 策略
- 文件系统标签恢复 (restorecon)
- 进程安全上下文设置

#### 2. 应用沙箱 (appspawn)
- 应用进程启动时设置标签
- 隔离应用数据目录
- 限制应用系统资源访问

#### 3. 应用安装 (bundle_framework)
- 安装时设置应用文件标签
- 管理应用权限上下文
- 更新应用标签配置

#### 4. 系统更新 (updater)
- 升级过程中标签恢复
- 确保系统文件标签正确
- 防止升级后权限混乱

### OH 特有定制点

| 定制点 | 原始行为 | OH 行为 | 目的 |
|-------|---------|--------|------|
| file_contexts | 单个配置文件 | 支持多个配置文件 | 模块化策略管理 |
| app_allow_config | 无 | 白名单配置 | 应用数据保护 |
| 构建系统 | Makefile | BUILD.gn | OH 构建集成 |

### 与上游版本的主要差异

1. **无传统 Patch 文件**
   - 定制通过条件编译实现
   - 新增功能通过独立模块实现
   - 维护成本更低，升级更容易

2. **OHOS_FC_INIT 宏**
   - 启用多文件 file_contexts 支持
   - 代码级定制，非配置级

3. **应用白名单**
   - 新增 app_allow_config 模块
   - 保护特定路径不被 restorecon

4. **构建适配**
   - 完整的 BUILD.gn 配置
   - 支持多镜像安装

---

## 模块组成

### 代码结构

```
third_party/selinux/
├── libsepol/              # 策略编译库
│   ├── src/              # 核心源文件
│   ├── cil/              # CIL 支持
│   └── include/          # 头文件
├── libselinux/           # SELinux 核心库
│   ├── src/              # 核心源文件
│   │   ├── app_allow_config.c   # 【OH 新增】应用白名单
│   │   ├── app_allow_config.h   # 【OH 新增】
│   │   └── ...
│   └── include/          # 头文件
├── checkpolicy/          # 策略编译器
├── secilc/               # CIL 编译器
└── BUILD.gn              # 【OH 特有】构建配置
```

### 关键 OH 文件

| 文件 | 说明 | 修改类型 |
|-----|------|---------|
| libselinux/src/app_allow_config.c | 应用白名单实现 | 新增 |
| libselinux/src/app_allow_config.h | 应用白名单头文件 | 新增 |
| libselinux/src/label.c | 支持 OHOS_FC_INIT | 修改 |
| libselinux/src/label_file.c | 支持 OHOS_FC_INIT | 修改 |
| libselinux/src/label_internal.h | 支持 OHOS_FC_INIT | 修改 |
| BUILD.gn | OH 构建配置 | 新增 |

---

## 相关文档链接

- [Patch 详细分析](02_Patches.md) - 深入了解 OH 特有修改
- [构建适配](03_Build_Integration.md) - BUILD.gn 配置详解
- [OH 使用场景](04_Usage_in_OH.md) - 依赖关系和使用方式

## 外部参考

- [SELinux Project 官方](https://github.com/SELinuxProject/selinux)
- [SELinux Notebook](https://github.com/SELinuxProject/selinux-notebook)
- [OpenHarmony 安全子系统文档](https://gitee.com/openharmony/docs)

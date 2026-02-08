# Toybox 库简介

> Toybox 是 OpenHarmony 系统的基础命令行工具集，提供 200+ 个常用 Linux 命令。

---

## 1.1 基本信息

### 原始库信息

| 项目 | 内容 |
|-----|------|
| **库名称** | toybox |
| **上游版本** | 0.8.12 |
| **许可证** | BSD Zero Clause License (0BSD) |
| **上游地址** | http://landley.net/toybox |
| **上游仓库** | https://github.com/landley/toybox |
| **维护者** | Rob Landley |
| **创建时间** | 2006 年 |

### OpenHarmony 组件信息

| 项目 | 内容 |
|-----|------|
| **OH 组件名称** | @ohos/toybox |
| **OH 组件版本** | 3.1 |
| **所属子系统** | thirdparty |
| **适配系统类型** | standard（标准系统）、small（轻量系统） |
| **ROM 占用** | 73KB |
| **RAM 占用** | 146KB |
| **安装位置** | system, ramdisk, updater 镜像 |

---

## 1.2 原始库功能

### 核心特性

Toybox 的设计理念：**"All-in-one Linux command line"**

- ✅ **简单**：代码结构清晰，易于理解和维护
- ✅ **小巧**：优化后的二进制文件仅约 73KB
- ✅ **快速**：经过性能优化的实现，启动速度快
- ✅ **符合标准**：遵循 POSIX-2008 和 LSB 4.1 规范
- ✅ **多调用二进制**：单个可执行文件，通过符号链接提供多个命令

### 工作原理

Toybox 采用**多调用二进制（multicall binary）**模式：

```
┌─────────────┐
│ toybox 可执行文件 │
└─────────────┘
       │
       ├─→ /bin/ls (符号链接)
       ├─→ /bin/cat (符号链接)
       ├─→ /bin/cp (符号链接)
       ├─→ /bin/mv (符号链接)
       ├─→ /bin/ps (符号链接)
       └─→ ... (200+ 命令)
```

当用户执行 `ls` 命令时：
1. Shell 执行 `/bin/ls`
2. `/bin/ls` 是符号链接，指向 `toybox` 可执行文件
3. toybox 检查 `argv[0]`（程序名称），发现是 "ls"
4. toybox 执行 `ls_main()` 函数

这种设计的优势：
- 📦 **节省空间**：单个二进制文件，而不是 200+ 个独立程序
- ⚡ **启动快速**：共享代码段，加载更快
- 🛠️ **易于维护**：统一的基础库和工具函数

### 提供的命令

Toybox 提供约 200+ 个命令，主要分类：

#### 基础文件操作
- `ls`, `cp`, `mv`, `rm`, `mkdir`, `rmdir`, `touch`, `ln`, `cat`, `head`, `tail`, `wc`, `sort`, `uniq`, `find`, `grep`, `sed`, `cut`, `paste`, `tee`, `stat`, `chmod`, `chown`, `chgrp`, `dd`, `tar`, `cpio`

#### 系统管理
- `ps`, `kill`, `killall`, `top`, `free`, `vmstat`, `dmesg`, `mount`, `umount`, `reboot`, `poweroff`, `halt`, `hostname`, `uname`, `uptime`, `date`, `df`, `du`, `sync`, `swapon`, `swapoff`

#### 网络工具
- `ifconfig`, `netstat`, `ping`, `netcat` (nc), `ftpget`, `ftpput`, `wget` (扩展), `telnet` (扩展), `traceroute` (扩展)

#### 文本处理
- `echo`, `printf`, `env`, `printenv`, `set`, `unset`, `export`, `expr`, `awk` (扩展), `diff` (扩展), `patch`, `tr` (扩展), `strings`, `hexedit`, `xxd`

#### 其他工具
- `base64`, `md5sum`, `sha1sum`, `sha256sum`, `sha384sum`, `sha512sum`, `gzip`, `gunzip`, `bzip2`, `bunzip2`, `zcat`, `tar`, `cpio`, `logger`, `which`, `whoami`, `id`, `groups`

---

## 1.3 在 OpenHarmony 中的定位

### 核心作用

Toybox 在 OpenHarmony 中扮演**基础命令行工具集**的角色：

1. **系统基础 shell 命令**
   - 提供 Linux/Unix 风格的命令行环境
   - 支持系统管理、文件操作、网络调试等日常任务
   - 为开发者提供熟悉的命令行工具

2. **系统服务支撑**
   - samgr（系统能力管理器）依赖 toybox 的命令进行进程管理和调试
   - 系统启动脚本使用 toybox 命令进行初始化

3. **内核 shell 环境**
   - 为 LiteOS_A 内核提供用户空间 shell 环境
   - 支持嵌入式设备的轻量级系统管理

4. **开发与调试**
   - 提供丰富的命令行工具用于开发和调试
   - 支持网络测试（ping, ifconfig, netstat）
   - 提供进程管理（ps, kill, top）

### 适用场景

#### 标准系统（standard）

```
┌─────────────────────────────────────────┐
│         OpenHarmony 标准系统           │
├─────────────────────────────────────────┤
│  应用层                               │
├─────────────────────────────────────────┤
│  框架层 (Ace, Graphic, Multimedia)   │
├─────────────────────────────────────────┤
│  系统服务层 (Samgr, ...)             │
│         ↓ 依赖 toybox                  │
├─────────────────────────────────────────┤
│  基础层 (HUKS, ... )                │
├─────────────────────────────────────────┤
│  Linux 内核                           │
└─────────────────────────────────────────┘

toybox 位置：/system/bin/toybox
主要用途：系统管理、开发调试、脚本执行
```

#### 轻量系统（small）

```
┌─────────────────────────────────────┐
│     OpenHarmony 轻量系统           │
├─────────────────────────────────────┤
│  应用层                             │
├─────────────────────────────────────┤
│  系统服务层 (Samgr_Lite)           │
│        ↓ 依赖 toybox                │
├─────────────────────────────────────┤
│  基础层                           │
├─────────────────────────────────────┤
│  LiteOS_A 内核                      │
└─────────────────────────────────────┘

toybox 位置：/bin/toybox
主要用途：嵌入式设备管理、调试
```

### 与 OH 的集成点

| 集成点 | 说明 |
|--------|------|
| **构建系统** | BUILD.gn 适配，支持标准系统和轻量系统 |
| **SELinux** | 集成 SELinux 安全框架（chcon, restorecon 命令） |
| **OpenSSL** | 扩展命令使用 OpenSSL 进行加密（wget, telnet） |
| **LiteOS_A** | 独立的适配层（porting/liteos_a/） |
| **安全加固** | PIE/RELRO/NX 链接标志，防止安全漏洞 |

---

## 1.4 OH 定制化概览

### 为什么要定制化？

Toybox 原生代码在 OpenHarmony 上存在以下问题：

1. **稳定性问题**
   - 某些边界条件下会崩溃（closedir(NULL)）
   - 终端显示异常（top 命令乱码）
   - 日志输出不完整（mv -v 不打印日志）

2. **兼容性问题**
   - 64 位系统上的类型问题
   - LiteOS_A 内核的 API 差异
   - 文件系统行为差异

3. **用户体验**
   - ls 命令显示不符合 OH 习惯
   - 块大小计算不一致
   - 排序逻辑不符合预期

4. **安全加固**
   - 需要添加 PIE/RELRO/NX 防护
   - su 命令需要权限限制
   - SELinux 集成

### 定制化策略

OpenHarmony 采用**分层定制化**策略：

```
┌─────────────────────────────────────┐
│    OH 特有代码                     │
│  (openharmony/su.c)              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    条件编译层                      │
│  (TOYBOX_OH_ADAPT 宏)           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    平台适配层                      │
│  (porting/liteos_a/)             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    上游代码                        │
│  (toybox 0.8.12)                │
└─────────────────────────────────────┘
```

### 定制化程度

| 指标 | 数值 | 占比 |
|-----|------|------|
| **OH 特有源文件** | 1 个独立 + 20+ 个适配修改 | ~3% |
| **条件编译点** | 95 处 | 中等 |
| **修改命令数** | 19 个 | ~10% |
| **新增代码量** | ~2000 行（su.c + 适配代码） | 低 |

### 关键修改

详见 [02_Patches.md](./02_Patches.md)。

#### 基础库层修复
- `lib/args.c`：64 位类型适配
- `lib/dirtree.c`：防止 closedir(NULL) 崩溃
- `lib/tty.c`：修复 top 终端乱码

#### 命令层修改
- `toys/posix/cp.c`：修复 mv -v 日志
- `toys/posix/ls.c`：排序和显示优化
- `toys/posix/ps.c`：进程信息显示调整
- `toys/posix/strings.c`：文件关闭优化

#### 平台层适配
- `porting/liteos_a/`：20+ 命令的 LiteOS_A 适配

---

## 1.5 与其他工具的对比

### vs BusyBox

| 特性 | Toybox | BusyBox |
|-----|---------|---------|
| **许可证** | 0BSD（无限制） | GPL v2（传染性） |
| **代码复杂度** | 简单清晰 | 复杂庞大 |
| **标准遵循** | 严格 POSIX | 部分兼容 |
| **Android 使用** | Android 6.0+ 默认使用 | Android 5.x 及更早版本 |
| **OpenHarmony 选择** | ✅ 使用 | ❌ 不使用 |

### vs GNU Coreutils

| 特性 | Toybox | GNU Coreutils |
|-----|---------|---------------|
| **大小** | ~73KB | ~2MB（所有工具） |
| **启动速度** | 快 | 较慢 |
| **功能完整性** | 基本完整 | 完整（更多选项） |
| **内存占用** | 低 | 较高 |
| **OpenHarmony 选择** | ✅ 使用（嵌入式友好） | ❌ 不使用（太大） |

---

## 1.6 总结

### Toybox 的价值

1. **嵌入式友好**：小巧、快速、低内存占用
2. **开发者友好**：熟悉的 Linux 命令，降低学习成本
3. **维护友好**：0BSD 许可证，代码简单清晰
4. **OH 定制化**：通过条件编译实现 OH 特定需求

### 核心优势

- 📦 **空间效率**：200+ 命令仅 73KB
- ⚡ **性能优异**：多调用二进制，启动快速
- 🔧 **易于维护**：简单的代码结构，清晰的模块划分
- 🛡️ **安全加固**：PIE/RELRO/NX，SELinux 集成

### 适用产品

Toybox 适用于所有 OpenHarmony 产品类型：
- ✅ 手机（phone）
- ✅ 平板（tablet）
- ✅ 穿戴设备（wearable）
- ✅ IoT 设备（ipcamera, rich）
- ✅ 标准/轻量系统（standard/small）

---

## 参考文档

- [02_Patches.md](./02_Patches.md) - OH Patch 详细分析
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估结果

---

## 外部参考

- Toybox 官方文档：http://landley.net/toybox
- Toybox 上游仓库：https://github.com/landley/toybox
- Toybox 贡献指南：http://landley.net/toybox/faq.html
- "Why Toybox?" 演讲：http://landley.net/talks/celf-2013.txt

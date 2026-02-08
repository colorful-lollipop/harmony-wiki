# xpm - 可执行权限管理器

## 1. 概述

### 1.1 模块定位

XPM（eXecutable Permission Manager）通过扩展内核能力，为应用的二进制和 ABC 代码提供运行时的管控，强制仅包含合法签名的代码才允许分配可执行权限。

**证据来源**: `xpm/README_zh.md:11-14`

```
xpm/README_zh.md:11-14
XPM主要通过三个功能实现上述能力：
1. 执行权限检查
2. XPM验签地址区
3. 代码完整性保护
```

### 1.2 安全目标

| 目标 | 说明 |
|------|------|
| 执行权限检查 | 强制验证文件代码签名合法性 |
| 验签地址区 | HAP 应用启动时保留验签地址范围 |
| 代码完整性 | 阻止运行时篡改已校验代码 |

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/xpm/
├── core/                       # XPM 管控代码
│   ├── xpm_module.c            # 模块初始化 (60行)
│   ├── xpm_common.c/h          # 公共工具
│   ├── xpm_security_hooks.c/h  # 安全 hooks (388行)
│   ├── xpm_hck_hooks.c/h      # HCK hooks
│   ├── xpm_misc_device.c/h    # 杂项设备接口
│   ├── xpm_debugfs.c/h        # 调试文件系统
│   ├── xpm_report.c/h         # 事件报告
│   └── xpm_log.h              # 日志
├── validator/                   # 签名检查模块
│   ├── exec_signature_info.c/h # 签名验证
│   └── elf_code_segment_info.c # ELF 代码段解析
├── developer/                  # 开发者模式
│   └── dsmm_developer.c/h
├── secureshield/               # 安全盾
│   └── dsmm_secureshield.c/h
├── Kconfig                     # 配置选项
├── Makefile                    # 构建配置
├── README_zh.md               # 文档
└── apply_xpm.sh               # 部署脚本
```

**证据来源**: `xpm/README_zh.md:35-48`

---

## 3. 核心功能

### 3.1 执行权限检查

在代码内存映射操作前，强制验证文件的代码签名合法性，拒绝将未包含合法签名的文件映射到可执行内存。

**证据来源**: `xpm/README_zh.md:15-18`

### 3.2 XPM 验签地址区

HAP 应用被拉起时，在进程地址空间内保留一段验签地址范围，任何尝试被映射到该地址范围的文件都会被校验代码签名。

**证据来源**: `xpm/README_zh.md:21-24`

### 3.3 代码完整性保护

基于写污点标记的代码执行权限冲突检查，为代码提供运行时的完整性保护。

**证据来源**: `xpm/README_zh.md:27-33`

| 页标识 | 说明 |
|--------|------|
| `readonly` | 只读代码页标记 |
| `writetainted` | 映射到写内存区域的页标记 |

---

## 4. 主要 API

### 4.1 模块生命周期

| 函数 | 功能 |
|------|------|
| `xpm_register_misc_device()` | 注册杂项设备 |
| `xpm_deregister_misc_device()` | 注销杂项设备 |
| `xpm_debugfs_init()` | 初始化调试fs |
| `xpm_register_security_hooks()` | 注册安全 hooks |

### 4.2 安全检查

| 函数 | 功能 |
|------|------|
| `xpm_check_ownerid_policy()` | 检查 ownerid 策略 |
| `xpm_get_process_cs_info()` | 获取进程签名信息 |
| `xpm_mmap_check()` | mmap 权限检查 |
| `xpm_mprotect_check()` | mprotect 权限检查 |

### 4.3 事件报告

```c
void report_mmap_event(enum xpm_mmap_fail_event event,
    enum xpm_mmap_type type,
    struct vm_area_struct *vma,
    unsigned long prot);
```

---

## 5. 管控策略

根据 SELinux 标签实施不同管控策略：

| 类型 | 签名检查 | 匿名可执行内存 |
|------|----------|----------------|
| 普通应用类 | 强制检查 | 限制申请 |
| webview 类 | 强制检查 | 允许 JIT |
| 调测类 | 不检查 | 允许 JIT |
| 沙箱类 | 不检查 | 允许 |

**证据来源**: `xpm/README_zh.md:61-68`

---

## 6. 构建配置

### 6.1 Kconfig 选项

| 配置项 | 说明 |
|--------|------|
| `CONFIG_SECURITY_XPM` | XPM 使能 |
| `CONFIG_SECURITY_XPM_DEBUG` | XPM 调试信息 |

**证据来源**: `xpm/README_zh.md:50-59`

```
xpm/README_zh.md:50-59
1. XPM使能: CONFIG_SECURITY_XPM=y
2. XPM禁用: CONFIG_SECURITY_XPM=n
3. XPM调试信息: CONFIG_SECURITY_XPM_DEBUG=y
```

### 6.2 依赖

| 依赖 | 说明 |
|------|------|
| `CONFIG_SECURITY_CODE_SIGN` | 代码签名支持 |
| SELinux | 策略标签系统 |

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [modules/code_sign.md](code_sign.md) | 代码签名（依赖模块） |
| [04_Security_Review.md](../04_Security_Review.md) | 安全风险评审 |

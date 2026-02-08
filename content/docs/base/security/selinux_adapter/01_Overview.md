# 项目概述

## 1. 项目定位

### 1.1 组件定位

SELinux Adapter 是 OpenHarmony 安全子系统的核心组件，提供基于 **SELinux (Security-Enhanced Linux)** 的**强制访问控制 (MAC, Mandatory Access Control)** 能力。

**证据来源**:
- `README.md:5` - "本部件负责对文件，属性，服务等系统资源提供强制访问控制保护"
- `bundle.json:3` - "security-enhanced linux(SELINUX) is a mandatory access control mechanism on linux"

### 1.2 核心能力

| 能力 | 说明 | 对应产物 |
|------|------|----------|
| **策略加载** | 加载编译后的 SELinux 二进制策略 (policy.31) | `libload_policy.so` |
| **文件标签管理** | 文件路径与 Security Context 的映射管理 | `librestorecon.so` |
| **HAP 上下文** | 应用安装/运行时的 Security Context 设置 | `libhap_restorecon.so` |
| **参数权限检查** | 系统参数读写的 SELinux 权限验证 | `libparaperm_checker.so` |
| **服务权限检查** | System Ability / HDF 服务的访问控制 | `libservice_checker.so` |

### 1.3 技术栈

| 层级 | 技术/组件 |
|------|-----------|
| 内核 | Linux Kernel SELinux |
| 用户空间 | libselinux (third_party/selinux) |
| 策略编译器 | checkpolicy, secilc, sefcontext_compile |
| 构建工具 | GN (Generate Ninja) |
| 依赖库 | cJSON, pcre2, FreeBSD (fts), hilog |

## 2. 项目边界

### 2.1 职责范围

```
✓ 负责:
  - SELinux 策略编译与加载
  - Security Context 编译与管理
  - 参数/服务/HAP 的访问控制检查
  - 文件标签恢复 (restorecon)
  - 策略合规性检查 (selinux_check)

✗ 不负责:
  - SELinux 内核实现 (由 third_party/selinux 提供)
  - 应用的业务逻辑访问控制
  - 用户态权限管理 (由 permission subsystem 负责)
```

### 2.2 外部依赖

| 依赖项 | 来源 | 用途 |
|--------|------|------|
| libselinux | third_party/selinux | SELinux 用户空间 API |
| checkpolicy | third_party/selinux | 策略语法检查与编译 |
| secilc | third_party/selinux | CIL 策略编译 |
| sefcontext_compile | third_party/selinux | 文件上下文编译 |
| pcre2 | third_party/pcre2 | 正则表达式匹配 |
| FreeBSD (fts) | third_party/FreeBSD | 目录遍历 |

## 3. 运行环境

### 3.1 支持平台

**当前仅支持**: RK3568 开发板

**证据来源**:
- `README.md:31` - "目前 Selinux 只支持 RK3568"
- `bundle.json:31-33` - adapted_system_type: ["standard"]

### 3.2 系统集成

```
系统启动流程:
1. init_lite 启动
2. LoadPolicy() 加载 /etc/selinux/targeted/policy/policy.31
3. 设置各服务进程的安全上下文 (scontext)
4. 运行时通过 selinux_adapter 库进行权限检查
```

### 3.3 运行时产物路径

| 产物 | 路径 | 说明 |
|------|------|------|
| 策略文件 | `/etc/selinux/targeted/policy/policy.31` | 二进制策略 |
| 文件上下文 | `/etc/selinux/targeted/contexts/file_contexts` | 文件标签规则 |
| 参数上下文 | `/etc/selinux/targeted/contexts/parameter_contexts` | 参数标签规则 |
| 服务上下文 | `/etc/selinux/targeted/contexts/service_contexts` | 服务标签规则 |
| HAP 上下文 | `/etc/selinux/targeted/contexts/sehap_contexts` | 应用上下文规则 |
| 模式配置 | `/etc/selinux/config` | enforcing/permissive |

## 4. 关键概念

### 4.1 Security Context (安全上下文)

SELinux 中每个进程/文件/服务都有一个安全标签，格式:

```
user:role:type:level
示例: u:r:hdcd:s0
```

| 字段 | 含义 |
|------|------|
| user | 安全用户 (通常为 u) |
| role | 角色 (如 r 表示进程) |
| type | 类型 (核心标识，如 hdcd, shell) |
| level | 敏感级别 (MLS, 通常为 s0) |

### 4.2 AVC (Access Vector Cache)

当 SELinux 拒绝访问时，会生成 AVC 告警:

```
audit: type=1400 audit(1502458430.566:4): avc: denied { open } 
  for pid=1658 comm="setenforce" path="/sys/fs/selinux/enforce"
  scontext=u:r:hdcd:s0 tcontext=u:object_r:selinuxfs:s0 tclass=file permissive=1
```

### 4.3 策略规则语法

```te
# 允许 hdcd 域访问 selinuxfs 类型的文件 (open 操作)
allow hdcd selinuxfs:file open;

# 禁止所有域访问 security_t
neverallow * security_t:file {read write};
```

## 5. 相关文档

| 文档 | 链接 |
|------|------|
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| API 接口 | [03_API.md](03_API.md) |
| 构建系统 | [04_Build.md](04_Build.md) |
| 安全评审 | [05_Security.md](05_Security.md) |
| 故障排查 | [06_Troubleshooting.md](06_Troubleshooting.md) |

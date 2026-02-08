# 安全评审

## 评审概述

本章节对 `kernel/linux/config` 仓库中的 Linux 内核配置文件进行安全评审，分析配置项对 OpenHarmony 系统安全性的影响。评审范围覆盖本仓库中的所有 defconfig 文件，包括 base_defconfig、standard_common_defconfig、small_common_defconfig 以及各开发板的平台配置文件。评审重点关注内核安全配置、内存安全、访问控制、安全审计等方面。

评审方法采用**代码证据驱动**的方式，所有安全结论都基于对配置文件的直接分析。评审依据包括：Linux 内核安全文档、KSPP（Kernel Self-Protection Project）建议配置、CIS（Center for Internet Security）Linux 基线指南，以及 OpenHarmony 自身的安全要求。

评审结论表明，本仓库的配置文件整体上遵循了较好的安全实践，特别是在 base_defconfig 中定义了明确的安全红线特性。但作为配置文件仓库，其安全性也依赖于构建系统的正确使用和补丁的及时合入。

## 攻击面分析

### 配置文件攻击面

内核配置文件本身不构成直接的攻击面，因为配置文件在构建时使用，不会在运行时被攻击者直接修改。但配置项决定了内核的攻击面大小，分析配置项有助于理解系统的安全边界。

**内核接口暴露**：标准系统的 `CONFIG_IKCONFIG=y` 和 `CONFIG_IKCONFIG_PROC=y` 选项会将完整的内核配置以可读文本形式暴露在 `/proc/config.gz` 中。虽然这有助于调试和问题定位，但也可能泄露敏感的系统信息，如编译时路径、调试选项、特定的驱动配置等。证据位置：`linux-5.10/type/standard_defconfig:72-73`。

**调试接口启用**：配置中的 `CONFIG_PRINTK=y`、`CONFIG_PRINTK_NMI=y`、`CONFIG_KALLSYMS=y` 和 `CONFIG_KALLSYMS_ALL=y` 选项启用了丰富的内核调试接口。这些接口在生产环境中可能被利用进行内核信息探测。建议在发布版本中适度限制这些接口的访问权限。

**内核模块加载**：`CONFIG_MODULES=y` 允许运行时加载内核模块，这为恶意代码注入提供了潜在途径。虽然配置文件层面无法完全防止运行时攻击，但 `CONFIG_MODULE_SIG=y` 和 `CONFIG_MODULE_SIG_FORCE=y` 提供了模块签名验证机制。证据位置：`linux-6.6/base_defconfig:98-99`。

### 外部接口攻击面

基于配置文件的分析，以下外部接口构成了主要的攻击面。

**网络协议栈**：`CONFIG_NET=y`、`CONFIG_INET=y` 和 `CONFIG_NETFILTER=y` 启用了完整的 TCP/IP 协议栈和网络过滤框架。网络接口是远程攻击的主要入口，虽然网络功能对于物联网设备通常是必需的，但仍需通过 iptables/nftables 等工具配置严格的访问控制策略。

**USB 驱动**：USB 子系统的配置（`CONFIG_USB=y` 等）需要与设备树配置配合使用。USB 接口可能通过恶意设备进行攻击，如 badUSB、恶意设备类驱动等。建议仅启用实际使用的 USB 设备类驱动。

**设备文件**：`CONFIG_INPUT=y`、`CONFIG_RTC_CLASS=y` 等配置启用了多种设备文件的访问路径。攻击者可能通过这些设备文件进行本地权限提升或信息泄露。正确的设备权限配置和 SELinux/AppArmor 策略是必要的防护措施。

### 内存攻击面

内存安全问题是最常见的内核漏洞类型，配置文件层面的防护措施尤为重要。

**栈保护**：`CONFIG_STACKPROTECTOR=y` 和 `CONFIG_STACKPROTECTOR_STRONG=y` 启用了内核栈保护机制，可以检测和防止栈溢出攻击。证据位置：`linux-6.6/base_defconfig:28-29`。

**堆保护**：Linux 内核的 SLAB/SLUB 分配器包含多种安全特性。`CONFIG_HARDENED_USERCOPY=y` 提供了用户空间拷贝边界检查，可防止堆溢出利用。证据位置：`linux-6.6/base_defconfig:12`。

**指针认证**：`CONFIG_ARM64_PTR_AUTH=y` 为 ARM64 架构提供了指针认证码（PAC）功能，可以有效检测和控制流劫持攻击。证据位置：`linux-6.6/base_defconfig:9`。

**控制流完整性**：`CONFIG_CFI_CLANG=y` 启用了基于编译器的控制流完整性保护，可以防止 JOP/ROP 攻击。证据位置：`linux-6.6/base_defconfig:15`。

## 信任边界与数据流

### 信任边界定义

在基于本仓库配置编译的 OpenHarmony 系统中，存在以下主要信任边界。

**内核空间与用户空间边界**：这是最根本的信任边界。内核代码运行在最高特权级（EL1/EL2），可以访问所有系统资源；用户空间代码运行在低特权级（EL0），受到沙箱机制的限制。配置文件通过各种 `CONFIG_*` 选项控制这一边界的强化程度。

**特权进程与普通进程边界**：通过 `CONFIG_USER_NS=y` 启用的用户命名空间允许普通进程获取部分特权能力。这一特性在容器化场景中有重要作用，但也可能被滥用于权限提升。`CONFIG_SECURITY_DMESG_RESTRICT=y` 限制了非特权用户访问内核日志的能力。证据位置：`linux-6.6/base_defconfig:93`。

**内核模块可信边界**：`MODULE_SIG_FORCE=y` 要求所有加载的模块必须具有有效签名，将模块加载信任边界限定为已签名的可信模块。证据位置：`linux-6.6/base_defconfig:99`。

### 数据流安全

数据流经信任边界时需要受到安全机制的保护。

**用户空间到内核空间的数据拷贝**：`CONFIG_HARDENED_USERCOPY=y` 确保用户空间向内核空间的数据拷贝在边界内进行，防止越界访问导致的内存破坏。证据位置：`linux-6.6/base_defconfig:12`。

**文件系统数据流**：`CONFIG_HMDFS_FS=y` 和 `CONFIG_HMDFS_FS_PERMISSION=y` 配置了 OpenHarmony 的分布式文件系统安全机制，确保文件访问权限的正确检查。证据位置：`linux-6.6/base_defconfig:74-75`。

**网络数据流**：`CONFIG_NETFILTER=y` 启用了网络过滤框架，支持配置防火墙规则控制进出网络数据流。iptables/nftables 规则定义了网络数据的信任边界。

## 可被利用点分析

### 安全风险一：IKCONFIG 信息泄露

**证据位置**：`linux-5.10/type/standard_defconfig:72-73`

```
CONFIG_IKCONFIG=y
CONFIG_IKCONFIG_PROC=y
```

**问题描述**：标准系统配置启用了 IKCONFIG 功能，将完整的内核配置文件暴露在 `/proc/config.gz`。该文件包含系统的大量敏感信息，包括：内核编译时使用的工具链路径、内核引导参数、内核调试选项配置、禁用的安全特性、未使用的驱动模块列表。

**利用路径**：攻击者通过本地访问或远程信息收集获取 `/proc/config.gz`，分析配置信息后可以有针对性地构造利用代码。例如，如果配置中禁用了某项安全保护，攻击者可以利用对应的已知漏洞。

**影响范围**：信息泄露属于低危风险，主要影响攻击者的信息收集阶段，不会直接导致系统被攻破。但结合其他漏洞可能提高攻击成功率。

**修复建议**：在发布版本（release build）中禁用 `CONFIG_IKCONFIG_PROC=y`，仅在调试版本中启用。可以通过 form 层的 `build_variant` 参数控制这一配置。

### 安全风险二：内核日志访问无限制

**证据位置**：`linux-6.6/base_defconfig:93`

```
CONFIG_SECURITY_DMESG_RESTRICT=y
```

**现状说明**：配置中启用了 `DMESG_RESTRICT` 限制，但该选项仅限制了非特权用户访问 dmesg。特权进程（如 root）仍可访问完整内核日志。

**问题描述**：内核日志可能包含敏感信息，如内核指针地址、密钥材料、用户数据片段等。长时间运行的系统日志可能积累大量敏感信息。

**利用路径**：具有 `CAP_SYSLOG` 能力的进程可以读取内核日志。通过日志中的信息，攻击者可以绕过 KASLR（内核地址空间布局随机化）保护，进行更精确的漏洞利用。

**影响范围**：中等风险。信息泄露可能辅助其他攻击，但需要一定的本地权限基础。

**修复建议**：考虑在应用层实现日志脱敏机制，过滤日志中的敏感信息。同时定期清理内核日志缓冲区，减少信息积累。

### 安全风险三：内核模块签名强制不完整

**证据位置**：`linux-6.6/base_defconfig:98-99`

```
CONFIG_MODULE_SIG=y
CONFIG_MODULE_SIG_FORCE=y
```

**现状说明**：虽然启用了模块签名强制验证，但该保护仅在启用 `CONFIG_MODULE_SIG=y` 时有效。如果底层密钥管理不当或签名密钥泄露，保护将失效。

**问题描述**：模块签名机制依赖于私钥的安全性。如果签名密钥被泄露，攻击者可以签名恶意模块；如果没有安全更新机制，无法及时吊销泄露的密钥。

**利用路径**：攻击者获取签名密钥后，可以构造恶意内核模块并加载执行，实现完美的内核级持久化。

**影响范围**：高风险。一旦签名密钥泄露，可能导致大规模的系统被植入后门。

**修复建议**：建立严格的签名密钥管理流程，定期轮换密钥；实现模块签名密钥的吊销机制；在关键安全场景中使用硬件安全模块（HSM）保护签名密钥。

### 安全风险四：HDF 驱动框架攻击面

**证据位置**：`linux-6.6/base_defconfig:60-67`

```
CONFIG_DRIVERS_HDF=y
CONFIG_DRIVERS_HDF_PLATFORM=y
CONFIG_DRIVERS_HDF_PLATFORM_GPIO=y
CONFIG_DRIVERS_HDF_PLATFORM_I2C=y
CONFIG_DRIVERS_HDF_PLATFORM_PWM=y
CONFIG_DRIVERS_HDF_PLATFORM_UART=y
CONFIG_DRIVERS_HDF_PLATFORM_SPI=y
CONFIG_DRIVERS_HDF_INPUT=y
```

**问题描述**：HDF 框架启用了多种外设驱动接口，包括 GPIO、I2C、PWM、UART、SPI 等。这些驱动运行在内核态，处理外设输入输出，可能存在输入验证不严、资源竞争等问题。

**利用路径**：恶意或异常的硬件设备可能通过这些接口向内核发送恶意数据，触发驱动中的漏洞。历史上多次内核漏洞与外设驱动相关。

**影响范围**：中高风险。取决于具体驱动的实现质量和硬件平台的支持情况。

**修复建议**：确保 HDF 驱动代码遵循安全编码规范，特别关注边界检查、整数溢出、资源释放等问题。定期进行 HDF 驱动的安全审计和模糊测试。

### 安全风险五：命名空间隔离不完整

**证据位置**：`linux-5.10/type/standard_defconfig:55-59`

```
CONFIG_NAMESPACES=y
CONFIG_UTS_NS=y
CONFIG_IPC_NS=y
CONFIG_PID_NS=y
CONFIG_NET_NS=y
```

**问题描述**：命名空间提供了进程隔离能力，但配置仅启用了基本命名空间，没有启用更多隔离能力（如时间命名空间、cgroup 命名空间等）。此外，命名空间的隔离能力依赖于容器运行时的正确配置。

**利用路径**：容器逃逸攻击可能利用命名空间配置的不足，实现从容器内访问宿主机资源。容器内的恶意进程可能通过未隔离的命名空间探测或攻击宿主机。

**影响范围**：中高风险。在容器化部署场景中尤为重要。

**修复建议**：根据实际安全需求评估是否需要启用更多命名空间类型；在容器运行时配置中确保命名空间的正确隔离配置；结合 seccomp 和 AppArmor/SELinux 提供多层防护。

### 安全风险六：SECCOMP 过滤能力受限

**证据位置**：`linux-6.6/base_defconfig:16-20`

```
CONFIG_HAVE_ARCH_SECCOMP=y
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
CONFIG_SECURITY_XPM=y
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
```

**问题描述**：虽然启用了 SECCOMP 过滤能力，但配置中缺少对 SECCOMP_FILTER_DEBUG 的支持，可能导致 SECCOMP 策略配置错误时难以调试和发现。

**利用路径**：SECCOMP 策略配置不当可能导致系统调用暴露过多，攻击者可能利用这些系统调用进行攻击。错误的策略可能导致拒绝服务或安全漏洞。

**影响范围**：中等风险。取决于应用 SECCOMP 策略的完整性。

**修复建议**：建议在调试版本中启用 SECCOMP_FILTER_DEBUG；在生产环境中使用经过严格测试的 SECCOMP 策略；定期审查和更新 SECCOMP 策略以适应应用更新。

## 安全配置亮点

### 内核强化配置

本仓库的 base_defconfig 中包含了多项内核强化配置，体现了较好的安全意识。

**指针认证**：`CONFIG_ARM64_PTR_AUTH=y` 为 ARM64 架构提供了硬件级别的指针认证，可以有效防止控制流劫持攻击。这一配置在支持的硬件上提供了强有力的安全保障。

**栈保护**：`CONFIG_STACKPROTECTOR_STRONG=y` 启用了更强的栈保护机制，可以检测多种栈溢出攻击模式。

**用户空间拷贝保护**：`CONFIG_HARDENED_USERCOPY=y` 启用了用户空间拷贝的边界检查，防止堆溢出等漏洞被利用。

**初始化内存保护**：`CONFIG_INIT_ON_ALLOC_DEFAULT_ON=y` 确保新分配的内存页在分配时清零，防止信息泄露。

### 模块安全

`CONFIG_MODULE_SIG=y` 和 `CONFIG_MODULE_SIG_FORCE=y` 的组合确保了模块加载的安全性。所有模块必须具有有效签名才能被加载，防止未经授权的代码注入内核。

### 日志访问控制

`CONFIG_SECURITY_DMESG_RESTRICT=y` 限制了非特权用户访问内核日志的能力，防止敏感内核信息被未授权用户获取。

## 安全配置建议

### 生产环境配置强化

基于以上分析，以下建议适用于生产环境的配置强化。

在内核信息泄露防护方面，建议在发布版本的 type 配置中禁用 `CONFIG_IKCONFIG_PROC=y` 和 `CONFIG_KALLSYMS_ALL=y`，减少内核信息暴露面。

在模块加载控制方面，建议确保 `CONFIG_MODULE_SIG_FORCE=y` 保持启用，同时建立模块签名密钥的安全管理机制。

在网络隔离方面，建议通过 `CONFIG_NETFILTER` 和网络命名空间实现网络隔离，在边界路由器配置严格的防火墙规则。

在容器安全方面，建议结合命名空间、cgroups、seccomp 和 AppArmor/SELinux 实现多层容器隔离。

### 构建时安全建议

在构建流程方面，建议确保构建环境的隔离性，防止构建系统被恶意软件污染；验证所有依赖的构建工具和脚本的完整性；使用可信的交叉编译工具链版本。

在补丁管理方面，建议及时合入 CVE 安全补丁；验证补丁来源的可靠性；在部署前进行补丁的兼容性测试。

### 运行时安全建议

在系统加固方面，建议在内核 cmdline 中启用 `slub_debug=P`、`kmalloc=` 等调试选项进行异常检测；配置 auditd 审计关键的系统调用和文件访问；启用 SELinux/AppArmor 并配置适当的策略。

在监控方面，建议部署内核完整性监控（IMA/EVM）；监控系统调用模式以检测异常行为；定期检查内核日志中的安全相关事件。

## 检查范围与局限性

### 本次评审的范围

本次安全评审的范围覆盖以下内容：`kernel/linux/config` 仓库中的所有 defconfig 文件，包括 base_defconfig、standard_common_defconfig、small_common_defconfig 以及各开发板的配置文件。评审内容侧重于配置项本身的安全性，不涉及配置的具体实现细节。

### 评审局限性

本次评审存在以下局限性。

第一，配置是静态文本文件，配置项的实际效果还取决于构建系统的正确使用和补丁的合入。评审无法验证构建流程的正确性。

第二，配置项之间的依赖关系和交互影响非常复杂，评审无法穷尽所有可能的配置组合场景。实际系统可能因配置组合而产生不同的安全特性。

第三，部分高级安全配置（如 `CONFIG_KASAN`、`CONFIG_UBSAN`）需要额外的内存开销和性能影响，评审未深入评估这些配置的工程可行性。

第四，评审未涉及具体硬件平台的安全特性，不同芯片平台可能有不同的安全能力和配置需求。

### 后续评审建议

建议在后续工作中扩展以下评审内容：对具体开发板配置进行针对性安全评估；分析 HDF 驱动框架的安全实现；评估 OpenHarmony 框架层与应用层的安全交互；进行实际渗透测试以验证安全配置的有效性。

# 术语表

本术语表汇总了本 Wiki 中使用的专业术语，按字母顺序排列，便于读者查阅和理解。

## 术语索引

**ARM64**：ARM 架构的 64 位版本，也称为 AArch64。在 OpenHarmony 中用于 RK3568 等现代芯片平台的配置。

**base_defconfig**：OpenHarmony 配置分层架构中的第一层（基础通用配置），包含 OpenHarmony 特性依赖的内核必选模块以及安全红线特性。该层配置具有不可覆盖性。

**BUILD.gn / BUILD.gn**：GN 构建系统的构建定义文件，用于描述构建目标和依赖关系。OpenHarmony 项目主要使用 GN 作为构建系统。

**build_type**：OpenHarmony 构建参数，用于指定系统形态类型（standard/small），决定加载 type 层的哪个配置文件。

**chip**：OpenHarmony 配置分层架构中的第四层（芯片平台配置），由具体芯片厂商提供，包含与特定芯片相关的内核配置。

**CVE**：Common Vulnerabilities and Exposures，通用漏洞披露。Linux 内核社区会定期发布 CVE 补丁修复已知安全漏洞。

**defconfig**：Linux 内核的标准配置文件格式，包含内核配置选项（CONFIG_*）及其取值。在 Linux 内核编译中，defconfig 文件记录了内核配置选项的期望值。

**device_type**：OpenHarmony 构建参数，用于在同芯片平台基础上区分不同产品形态，决定是否加载 product 层的配置。

**form**：OpenHarmony 配置分层架构中的第三层（版本形态配置），用于区分调试版本和发布版本等不同版本类型。

**HDF**：Hardware Driver Foundation，硬件驱动框架。OpenHarmony 提供的统一硬件驱动抽象层，其内核部分以补丁形式与内核源码配合使用。

**Hi3516DV300**：海思半导体推出的 Hi3516DV300 芯片，广泛应用于 OpenHarmony 标准系统开发板（Hispark Taurus）。

**IPC**：Inter-Process Communication，进程间通信。Linux 内核提供多种 IPC 机制，包括管道、消息队列、共享内存、信号量、套接字等。

**Kconfig**：Linux 内核配置系统的描述语言，用于定义配置选项及其依赖关系。Kconfig 文件通常位于内核源码的各个子目录中。

**KSPP**：Kernel Self-Protection Project，Linux 内核自我保护项目。该项目旨在提升 Linux 内核的安全防护能力，提供了一系列安全配置建议。

**Linux LTS**：Linux Long Term Support，长期支持版本。Linux 社区会为 LTS 版本提供多年的安全更新和 bug 修复。OpenHarmony 目前基于 4.19.y 和 5.10.y LTS 版本。

**MODULE_SIG**：Linux 内核模块签名机制，用于验证加载的内核模块是否来自可信来源，防止未经授权的代码注入内核。

**N-API**：Native API，本地应用程序接口。在 OpenHarmony 中通常指 C/C++ 暴露给 JavaScript/ArkTS 的原生接口。

**OHOS**：OpenHarmony Operating System，开放鸿蒙操作系统。

**OpenHarmony**：开放原子开源基金会旗下的开源操作系统项目，支持多种设备形态。

**product**：OpenHarmony 配置分层架构中的第五层（产品类型配置），用于在同芯片平台基础上区分不同产品形态。

**RK3568**：瑞芯微半导体推出的 RK3568 芯片，支持 ARM64 架构，广泛应用于 OpenHarmony 开发板。

**standard**：标准系统，OpenHarmony 的一种系统形态，适用于功能丰富的智能设备。标准系统运行完整的服务框架和应用运行环境。

**small**：小系统，OpenHarmony 的一种系统形态，适用于资源受限的物联网设备。小系统运行轻量级的系统服务。

**type**：OpenHarmony 配置分层架构中的第二层（系统形态配置），区分标准系统和小系统的配置差异。

**uImage**：U-Boot 引导的 Linux 内核镜像格式，包含标准 Linux zImage 和 U-Boot 引导头。适用于 ARM32 等老平台。

## 配置项前缀说明

Linux 内核配置项都以 `CONFIG_` 为前缀，本 Wiki 中为简洁起见通常省略该前缀。以下是常见配置项类别的说明。

**CONFIG_ARM64_***：ARM64 架构相关的配置选项，如 `CONFIG_ARM64_PTR_AUTH`（指针认证）。

**CONFIG_FORTIFY_SOURCE**：编译器提供的缓冲区溢出检测功能。

**CONFIG_HARDENED_***：内核强化相关配置，如 `CONFIG_HARDENED_USERCOPY`（用户空间拷贝边界检查）。

**CONFIG_KASAN**：内核地址 sanitizer，用于检测内存访问错误。

**CONFIG_NETFILTER**：网络过滤框架，支持配置防火墙规则。

**CONFIG_SECCOMP***：安全计算模式，限制进程可用的系统调用。

**CONFIG_SECURITY_***：通用安全框架配置，包括 SELinux、AppArmor、SMACK 等。

**CONFIG_SLUB / CONFIG_SLAB**：内核内存分配器配置，影响内存分配的性能和安全性。

## 常见缩写

| 缩写 | 全称 | 说明 |
|------|------|------|
| ABI | Application Binary Interface | 应用程序二进制接口 |
| API | Application Programming Interface | 应用程序编程接口 |
| ELF | Executable and Linkable Format | 可执行和可链接文件格式 |
| GPIO | General-Purpose Input/Output | 通用输入输出 |
| I2C | Inter-Integrated Circuit | 集成电路总线 |
| IPC | Inter-Process Communication | 进程间通信 |
| PAC | Pointer Authentication Code | 指针认证码 |
| ROP | Return-Oriented Programming | 返回导向编程 |
| SPI | Serial Peripheral Interface | 串行外设接口 |
| UART | Universal Asynchronous Receiver/Transmitter | 通用异步收发传输器 |

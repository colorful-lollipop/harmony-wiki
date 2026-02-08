# libbpf - OpenHarmony 第三方库

> **libbpf** 是 eBPF（Extended Berkeley Packet Filter）技术的核心库，用于在 OpenHarmony 系统中开发和使用 BPF 程序。

---

## 快速导航

| 文档 | 说明 | 适合读者 |
|------|------|---------|
| [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | 新用户 |
| [01_Overview.md](01_Overview.md) | libbpf 简介与 OH 定位 | 所有人 |
| [02_Patches.md](02_Patches.md) | Patch 分析 | 维护者 |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配详解 | 构建系统开发者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系与使用场景 | 应用开发者 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异 | 应用开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

---

## libbpf 在 OpenHarmony 中的定位

libbpf 是 OpenHarmony 中 **唯一** 的 eBPF 用户态库，提供以下核心能力：

### 核心功能

- **BPF 程序加载**: 将编译后的 eBPF 字节码加载到内核
- **BPF 程序附着**: 将 BPF 程序附加到内核钩子、跟踪点等
- **BTF 类型系统**: 支持 BPF CO-RE（Compile Once – Run Everywhere）
- **高性能数据通信**: Ring Buffer 和 Perf Buffer

### 在 OH 中的应用场景

| 场景 | 使用模块 | 具体功能 |
|------|---------|----------|
| **网络安全** | netmanager_base | 网络防火墙、流量统计 |
| **性能分析** | hiebpf | 系统性能追踪、内核探针 |
| **性能数据流** | smartperf_host | 性能数据流处理 (hiperf) |

---

## OH 适配概述

### 适配方式

libbpf 在 OpenHarmony 中采用 **构建系统适配** 策略，无源代码 Patch：

- ✅ 使用 ELFIO 库替代 libelf
- ✅ 仅构建动态共享库（`.so`）
- ✅ 禁用 linker.c 和 usdt.c 功能
- ✅ 适配标准系统（standard）

### 关键配置

```gn
ohos_shared_library("libbpf") {
  external_deps = [ "elfio:elfio", "zlib:libz" ]
  configs = [ ":libbpf_config" ]
  branch_protector_ret = "pac_ret"
  install_enable = true
}
```

### 版本信息

| 项目 | 信息 |
|------|------|
| **上游仓库** | https://github.com/libbpf/libbpf |
| **OH 版本** | v1.3.4 |
| **许可证** | BSD-2-Clause OR LGPL-2.1 |
| **子系统** | thirdparty |

---

## 快速开始

### 1. 引入依赖

在模块的 `BUILD.gn` 中添加：

```gn
external_deps = [ "libbpf:libbpf" ]
```

### 2. 使用示例

```c
#include "libbpf.h"

// 加载并验证 BPF 程序
struct bpf_object *obj = bpf_object__open("my_prog.o");
if (!obj) {
    return -1;
}

// 加载到内核
if (bpf_object__load(obj)) {
    return -1;
}

// 查找并附加程序
struct bpf_program *prog = bpf_object__find_program_by_name(obj, "my_kprobe");
if (!prog) {
    return -1;
}

struct bpf_link *link = bpf_program__attach(prog);
if (!link) {
    return -1;
}

// 清理资源
bpf_link__destroy(link);
bpf_object__close(obj);
```

### 3. BPF 程序示例

```c
// kprobe.bpf.c
char LICENSE[] SEC("license") = "Dual BSD/GPL";

SEC("kprobe/do_unlinkat")
int kprobe_do_unlinkat(int dfd, struct filename *name)
{
    pid_t pid = bpf_get_current_pid_tgid() >> 32;
    bpf_printk("pid = %d, unlinkat called\n", pid);
    return 0;
}
```

---

## 文档目录

```
wiki/
├── README.md                   # 本文档 - 库概览
├── SUMMARY.md                  # 阅读路线建议
├── 01_Overview.md             # libbpf 简介 + OH 定位
├── 02_Patches.md             # Patch 分析（说明无实际 Patch）
├── 03_Build_Integration.md    # OH 构建适配详解
├── 04_Usage_in_OH.md        # 依赖关系与使用场景
├── 05_API_Differences.md     # API 差异（说明无差异）
├── 06_Security.md            # 安全风险分析
└── _work/
    ├── ASSESSMENT.md         # 项目评估结果
    ├── PLAN.md              # 任务进度
    └── NOTES.md            # 分析过程记录
```

---

## 参考资源

- **上游文档**: https://libbpf.readthedocs.io/en/latest/
- **BPF CO-RE 指南**: https://nakryiko.com/posts/bpf-core-reference-guide/
- **libbpf-bootstrap**: https://github.com/libbpf/libbpf-bootstrap
- **OH 示例**: 查看 `README_OH.md`

---

**版权**: BSD-2-Clause OR LGPL-2.1
**维护者**: xiazhonglin@huawei.com

# libbpf 概述

> libbpf 是 eBPF（Extended Berkeley Packet Filter）技术的核心用户态库，OpenHarmony 中的唯一 eBPF 开发框架。

---

## 1. 库基本信息

### 1.1 原始库信息

| 项目 | 信息 |
|------|------|
| **名称** | libbpf |
| **上游仓库** | https://github.com/libbpf/libbpf |
| **权威源码** | Linux 内核 bpf-next 树的 `tools/lib/bpf` |
| **OH 版本** | v1.3.4 |
| **许可证** | BSD-2-Clause OR LGPL-2.1 (双许可，可任选其一) |
| **上游同步** | CHECKPOINT-COMMIT: `750011e239a50873251c16207b0fe78eabf8577e` |

### 1.2 OpenHarmony 集成信息

| 项目 | 信息 |
|------|------|
| **组件名称** | @ohos/libbpf |
| **子系统** | thirdparty |
| **适配系统** | standard |
| **构建产物** | libbpf.so (动态共享库) |
| **依赖组件** | elfio, zlib |
| **维护者** | xiazhonglin@huawei.com |

---

## 2. libbpf 是什么

### 2.1 核心定位

libbpf 是 **Linux 内核 eBPF 子系统的用户态配套库**，提供：

- ✅ **BPF 程序加载**: 将编译后的 eBPF 字节码加载到内核
- ✅ **程序验证**: 协助内核验证 BPF 程序安全性
- ✅ **程序附着**: 将 BPF 程序附加到内核钩子、跟踪点、网络接口等
- ✅ **数据通信**: 提供高性能的内核态→用户态数据通道
- ✅ **BTF 类型系统**: 支持 BPF CO-RE（Compile Once – Run Everywhere）

### 2.2 在 Linux 生态中的位置

```
Linux 内核 (bpf-next)
├── kernel/bpf/           ← eBPF 子系统核心
├── tools/lib/bpf/        ← libbpf 权威源码
└── .github/libbpf        ← 自动同步的 GitHub 镜像
                           ↓
                      OpenHarmony
                      third_party/libbpf
```

**设计理念**: libbpf 是**内核无关**的，可以跨多个内核版本工作，独立于 Linux 内核进行版本化管理。

---

## 3. 核心功能

### 3.1 BPF 程序生命周期管理

```c
// 1. 打开 BPF 对象文件
struct bpf_object *obj = bpf_object__open("prog.o");

// 2. 加载到内核并验证
bpf_object__load(obj);

// 3. 查找程序并附加
struct bpf_program *prog = bpf_object__find_program_by_name(obj, "my_kprobe");
struct bpf_link *link = bpf_program__attach(prog);

// 4. 清理资源
bpf_link__destroy(link);
bpf_object__close(obj);
```

### 3.2 支持的 BPF 程序类型

| 类型 | SEC() 标记 | 用途 |
|------|-----------|------|
| **kprobe/kretprobe** | `kprobe/function_name` | 内核函数跟踪 |
| **tracepoint** | `tracepoint/category/event` | 内核跟踪点 |
| **uprobe/uretprobe** | `uprobe/binary:offset` | 用户态函数跟踪 |
| **XDP** | `xdp` | 网络包处理（高性能） |
| **TC** | `tc` | 流量控制 |
| **Socket Filter** | `socket` | socket 过滤 |
| **cgroup/skb** | `cgroup/skb` | cgroup 网络钩子 |
| **LSM** | `lsm/` | Linux 安全模块 |

### 3.3 BPF CO-RE（Compile Once – Run Everywhere）

**问题**: 传统 BPF 程序需要为每个内核版本重新编译（依赖内核头文件）。

**CO-RE 解决方案**:
- 利用 BTF（BPF Type Format）类型信息
- 通过 BPF 重定位实现内核无关性
- **一次编译，跨内核运行**

**OH 中的应用**:
- netmanager_base 的网络防火墙程序可以在不同 OH 内核版本间兼容
- hiebpf 的性能追踪程序无需为每个内核版本重新编译

### 3.4 高性能数据通道

#### Ring Buffer
- 零拷贝、无锁设计
- 支持批量数据传输
- 适合高频事件追踪

#### Perf Buffer
- 基于 perf_event 机制
- 兼容旧版本内核
- 适合低频事件通知

```c
// Ring Buffer 示例
struct ring_buffer *rb = ring_buffer__new(map_fd, handle_event, NULL, NULL);
ring_buffer__poll(rb, 100 /* timeout_ms */);
ring_buffer__free(rb);
```

---

## 4. 在 OpenHarmony 中的定位与作用

### 4.1 系统定位

libbpf 在 OpenHarmony 中是 **eBPF 开发的唯一用户态框架**，承担以下角色：

| 角色 | 说明 |
|------|------|
| **内核扩展桥梁** | 用户态程序与内核 eBPF 子系统的桥梁 |
| **性能分析基础** | 为系统性能分析工具（hiebpf）提供底层支持 |
| **网络安全引擎** | 为网络防火墙（netmanager_base）提供 BPF 程序加载能力 |
| **可观测性工具** | 支持系统级追踪和监控 |

### 4.2 与上游的关系

**重要**: libbpf 在 OpenHarmony 中是**纯净的上游版本**，无源代码修改。

| 对比项 | 上游 | OH 版本 |
|--------|------|---------|
| **源代码** | 完整 | 完整（纯净上游） |
| **Patch** | 无 | 无（只有构建适配） |
| **API** | 标准 | 标准（无 OH 特定 API） |
| **功能** | 完整 | 部分禁用（linker.c, usdt.c） |
| **构建系统** | Makefile | BUILD.gn (GN) |

### 4.3 适配策略

OpenHarmony 通过 **构建系统适配** 集成 libbpf，而非源代码 Patch：

#### 构建层适配
- ✅ 使用 ELFIO 库替代 libelf（`HAVE_ELFIO`）
- ✅ 仅构建动态共享库（`.so`）
- ✅ 禁用 linker.c（BPF 静态链接器）
- ✅ 禁用 usdt.c（用户态动态追踪）
- ✅ ARM64 PAC-RET 分支保护

#### 编译器配置
- 大量警告抑制（内核编程模式与用户态编译器标准不符）
- 保留帧指针（`-fno-omit-frame-pointer`）
- 禁用优化（`-fno-inline`, `-fno-optimize-sibling-calls`）

---

## 5. 目录结构概览

```
libbpf/
├── BUILD.gn                    # OH 构建脚本
├── bundle.json                 # OH 组件元数据
├── README_OH.md                # OH 集成文档（中文）
│
├── src/                        # 核心源代码
│   ├── libbpf.c/h             # 主 API 实现 (375KB/71KB)
│   ├── bpf.c/h                # BPF 系统调用封装
│   ├── btf.c/h               # BTF 类型系统
│   ├── btf_dump.c            # BTF 打印输出
│   ├── elf.c                 # ELF 文件处理
│   ├── netlink.c             # Netlink 通信
│   ├── gen_loader.c          # BPF 加载器生成
│   ├── relo_core.c/h        # CO-RE 重定位
│   ├── ringbuf.c            # Ring Buffer 实现
│   └── ...                 # 其他辅助模块
│
├── include/                    # 公共头文件
│   ├── uapi/linux/          # 内核 UAPI (bpf.h, btf.h 等)
│   ├── linux/               # 内核兼容层
│   └── asm/                # 架构相关
│
├── docs/                       # 官方文档
├── scripts/                    # 维护脚本 (sync-kernel.sh)
└── ci/                         # CI/CD 配置
```

---

## 6. 核心源文件说明

| 文件 | 大小 | 功能 |
|------|------|------|
| **libbpf.c** | 375KB | 核心库实现，对象加载、程序附着等主 API |
| **libbpf.h** | 71KB | 公共 API 头文件，200+ 个接口 |
| **bpf.c** | 36KB | BPF 系统调用封装 |
| **btf.c** | 139KB | BTF 类型系统实现 |
| **btf_dump.c** | 70KB | BTF 类型打印输出 |
| **elf.c** | 18KB | ELF 文件解析 |
| **netlink.c** | - | Netlink 与内核通信 |
| **relo_core.c** | - | BPF CO-RE 重定位逻辑 |
| **ringbuf.c** | - | Ring Buffer 实现 |

---

## 7. 快速开始

### 7.1 引入依赖

```gn
# 在模块的 BUILD.gn 中
external_deps = [ "libbpf:libbpf" ]
```

### 7.2 编写 BPF 程序

```c
// myprog.bpf.c
#include "bpf_helpers.h"

char LICENSE[] SEC("license") = "Dual BSD/GPL";

SEC("kprobe/do_unlinkat")
int kprobe_do_unlinkat(int dfd, struct filename *name)
{
    pid_t pid = bpf_get_current_pid_tgid() >> 32;
    bpf_printk("pid = %d, unlinkat called\n", pid);
    return 0;
}
```

### 7.3 用户态程序

```c
#include "libbpf.h"

int main() {
    // 加载 BPF 程序
    struct bpf_object *obj = bpf_object__open("myprog.bpf.o");
    if (bpf_object__load(obj)) {
        return -1;
    }

    // 附加到内核
    struct bpf_program *prog = bpf_object__find_program_by_name(obj,
                                                          "kprobe_do_unlinkat");
    struct bpf_link *link = bpf_program__attach(prog);
    if (!link) {
        return -1;
    }

    // 等待事件
    printf("BPF program attached, press Ctrl+C to exit...\n");
    pause();

    // 清理
    bpf_link__destroy(link);
    bpf_object__close(obj);
    return 0;
}
```

---

## 8. 参考资料

### 官方文档

- **libbpf API 文档**: https://libbpf.readthedocs.io/en/latest/api.html
- **BPF CO-RE 指南**: https://nakryiko.com/posts/bpf-core-reference-guide/
- **libbpf 开发指南**: https://nakryiko.com/posts/bcc-to-libbpf-howto-guide/
- **libbpf-bootstrap**: https://github.com/libbpf/libbpf-bootstrap

### 内核资源

- **Linux 内核 bpf-next**: https://git.kernel.org/pub/scm/linux/kernel/git/bpf/bpf-next.git/
- **BPF 邮件列表**: bpf@vger.kernel.org

### OpenHarmony 资源

- **README_OH.md**: OH 集成文档（包含示例代码）
- **scripts/sync-kernel.sh**: 内核同步脚本
- **SYNC.md**: 同步说明

---

**下一步**: 阅读 [03_Build_Integration.md](03_Build_Integration.md) 了解 OH 构建适配详情

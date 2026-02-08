# VIXL - ARMv8 运行时代码生成库

## 库概述

VIXL (VIXL: ARMv8 Runtime Code Generation Library) 是一个用于在运行时生成和解码 ARM 指令的高性能库。在 OpenHarmony 中，VIXL 主要服务于**方舟编译器 (arkcompiler)** 的代码生成后端。

**版本**: 7.0.0  
**许可证**: BSD 3-Clause  
**上游地址**: https://github.com/Linaro/vixl

## OpenHarmony 适配特点

### 无需 Patch

VIXL 库在 OpenHarmony 中**无需任何源代码修改**即可使用，这得益于：

- VIXL 本身是架构无关的代码生成库，不依赖特定操作系统
- OpenHarmony 仅使用 VIXL 的基础指令生成功能
- 所有 OH 特定适配通过 BUILD.gn 构建配置完成

### 核心适配内容

| 适配项 | 说明 |
|-------|------|
| **构建系统** | GN 构建系统适配 |
| **内存分配** | 代码缓冲区使用 mmap 分配 (`VIXL_CODE_BUFFER_MMAP`) |
| **编译器标识** | Panda 构建标识 (`PANDA_BUILD`) |
| **工具链兼容** | OHOS 工具链缺失函数重实现 |

## 主要使用者

- **方舟编译器 (arkcompiler)**: VIXL 的主要使用者，用于 AArch64/AArch32 指令生成
- **aotdump 工具**: AOT 镜像反编译工具
- **pasteboard 服务**: 剪贴板服务的测试用例

## 文档导航

### 快速入门

1. **[01_Overview.md](01_Overview.md)** - 库功能简介
2. **[03_Build_Integration.md](03_Build_Integration.md)** - 构建适配说明
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** - 在 OH 中的使用方式

### 进阶内容

4. **[05_API_Differences.md](05_API_Differences.md)** - API 使用说明

## 编译与集成

### 编译目标

```bash
./build.sh --product-name rk3568 --build-target libvixl_frontend_static
```

### 依赖添加

在 BUILD.gn 中添加依赖：
```gn
external_deps += [ "vixl:libvixl" ]
```

## 相关信息

- **上游文档**: [VIXL GitHub](https://github.com/Linaro/vixl)
- **指令支持列表**: `doc/aarch64/supported-instructions-aarch64.md`
- **构建产物**: `out/rk3568/obj/third_party/vixl/libvixl_frontend_static.a`

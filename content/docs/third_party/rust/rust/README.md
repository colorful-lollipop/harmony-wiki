# Rust Toolchain for OpenHarmony

## 库概览

本工具链基于 **Rust 1.72.0** 与 **LLVM 15.0.4** 构建，为 OpenHarmony 提供了完整的 Rust 交叉编译支持。

| 项目 | 值 |
|------|-----|
| **版本** | Rust 1.72.0 ( nightly ) |
| **许可证** | Apache-2.0 / MIT |
| **上游地址** | https://github.com/rust-lang/rust |
| **维护者** | qilin.wang@huawei.com |

## OpenHarmony 适配概述

此工具链实现了三个 OpenHarmony 目标平台的交叉编译支持：

| 目标平台 | 架构 | 说明 |
|---------|------|------|
| `aarch64-unknown-linux-ohos` | ARM64 | 适用于高端手机、平板等设备 |
| `armv7-unknown-linux-ohos` | ARMv7-A | 适用于 IoT 设备、智能穿戴等 |
| `x86_64-unknown-linux-ohos` | x86_64 | 适用于 x86 模拟器开发 |

### 主要适配内容

1. **目标三元组定义** - 在 `compiler/rustc_target/src/spec/` 中新增三个目标文件
2. **TLS 仿真支持** - OpenHarmony 无原生线程本地存储，使用仿真方案
3. **musl libc 集成** - 使用 musl 作为 C 标准库基础
4. **Clang 包装器** - 提供 OHOS SDK 集成的编译器包装脚本
5. **CI/CD 流程** - 完整的自动化构建和测试脚本

## 文档导航

### 核心文档

| 文档 | 说明 |
|------|------|
| [README.md](./README.md) | 本文档 |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 |
| [01_Overview.md](./01_Overview.md) | 原始库与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | **核心** - Patch 详细分析 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建系统适配 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 使用场景与依赖 |
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异 |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估结果 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度跟踪 |

## 快速开始

### 构建工具链

```bash
# 克隆 OpenHarmony 构建仓库
git clone https://gitee.com/openharmony/build.git
cd build

# 下载依赖
export PYTHONIOENCODING=utf-8
bash build/prebuilts_download.sh
pip3 install requests
python3 ./build/scripts/download_sdk.py --branch OpenHarmony-5.1.0-Release --product-name ohos-sdk-full

# 同步代码
repo init -u https://gitee.com/OpenHarmony/manifest.git -b master -m rust-toolchain.xml
repo sync -c
repo forall -c 'git lfs pull'

# 构建 Rust 工具链
bash third_party/rust/rust/rust-build/ohos_ci_build.sh
```

### 使用工具链

```bash
# 编译 Rust 程序到 OHOS 目标
rustc --target aarch64-unknown-linux-ohos your_program.rs

# 或使用 cargo
cargo build --target aarch64-unknown-linux-ohos
```

## 相关资源

- [Rust 官方文档](https://doc.rust-lang.org/)
- [OpenHarmony 构建子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-rust-toolchain.md)
- [Rust 平台支持](https://doc.rust-lang.org/rustc/platform-support.html)

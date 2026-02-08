# mksh (MirBSD Korn Shell) - OpenHarmony 第三方库文档

## 库概览

mksh 是 MirBSD Korn Shell 的缩写，是一个命令解释器（shell），用于 shell 命令交互和 shell 脚本语言执行。它是 Shell 语言的超集，同时兼容原本的 Korn shell（ksh93）。

**在 OpenHarmony 中的定位**：
- 作为系统的标准 shell 提供命令行交互能力
- 为系统服务（如 samgr_lite）提供脚本执行环境
- 轻量级设计（ROM 172KB，RAM 344KB），适合嵌入式设备

## OH 适配概述

mksh 库的 OH 适配采用**源码级直接集成**方式，而非传统 Patch 文件方式：

| 适配类型 | 详情 |
|----------|------|
| **适配宏** | `MKSH_OH_ADAPT`（通用适配）、`MKSH_TERMINAL_EXT`（终端扩展） |
| **适配文件** | edit.c、exec.c、funcs.c、main.c |
| **适配数量** | 5 处代码修改 |
| **适配内容** | 稳定性修复、终端功能增强 |

### 主要适配内容

1. **稳定性修复**
   - `exec -a0` 命令空指针崩溃修复
   - 管道写入重试机制，提高 I/O 可靠性

2. **终端功能增强**
   - 终端清除字符串优化
   - 终端扩展键绑定支持
   - 窗口大小动态更新

3. **构建系统适配**
   - GN 构建系统集成
   - OHOS 特定编译配置
   - 安全编译选项强化

## 文档导航

| 文档 | 内容 | 建议阅读对象 |
|------|------|--------------|
| [SUMMARY.md](SUMMARY.md) | 文档阅读路线建议 | 所有读者 |
| [01_Overview.md](01_Overview.md) | 原始库功能简介 | 初次接触该库 |
| [02_Patches.md](02_Patches.md) | OH 适配代码详解 | 需要理解适配细节 |
| [03_Build_Integration.md](03_Build_Integration.md) | 构建配置说明 | 构建系统开发者 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用方式 | 系统集成开发者 |
| [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

## 快速开始

### 在 OH 中使用 mksh

```gn
# 在 BUILD.gn 中添加依赖
public_deps += ["//third_party/mksh"]
```

### 终端使用

```bash
# 进入 mksh shell
./bin/mksh

# 查看版本
echo $KSH_VERSION
```

## 版本信息

| 项目 | 信息 |
|------|------|
| **上游版本** | R59c |
| **OH 版本** | 3.1 |
| **许可证** | MirOS License |
| **上游地址** | https://www.mirbsd.org/mksh.htm |

## 贡献者

| 角色 | 联系 |
|------|------|
| OH 维护者 | maguangyao@huawei.com |

## 相关链接

- [上游项目](https://www.mirbsd.org/mksh.htm)
- [上游 GitHub](https://github.com/MirBSD/mksh)
- [上游 FAQ](http://www.mirbsd.org/mksh-faq.htm)
- [OH Bundle 配置](../bundle.json)
- [OH BUILD.gn](../BUILD.gn)

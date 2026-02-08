# OpenHarmony Kernel Linux Patches Wiki

> 本 Wiki 由 OpenHarmony 工程 Wiki 生成工具自动生成

## 文档覆盖范围

本文档涵盖 `kernel_linux_patches` 仓库的完整工程分析，包括：

### 已覆盖内容

1. **项目概述** - 仓库定位、核心能力、运行环境
2. **目录结构** - 模块划分、文件组织、职责说明
3. **支持的板卡** - 芯片平台、开发板列表、适配状态
4. **补丁分析** - HDF 补丁、SOC 补丁、配置补丁详解
5. **构建系统** - GN 构建配置、构建流程、产物说明
6. **安全评审** - 潜在风险、修复建议、注意事项

### 未覆盖内容

- Linux 内核源码（不在本仓库范围内）
- 用户态 N-API 接口（本仓库为内核补丁，无用户态 API）
- 完整的 CVE 补丁列表（仅包含 OpenHarmony 相关补丁）

## 文档更新方式

### 触发更新的场景

当仓库发生以下变更时，建议更新本文档：

1. 新增支持的芯片平台或开发板
2. 新增或修改补丁类型
3. 修改构建配置或构建流程
4. 新增安全相关补丁

### 更新步骤

1. 克隆仓库：`git clone https://gitee.com/openharmony/kernel_linux_patches.git`
2. 进入 Wiki 目录：`cd kernel_linux_patches/wiki`
3. 更新对应章节的 Markdown 文件
4. 提交更改并推送到远程仓库

## 生成信息

- **生成时间**: 2026-02-06
- **仓库版本**: 基于当前仓库状态
- **内核版本**: Linux 4.19.y / 5.10.y / 6.6.y

## 联系方式

如有问题或建议，请通过 OpenHarmony 社区渠道反馈：

- [OpenHarmony Gitee](https://gitee.com/openharmony)
- [OpenHarmony 社区](https://www.openharmony.cn/)

## 贡献指南

欢迎为本 Wiki 贡献内容：

1. Fork 本仓库
2. 创建分支：`git checkout -b wiki-update`
3. 修改 Wiki 文件
4. 提交更改：`git commit -m "docs: update wiki content"`
5. 推送并创建 Pull Request

## 许可证

本文档遵循 [Apache License 2.0](./LICENSE) 许可证。

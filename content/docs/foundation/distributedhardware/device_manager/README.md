# DeviceManager Wiki

## 概述

本 Wiki 是 OpenHarmony [DeviceManager](../README_zh.md) 组件的工程文档，旨在帮助开发者快速理解项目架构、API 使用、构建配置以及安全考量。

**项目定位**：DeviceManager 提供账号无关的分布式设备的认证组网能力，为开发者提供了一套用于分布式设备间监听、发现和认证的接口。

## 覆盖范围

### 已覆盖内容

#### 基础理解
- [项目概览](01_Overview.md)：核心能力、依赖组件、适用场景
- [架构说明](02_Architecture.md)：组件图、数据流、线程模型
- [代码地图](03_CodeMap.md)：目录结构、关键文件定位

#### API 参考
- [N-API 参考](03_N_API_Reference.md)：JS 接口清单、注册机制、调用链
- [内部 API](04_Inner_API.md)：模块接口、依赖方向、稳定性标注

#### 安全分析 ⭐ 新增
- [攻击面分析](05_AttackSurface.md)：N-API/IPC/网络攻击面地图 ⭐
- [安全风险评估](06_SecurityReview.md)：详细风险分析、修复建议 ⭐
- [安全风险评审](07_Security_Review.md)：原始安全评审

#### 工程实现
- [GN 构建配置](05_GN_Build.md)：Targets 列表、依赖关系
- [编译产物说明](06_Artifacts.md)：.so/.hap 文件、安装路径

#### 附录
- [调用链图](appendix/Callgraphs.md)：关键调用链
- [配置开关](appendix/Config_Flags.md)：编译配置说明

### 文档统计
- 核心文档：11 篇
- 附录文档：2 篇
- 工作区文档：3 篇
- **总计：16 篇**

## 更新方式

本 Wiki 随代码变更自动生成。更新方式：

```bash
# 重新生成 Wiki（需运行 Wiki 生成 Agent）
./generate_wiki.sh

# 或手动触发
python3 scripts/generate_wiki.py
```

**注意**：仅更新 `wiki/` 目录下的文件，不修改源码。

## 代码证据要求

所有关键结论均可在仓库中找到直接证据：
- 文件路径（必要时包含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

无法确认的结论标记为 `TODO(需确认)` 并说明缺少的证据。

## 新人阅读建议

建议按照以下顺序阅读：

1. [README.md](README.md) → 了解 Wiki 覆盖范围
2. [01_Overview.md](01_Overview.md) → 理解项目定位与核心能力
3. [02_Architecture.md](02_Architecture.md) → 掌握整体架构
4. [03_N_API_Reference.md](03_N_API_Reference.md) → 学习 API 使用方法
5. 根据需要查阅其他文档

## 相关链接

- [OpenHarmony DeviceManager 源码](../README_zh.md)
- [API 文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-distributedservice-kit/js-apis-distributedDeviceManager.md)
- [仓颉 FFI 接口](../interfaces/cj/)
- [内部 Native 接口](../interfaces/inner_kits/native_cpp/)

## 生成信息

- **生成时间**：2025-02-06
- **文档版本**：v1.0
- **代码版本**：OpenHarmony DeviceManager 主线版本

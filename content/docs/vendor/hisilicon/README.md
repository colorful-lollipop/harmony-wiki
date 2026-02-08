# Hisilicon Vendor Wiki 索引

## 文档说明

本文档为 OpenHarmony Hisilicon Vendor 仓库的工程 Wiki，旨在帮助开发者快速理解项目结构、配置体系和使用方法。

### 覆盖范围

本 Wiki 覆盖以下内容：

- **项目定位与架构**：仓库目标、产品系列、模块职责
- **目录结构**：各目录用途与组织方式
- **配置体系**：HDF 配置、产品配置、GN 构建等
- **Demo 代码**：各开发板的示例程序
- **构建产物**：编译输出与部署方式
- **安全风险**：配置安全分析与建议

### 未覆盖范围

- **SOC 相关配置**（请参考 `device_soc_hisilicon` 仓库）
- **内核源码**（请参考对应内核仓库）
- **N-API 接口**（本仓库不包含 N-API）

### 更新方式

本 Wiki 由代码分析自动生成。如需更新：

1. 修改/新增配置文件后，运行 Wiki 生成工具
2. 验证文档链接正确性
3. 提交更改

### 生成信息

- **生成时间**：2026-02-06
- **仓库版本**：master 分支
- **分析工具**：OpenHarmony Wiki Generator

---

## 快速导航

### 新人阅读路线

建议阅读顺序：

1. [项目概述](01_Overview.md) - 了解项目定位与目标
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [产品系列](03_Products.md) - 了解支持的开发板
4. [配置体系](04_Configuration.md) - 掌握配置方法
5. [Demo 示例](05_Demos.md) - 学习示例代码
6. [构建指南](06_Build.md) - 了解构建流程
7. [安全指南](07_Security.md) - 关注安全风险

### 按功能查找

| 功能 | 文档 |
|-----|------|
| 了解项目背景 | [项目概述](01_Overview.md) |
| 查找文件位置 | [目录结构](02_Directory_Structure.md) |
| 了解开发板特性 | [产品系列](03_Products.md) |
| 修改驱动配置 | [HDF 配置](04_Configuration.md#hdf-配置) |
| 添加预装应用 | [预安装配置](04_Configuration.md#预安装配置) |
| 学习 WiFi 使用 | [WiFi Demo](05_Demos.md#easy_wifi_demo) |
| 编译项目 | [构建指南](06_Build.md) |
| 安全加固 | [安全指南](07_Security.md) |

---

## 相关链接

### 内部导航

- [SUMMARY.md](./SUMMARY.md) - 全站导航

### 官方资源

- [OpenHarmony 官方文档](https://www.harmonyos.com/)
- [海思开发者资源](https://device.harmonyos.com/)
- [device_board_hisilicon](https://gitee.com/openharmony/device_board_hisilicon)
- [device_soc_hisilicon](https://gitee.com/openharmony/device_soc_hisilicon)

---

## 贡献指南

欢迎贡献和改进本 Wiki。请遵循以下原则：

1. **证据充分**：关键结论需有代码路径作为证据
2. **结构清晰**：保持文档层次分明
3. **术语统一**：使用项目统一的术语
4. **及时更新**：代码变更后同步更新文档

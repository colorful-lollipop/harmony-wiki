# Wifi IoT Sample 应用 Wiki

本文档为 OpenHarmony wifi-iot 示例应用的完整技术文档。

## 文档范围

**已覆盖内容**:
- 项目定位与架构概览
- 目录结构与模块职责
- GN 构建系统配置
- SAMGR_Lite 服务框架示例
- IoT 硬件操作示例（GPIO）
- 对外 API 接口文档（C 层）
- 编译产物与安装说明
- 安全风险评审

**未覆盖内容**:
- 详细的内核 API 文档（请参考 OpenHarmony 官方文档）
- 具体硬件板级的配置说明
- 第三方依赖库的详细文档

## 生成信息

- **生成时间**: 2026-02-05
- **项目版本**: 4.0.2
- **仓库地址**: https://gitee.com/openharmony/applications_sample_wifi_iot
- **许可证**: Apache License 2.0

## 文档维护

本文档基于源代码自动/手动生成。当以下情况发生时，需要更新文档：
- 新增/删除源代码模块
- 修改 GN 构建配置
- 更新依赖组件
- 新增对外接口

更新建议：
1. 运行完整代码扫描
2. 验证所有文件路径和符号引用
3. 更新相关章节的修改日期
4. 检查交叉引用的有效性

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [SAMGR_Lite 框架文档](https://gitee.com/openharmony/systemabilitymgr_samgr_lite)
- [LiteOS-M 内核文档](https://gitee.com/openharmony/kernel_liteos_m)

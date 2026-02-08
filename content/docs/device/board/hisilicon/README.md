# OpenHarmony Hisilicon Board Wiki

> 本 Wiki 为 OpenHarmony Hisilicon 板卡仓库的完整技术文档
>
> 生成时间: 2026-02-06 10:14:07
>
> 仓库路径: `/Volumes/lexar/code/d/work/oh/device/board/hisilicon`

---

## 文档覆盖范围

### ✅ 已覆盖内容
- 项目概述与板卡介绍
- 目录结构与模块职责
- 架构说明（板卡初始化、安全启动、驱动 HAL）
- GN 构建系统与 Targets
- 编译产物与运行时加载关系
- 安全风险评审（Secure Boot、硬件访问控制）
- 常见构建/运行/调试问题

### ❌ 未覆盖内容
- N-API 模块文档（本仓库为板卡级，不包含应用层 API）
- IPC/ServiceAbility 文档（本仓库不包含系统服务）
- 应用层权限管理（位于框架层）
- 业务功能文档（如相机应用、音频应用等）

---

## 如何更新文档

1. **代码变更后**: 对应章节需要更新时，请保留代码证据（文件路径、行号、符号名）
2. **新增板卡**: 需要在以下文档中更新
   - `00_Overview.md` - 添加板卡信息表
   - `01_Directory_Structure.md` - 添加目录结构分析
   - `03_Architecture.md` - 添加架构说明
   - `04_GN_Targets.md` - 添加构建目标
3. **新增配置/特性**: 更新对应的架构和 GN Targets 文档
4. **安全发现**: 更新 `06_Security_Review.md`，保持证据链完整

---

## 文档生成信息

- **生成工具**: OpenHarmony Wiki Generation Agent (Sisyphus)
- **生成依据**: 代码静态分析 + GN 构建系统解析
- **证据来源**: 仓库源代码、配置文件、构建脚本
- **验证方式**: 所有结论基于代码证据，不含推测内容

---

## 文档维护原则

1. **证据优先**: 每个关键结论必须提供代码证据（路径 + 符号 + 行号）
2. **不推测**: 无法确认的内容必须标注 `TODO(需确认)` 并说明缺失证据
3. **排除测试**: 所有文档不引用测试相关内容
4. **中文优先**: 默认使用中文文档，特殊情况可使用英文

---

## 联系与反馈

如有疑问或发现文档错误，请通过以下方式反馈:
- 提交 Issue 到板卡仓库
- 提交 PR 更新 Wiki 文档

---

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Hisilicon SoC 仓库](https://gitee.com/openharmony/device_soc_hisilicon)
- [Hisilicon Vendor 仓库](https://gitee.com/openharmony/vendor_hisilicon)
- [U-Boot 开源项目](https://gitee.com/openharmony/third_party_u-boot)

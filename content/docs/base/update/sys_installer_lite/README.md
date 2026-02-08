# sys_installer_lite 工程 Wiki

## 文档说明

本文档是 OpenHarmony `sys_installer_lite` 组件的工程 Wiki，旨在帮助开发者快速理解项目架构、接口定义和安全特性。

## 文档生成信息

- **生成时间**: 2026-02-06
- **版本**: 4.0.2 (bundle.json)
- **仓库路径**: `base/update/sys_installer_lite`
- **代码提交**: HEAD

## 覆盖范围

### ✅ 已覆盖

- 项目定位与核心能力
- 目录结构与模块职责
- 对外 C API 完整清单（15个函数）
- HAL 接口定义（19个函数）
- 架构组件图与数据流
- GN 构建目标与产物
- 安全风险评审（攻击面、可被利用点）

### ⚠️ 需厂商实现部分（本仓库未包含）

- HAL 具体实现（`hal_update` 库）
- 芯片厂商适配层（board 层实现）
- 公钥存储与读取机制

### ❌ 未覆盖（按规则排除）

- 单元测试代码（test/ 目录）
- 构建系统测试用例

## 文档更新方式

本文档基于代码分析自动生成。当代码变更时：

1. 检查 `bundle.json` 中的版本号是否变更
2. 重新扫描关键头文件（hota_updater.h, hal_hota_board.h）
3. 更新 API 清单和调用链
4. 校验安全风险点是否修复或新增

## 相关链接

- [OpenHarmony Update 子系统文档](https://gitcode.com/openharmony/docs/blob/master/en/readme/update.md)
- [device_hisilicon_hardware HAL 示例](https://gitcode.com/openharmony/device_hisilicon_hardware)

---

**注意**: 本文档中的所有代码证据均基于仓库实际文件路径和行号，关键结论可追溯验证。

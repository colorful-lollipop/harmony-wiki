# 工具集说明

## 概述

集成验证部件提供了丰富的工具集，用于支持测试、分析、构建等多种场景。这些工具位于 `tools/` 目录下，是 OpenHarmony 开发工作流的重要组成部分。

## 工具分类总览

| 工具分类 | 目录 | 主要功能 |
|----------|------|----------|
| 资源分析 | rom_ram_analyzer | ROM/RAM 使用分析 |
| 自动化测试 | fotff | OTA 升级流程测试框架 |
| 开源合规 | opensource_tools | 许可证检查与合规验证 |
| 代码管控 | code_access_control | 代码访问控制检查 |
| 依赖分析 | components_deps | 组件依赖关系分析 |
| 依赖守护 | deps_guard | 依赖关系守护 |
| 门禁检查 | gated_check_in | 门禁配置检查 |
| 精准构建 | precise_build | 精准构建工具 |
| 启动守护 | startup_guard | 启动流程守护 |
| ArkUI 工具 | arkui_tools | ArkUI 相关工具 |

## ROM/RAM 分析工具

### 工具概述

**目录**：`tools/rom_ram_analyzer/`

**功能**：分析二进制文件的 ROM 和 RAM 占用情况，帮助开发者优化资源使用。

### 版本支持

| 版本类型 | 目录 | 说明 |
|----------|------|------|
| 标准版 | `tools/rom_ram_analyzer/standard/` | 适用于标准系统设备 |
| 轻量版 | `tools/rom_ram_analyzer/lite_small/` | 适用于轻量系统设备 |

### 核心脚本

| 脚本名 | 功能 |
|--------|------|
| `rom_analyzer.py` | ROM 使用分析器 |
| `ram_analyzer.py` | RAM 使用分析器 |

### 使用方法

```bash
# ROM 分析
python3 tools/rom_ram_analyzer/standard/rom_analyzer.py [参数]

# RAM 分析
python3 tools/rom_ram_analyzer/standard/ram_analyzer.py [参数]
```

### 依赖包目录

**目录结构**：
```
pkgs/
├── rom_ram_baseline_collector.py  # 基线数据收集器
├── basic_tool.py                 # 基础工具函数
├── gn_common_tool.py             # GN 构建系统通用工具
├── simple_excel_writer.py        # Excel 输出工具
└── simple_yaml_tool.yaml         # YAML 配置工具
```

## 自动化测试框架

### 工具概述

**目录**：`tools/fotff/`

**全称**：Full OTA Flow Framework

**功能**：完整的 OTA（Over-The-Air）升级流程测试框架，支持自动化端到端测试。

### 核心特性

- 自动化测试流程控制
- 测试结果收集与分析
- 支持多种测试场景
- 与 CI/CD 流程集成

## 开源合规工具

### 工具概述

**目录**：`tools/opensource_tools/`

**功能**：开源许可证合规检查，确保项目使用开源软件的合规性。

### 核心脚本

| 脚本名 | 功能 |
|--------|------|
| `generate_readme_opensource.py` | 生成开源软件 README |
| `validate_readme_opensource.py` | 验证开源软件 README 合规性 |
| `spdx_license_matcher.py` | SPDX 许可证匹配器 |

### 许可证数据

**数据文件**：`tools/opensource_tools/data/spdx.json`

**说明**：包含 SPDX 标准许可证信息，用于许可证识别和匹配。

### 使用方法

```bash
# 生成开源软件 README
python3 tools/opensource_tools/src/generate_readme_opensource.py

# 验证 README 合规性
python3 tools/opensource_tools/src/validate_readme_opensource.py

# 许可证匹配
python3 tools/opensource_tools/src/spdx_license_matcher.py [选项]
```

## 代码访问控制工具

### 工具概述

**目录**：`tools/code_access_control/`

**功能**：代码访问控制检查，确保代码符合访问控制策略。

### 检查脚本

| 脚本名 | 功能 |
|--------|------|
| `check_arkweb_hard_coded.py` | 检查硬编码问题 |
| `part_compile_build.py` | 部件编译构建检查 |
| `sdk_check_comment.py` | SDK 注释规范检查 |

## 组件依赖分析工具

### 工具概述

**目录**：`tools/components_deps/`

**功能**：分析组件间的依赖关系，帮助理解系统结构和优化依赖管理。

### 主要能力

- 依赖关系可视化
- 循环依赖检测
- 依赖版本管理

## 依赖守护工具

### 工具概述

**目录**：`tools/deps_guard/`

**功能**：守护依赖关系，防止不合理的依赖变更影响系统稳定性。

### 主要能力

- 依赖变更监控
- 依赖合规性检查
- 异常依赖告警

## 门禁检查工具

### 工具概述

**目录**：`tools/gated_check_in/`

**功能**：检查门禁配置的正确性和完整性。

### 主要能力

- 用例配置验证
- 门禁规则检查
- 配置格式校验

## 精准构建工具

### 工具概述

**目录**：`tools/precise_build/`

**功能**：提供精准构建能力，减少不必要的构建开销，提高构建效率。

### 主要能力

- 变更影响分析
- 增量构建支持
- 构建依赖优化

## 启动守护工具

### 工具概述

**目录**：`tools/startup_guard/`

**功能**：监控和守护系统启动流程，确保系统启动的稳定性和可靠性。

### 主要能力

- 启动流程监控
- 启动时间分析
- 启动异常检测

## ArkUI 工具

### 工具概述

**目录**：`tools/arkui_tools/`

**功能**：提供 ArkUI 开发相关的辅助工具。

### 核心脚本

| 脚本名 | 功能 |
|--------|------|
| `arkui_hfile_counts.py` | ArkUI 资源文件统计 |

## 主检查入口

### check.py

**路径**：`tools/check.py`

**功能**：提供统一的命令行检查入口，整合各类检查工具。

### 使用方法

```bash
# 执行所有检查
python3 tools/check.py

# 执行特定检查
python3 tools/check.py --tool [工具名]

# 查看帮助
python3 tools/check.py --help
```

## 工具使用建议

### 日常开发

- 使用 `rom_ram_analyzer` 定期检查资源占用
- 使用 `code_access_control` 确保代码质量

### CI/CD 集成

- 在门禁流程中使用 `gated_check_in` 验证配置
- 使用 `fotff` 执行自动化测试
- 使用 `deps_guard` 监控依赖变更

### 版本发布

- 使用 `precise_build` 优化构建流程
- 使用 `startup_guard` 验证启动性能
- 使用 `opensource_tools` 确保合规性

## 相关文档

- [概览](./index.md)
- [目录结构与模块职责](./01_Directory_Structure.md)
- [用例管理](./02_Test_Cases.md)
- [部署与使用指南](./04_Deployment.md)
- [架构设计](./05_Architecture.md)

# OpenHarmony Theme 应用 Wiki

## 文档覆盖范围

本 Wiki 文档覆盖 OpenHarmony `applications_theme` 仓库的完整工程信息，包括：

### 已覆盖内容

| 类别 | 覆盖范围 |
|------|----------|
| **项目定位** | 主题应用的核心功能、运行环境、关键概念 |
| **目录结构** | 按职责归类的模块结构和文件组织 |
| **架构说明** | 组件图、数据流、关键时序（Mermaid） |
| **对外 API** | N-API 接口清单（本项目无 N-API，纯 ETS 项目） |
| **内部 API** | 模块接口、依赖方向、生命周期机制 |
| **GN Targets** | 构建目标梳理（本项目使用 hvigor + Node.js） |
| **编译产物** | .hap 安装包、运行时加载关系 |
| **安全评审** | 攻击面、信任边界、风险清单 |
| **问题定位** | 常见构建/运行/调试问题与解决路径 |

### 未覆盖内容

| 类别 | 说明 |
|------|------|
| **测试代码** | 按照 Wiki 生成规范，测试相关内容不纳入文档 |
| **第三方依赖** | Node.js 生态依赖（hvigor 等）的内部实现 |
| **IDE 配置** | 开发工具配置、调试技巧等非工程文档 |

## 文档更新方式

本 Wiki 文档基于代码自动生成，更新方式如下：

### 触发更新条件

当以下内容发生变化时，建议更新 Wiki：

1. **新增/删除模块** - `product/phone/` 或 `product/pad/` 目录结构变化
2. **新增系统 API 调用** - 引入新的 `@ohos.*` import 语句
3. **权限配置变更** - `module.json5` 中的 `requestPermissions` 变化
4. **构建配置调整** - `build-profile.json5` 或 `hvigorfile.js` 变更
5. **架构重构** - 组件职责、调用关系变化

### 手动更新步骤

```bash
# 1. 进入仓库根目录
cd /path/to/applications_theme

# 2. 更新本 Wiki 目录下的相关文件
#    修改对应 .md 文件，遵循本 Wiki 的文档规范

# 3. 更新 SUMMARY.md 导航链接（如有新增页面）
```

## 文档生成信息

- **生成时间**: 2026-02-06
- **生成方式**: 基于代码扫描和文档模板自动生成
- **适用版本**: OpenHarmony 3.2+ (SDK 9)
- **文档语言**: 中文（默认）

## 快速导航

- [首页](/wiki/index.md)
- [项目概览](/wiki/01_Project_Overview.md)
- [目录结构](/wiki/02_Directory_Structure.md)
- [架构说明](/wiki/03_Architecture.md)
- [API 文档](/wiki/04_API.md)
- [内部 API](/wiki/05_Inner_API.md)
- [构建配置](/wiki/06_Build.md)
- [编译产物](/wiki/07_Artifacts.md)
- [安全评审](/wiki/08_Security.md)
- [问题定位](/wiki/09_Troubleshooting.md)

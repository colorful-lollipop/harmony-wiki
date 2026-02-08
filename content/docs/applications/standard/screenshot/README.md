# OpenHarmony ScreenShot 应用 Wiki

## 文档覆盖范围

本文档覆盖 OpenHarmony 系统截屏应用（ScreenShot）的完整技术实现，包括：

- **项目定位与架构**: 应用边界、核心能力、运行环境
- **模块职责**: 各模块的职责边界与依赖关系
- **API 分析**: 系统 API 调用链与接口说明
- **构建配置**: hvigor 构建配置与模块依赖
- **安全风险**: 攻击面分析与风险评估

## 文档更新方式

本文档基于代码仓库实时生成。当以下内容发生变化时，建议更新对应文档：

| 变更类型 | 需要更新的文档 |
|---------|--------------|
| 新增/修改 ServiceAbility | `02_Architecture.md`, `03_Modules.md` |
| 新增/修改权限声明 | `04_API.md`, `06_Security.md` |
| 修改构建配置 | `05_Build.md` |
| 新增系统 API 调用 | `04_API.md` |

**更新命令**:
```bash
# 重新扫描并生成文档（需运行 Wiki 生成 Agent）
```

## 文档生成信息

- **生成时间**: 2024-02-06
- **代码版本**: 当前仓库 HEAD
- **覆盖范围**: 不包含测试代码（test/, tests/, *_test.* 等）
- **文档语言**: 中文（除非另有说明）

## 相关资源

- **代码仓库**: `applications/standard/screenshot`
- **OpenHarmony 官方文档**: https://gitee.com/openharmony/docs
- **API 参考**: https://developer.harmonyos.com

## 阅读建议

1. **新人入门**: 按 `SUMMARY.md` 导航顺序阅读
2. **开发者**: 直接查阅 `03_Modules.md` 和 `04_API.md`
3. **安全评审**: 重点阅读 `06_Security.md`
4. **构建调试**: 参考 `05_Build.md` 和 `07_FAQ.md`

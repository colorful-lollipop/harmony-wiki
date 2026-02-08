# OpenHarmony UI Appearance Wiki

> UI Appearance 子系统深色模式服务文档

## 文档覆盖范围

本文档覆盖 OpenHarmony `arkui/ui_appearance` 子系统的完整技术实现，包括：

- **项目概述**: 定位、边界、核心能力
- **架构设计**: 组件图、数据流、线程模型、调用链
- **N-API 接口**: JS API 详细文档、参数校验、错误码
- **Inner API**: 内部模块接口、依赖关系
- **GN 构建**: 编译目标、依赖关系、产物说明
- **安全评审**: 攻击面分析、风险评估

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航
├── 01_Overview.md         # 项目概述
├── 02_Architecture.md     # 架构设计
├── 03_NAPI.md             # N-API 接口
├── 03_CodeMap.md          # 代码地图
├── 04_InnerAPI.md         # 内部 API
├── 05_AttackSurface.md    # 攻击面分析
├── 05_Build.md            # 构建与产物
├── 06_Security.md        # 安全评审
└── figures/               # 架构图资源
```

## 适用范围

本文档适用于以下场景：

1. **二次开发**: 需要理解或扩展 UI Appearance 功能
2. **问题定位**: 需要排查深色模式相关问题
3. **安全审计**: 需要了解安全边界和风险
4. **集成对接**: 需要调用 N-API 接口

**前置知识要求**:
- OpenHarmony 系统架构基础
- N-API 开发模式
- System Ability (SA) 框架
- GN 构建系统

## 关键链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [UIAppearance API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis/js-apis-uiappearance.md)
- [ArkUI 框架子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/ArkUI%E6%A1%86%E6%9E%B6%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [arkui_napi 仓库](https://gitee.com/openharmony/arkui_napi)
- [arkui_ace_engine 仓库](https://gitee.com/openharmony/arkui_ace_engine)

## 代码证据说明

本文档所有关键结论均可在代码仓库中找到直接证据，包括：

- **文件路径**: 精确到 `path:line` 行号
- **符号定义**: 函数、类、宏、enum
- **调用链**: 完整的代码执行路径

无法确认的结论已标注 `TODO(需确认)`。

## 更新说明

### 更新时机

当发生以下变更时，应更新本文档：

1. 新增 N-API 接口
2. 修改现有 API 的参数或返回值
3. 架构重构或模块职责变更
4. 新增或移除依赖组件
5. 安全相关变更

### 更新方式

1. 更新 `wiki/_work/NOTES.md` 记录变更事实
2. 更新对应模块的文档
3. 更新 `SUMMARY.md` 链接
4. 运行一致性检查

### 版本信息

| 项目 | 值 |
|------|-----|
| **组件版本** | 3.2 |
| **系统能力** | SystemCapability.ArkUI.UiAppearance |
| **文档生成时间** | 2026-02-07 |
| **维护者** | arkui-ui_appearance 组件团队 |

## 排除范围

以下内容不在本文档覆盖范围：

- 测试代码 (`test/` 目录)
- 产品化应用定制外观
- 单元测试实现细节
- CI/CD 流程文档

---

*本文档基于 OpenHarmony 源码 `arkui/ui_appearance` 生成*

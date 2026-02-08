# 分布式文件服务（DFS Service）Wiki

> 本 Wiki 由代码自动生成，最后更新于 2026-02-06。

## 文档覆盖范围

本 Wiki 覆盖 OpenHarmony 分布式文件服务（`@ohos/dfs_service`）的核心内容：

| 模块 | 状态 | 说明 |
|------|------|------|
| 项目概览 | ✅ | 定位、能力、约束 |
| 架构设计 | ✅ | 组件、数据流、线程模型 |
| 对外 N-API | ✅ | JS API、NDK API、ANI |
| 内部 Inner API | ✅ | 系统服务间接口 |
| GN 构建配置 | ✅ | targets、依赖、产物 |
| 编译产物 | ✅ | .so、可执行文件、安装路径 |
| 安全风险评审 | ⏳ | 待 Phase 6 完成 |
| 配置说明 | ⏳ | 待补充 |

## 文档更新方式

### 何时需要更新 Wiki

当发生以下变更时，应更新对应文档：

| 变更类型 | 更新文档 | 触发条件 |
|----------|----------|----------|
| 新增 N-API | `02_N-API.md` | 添加 `NAPI_MODULE` 或 `.ndk.json` |
| 新增 SA 服务 | `01_Architecture.md`、`04_Build.md` | 添加 `REGISTER_SYSTEM_ABILITY_BY_ID` |
| 新增 GN target | `04_Build.md`、`05_Artifacts.md` | 修改 `BUILD.gn` |
| 新增权限校验 | `02_N-API.md`、`06_Security.md` | 新增 `CheckCallerPermission` 调用 |
| 新增安全风险 | `06_Security.md` | 发现新的攻击面 |

### 更新步骤

```bash
# 1. 确保在仓库根目录
cd /foundation/filemanagement/dfs_service

# 2. 执行全局扫描（可选，用于验证变更）
# 3. 更新对应 md 文件
# 4. 更新 SUMMARY.md（如果新增页面）
# 5. 提交变更
```

### 质量检查清单

- [ ] API 清单包含文件路径和行号
- [ ] GN target 包含 deps 和 output
- [ ] 安全风险包含代码证据
- [ ] 链接到 `SUMMARY.md` 中的页面均存在
- [ ] 无测试代码引用（`test/`、`*_test.*`）

## 相关链接

- **OpenHarmony 官网**：https://www.openharmony.cn/
- **源码仓库**：https://gitee.com/openharmony/communication_dsoftbus
- **分布式文件系统架构图**：见 `README_zh.md`

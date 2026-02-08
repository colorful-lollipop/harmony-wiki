# Face Authentication Wiki

本 Wiki 是 OpenHarmony 人脸认证模块的工程文档。

## 覆盖范围

| 章节 | 状态 | 说明 |
|------|------|------|
| 01_Overview | ✅ | 项目定位、核心能力 |
| 02_Directory_Structure | ✅ | 目录结构与模块职责 |
| 03_Architecture | ✅ | 架构图、数据流、时序 |
| 04_NAPI | ✅ | JS/ETS API 清单 |
| 05_Inner_API | ✅ | 内部模块接口 |
| 06_GN_Targets | ✅ | GN 构建目标 |
| 07_Build_Artifacts | ✅ | 编译产物 |
| 08_Security_Review | ✅ | 安全风险评审 |
| 09_FAQ | ✅ | 常见问题 |

## 更新方式

当代码变更时，需同步更新以下内容：

1. **N-API 变更**: 更新 `04_NAPI.md`
   - 新增/删除 JS API
   - 修改参数/返回值

2. **架构变更**: 更新 `03_Architecture.md`
   - 新增模块
   - 修改调用链

3. **构建变更**: 更新 `06_GN_Targets.md` 和 `07_Build_Artifacts.md`
   - 新增/删除 targets
   - 修改依赖关系

4. **安全变更**: 更新 `08_Security_Review.md`
   - 新增风险点
   - 更新修复建议

## 生成信息

- **生成时间**: 2024-02-06
- **代码版本**: face_auth v4.0
- **文档版本**: 1.0

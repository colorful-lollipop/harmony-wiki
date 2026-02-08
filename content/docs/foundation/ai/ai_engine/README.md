# AI Engine Wiki

## 文档覆盖范围

本 Wiki 文档覆盖 OpenHarmony AI Engine 子系统的完整工程信息，包括：

### 覆盖内容
- **项目定位与核心能力**：AI 子系统的设计目标、适用范围
- **目录结构与模块职责**：各模块的功能边界和依赖关系
- **架构说明**：组件图、数据流、线程模型、关键时序
- **对外 API**：SDK 接口定义、参数说明、错误码
- **内部 API**：模块接口、依赖方向、稳定性标注
- **GN 构建系统**：targets 列表、依赖、产物
- **编译产物**：so/a/可执行文件、安装路径、加载关系
- **安全风险评审**：攻击面、信任边界、风险点与修复建议
- **常见问题**：构建、运行、调试问题的定位路径

### 未覆盖内容
- **测试代码**：单元测试、集成测试代码不作为业务证据
- **第三方依赖**：仅记录必要的外部依赖信息
- **历史变更记录**：请参考 git log

## 更新方式

本 Wiki 随代码变更同步更新。建议在以下场景更新文档：

1. **新增模块**：添加模块职责描述和接口文档
2. **接口变更**：更新 API 清单表和调用链
3. **构建调整**：更新 target 列表和产物映射
4. **安全问题修复**：更新安全评审章节

**更新步骤**：
1. 在 `_work/NOTES.md` 中记录发现
2. 更新对应章节的 `.md` 文件
3. 验证 `SUMMARY.md` 链接正确性
4. 提交变更

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于 OpenHarmony AI Engine v3.1
- **文档语言**: 中文 (默认)
- **负责团队**: OpenHarmony AI 子系统

## 相关链接

- [OpenHarmony AI 子系统](https://gitee.com/openharmony/docs/blob/master/en/readme/ai.md)
- [AI Engine Framework 开发指南](https://gitee.com/openharmony/docs/blob/master/en/device-dev/subsystems/subsys-ai-aiframework-devguide.md)
- [build_lite 构建系统](https://gitee.com/openharmony/build_lite/blob/master/README.md)
- [SAMGR Lite](https://gitee.com/openharmony/distributedschedule_samgr_lite/blob/master/README.md)

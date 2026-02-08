# OpenHarmony Intelligent Voice Framework - 工程 Wiki

> **版本**: 4.0  
> **生成时间**: 2026-02-06  
> **最后更新**: 2026-02-06  
> **维护者**: AI Engineering Team

---

## 文档覆盖范围

本文档为 OpenHarmony Intelligent Voice Framework（智能语音框架）的完整工程 Wiki，旨在帮助开发者快速理解项目架构、API 使用、构建系统和安全机制。

### 覆盖内容

- **项目概览**：定位、边界、核心能力、基本概念
- **目录结构**：模块职责、代码组织方式
- **架构设计**：组件图、数据流、线程模型、关键时序
- **对外 API**：N-API 接口清单、参数校验、错误码
- **内部架构**：Inner API、模块依赖、生命周期
- **构建系统**：GN Targets、编译产物、安装路径
- **安全评审**：攻击面、信任边界、风险点、修复建议
- **常见问题**：构建/运行/调试问题定位

### 未覆盖内容

- 测试代码细节（`tests/`, `llt/`, `*_test.*`）
- 具体算法实现细节
- 第三方依赖（HDI 驱动层）内部实现
- 特定设备硬件相关配置

### 阅读建议

**新人入门路线**（推荐顺序）：
1. `01_Overview.md` - 项目概览
2. `03_Directory_Structure.md` - 目录结构
3. `04_NAPI_Reference.md` - N-API 使用
4. `02_Architecture.md` - 架构设计

**开发者参考**（按需查阅）：
- `06_Build_System.md` - 构建配置
- `07_Artifacts.md` - 产物说明
- `08_Security_Review.md` - 安全机制
- `appendix/Callgraphs.md` - 调用链

---

## 项目基本信息

| 属性 | 值 |
|------|-----|
| **仓库路径** | `foundation/ai/intelligent_voice_framework` |
| **子系统** | `ai` |
| **系统能力** | `SystemCapability.AI.IntelligentVoice.Core` |
| **许可证** | Apache License 2.0 |
| **ROM 占用** | ~675KB |
| **RAM 占用** | ~7680KB |
| **SA ID** | 312 |
| **进程名** | `intell_voice_service` |

---

## 核心能力

Intelligent Voice Framework 提供以下核心能力：

1. **语音注册（Enrollment）**：将用户的唤醒词转换为声学模型和声纹特征
2. **语音唤醒（Wakeup）**：检测当前说话者是否为注册用户，系统唤醒
3. **声音触发（Sound Trigger）**：DSP 模型加载/卸载、DSP 算法控制
4. **系统事件感知**：监听开机解锁、屏幕开关等系统事件
5. **并发策略管理**：智能语音服务的并发管理

---

## 与本 Wiki 相关的文档链接

- [项目概览](./01_Overview.md)
- [架构设计](./02_Architecture.md)
- [目录结构](./03_Directory_Structure.md)
- [N-API 参考](./04_NAPI_Reference.md)
- [Inner API 参考](./05_Inner_API.md)
- [构建系统](./06_Build_System.md)
- [编译产物](./07_Artifacts.md)
- [安全评审](./08_Security_Review.md)
- [关键调用链](./appendix/Callgraphs.md)
- [配置开关](./appendix/Config_Flags.md)

---

## 如何更新本文档

本文档基于代码自动生成。如需更新：

1. **代码变更后**：运行文档生成脚本（如有）
2. **新增 API**：更新 `interfaces/kits/js/@ohos.ai.intelligentVoice.d.ts` 后重新生成
3. **新增模块**：更新对应 `BUILD.gn` 和模块文档
4. **安全变更**：更新 `08_Security_Review.md` 的风险清单

### 文档维护原则

- 所有关键结论必须可追溯到代码证据（路径+符号）
- N-API 变更必须同步更新 API 清单表
- 安全评审需要包含最新的攻击面分析
- 忽略测试相关内容作为业务证据

---

## 反馈与贡献

如发现文档错误或遗漏，请：

1. 检查 `wiki/_work/NOTES.md` 中的待办项
2. 提交 Issue 描述问题
3. 或直接发起 PR 修改文档

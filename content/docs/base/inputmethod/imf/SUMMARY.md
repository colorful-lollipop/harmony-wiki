# 文档导航

本文档提供 OpenHarmony 输入法框架的完整指南，根据您的角色选择阅读路线。

---

## 🔰 新人学习路线

适合快速理解项目、上手开发的开发者。

| 步骤 | 文档 | 目标 | 预计时间 |
|------|------|------|----------|
| 1 | [00_Overview.md](./00_Overview.md) | 理解 IMF 是什么、能做什么 | 5 分钟 |
| 2 | [01_Architecture.md](./01_Architecture.md) | 理解四大模块如何协作 | 15 分钟 |
| 3 | [02_N-API.md](./02_N-API.md) | 学会调用 JS API | 20 分钟 |
| 4 | [03_Inner_API.md](./03_Inner_API.md) | 了解内部 C++ 接口 | 15 分钟 |
| 5 | [04_Build.md](./04_Build.md) | 了解构建系统和产物 | 10 分钟 |

**路线目标**：
- ✅ 能在 5 分钟内理解项目定位和用途
- ✅ 能在 15 分钟内找到核心代码位置
- ✅ 能在 30 分钟内理解基本架构

---

## 🔒 安全研究路线

适合进行安全审计、漏洞分析的研究员。

| 步骤 | 文档 | 目标 | 重点内容 |
|------|------|------|----------|
| 1 | [00_Overview.md](./00_Overview.md) | 了解项目类型和暴露面 | SystemAbility 模式、N-API 入口 |
| 2 | [01_Architecture.md](./01_Architecture.md) | 理解信任边界 | 应用 → IMSA → IME 数据流 |
| 3 | [05_Security.md](./05_Security.md) | 全面安全分析 | 攻击面、风险点、利用路径 |
| 4 | [02_N-API.md](./02_N-API.md) | 审查 JS 接口 | 参数校验、权限检查 |
| 5 | [04_Build.md](./04_Build.md) | 了解产物和部署 | 文件权限、安装路径 |

**路线目标**：
- ✅ 快速识别所有外部输入入口
- ✅ 定位敏感操作和权限检查点
- ✅ 每个风险都有可利用性评估和修复建议

---

## 📚 完整文档索引

## 完整文档列表

### 核心文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](./README.md) | Wiki 说明与更新方式 | 必读 |
| [00_Overview.md](./00_Overview.md) | 项目定位、核心能力、运行环境 | 高 |
| [01_Architecture.md](./01_Architecture.md) | 组件图、数据流、线程模型 | 高 |
| [02_N-API.md](./02_N-API.md) | N-API 接口清单与使用 | 高 |
| [03_Inner_API.md](./03_Inner_API.md) | Inner API 接口与依赖 | 中 |
| [04_Build.md](./04_Build.md) | GN Targets、编译产物、安装路径 | 高 |
| [05_Security.md](./05_Security.md) | 攻击面、信任边界、风险点 | 高 |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链 |

## API 快速索引

### JS API (N-API)

- `inputMethod` - 输入法管理
- `InputMethodController` - 输入控制器
- `inputMethodList` - 输入法列表
- `panel` - 输入法面板
- `inputMethodEngine` - 输入法引擎
- `keyboardPanelManager` - 键盘面板管理器

### Inner API

- `InputMethodController` - 控制器接口
- `InputMethodAbility` - 能力接口
- `ImfHook` - Hook 接口

### NDK C API

- `native_inputmethod_types.h` - 类型定义
- `native_text_changed_listener.h` - 文本监听
- `native_message_handler_callback.h` - 消息处理

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme.md)
- [inputmethod_imf 仓库](gitee:///base/inputmethod/imf)

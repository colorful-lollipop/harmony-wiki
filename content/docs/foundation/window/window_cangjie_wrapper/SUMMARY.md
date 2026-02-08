# window_cangjie_wrapper Wiki 文档导航

> 新人阅读顺序与文档索引

---

## 📖 新人阅读路径

### 第一阶段：快速入门（30 分钟）
1. 📋 [README.md](./README.md) - 文档概览与更新方式
2. 📖 [00_Overview.md](./00_Overview.md) - 项目定位、边界、核心能力
3. 📁 [01_Project_Positioning.md](./01_Project_Positioning.md) - 运行环境、关键概念

### 第二阶段：架构理解（60 分钟）
4. 📊 [02_Directory_Structure.md](./02_Directory_Structure.md) - 目录结构与模块职责
5. 🏗️ [03_Architecture.md](./03_Architecture.md) - 组件图、数据流、线程模型

### 第三阶段：API 掌握（90 分钟）
6. 🎯 [04_External_API.md](./04_External_API.md) - **对外仓颉 API（重点）**
7. 🔄 [05_Internal_API.md](./05_Internal_API.md) - 内部 API、模块接口、稳定性

### 第四阶段：构建与部署（45 分钟）
8. 🔧 [06_GN_Targets.md](./06_GN_Targets.md) - GN targets、依赖关系
9. 📦 [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 编译产物、安装路径

### 第五阶段：安全与维护（45 分钟）
10. 🔒 [08_Security_Review.md](./08_Security_Review.md) - **安全风险评审（重点）**
11. ❓ [09_FAQ.md](./09_FAQ.md) - 常见构建/运行/调试问题

### 附录（可选）
12. 📜 [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 关键调用链
13. ⚙️ [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 关键宏/feature flags

---

## 📚 文档索引

| 序号 | 文档 | 页数 | 阅读时间 | 关键词 |
|------|--------|--------|----------|--------|
| 1 | [README.md](./README.md) | 1 | 文档概览、更新方式 |
| 2 | [00_Overview.md](./00_Overview.md) | 5 | 项目定位、边界、核心能力 |
| 3 | [01_Project_Positioning.md](./01_Project_Positioning.md) | 3 | 运行环境、关键概念 |
| 4 | [02_Directory_Structure.md](./02_Directory_Structure.md) | 4 | 目录结构、模块职责 |
| 5 | [03_Architecture.md](./03_Architecture.md) | 8 | 组件图、数据流、线程模型 |
| 6 | [04_External_API.md](./04_External_API.md) | 12 | **仓颉 API 清单** |
| 7 | [05_Internal_API.md](./05_Internal_API.md) | 5 | 内部 API、依赖方向 |
| 8 | [06_GN_Targets.md](./06_GN_Targets.md) | 4 | GN targets、依赖 |
| 9 | [07_Build_Artifacts.md](./07_Build_Artifacts.md) | 3 | 编译产物、安装路径 |
| 10 | [08_Security_Review.md](./08_Security_Review.md) | 6 | **安全风险评审** |
| 11 | [09_FAQ.md](./09_FAQ.md) | 4 | 常见问题、定位路径 |
| 12 | [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 3 | 关键调用链（可选） |
| 13 | [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 2 | 关键宏（可选） |

---

## 🔍 主题索引

### 按主题快速查找

#### 窗口管理
- 窗口创建与查找：[04_External_API.md](./04_External_API.md#窗口创建与查找)
- 窗口属性设置：[04_External_API.md](./04_External_API.md#窗口属性设置)
- 窗口生命周期：[04_External_API.md](./04_External_API.md#窗口生命周期)
- WindowStage 管理：[04_External_API.md](./04_External_API.md#windowstage-管理)
- 回调机制：[04_External_API.md](./04_External_API.md#回调机制)

#### 显示设备管理
- Display 查询：[04_External_API.md](./04_External_API.md#display-查询)
- 折叠屏管理：[04_External_API.md](./04_External_API.md#折叠屏管理)
- 回调监听：[04_External_API.md](./04_External_API.md#display-回调)

#### 架构与构建
- 模块依赖：[03_Architecture.md](./03_Architecture.md#模块依赖关系)
- FFI 接口：[03_Architecture.md](./03_Architecture.md#ffi-foreign-function-interface)
- GN 配置：[06_GN_Targets.md](./06_GN_Targets.md)

#### 安全
- 输入校验：[08_Security_Review.md](./08_Security_Review.md#输入校验)
- 权限检查：[08_Security_Review.md](./08_Security_Review.md#权限检查)
- 内存安全：[08_Security_Review.md](./08_Security_Review.md#内存安全)

---

## 📊 文档统计

| 类别 | 数量 |
|--------|--------|
| 主文档 | 11 篇 |
| 附录文档 | 2 篇 |
| API 清单条目 | ~150 个 |
| 枚举类型 | 15 个 |
| 错误码 | 14 个 |

---

## 🎯 学习建议

### 对于开发者
1. **重点掌握** `ohos.window` 和 `ohos.display` 模块的公共 API
2. **理解** FFI 调用机制和错误处理模式
3. **注意** 窗口生命周期管理，避免资源泄露

### 对于代码贡献者
1. **遵循** 现有代码风格和注释规范
2. **使用** `@!APILevel` 注解标注 API 变更
3. **更新** Wiki 文档以反映新功能

---

## ⚠️ 重要说明

- 本文档基于代码仓库 **2025-02-06** 的状态生成
- 代码持续演进，文档可能滞后，请以最新代码为准
- 测试用例（test/ 目录）不在本文档覆盖范围内
- Native 层实现（window_manager 子系统）不在此仓库中

---

**最后更新**: 2025-02-06

# 文档导航

本文档为 OpenHarmony font_manager 子系统的完整 Wiki，涵盖架构设计、API 参考、构建配置和安全评审。

## 快速开始

- 📖 [项目概述](00_Overview.md) - 了解 font_manager 的定位和核心能力
- 🚀 [API 参考](02_NAPI_Reference.md) - 查看 JS API 使用方法

## 目录

### 核心文档

| 章节 | 标题 | 说明 | 受众 |
|------|------|------|------|
| [00](00_Overview.md) | 项目概览 | 功能定位、目录结构、系统能力、运行环境 | 👶 新人 |
| [01](01_Architecture.md) | 架构说明 | 组件图、数据流、线程模型、关键时序 | 👶 新人 |
| [02](02_NAPI_Reference.md) | N-API 参考 | 导出 JS API 清单、参数校验、错误码 | 👶 新人 |
| [03](03_Inner_API.md) | 内部 API | 核心模块接口、依赖方向、生命周期 | 👶 新人 |
| [04](04_Build_Targets.md) | 构建目标 | GN targets 列表、依赖关系、编译产物 | 👷 开发者 |
| [05](05_Artifacts.md) | 编译产物 | .so/.a 文件、安装路径、加载关系 | 👷 开发者 |
| [06](06_Security_Review.md) | 安全评审 | 攻击面、信任边界、风险点与修复建议 | 🛡️ 安全研究员 |
| [07](07_Troubleshooting.md) | 故障排查 | 常见问题、定位路径、调试方法 | 👷 开发者 |

### 附录

| 附录 | 标题 | 说明 | 状态 |
|------|------|------|------|
| [A](appendix/Callgraphs.md) | 关键调用链 | 入口→核心逻辑的完整调用链 | 🚧 待补充 |

---

## 📚 阅读路线图

### 🎯 场景 1：新人学习路线

**目标**：快速理解项目、掌握 API 使用、熟悉架构设计

```
第 1 步：项目认知（15 分钟）
└─ 00_Overview.md
   ├─ 项目定位：什么是 font_manager？
   ├─ 能力边界：能做什么/不能做什么
   └─ 运行环境：SystemAbility、权限要求

第 2 步：架构理解（30 分钟）
└─ 01_Architecture.md
   ├─ 分层架构：ArkTS → N-API → Client → Server → Core
   ├─ 数据流：安装/卸载完整流程
   └─ 时序图：跨组件调用链

第 3 步：API 使用（30 分钟）
└─ 02_NAPI_Reference.md
   ├─ installFont：安装字体
   ├─ uninstallFont：卸载字体
   ├─ dataMigration：数据迁移
   └─ 错误码：14个错误码详解

第 4 步：内部实现（30 分钟）
└─ 03_Inner_API.md
   ├─ 模块职责：Client/Server/Core 各做什么
   ├─ 接口契约：稳定接口 vs 内部实现
   └─ 生命周期：资源创建和释放

第 5 步：构建与部署（15 分钟）
└─ 04_Build_Targets.md + 05_Artifacts.md
   ├─ GN targets：如何编译
   ├─ 编译产物：生成哪些 .so 文件
   └─ 安装路径：文件部署到哪
```

**时间预估**：2 小时

---

### 🛡️ 场景 2：安全研究路线

**目标**：识别攻击面、分析信任边界、定位安全风险

```
第 1 步：攻击面分析（30 分钟）
└─ 06_Security_Review.md - 攻击面分析章节
   ├─ 外部输入清单：
   │  ├─ N-API 参数：fontPath, fontName
   │  ├─ IPC 数据：FileDescriptor
   │  └─ 文件输入：字体文件本身
   └─ 敏感操作清单：
      ├─ 文件写入：/data/service/el1/{userId}/for-all-app/fonts/
      ├─ 配置修改：install_fontconfig.json
      └─ 权限校验：ohos.permission.UPDATE_FONT

第 2 步：信任边界映射（30 分钟）
└─ 06_Security_Review.md - 信任边界章节
   ├─ 信任起点：沙箱内应用
   ├─ 跨越点：
   │  ├─ N-API 层：参数校验和类型转换
   │  ├─ Client 层：路径规范化和 fd 验证
   │  ├─ Server 层：权限校验和业务逻辑
   │  └─ Core 层：文件操作和配置管理
   └─ 信任终点：字体文件存储目录

第 3 步：安全风险审计（60 分钟）
└─ 06_Security_Review.md - 安全风险清单
   ├─ 已缓解风险（5个）：
   │  ├─ R1: 权限校验 ✅
   │  ├─ R2: 路径遍历防护 ✅
   │  ├─ R3: 文件描述符验证 ✅
   │  ├─ R4: 文件数量限制 ✅
   │  └─ R5: 字体文件格式验证 ⚠️
   └─ 潜在风险（5个）：
      ├─ R6: TOCTOU 竞态条件（中危）
      ├─ R7: 字体配置文件注入（低危）
      ├─ R8: SA 空闲自动卸载（低危）
      ├─ R9: 临时文件处理（低危）
      └─ R10: 内存安全（已缓解）

第 4 步：漏洞利用路径分析（60 分钟）
└─ 深入代码分析
   ├─ 跟踪输入处理链：
   │  N-API → Client → Server → Core → 文件系统
   ├─ 定位校验薄弱点：
   │  ├─ access() 和 fopen() 之间的时间窗口
   │  ├─ realpath() 的符号链接处理
   │  └─ JSON 解析的注入风险
   └─ 分析修复建议可行性：
      ├─ 优先级排序
      ├─ 代码改动量评估
      └─ 向后兼容性影响

第 5 步：PoC 开发（120 分钟）
└─ 编写漏洞验证代码
   ├─ TOCTOU 竞态条件 PoC
   ├─ 路径遍历测试用例
   └─ 权限绕过验证
```

**时间预估**：5 小时（不含 PoC 开发）

---

### 🐛 场景 3：Bug 修复路线

**目标**：定位问题根因、理解代码逻辑、安全修复

```
第 1 步：问题定位（15 分钟）
└─ 07_Troubleshooting.md
   ├─ 根据错误码查找常见问题
   ├─ 定位日志关键字
   └─ 确定可疑模块

第 2 步：调用链追踪（30 分钟）
└─ 01_Architecture.md + 03_Inner_API.md
   ├─ 从入口点开始追踪：
   │  └─ JS API → N-API → Client → Server → Core
   ├─ 记录关键函数和返回值
   └─ 画出数据流转图

第 3 步：代码审计（30 分钟）
└─ 直接阅读源代码
   ├─ 读取关键实现文件
   ├─ 检查错误处理路径
   └─ 对比安全评审中的风险点

第 4 步：修复验证（30 分钟）
└─ 02_NAPI_Reference.md + 06_Security_Review.md
   ├─ 遵循 API 规范
   ├─ 检查安全风险
   └─ 单元测试验证
```

**时间预估**：2 小时

---

### 🏗️ 场景 4：功能开发路线

**目标**：理解现有架构、遵循设计模式、实现新功能

```
第 1 步：架构理解（45 分钟）
└─ 01_Architecture.md + 03_Inner_API.md
   ├─ 分层设计：每层的职责和约束
   ├─ 接口契约：稳定接口 vs 内部实现
   └─ 依赖关系：模块间如何交互

第 2 步：API 设计（30 分钟）
└─ 02_NAPI_Reference.md
   ├─ 参考现有 API 设计
   ├─ 遵循命名规范
   ├─ 定义错误码
   └─ 编写使用示例

第 3 步：实现编码（90 分钟）
└─ 直接实现
   ├─ N-API 层：实现 JS 绑定
   ├─ Client 层：实现 IPC 调用
   ├─ Server 层：实现权限校验和业务逻辑
   └─ Core 层：实现核心功能

第 4 步：安全审查（60 分钟）
└─ 06_Security_Review.md
   ├─ 检查输入验证
   ├─ 检查权限校验
   ├─ 检查敏感操作
   └─ 检查潜在风险

第 5 步：构建测试（30 分钟）
└─ 04_Build_Targets.md + 05_Artifacts.md
   ├─ 更新 BUILD.gn
   ├─ 编译验证
   └─ 单元测试
```

**时间预估**：4.5 小时

---

## 🔗 相关链接

### 官方文档

| 链接 | 说明 |
|------|------|
| [OpenHarmony 官方文档](https://gitee.com/openharmony/docs) | 系统级文档和开发指南 |
| [font_manager 仓库](https://gitee.com/openharmony/global_font_manager) | 源代码仓库 |
| [SystemAbility 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/standard/systemability/sysa-development.md) | SA 开发最佳实践 |
| [N-API 开发指南](https://gitee.com/openharmony/napi) | N-API 接口规范 |
| [IPC 通信指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/standard/ipc/ipc-overview.md) | Binder IPC 开发 |

### 系统能力

| 系统能力 | 说明 |
|---------|------|
| `SystemCapability.Global.FontManager` | 字体管理系统能力 |
| `ohos.permission.UPDATE_FONT` | 更新字体权限 |

---

## 📝 文档更新日志

| 日期 | 版本 | 更新内容 |
|------|------|---------|
| 2025-02-07 | 2.0 | 完善双路线导航，新增场景化阅读路线图 |
| 2025-02-06 | 1.0 | 初始版本，完成核心文档 |

---

## 💡 使用建议

### 新手开发者

1. **顺序阅读**：按照"新人学习路线"逐步阅读
2. **动手实践**：边读边写代码，加深理解
3. **提问反馈**：遇到问题查看 FAQ 或提交 Issue

### 安全研究员

1. **快速定位**：直接阅读"安全研究路线"相关章节
2. **代码审计**：使用代码编辑器同步查看源文件
3. **工具辅助**：使用静态分析工具和 Fuzz 测试

### 项目维护者

1. **保持同步**：代码变更时更新对应文档
2. **证据更新**：确保代码路径和行号准确
3. **版本标注**：重要变更在文档中标注版本号

---

**文档维护**：Font Manager 团队
**最后更新**：2025-02-07

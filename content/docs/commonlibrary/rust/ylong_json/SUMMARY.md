# SUMMARY

ylong_json 工程 Wiki 导航

## 📚 推荐阅读路径

### 新人学习路线（快速上手，30 分钟理解）

适合：系统服务层开发者、需要使用 JSON 解析能力的 C/Rust 开发者

**第一步（5 分钟）**：了解项目定位和核心能力
→ [项目概览](00_Overview.md)
  - 一句话定义：高性能 JSON 解析和序列化库
  - 核心能力：JSON 解析/生成、多底层数据结构、三种接口形式
  - 性能对比：优于 cJSON（测试数据）
  - 适用场景和边界

**第二步（10 分钟）**：理解整体架构和设计
→ [架构说明](01_Architecture.md)
  - 组件架构图
  - 数据流图（解析和序列化流程）
  - 模块依赖关系
  - 状态机解析流程

**第三步（10 分钟）**：掌握对外接口
→ [对外 API (C FFI)](03_Public_API.md) - 如果使用 C 接口
→ [内部 API (Rust)](04_Internal_API.md) - 如果使用 Rust 接口
  - 接口清单表
  - 参数和返回值
  - 内存管理规则
  - 错误处理

**第四步（5 分钟）**：了解构建和集成
→ [GN Targets](05_GN_Targets.md)
→ [编译产物](06_Build_Artifacts.md)
  - 如何集成到 OpenHarmony 项目
  - Feature flags 选择

---

### 安全研究路线（深度审计，识别攻击面）

适合：安全审计人员、安全架构师

**第一步（10 分钟）**：识别攻击面
→ [安全风险分析](07_Security_Analysis.md) - 从"攻击面清单"开始
  - 外部输入点：C FFI 接口、JSON 字符串
  - 敏感操作：内存分配/释放、文件访问
  - 信任边界图

**第二步（30 分钟）**：深度风险分析
→ [安全风险分析](07_Security_Analysis.md) - 阅读"可被利用点"
  - 空指针解引用风险
  - 内存泄漏风险
  - use-after-free 风险
  - 整数溢出风险
  - 递归深度攻击
  - UTF-8 处理不安全
  - 拒绝服务（大输入）
  - 类型混淆风险

**第三步（10 分钟）**：修复建议和总体评估
→ [安全风险分析](07_Security_Analysis.md) - 阅读"修复建议汇总"和"总体评估"
  - 高优先级修复项
  - 中优先级修复项
  - 安全等级评估

**第四步（深入源代码）**：验证风险
→ [对外 API (C FFI)](03_Public_API.md) - 查看接口定义
→ [目录结构](02_Directory_Structure.md) - 定位关键文件
→ [架构说明](01_Architecture.md) - 理解实现逻辑
→ 源代码审查（`src/adapter.rs`、`src/states.rs` 等）

---

## 📖 完整文档目录

### 快速开始

- [Wiki 说明](README.md)
- [项目概览](00_Overview.md)

### 核心文档

- [架构说明](01_Architecture.md)
  - 组件架构图
  - 数据流图
  - 模块依赖关系
- [目录结构](02_Directory_Structure.md)
  - 顶层目录
  - src/ 目录详解
  - 模块职责

### API 文档

- [对外 API (C FFI)](03_Public_API.md)
  - 接口清单表
  - 参数校验规则
  - 错误码说明
  - 内存管理
- [内部 API (Rust)](04_Internal_API.md)
  - JsonValue API
  - Array/Object API
  - Serde 集成

### 构建与产物

- [GN Targets](05_GN_Targets.md)
  - 构建目标列表
  - Feature flags
- [编译产物](06_Build_Artifacts.md)
  - 输出文件
  - 安装路径

### 安全与问题排查

- [安全风险分析](07_Security_Analysis.md)
  - 攻击面清单
  - 可被利用点
  - 修复建议
- [常见问题](08_Troubleshooting.md)
  - 构建问题
  - 运行时问题

### 附录

- [关键调用链](appendix/Callgraphs.md)
- [配置选项](appendix/Config_Flags.md)

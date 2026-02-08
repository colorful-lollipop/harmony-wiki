# Wiki 目录

## 快速导航

- [Wiki 首页](README.md) - 使用指南与术语表
- [项目概述](00_Overview.md) - 定位、能力、运行环境
- [架构说明](01_Architecture.md) - 组件图、数据流、关键时序

## 核心文档

### 代码结构
- [目录结构与模块职责](02_Directory_Structure.md) - 不含测试

### API 参考
- [对外 API - N-API 层](03_NAPI_Reference.md)
  - MessageSequence API
  - Ashmem API
  - Parcelable 接口
  - 错误码与异常

### 实现细节
- [内部 API 与架构](04_Internal_API.md)
  - 模块职责与边界
  - 类关系图
  - FFI 层接口

### 构建系统
- [GN Targets 与编译产物](05_GN_Targets.md)
  - 构建目标列表
  - 依赖关系
  - 产物映射

### 安全评估
- [安全风险评审](06_Security_Analysis.md)
  - 攻击面分析
  - 信任边界
  - 可被利用点
  - 修复建议

### 运维指南
- [编译产物与运行时](07_Build_Artifacts.md)
- [常见问题与调试](08_Troubleshooting.md)

## 附录

- [错误码完整列表](appendix/Error_Codes.md)

## 双路线导航

### 路线一：新人学习路线（5天入门）

**第一天：了解项目（5分钟）**
1. [Wiki 首页](README.md) - 了解文档结构
2. [项目概述](00_Overview.md) - 理解项目定位
3. [目录结构](02_Directory_Structure.md) - 熟悉代码组织

**第二天：掌握 API（30分钟）**
1. [对外 API](03_NAPI_Reference.md) - 学习核心 API
2. 参考 [错误码列表](appendix/Error_Codes.md) - 了解异常处理

**第三天：理解实现（1小时）**
1. [架构说明](01_Architecture.md) - 理解整体设计
2. [内部 API](04_Internal_API.md) - 了解实现细节

**第四天：构建与安全（30分钟）**
1. [GN Targets](05_GN_Targets.md) - 了解构建系统
2. [安全风险评审](06_Security_Analysis.md) - 了解安全考量

**第五天：实践（按需）**
1. [编译产物](07_Build_Artifacts.md)
2. [常见问题](08_Troubleshooting.md)

---

### 路线二：安全研究路线（快速上手）

**阶段一：攻击面识别（15分钟）**
1. [安全风险评审](06_Security_Analysis.md)
   - 阅读「攻击面清单」
   - 阅读「信任边界」
2. [项目概述](00_Overview.md)
   - 了解「对外暴露面」

**阶段二：输入入口分析（30分钟）**
1. [对外 API](03_NAPI_Reference.md)
   - MessageSequence 写入方法（外部输入点）
   - Ashmem 创建/操作方法
   - 文件描述符传递方法
2. [内部 API](04_Internal_API.md)
   - FFI 层接口调用链

**阶段三：漏洞深度分析（1小时）**
1. [安全风险评审](06_Security_Analysis.md)
   - 7个可被利用点详细分析
   - 代码路径 + 行号 + 触发路径
2. 源码验证
   - `ohos/rpc/ashmem.cj:89-101` (R1: size未校验)
   - `ohos/rpc/ashmem.cj:233-244` (R2: 边界缺失)
   - `ohos/rpc/message_sequence.cj:1062-1083` (R4: FD无校验)

**阶段四：修复建议验证（30分钟）**
1. [安全风险评审](06_Security_Analysis.md)
   - 阅读「安全建议汇总」
2. 评估修复优先级
   - 高优先级：Ashmem参数校验
   - 中优先级：FD验证、边界检查
   - 低优先级：类型安全、长度限制

---

## 快速参考

### 新人常用
| 需求 | 推荐文档 |
|------|----------|
| 了解项目是什么 | [00_Overview.md](00_Overview.md) |
| 查找 API 用法 | [03_NAPI_Reference.md](03_NAPI_Reference.md) |
| 查看错误码含义 | [appendix/Error_Codes.md](appendix/Error_Codes.md) |
| 理解代码结构 | [02_Directory_Structure.md](02_Directory_Structure.md) |
| 解决构建问题 | [08_Troubleshooting.md](08_Troubleshooting.md) |

### 安全研究常用
| 需求 | 推荐文档 |
|------|----------|
| 识别攻击面 | [06_Security_Analysis.md](06_Security_Analysis.md) - 攻击面清单 |
| 查找漏洞点 | [06_Security_Analysis.md](06_Security_Analysis.md) - 可被利用点分析 |
| 验证代码证据 | [04_Internal_API.md](04_Internal_API.md) + 源码 |
| 了解修复建议 | [06_Security_Analysis.md](06_Security_Analysis.md) - 安全建议汇总 |
| 查看代码地图 | [02_Directory_Structure.md](02_Directory_Structure.md) |

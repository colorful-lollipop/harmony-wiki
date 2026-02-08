# 文档导航

本文档为 OpenHarmony `window_manager_lite` 的完整技术文档，面向两类读者：**新人学习者**和**安全研究员**。

---

## 新人学习路线

**目标**：快速理解项目是什么、能做什么、怎么用、架构如何组织

### 阶段一：入门（5分钟）
1. [项目概述](./01_Overview.md) - 了解项目定位、核心能力、运行环境

### 阶段二：结构（15分钟）
2. [目录结构](./02_Directory_Structure.md) - 掌握模块划分和文件组织
3. [架构设计](./03_Architecture.md) - C/S 架构、组件图、数据流、线程模型

### 阶段三：接口（30分钟）
4. [Inner API](./04_Inner_API.md) - 模块间接口详解和使用方式
5. [IMS 输入管理](./06_IMS.md) - 输入事件处理流程

### 阶段四：构建
6. [GN 构建配置](./05_GN_Build.md) - 编译目标、产物、依赖关系

### 阶段五：深入
7. [安全风险评审](./07_Security.md) - 了解安全注意事项

---

## 安全研究路线

**目标**：识别攻击面、信任边界、潜在漏洞点和利用路径

### 阶段一：概览
1. [项目概述](./01_Overview.md) - 快速了解项目定位和对外暴露面
2. [架构设计](./03_Architecture.md) - 理解信任边界和 C/S 通信机制

### 阶段二：攻击面
3. [攻击面分析](./05_AttackSurface.md) - 详细的外部输入入口和敏感操作清单
   - IPC 接口清单（14个调用点）
   - 输入数据来源
   - 敏感操作位置

### 阶段三：安全评估
4. [安全风险评审](./07_Security.md) - 风险评估、可被利用点、修复建议
   - 高风险：Screenshot 权限、窗口创建权限
   - 中风险：参数边界、ID 验证
   - 内存安全分析

### 附录
5. [目录结构](./02_Directory_Structure.md) - 代码定位参考
6. [Inner API](./04_Inner_API.md) - 接口实现细节

---

## 快速索引

### 按主题分类

| 主题 | 相关章节 |
|------|----------|
| **项目入门** | [01_Overview](./01_Overview.md) |
| **代码地图** | [02_Directory_Structure](./02_Directory_Structure.md) |
| **架构理解** | [03_Architecture](./03_Architecture.md) |
| **API 使用** | [04_Inner_API](./04_Inner_API.md) |
| **构建配置** | [05_GN_Build](./05_GN_Build.md) |
| **输入系统** | [06_IMS](./06_IMS.md) |
| **攻击面分析** | [05_AttackSurface](./05_AttackSurface.md) |
| **安全评估** | [07_Security](./07_Security.md) |

### 按代码位置分类

| 代码位置 | 说明 | 相关文档 |
|----------|------|----------|
| `services/wms/` | WMS 服务端实现 | [03_Architecture](./03_Architecture.md)、[07_Security](./07_Security.md) |
| `services/ims/` | IMS 服务端实现 | [06_IMS](./06_IMS.md) |
| `frameworks/wms/` | WMS 客户端实现 | [04_Inner_API](./04_Inner_API.md) |
| `frameworks/ims/` | IMS 客户端实现 | [06_IMS](./06_IMS.md) |
| `interfaces/innerkits/` | 模块间接口定义 | [04_Inner_API](./04_Inner_API.md) |
| `BUILD.gn` | 构建配置 | [05_GN_Build](./05_GN_Build.md) |

### 关键代码证据速查

| 功能 | 文件路径 | 行号 |
|------|----------|------|
| IPC 请求处理 | `services/wms/lite_wms.cpp` | 31-80 |
| 权限检查 | `services/wms/lite_wms.cpp` | 216-228 |
| 窗口创建 | `services/wms/lite_wms.cpp` | 186-199 |
| 服务注册 | `services/wms/samgr_wms.cpp` | 85-91 |
| 输入分发 | `services/ims/input_event_distributer.h` | 26-75 |

---

## 受众满意度检查清单

### 新人视角
- [x] 能在 5 分钟内理解项目定位和用途 → [01_Overview](./01_Overview.md)
- [x] 能在 15 分钟内找到核心代码位置 → [02_Directory_Structure](./02_Directory_Structure.md)
- [x] 能在 30 分钟内理解基本架构 → [03_Architecture](./03_Architecture.md)

### 安全研究员视角
- [x] 能快速识别所有外部输入入口 → [03_Architecture](./03_Architecture.md) + [07_Security](./07_Security.md)
- [x] 能定位敏感操作和权限检查点 → [07_Security](./07_Security.md)
- [x] 每个风险都有可利用性评估 → [07_Security](./07_Security.md)

# Data Share Wiki 导航

## 新人阅读顺序

### 第一步：了解项目（15分钟）
1. [概览](01_Overview.md) - 项目定位、核心能力、运行环境
2. [目录结构](02_Directory_Structure.md) - 模块职责与文件组织

### 第二步：理解接口（30分钟）
3. [对外 N-API](04_NAPI_Reference.md) - JS API 清单、参数、错误码
4. [内部 API](06_Inner_API.md) - C++ 接口定义、模块依赖

### 第三步：深入架构（45分钟）
5. [架构设计](05_Architecture.md) - 组件图、数据流、线程模型
6. [关键调用链](appendix/Callgraphs.md) - 入口→核心逻辑调用链

### 第四步：构建与安全（30分钟）
7. [构建系统](07_Build_System.md) - GN Targets、编译产物、依赖
8. [安全分析](08_Security_Analysis.md) - 攻击面、风险点、修复建议

### 附录速查
- [配置开关](appendix/Config_Flags.md) - 关键宏与 feature flags
- [错误码速查](appendix/Error_Codes.md) - 完整错误码列表

---

## 按角色导航

### 我是 JS/ArkTS 开发者
- [N-API 文档](04_NAPI_Reference.md) - 完整的 JS API 参考
- [架构设计 - N-API 层](05_Architecture.md#n-api-层) - 理解 JS 到 C++ 的调用链
- [错误码速查](appendix/Error_Codes.md) - 快速定位问题

### 我是 Native 开发者
- [内部 API](06_Inner_API.md) - C++ 接口完整定义
- [架构设计 - Native 层](05_Architecture.md#native-层) - 理解模块划分
- [关键调用链](appendix/Callgraphs.md) - 跟踪代码执行流程

### 我是架构师
- [架构设计](05_Architecture.md) - 完整架构图和线程模型
- [构建系统](07_Build_System.md) - 模块依赖关系
- [安全分析](08_Security_Analysis.md) - 安全设计评估

### 我是安全工程师
- [安全分析](08_Security_Analysis.md) - 完整安全评审
- [架构设计 - IPC 层](05_Architecture.md#ipc-通信层) - 信任边界分析
- [权限相关代码](06_Inner_API.md#权限模块) - 权限检查点梳理

### 我是构建工程师
- [构建系统](07_Build_System.md) - 完整构建配置
- [GN Targets 清单](07_Build_System.md#target-清单) - 所有构建目标
- [产物映射](07_Build_System.md#产物映射) - 输出文件路径

---

## 文档清单

| 文档 | 路径 | 状态 | 最后更新 |
|------|------|------|----------|
| README | [README.md](README.md) | ✅ | 2025-02-06 |
| 概览 | [01_Overview.md](01_Overview.md) | ✅ | 2025-02-06 |
| 目录结构 | [02_Directory_Structure.md](02_Directory_Structure.md) | ✅ | 2025-02-06 |
| N-API 参考 | [04_NAPI_Reference.md](04_NAPI_Reference.md) | ✅ | 2025-02-06 |
| 架构设计 | [05_Architecture.md](05_Architecture.md) | ✅ | 2025-02-06 |
| 内部 API | [06_Inner_API.md](06_Inner_API.md) | ✅ | 2025-02-06 |
| 构建系统 | [07_Build_System.md](07_Build_System.md) | ✅ | 2025-02-06 |
| 安全分析 | [08_Security_Analysis.md](08_Security_Analysis.md) | ✅ | 2025-02-06 |
| 调用链附录 | [appendix/Callgraphs.md](appendix/Callgraphs.md) | ✅ | 2025-02-06 |
| 错误码附录 | [appendix/Error_Codes.md](appendix/Error_Codes.md) | ✅ | 2025-02-06 |
| 配置开关附录 | [appendix/Config_Flags.md](appendix/Config_Flags.md) | ✅ | 2025-02-06 |

---

## 快速链接

### 核心代码路径
- N-API 入口: `frameworks/js/napi/dataShare/src/native_datashare_module.cpp`
- DataShareHelper: `interfaces/inner_api/consumer/include/datashare_helper.h`
- 权限检查: `frameworks/native/permission/src/data_share_permission.cpp`
- 服务端实现: `frameworks/native/provider/src/datashare_stub_impl.cpp`
- 错误码定义: `interfaces/inner_api/common/include/datashare_errno.h`

### 构建配置
- 主构建文件: `interfaces/inner_api/BUILD.gn`
- N-API 构建: `frameworks/js/napi/dataShare/BUILD.gn`
- 路径变量: `datashare.gni`

### 外部依赖
- `ability_runtime` - Ability 框架
- `access_token` - 权限管理
- `ipc` - IPC 通信
- `kv_store` - 分布式数据
- `bundle_framework` - 包管理

---

## 反馈与更新

如发现文档错误或有改进建议，请：
1. 在代码仓库提交 Issue
2. 联系 Data Share 子系统维护者
3. 参考 [README.md](README.md) 中的更新方式

**当前文档版本**: 3.2.0 (对应代码版本)

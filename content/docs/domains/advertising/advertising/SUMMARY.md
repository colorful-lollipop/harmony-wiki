# Advertising 子系统文档导航

## 新人阅读顺序（推荐）

```
1️⃣  index.md          → 项目概览
2️⃣  Architecture.md    → 架构理解
3️⃣  NAPI_Reference.md → API 使用
4️⃣  Build_Configuration.md → 构建配置
5️⃣  Security_Review.md → 安全考量
```

## 安全研究员阅读顺序（推荐）

```
1️⃣  index.md          → 项目定位
2️⃣  Security_Review.md → 攻击面与风险
3️⃣  Architecture.md    → 数据流与信任边界
4️⃣  NAPI_Reference.md → API 安全考量
5️⃣  Build_Configuration.md → 编译安全选项
```

## 完整文档列表

### 入门指南
- [README](README.md) - 文档说明与更新方式
- [项目概览](index.md) - 定位、能力、关键概念

### 架构设计
- [架构说明](Architecture.md) - 组件图、数据流、线程模型
- [内部 API](Inner_API.md) - 模块接口、依赖方向

### API 参考
- [N-API 接口文档](NAPI_Reference.md) - JS API 清单、参数、错误码

### 工程配置
- [构建配置](Build_Configuration.md) - GN Targets、编译产物、安装路径

### 安全评估
- [安全风险评审](Security_Review.md) - 攻击面、风险点、修复建议

### 工作文件（内部使用）
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估（Phase 0）
- [_work/NOTES.md](_work/NOTES.md) - 代码证据汇总
- [_work/PLAN.md](_work/PLAN.md) - 任务进度追踪
- [_work/QUALITY_REPORT.md](_work/QUALITY_REPORT.md) - 质量报告

## 快速跳转

### 关键文件位置
| 内容 | 路径 |
|-----|------|
| N-API 入口 | `frameworks/js/napi/ads/src/ad_init.cpp:51` |
| 广告加载 | `frameworks/js/napi/ads/src/ad_load_service.cpp:113` |
| IPC 代理 | `common/ipc/src/ad_load_proxy.cpp:36` |
| 错误码 | `common/error_code/ad_inner_error_code.h` |

### 关键常量
| 常量 | 值 | 说明 |
|-----|---|------|
| `ADVERTISING_ID` | 6104 | SA 系统能力 ID |
| `USER_ID` | -1 | 默认用户 ID |
| `CONNECT_TIME_OUT` | 3s | 连接超时 |

## 代码证据索引

所有关键结论均可在代码中找到证据：

- **N-API 注册**: `ad_init.cpp:51-89`
- **导出函数**: `advertising.cpp:871-888`
- **AdLoader 类**: `advertising.h:120-127`
- **IPC 通信**: `ad_load_proxy.cpp:36-80`
- **构建配置**: `BUILD.gn` 文件 (9个)

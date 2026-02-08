# 文档导航

## 新人阅读路线

```
建议阅读顺序:
1. README.md (本文档说明)
2. 01_Overview.md (项目概览)
3. 02_Architecture.md (架构设计)
4. 03_NAPI_Reference.md (API接口)
5. 04_Build_Guide.md (构建配置)
6. 05_Security_Review.md (安全评审)
```

## 完整目录

### 入门指南
- [README](./README.md) - 文档说明与更新日志

### 核心文档
- [01_Overview.md](./01_Overview.md) - 项目定位与核心能力
- [02_Architecture.md](./02_Architecture.md) - 系统架构与组件关系
- [03_NAPI_Reference.md](./03_NAPI_Reference.md) - N-API 接口参考
- [04_Build_Guide.md](./04_Build_Guide.md) - GN 构建与产物
- [05_Security_Review.md](./05_Security_Review.md) - 安全风险评估

### 附录
- [附录A: 接口清单](./appendix/A_API_List.md) - 完整API列表
- [附录B: 错误码参考](./appendix/B_Error_Codes.md) - 错误码定义
- [附录C: SA配置说明](./appendix/C_SA_Config.md) - Service Ability配置

## 模块索引

| 模块 | 路径 | 职责 |
|------|------|------|
| ANS Client | `frameworks/ans/` | 通知客户端库 |
| ANS Service | `services/ans/` | 通知核心服务 |
| N-API | `frameworks/js/napi/` | JS接口绑定 |
| ETS/ANI | `frameworks/ets/ani/` | ArkTS接口绑定 |
| Inner API | `interfaces/inner_api/` | 内部模块接口 |
| NDK | `interfaces/ndk/` | C接口 |

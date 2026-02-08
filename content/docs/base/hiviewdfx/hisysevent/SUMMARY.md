# HiSysEvent Wiki 导航

## 新人阅读路线

```
推荐阅读顺序:
1. 00_Overview.md  → 项目概览
2. 01_Architecture.md → 架构理解
3. 根据需要选择:
   - 02_N-API.md  (JS/ArkTS 开发者)
   - 03_CPP_API.md (Native 开发者)
4. 04_Build.md    → 构建配置
5. 05_Security.md → 安全注意事项
6. 06_Troubleshooting.md → 故障排查
```

## 文档索引

### 快速入门
- [首页](README.md)
- [概览](00_Overview.md)

### 架构与设计
- [架构说明](01_Architecture.md)

### API 参考
- [N-API 参考](02_N-API.md) - JS/ArkTS 接口
- [C++ API 参考](03_CPP_API.md) - Native 接口

### 工程实践
- [构建与编译](04_Build.md)
- [安全风险评审](05_Security.md)
- [故障排查](06_Troubleshooting.md)

## 代码导航

| 模块 | 路径 | 主要文件 |
|------|------|----------|
| N-API 实现 | `interfaces/js/kits/napi/` | `napi_hisysevent_js.cpp` |
| C++ 核心 | `interfaces/native/innerkits/hisysevent/` | `hisysevent.h`, `hisysevent.cpp` |
| 事件管理 | `interfaces/native/innerkits/hisysevent_manager/` | `hisysevent_manager.h` |
| 构建配置 | `interfaces/native/innerkits/*/BUILD.gn` | BUILD.gn 文件 |

## 版本信息

- **系统能力**: SystemCapability.HiviewDFX.HiSysEvent
- **组件版本**: 3.1
- **支持设备**: standard (标准设备)

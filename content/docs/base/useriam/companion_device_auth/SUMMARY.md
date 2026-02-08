# SUMMARY - 伴随设备认证Wiki导航

> 新人阅读顺序与全站导航

---

## 推荐阅读顺序

### 第一阶段：建立整体认知 (30分钟)
1. [README.md](./README.md) - 了解文档范围与结构
2. [00_Overview.md](./00_Overview.md) - 理解项目定位、边界、核心能力

### 第二阶段：理解对外接口 (1小时)
3. [02_NAPI_Reference.md](./02_NAPI_Reference.md) - JS API使用方式

### 第三阶段：深入架构实现 (2小时)
4. [01_Architecture.md](./01_Architecture.md) - 组件关系与数据流
5. [03_Inner_API.md](./03_Inner_API.md) - 内部模块接口

### 第四阶段：工程实践 (按需)
6. [04_GN_Targets.md](./04_GN_Targets.md) - 构建与产物
7. [05_Security.md](./05_Security.md) - 安全分析
8. [06_Troubleshooting.md](./06_Troubleshooting.md) - 问题排查

### 附录参考
9. [appendix/Callgraphs.md](./appendix/Callgraphs.md) - 调用链
10. [appendix/Config_Flags.md](./appendix/Config_Flags.md) - 配置项

---

## 按角色导航

### 应用开发者 (JS/TS)
- 必读: [02_NAPI_Reference.md](./02_NAPI_Reference.md)
- 参考: [00_Overview.md](./00_Overview.md) 中的使用场景

### 系统开发者 (Native/C++)
- 必读: [01_Architecture.md](./01_Architecture.md)
- 必读: [03_Inner_API.md](./03_Inner_API.md)
- 参考: [04_GN_Targets.md](./04_GN_Targets.md)

### 安全工程师
- 必读: [05_Security.md](./05_Security.md)
- 参考: [01_Architecture.md](./01_Architecture.md) 信任边界部分

### 构建/CI工程师
- 必读: [04_GN_Targets.md](./04_GN_Targets.md)
- 参考: [appendix/Config_Flags.md](./appendix/Config_Flags.md)

---

## 文档索引

### 核心文档

| 文档 | 内容 | 代码证据 |
|------|------|----------|
| [00_Overview.md](./00_Overview.md) | 项目定位、应用场景、目录结构 | `README_ZH.md`, `bundle.json` |
| [01_Architecture.md](./01_Architecture.md) | 组件图、数据流、线程模型 | `services/`, `frameworks/` |
| [02_NAPI_Reference.md](./02_NAPI_Reference.md) | JS API清单、参数、错误码 | `frameworks/js/napi/src/` |
| [03_Inner_API.md](./03_Inner_API.md) | 内部接口、依赖关系 | `services/*/inc/`, `interfaces/` |
| [04_GN_Targets.md](./04_GN_Targets.md) | GN目标、产物、安装路径 | `BUILD.gn`, `*.gni` |
| [05_Security.md](./05_Security.md) | 攻击面、风险点、修复建议 | 权限检查点代码 |
| [06_Troubleshooting.md](./06_Troubleshooting.md) | 常见问题、调试方法 | 日志标签、错误码定义 |

### 附录

| 文档 | 内容 |
|------|------|
| [appendix/Callgraphs.md](./appendix/Callgraphs.md) | 关键调用链图示 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 编译配置与宏定义 |

---

## 快速链接

### 关键源码位置
- N-API入口: `frameworks/js/napi/src/companion_device_auth_entry.cpp:607`
- 服务入口: `services/service_entry/src/companion_device_auth_service.cpp`
- 权限检查: `frameworks/js/napi/src/companion_device_auth_entry.cpp:35-60`
- IPC接口: `frameworks/native/ipc/idl/ICompanionDeviceAuth.idl`

### 关键配置
- SA配置: `sa_profile/945.json`
- 构建配置: `companion_device_auth.gni`
- 组件配置: `bundle.json`

### 错误码参考
- 定义位置: `common/inc/common_defines.h:28-54`
- JS错误码映射: `frameworks/js/napi/src/companion_device_auth_napi_helper.cpp:45-53`

---

**提示**: 所有文档中的代码引用均可通过路径和行号在源代码中找到对应实现。


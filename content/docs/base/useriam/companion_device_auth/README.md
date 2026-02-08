# companion_device_auth 工程 Wiki

> 基于代码证据的OpenHarmony伴随设备认证子系统技术文档
> 生成时间: 2025-02-06
> 更新时间: 2026-02-07
> 版本: 4.0

---

## 文档范围

本文档覆盖 `base/useriam/companion_device_auth` 仓库的完整技术分析，包括：

- **项目定位与边界**: 伴随设备认证在OpenHarmony用户认证体系中的角色
- **架构设计**: 组件图、数据流、线程模型、关键时序
- **对外API**: JS/N-API接口、权限、参数、错误码
- **内部API**: 模块接口、依赖方向、稳定性
- **构建系统**: GN目标、编译产物、安装路径
- **安全风险**: 攻击面、信任边界、可利用点分析
- **调试指南**: 常见问题与定位方法

## 未覆盖范围

- 测试代码 (`test/` 目录下的所有内容)
-  fuzz测试相关实现细节
- 特定硬件TEE实现细节
- 与厂商定制相关的扩展接口

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 导航与阅读指南
├── 00_Overview.md               # 项目概览
├── 01_Architecture.md           # 架构说明
├── 02_NAPI_Reference.md         # N-API接口文档
├── 03_Inner_API.md              # 内部API文档
├── 04_GN_Targets.md             # 构建目标与产物
├── 05_Security.md               # 安全风险评审
├── 06_Troubleshooting.md        # 问题排查
└── appendix/
    ├── Callgraphs.md            # 关键调用链
    └── Config_Flags.md          # 配置与宏定义
```

## 更新方式

本文档基于代码仓库的静态分析生成。当代码变更时，建议：

1. 关键接口变更需同步更新接口文档
2. 新增模块需补充架构说明
3. 安全相关改动需重新评审风险点
4. 构建系统变更需更新GN文档

## 关键代码路径速查

| 内容 | 路径 |
|------|------|
| 版本/组件配置 | `bundle.json` |
| 构建配置 | `companion_device_auth.gni` |
| N-API实现 | `frameworks/js/napi/src/` |
| IPC接口定义 | `frameworks/native/ipc/idl/` |
| 服务入口 | `services/service_entry/src/companion_device_auth_service.cpp` |
| 安全代理 | `services/security_agent/` |
| 跨设备交互 | `services/cross_device_interaction/` |

## 术语表

| 术语 | 说明 |
|------|------|
| 伴随设备(Companion Device) | 与主设备绑定的辅助设备，如手表、耳机 |
| 主设备(Host Device) | 被认证的目标设备，如手机、PC |
| Template | 伴随设备与主设备的绑定凭证 |
| SA | SystemAbility，系统能力服务 |
| N-API | Node-API，JS与C++绑定接口 |
| SoftBus | OpenHarmony分布式软总线 |
| IDL | Interface Definition Language，接口定义语言 |

---

**License**: Apache License 2.0 (与源代码一致)


# 分布式屏幕 Wiki 导航

> 双路线设计：新人学习路线 和 安全研究路线

---

## 🎓 新人学习路线 (推荐)

**目标**: 5分钟理解项目 → 15分钟找到代码 → 30分钟理解架构

| 顺序 | 文档 | 阅读重点 | 预计时间 |
|------|------|----------|----------|
| 1 | [项目概览](00_Overview.md) | 项目定位、核心能力、双SA架构 | 5分钟 |
| 2 | [架构设计](01_Architecture.md) | 组件图、数据流图、主控端/被控端协作 | 15分钟 |
| 3 | [目录结构](02_Directory_Structure.md) | 模块职责、核心文件定位 | 10分钟 |
| 4 | [对外接口](03_Interfaces.md) | IPC接口定义、调用方式、权限要求 | 15分钟 |
| 5 | [内部接口](04_Inner_APIs.md) | 模块间接口和调用链 | 10分钟 |
| 6 | [构建系统](05_Build_System.md) | GN构建配置、编译命令 | 10分钟 |
| 7 | [编译产物](06_Products.md) | 产物清单、安装路径 | 5分钟 |

---

## 🔒 安全研究路线 (推荐)

**目标**: 快速识别攻击面 → 定位漏洞点 → 评估可利用性

| 顺序 | 文档 | 阅读重点 | 关键证据 |
|------|------|----------|----------|
| 1 | [项目概览](00_Overview.md) | 双SA架构、信任边界 | SA 4807/4808 |
| 2 | [安全风险](07_Security.md) | 攻击面清单、6+可利用点 | 含高危漏洞 |
| 3 | [对外接口](03_Interfaces.md) | IPC输入点、权限检查 | Stub实现 |
| 4 | [架构设计](01_Architecture.md) | 数据流、信任边界跨越 | SoftBus通信 |
| 5 | [内部接口](04_Inner_APIs.md) | 敏感操作、内部调用链 | 权限传播 |

**关键发现速览**:
- ⚠️ **高危**: ConfigDistributedHardware 接口缺少权限检查
- ⚠️ **高危**: 多处内存分配未使用 nothrow
- ⚠️ **中危**: Sink端订阅接口缺乏权限保护
- ✅ 编译安全选项完整 (CFI/UBSan/栈保护)

---

## 完整文档索引

### 核心文档
- [00_项目概览](00_Overview.md) - 项目定位、核心能力、运行环境
- [01_架构设计](01_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [02_目录结构](02_Directory_Structure.md) - 模块职责和文件组织
- [03_对外接口](03_Interfaces.md) - Native SDK接口定义和使用方式
- [04_内部接口](04_Inner_APIs.md) - 模块间接口和依赖关系
- [05_构建系统](05_Build_System.md) - GN构建配置和targets详解
- [06_编译产物](06_Products.md) - 产物清单和安装路径
- [07_安全风险](07_Security.md) - 攻击面分析、可利用点、修复建议
- [08_问题排查](08_Troubleshooting.md) - 常见问题和调试方法

---

## 附录

- [关键调用链](appendix/Callgraphs.md)
- [配置参数](appendix/Config_Flags.md)

---

## 核心信息速查

### SA ID
| 服务 | ID | 产物 |
|------|-----|------|
| Source (主控端) | 4807 | libdistributed_screen_source.z.so |
| Sink (被控端) | 4808 | libdistributed_screen_sink.z.so |

### 关键文件路径
```
SA配置:    sa_profile/{4807,4808}.json
IPC接口:   common/include/dscreen_ipc_interface_code.h
错误码:    common/include/dscreen_errcode.h
常量定义:  common/include/dscreen_constants.h
Source接口: interfaces/innerkits/native_cpp/screen_source/include/idscreen_source.h
Sink接口:   interfaces/innerkits/native_cpp/screen_sink/include/idscreen_sink.h
```

### 构建入口
```
bundle.json 定义 11 个 sub_component
distributedscreen.gni 定义构建变量
```

---

*最后更新: 2025-02-06*
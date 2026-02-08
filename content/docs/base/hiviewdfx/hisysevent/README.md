# HiSysEvent 组件 Wiki

## 概述

本文档是 OpenHarmony DFX 子系统下 **HiSysEvent** 组件的工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、构建流程以及安全注意事项。

## 覆盖范围

| 文档 | 内容说明 | 证据来源 |
|------|----------|----------|
| [SUMMARY](SUMMARY.md) | 全站导航索引 | - |
| [概览](00_Overview.md) | 项目定位、核心能力、运行环境 | `README_zh.md`, `bundle.json` |
| [架构说明](01_Architecture.md) | 组件图、数据流、线程模型 | 代码目录结构分析 |
| [N-API 参考](02_N-API.md) | JS/ArkTS API 完整文档 | `napi_hisysevent_js.cpp:357-365` |
| [C++ API 参考](03_CPP_API.md) | Native 接口完整文档 | `hisysevent.h`, `hisysevent_c.h`, `def.h` |
| [构建与编译](04_Build.md) | GN targets、编译产物、依赖关系 | `BUILD.gn`, `bundle.json` |
| [安全风险评审](05_Security.md) | 攻击面、信任边界、修复建议 | 代码安全机制分析 |
| [故障排查](06_Troubleshooting.md) | 常见问题与定位路径 | 错误码定义、业务逻辑 |

## 未覆盖范围

以下内容不在本文档范围内（需参考其他官方文档）：

1. **SysEventImpl 服务端实现**: 属于系统服务层，详细 IPC 协议和 SA 注册流程
2. **HDF 驱动集成**: 底层硬件抽象层实现
3. **云端大数据分析**: 事件数据上报后的云端处理流程
4. **IDE 集成开发**: DevEco Studio 中的图形化配置
5. **性能调优指南**: 详细的性能测试数据和优化建议

## 术语一致性

本文档使用以下核心术语，与代码保持一致：

| 术语 | 定义 | 代码对应 |
|------|------|----------|
| Domain | 事件所属系统模块 | `HiSysEvent::Domain` |
| EventType | 事件类型枚举 | `HiSysEvent::EventType` |
| N-API | Node.js Native API 封装层 | `napi_*` 函数 |
| Innerkit | 内部子系统 C++ 接口 | `interfaces/native/innerkits/` |
| SA | System Ability | IPC 服务端 |
| Writer | 事件写入者 | 调用 `Write()` 的主体 |

## 证据标准

所有关键结论均提供代码证据：

| 结论类型 | 证据要求 |
|----------|----------|
| API 签名 | 头文件路径 + 行号 |
| 错误码定义 | `def.h` 文件路径 + 行号 |
| 构建配置 | `BUILD.gn` 文件路径 + target 定义 |
| 安全机制 | 实现文件路径 + 行号 |
| 目录结构 | 代码仓库实际路径 |

## 更新方式

本 Wiki 由 OpenHarmony 工程 Agent 自动生成。建议在以下场景重新扫描：

### 触发条件

| 场景 | 操作 |
|------|------|
| 新增 N-API 接口 | 重新扫描 `interfaces/js/kits/napi/` |
| 新增 C++ 接口 | 重新扫描 `interfaces/native/innerkits/` |
| 修改构建配置 | 重新扫描 `BUILD.gn` |
| 安全机制变更 | 更新 `05_Security.md` |

### 手动更新步骤

```bash
# 1. 克隆仓库到工作区
git clone <hisysevent-repo>

# 2. 运行 Wiki 生成工具
python3 generate_wiki.py --input <repo-path> --output ./wiki

# 3. 校验链接和引用
./verify_links.sh ./wiki

# 4. 提交更新
git add .
git commit -m "docs: update Wiki for new features"
```

**最后更新时间**: 2026-02-06

**源代码位置**: `//base/hiviewdfx/hisysevent`

## 相关链接

- [OpenHarmony DFX 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- [HiSysEvent C++ API 头文件](interfaces/native/innerkits/hisysevent/include/hisysevent.h)
- [HiSysEvent N-API 实现](interfaces/js/kits/napi/src/napi_hisysevent_js.cpp)

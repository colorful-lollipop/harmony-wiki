# SUMMARY

## 新人阅读路线

### 第一步：了解项目（15分钟）
1. [README](README.md) - 本文档说明
2. [00_Overview](00_Overview.md) - 项目定位与核心能力
3. [01_Directory_Structure](01_Directory_Structure.md) - 目录结构

### 第二步：理解架构（30分钟）
4. [02_Architecture](02_Architecture.md) - 架构设计
   - 组件图
   - 数据流
   - 关键时序

### 第三步：掌握接口（45分钟）
5. [03_NAPI_Interface](03_NAPI_Interface.md) - JS API 接口
   - API 清单
   - 调用链
   - 错误码
6. [04_Inner_API](04_Inner_API.md) - 内部 API
   - 模块接口
   - 依赖方向

### 第四步：工程实践（30分钟）
7. [05_GN_Targets](05_GN_Targets.md) - 构建系统
   - Targets 列表
   - 编译产物
8. [06_Security_Analysis](06_Security_Analysis.md) - 安全风险
   - 攻击面
   - 修复建议
9. [07_Troubleshooting](07_Troubleshooting.md) - 问题定位

## 快速参考

### 按主题索引

| 主题 | 文档 |
|------|------|
| 项目定位 | [00_Overview](00_Overview.md) |
| 目录结构 | [01_Directory_Structure](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture](02_Architecture.md) |
| JS API | [03_NAPI_Interface](03_NAPI_Interface.md) |
| C/C++ API | [04_Inner_API](04_Inner_API.md) |
| 构建系统 | [05_GN_Targets](05_GN_Targets.md) |
| 安全审计 | [06_Security_Analysis](06_Security_Analysis.md) |
| 问题排查 | [07_Troubleshooting](07_Troubleshooting.md) |

### 关键文件速查

| 文件 | 路径 | 说明 |
|------|------|------|
| Syscap定义 | `include/codec_config/syscap_define.h` | 所有系统能力枚举 |
| 主工具头 | `include/syscap_tool.h` | 核心编解码接口 |
| 内部API头 | `interfaces/inner_api/syscap_interface.h` | 内部接口定义 |
| N-API实现 | `napi/napi_query_syscap.cpp` | JS接口实现 |
| 主构建文件 | `BUILD.gn` | GN构建配置 |
| 组件配置 | `bundle.json` | 部件配置 |

### 关键符号速查

| 符号 | 位置 | 说明 |
|------|------|------|
| `SyscapNum` | `syscap_define.h:36` | 系统能力枚举 |
| `g_arraySyscap` | `syscap_define.h:414` | 系统能力映射表 |
| `PCIDMain` | `create_pcid.h:23` | PCID主结构 |
| `RPCIDHead` | `context_tool.h:36` | RPCID头部 |
| `EncodeOsSyscap` | `syscap_interface.h:46` | 编码OS Syscap |
| `ComparePcidString` | `syscap_interface.h:70` | 比较PCID字符串 |

## 附录

- [A. 调用链详情](appendix/Callgraphs.md)
- [B. 配置标志](appendix/Config_Flags.md)

---

*最后更新: 2025-02-06*

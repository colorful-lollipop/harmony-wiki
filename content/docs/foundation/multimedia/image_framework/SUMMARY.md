# SUMMARY

## Image Framework 工程 Wiki

### 新人入门路线
1. [README](README.md) - 本文档说明
2. [项目概览](00_Overview.md) - 了解项目定位与核心能力
3. [目录结构](01_Directory_Structure.md) - 熟悉代码组织
4. [架构说明](02_Architecture.md) - 理解组件关系与数据流

### API 参考
- [N-API 接口文档](03_NAPI_Reference.md)
  - PixelMap API
  - ImageSource API
  - ImagePacker API
  - ImageReceiver API
  - Picture API
- [内部 API 文档](04_Inner_API.md)
  - C++ 核心类
  - 插件接口
  - 内存管理

### 构建与产物
- [GN 构建目标](05_GN_Targets.md) - 完整构建系统参考
- [编译产物](06_Build_Artifacts.md) - 输出文件清单

### 高级主题
- [安全风险分析](07_Security_Risks.md) - 攻击面与漏洞分析
- [常见问题](08_FAQ.md) - 调试与问题定位

### 附录
- [调用链](appendix/Callgraphs.md) - 关键调用路径
- [配置标志](appendix/Config_Flags.md) - 编译选项参考

---

## 快速导航

### 按主题
| 主题 | 文档 |
|------|------|
| 项目介绍 | [00_Overview.md](00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| JS API | [03_NAPI_Reference.md](03_NAPI_Reference.md) |
| Native API | [04_Inner_API.md](04_Inner_API.md) |
| 构建系统 | [05_GN_Targets.md](05_GN_Targets.md) |
| 产物说明 | [06_Build_Artifacts.md](06_Build_Artifacts.md) |
| 安全分析 | [07_Security_Risks.md](07_Security_Risks.md) |
| FAQ | [08_FAQ.md](08_FAQ.md) |

### 按角色
| 角色 | 推荐阅读 |
|------|----------|
| **应用开发者** | 00_Overview → 03_NAPI_Reference → 08_FAQ |
| **系统开发者** | 01_Directory_Structure → 02_Architecture → 04_Inner_API |
| **安全审计** | 07_Security_Risks → 02_Architecture → 04_Inner_API |
| **构建维护** | 05_GN_Targets → 06_Build_Artifacts → appendix/Config_Flags |

### 关键代码路径
| 组件 | 路径 |
|------|------|
| N-API 注册 | `frameworks/kits/js/common/native_module_ohos_image.cpp` |
| PixelMap | `interfaces/innerkits/include/pixel_map.h` |
| ImageSource | `interfaces/innerkits/include/image_source.h` |
| ImagePacker | `interfaces/innerkits/include/image_packer.h` |
| 插件管理 | `plugins/manager/include/plugin_server.h` |
| 构建配置 | `BUILD.gn`, `ide/image_decode_config.gni` |

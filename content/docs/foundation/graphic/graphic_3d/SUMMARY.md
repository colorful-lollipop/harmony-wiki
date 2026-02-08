# Wiki 导航 - SUMMARY

## 快速导航

```
wiki/
├── README.md                 # 本Wiki说明
├── SUMMARY.md               # 本文档 - 全站导航
├── index.md                 # 首页/项目概览
├── 01_Directory_Structure.md # 目录结构与模块职责
├── 02_Architecture.md       # 架构设计
├── 03_NAPI_Reference.md     # 对外N-API接口
├── 04_Internal_API.md       # 内部API
├── 05_GN_Build.md           # GN构建系统
├── 06_Security.md           # 安全风险评审
├── 07_FAQ.md                # 常见问题
└── appendix/
    ├── Callgraphs.md        # 关键调用链
    ├── Config_Flags.md      # 配置宏说明
    └── Symbol_Index.md      # 符号索引
```

## 新人阅读顺序

### 第一阶段：建立整体认知（30分钟）
1. **[项目概览](index.md)** - 了解项目定位、核心能力
2. **[目录结构](01_Directory_Structure.md)** - 熟悉代码组织方式

### 第二阶段：理解核心架构（1小时）
3. **[架构设计](02_Architecture.md)** - ECS架构、数据流、线程模型

### 第三阶段：按需深入
4. 应用开发者 → **[N-API接口](03_NAPI_Reference.md)**
5. 系统开发者 → **[内部API](04_Internal_API.md)** → **[GN构建](05_GN_Build.md)**
6. 安全评审 → **[安全风险](06_Security.md)**

## 主题导航

### 接口文档
| 主题 | 文档 |
|------|------|
| JS API (N-API) | [03_NAPI_Reference.md](03_NAPI_Reference.md) |
| ETS API (Taihe) | [03_NAPI_Reference.md](03_NAPI_Reference.md#ets-taihe接口) |
| 内部C++ API | [04_Internal_API.md](04_Internal_API.md) |

### 实现文档
| 主题 | 文档 |
|------|------|
| ECS系统 | [02_Architecture.md](02_Architecture.md#ecs架构) |
| 渲染管线 | [02_Architecture.md](02_Architecture.md#渲染管线) |
| 插件系统 | [02_Architecture.md](02_Architecture.md#插件系统) |
| 资源管理 | [02_Architecture.md](02_Architecture.md#资源管理) |

### 工程文档
| 主题 | 文档 |
|------|------|
| 构建系统 | [05_GN_Build.md](05_GN_Build.md) |
| 编译产物 | [05_GN_Build.md](05_GN_Build.md#编译产物清单) |
| 安全风险 | [06_Security.md](06_Security.md) |
| 问题排查 | [07_FAQ.md](07_FAQ.md) |

## 附录索引

- **[关键调用链](appendix/Callgraphs.md)** - 入口到核心逻辑的调用路径
- **[配置宏](appendix/Config_Flags.md)** - Feature flags与编译选项
- **[符号索引](appendix/Symbol_Index.md)** - 关键类/函数/宏索引

## 外部链接

### 相关仓库
- [graphic_graphic_2d](https://gitee.com/openharmony/graphic_graphic_2d) - 2D图形子系统
- [arkui_ace_engine](https://gitee.com/openharmony/arkui_ace_engine) - ArkUI框架
- [third_party_vulkan-headers](https://gitee.com/openharmony/third_party_vulkan-headers) - Vulkan头文件
- [third_party_vulkan-loader](https://gitee.com/openharmony/third_party_vulkan-loader) - Vulkan加载器

### 官方文档
- [OpenHarmony 图形子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-arkgraphics3d/)
- [N-API开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/napi/)

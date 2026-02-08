# 目录结构

> ⚠️ 本文档基于 README 描述的标准结构推断，实际代码仓库可能有差异

## 标准目录结构

```
distributedfile/
│
├── figures/                          # 架构图等静态资源
│   └── distributed-file-subsystem-architecture.png
│
├── interfaces/                       # API 接口层（N-API 绑定）
│   └── kits/                        # 对外暴露的 JS API 套件
│       └── fileio/                  # fileio 模块
│           ├── napi_*.cpp           # N-API 实现
│           ├── napi_*.h             # 头文件
│           └── BUILD.gn             # 构建配置
│
├── utils/                           # 公共组件
│   ├── filemgmt_libhilog/           # 日志组件
│   │   ├── include/                 # 头文件
│   │   ├── src/                     # 源文件
│   │   └── BUILD.gn
│   │
│   └── filemgmt_libn/               # 平台抽象库（LibN）
│       ├── include/                 # 头文件
│       │   ├── napi/               # N-API 辅助
│       │   └── types/               # 类型定义
│       ├── src/                     # 源文件
│       └── BUILD.gn
│
├── services/                        # 服务层实现（可能在其他仓库）
│   ├── distributed/                 # 分布式文件服务
│   └── local/                       # 本地文件服务
│
├── BUILD.gn                          # 根构建入口
├── ohos.build                       # OpenHarmony 构建配置
│
└── README.md / README_zh.md          # 项目说明
```

## 模块职责

| 目录/模块 | 职责 | 稳定性 |
|-----------|------|--------|
| `interfaces/kits/fileio/` | N-API 绑定层，对外暴露 JS API | 稳定 |
| `utils/filemgmt_libhilog/` | 日志组件，跨平台日志接口 | 稳定 |
| `utils/filemgmt_libn/` | LibN 抽象库，类型系统/内存管理 | 稳定 |
| `services/` | 核心文件 I/O 逻辑 | 稳定 |

## 关键文件模式

### N-API 实现文件

```
interfaces/kits/fileio/
├── napi_fileio.cpp          # 主入口，模块注册
├── napi_file.cpp            # 文件操作绑定
├── napi_dir.cpp             # 目录操作绑定
├── napi_stream.cpp          # 流操作绑定
├── napi_stat.cpp            # 统计操作绑定
└── napi_common.cpp          # 公共辅助函数
```

### LibN 库文件

```
utils/filemgmt_libn/
├── include/napi/
│   ├── napi_util.h          # N-API 辅助函数
│   ├── napi_env.h           # N-API 环境封装
│   └── napi_callback.h      # 回调封装
│
├── src/
│   ├── napi_util.cpp
│   ├── napi_env.cpp
│   └── ...
│
└── BUILD.gn
```

## 头文件包含关系（推断）

```
napi_fileio.cpp
├── <js_native_api.h>        # N-API 底层
├── "napi_file.h"           # 本模块定义
├── "utils/filemgmt_libn/include/napi/napi_util.h"  # LibN 抽象
└── "utils/filemgmt_libhilog/include/log.h"         # 日志
```

## 排除的目录

以下目录不计入文档分析范围：

| 目录模式 | 说明 |
|----------|------|
| `test/` | 单元测试 |
| `tests/` | 集成测试 |
| `unittest/` | 单元测试 |
| `*_test.*` | 测试文件 |
| `*_fuzzer.*` | 模糊测试 |
| `build/` | 构建产物 |

## 目录变迁（待补充）

| 版本 | 变更内容 |
|------|----------|
| 1.0 | 初始结构（基于 README 推断） |

## 参考

- [项目概览](00_Overview.md)
- [N-API 文档](01_APIs.md)
- [构建系统](03_Build_System.md)

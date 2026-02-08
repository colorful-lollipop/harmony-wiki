# ets_utils 组件 Wiki

> OpenHarmony Commonlibrary 子系统 - ETS 工具库

## 项目信息

| 属性 | 值 |
|------|-----|
| **组件名称** | @ohos/ets_utils |
| **版本** | "" |
| **许可证** | Apache License 2.0 |
| **仓库** | https://gitee.com/openharmony/js_api_module |
| **子系统** | commonlibrary |
| **SysCap** | SystemCapability.Utils.Lang |
| **ROM** | 1400KB |
| **RAM** | ~4096KB |

## 概述

ets_utils 是 OpenHarmony 系统中的重要组件，提供丰富的工具类 API 给 ArkTS 应用调用。该组件包含四个主要子模块：

1. **js_api_module**: 基础 API（URL、XML、Buffer）
2. **js_util_module**: 实用工具（容器、JSON、编码）
3. **js_sys_module**: 系统级 API（进程、定时器、控制台）
4. **js_concurrent_module**: 并发支持（Worker、Taskpool）

## 目录结构

```
ets_utils/
├── js_api_module/           # API 模块
│   ├── url/                 # URL 解析
│   ├── uri/                 # URI 处理
│   ├── xml/                 # XML 解析与序列化
│   ├── buffer/              # 缓冲区操作
│   ├── fastbuffer/          # 高性能缓冲区
│   └── convertxml/          # XML 转换
├── js_util_module/          # 工具模块
│   ├── container/           # 容器类 (15种)
│   ├── util/                # 工具类
│   ├── json/                # JSON 处理
│   ├── collections/         # 集合工具
│   └── stream/              # 流处理
├── js_sys_module/           # 系统模块
│   ├── process/             # 进程管理
│   ├── timer/               # 定时器
│   ├── console/             # 控制台
│   └── dfx/                 # 调试工具
├── js_concurrent_module/    # 并发模块
│   ├── worker/              # Worker 线程
│   ├── taskpool/            # 任务池
│   └── utils/               # 并发工具
├── platform/                # 平台抽象层
└── wiki/                    # 文档目录
```

## 快速开始

### 导入方式

```typescript
// 方式一: 导入整个模块
import util from '@ohos.util'

// 方式二: 按需导入
import { Buffer, TextEncoder } from '@ohos.buffer'
import { Worker } from '@ohos.worker'
```

### 特性亮点

- **N-API 桥接**: 基于 Node.js N-API 实现高性能 native 调用
- **安全加固**: CFI + PAC 返回地址保护
- **类型安全**: 使用 type tag 进行运行时类型检查
- **跨平台**: 支持 OpenHarmony、Android、iOS

## 文档导航

请参阅 [SUMMARY.md](./SUMMARY.md) 获取完整的文档导航和阅读路线。

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs/blob/master/README_zh.md)
- [ArkTS API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-arkts/README_zh.md)
- [ets_utils 源码仓库](https://gitee.com/openharmony/js_api_module)

---

*最后更新: 2026-02-07*

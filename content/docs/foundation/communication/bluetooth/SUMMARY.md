# Bluetooth 模块 Wiki 导航

> **新人推荐阅读顺序**: [概览 → 目录 → API → 构建 → 安全](wiki/README.md#阅读建议新人路线)

## 核心文档

| 文档 | 说明 | 状态 |
|------|------|------|
| [README](README.md) | 文档覆盖范围、更新方式、阅读建议 | ✅ |
| [00_Overview](00_Overview.md) | 项目定位、边界、核心能力、运行环境 | ✅ |
| [01_Directory_Structure](01_Directory_Structure.md) | 目录结构、模块职责划分 | ✅ |
| [02_Architecture](02_Architecture.md) | 组件图、数据流、线程模型、IPC 架构 | ⏳ |
| [03_API_Reference](03_API_Reference.md) | API 总览与使用指南 | ⏳ |
| [04_Build_System](04_Build_System.md) | GN targets、编译产物、依赖关系 | ⏳ |
| [05_Security_Review](05_Security_Review.md) | 攻击面、风险点、修复建议 | ⏳ |
| [06_Troubleshooting](06_Troubleshooting.md) | 常见构建/运行/调试问题 | ⏳ |

## API 详细参考

| 模块 | N-API | C API | 说明 |
|------|-------|-------|------|
| BLE | [03_NAPI_Reference.md#ble-模块](03_API_Reference.md#ble-模块) | [03_CAPI_Reference.md#ble](03_API_Reference.md#ble) | 低功耗蓝牙 |
| GAP | [03_NAPI_Reference.md#gap-模块](03_API_Reference.md#gap-模块) | [03_CAPI_Reference.md#gap](03_API_Reference.md#gap) | 通用访问协议 |
| GATT | [03_NAPI_Reference.md#gatt-模块](03_API_Reference.md#gatt-模块) | [03_CAPI_Reference.md#gatt](03_API_Reference.md#gatt) | 属性协议 |
| A2DP | [03_NAPI_Reference.md#a2dp-模块](03_API_Reference.md#a2dp-模块) | - | 高级音频分发 |
| HFP | [03_NAPI_Reference.md#hfp-模块](03_API_Reference.md#hfp-模块) | - | 免提协议 |
| HID | [03_NAPI_Reference.md#hid-模块](03_API_Reference.md#hid-模块) | - | 人体学接口设备 |
| PAN | [03_NAPI_Reference.md#pan-模块](03_API_Reference.md#pan-模块) | - | 个人局域网 |
| OPP | [03_NAPI_Reference.md#opp-模块](03_API_Reference.md#opp-模块) | - | 对象推送协议 |
| PBAP | [03_NAPI_Reference.md#pbap-模块](03_API_Reference.md#pbap-模块) | - | 电话簿访问 |
| MAP | [03_NAPI_Reference.md#map-模块](03_API_Reference.md#map-模块) | - | 消息访问协议 |
| Socket | [03_NAPI_Reference.md#socket-模块](03_API_Reference.md#socket-模块) | - | RFCOMM 套接字 |

## 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 关键宏/feature flags |

## 快速索引

### 按代码位置

| 位置 | 对应章节 |
|------|----------|
| `frameworks/js/napi/` | [03_API_Reference.md](03_API_Reference.md) |
| `frameworks/c_api/` | [03_CAPI_Reference.md](03_API_Reference.md) |
| `frameworks/inner/ipc/` | [02_Architecture.md#ipc-架构](02_Architecture.md) |
| `interfaces/inner_api/` | [02_Architecture.md#内部接口](02_Architecture.md) |

### 按任务类型

| 任务 | 参考文档 |
|------|----------|
| 添加新 N-API | [03_NAPI_Reference.md#新增模块指南](03_API_Reference.md) |
| 理解 IPC 调用 | [02_Architecture.md#数据流](02_Architecture.md) |
| 修改 GN 构建 | [04_Build_System.md](04_Build_System.md) |
| 安全代码审查 | [05_Security_Review.md](05_Security_Review.md) |
| 排查构建问题 | [06_Troubleshooting.md](06_Troubleshooting.md) |

---

[返回项目根目录](../README.md)

# Wiki 导航索引

> 全站导航与新人阅读路线

## 文档索引

| 编号 | 文档 | 描述 | 适用角色 |
|------|------|------|----------|
| [00](00_Overview.md) | 项目概览 | 项目定位、能力边界、关键概念 | 所有开发者 |
| [01](01_Architecture.md) | 系统架构 | 组件图、数据流、线程模型 | 架构师、高级开发者 |
| [02](02_API_Reference.md) | API 参考 | Cangjie API 接口、使用示例 | 应用开发者 |
| [03](03_Build.md) | 构建配置 | GN targets、编译产物、安装路径 | 构建工程师 |
| [04](04_Security.md) | 安全评审 | 攻击面、风险点、修复建议 | 安全工程师 |

## 新人阅读路线

### 路线一：应用开发者快速上手

```
1. [00_Overview.md] → 了解项目定位和能力边界
2. [02_API_Reference.md] → 掌握 API 使用方法
3. [04_Security.md] → 注意安全注意事项
```

### 路线二：系统开发者深入理解

```
1. [00_Overview.md] → 项目整体认知
2. [01_Architecture.md] → 深入架构设计
3. [03_Build.md] → 理解构建流程
4. [02_API_Reference.md] → API 实现细节
```

### 路线三：构建工程师配置

```
1. [00_Overview.md] → 项目基本信息
2. [03_Build.md] → 详细构建配置
3. [01_Architecture.md] → 产物依赖关系
```

## 关键跳转链接

### 代码位置索引

| 分类 | 路径 | 相关文档 |
|------|------|----------|
| Cangjie Kit | `kit/LocationKit/` | [02](02_API_Reference.md)、[03](03_Build.md) |
| 核心实现 | `ohos/geo_location_manager/` | [01](01_Architecture.md)、[02](02_API_Reference.md) |
| 构建配置 | `BUILD.gn`、`kit/**/BUILD.gn`、`ohos/**/BUILD.gn` | [03](03_Build.md) |
| Mock 代码 | `mock/ohos.geo_location_manager.cj` | [01](01_Architecture.md) |

### API 快速查找

| API | 类 | 文档章节 |
|-----|-----|----------|
| `getCurrentLocation()` | GeoLocationManager | [02.1](02_API_Reference.md#21-geolocationmanager-类) |
| `isLocationEnabled()` | GeoLocationManager | [02.1](02_API_Reference.md#21-geolocationmanager-类) |
| `Location` | 数据类 | [02.2](02_API_Reference.md#22-location-类) |
| `CurrentLocationRequest` | 请求配置 | [02.4](02_API_Reference.md#24-currentlocationrequest-类) |
| `SingleLocationRequest` | 请求配置 | [02.5](02_API_Reference.md#25-singlelocationrequest-类) |

### 外部参考链接

- [项目 README](../README.md) - 原始项目说明
- [中文 README](../README_zh.md) - 中文项目说明
- [OpenHarmony Location 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/Dev_Guide/source_zh_cn/location/cj-location-guidelines.md)
- [LocationKit API 参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_zh_cn/apis/LocationKit)
- [OpenHarmony 代码贡献指南](https://gitcode.com/openharmony/docs/blob/master/zh-cn/contribute/参与贡献.md)

## 术语表

| 术语 | 英文 | 说明 |
|------|------|------|
| 仓颉 | Cangjie | OpenHarmony 的应用开发框架 |
| GNSS | Global Navigation Satellite System | 全球导航卫星系统 |
| FFI | Foreign Function Interface | 外部函数接口 |
| N-API | Native API | Native API 接口 |
| syscap | System Capability | 系统能力标识 |

## 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| 1.0 | 2025-02-06 | 初始版本，完成所有文档框架 |

## 贡献者

欢迎通过 Pull Request 贡献文档改进。

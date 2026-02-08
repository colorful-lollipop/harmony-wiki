# Sensor_Lite Wiki 导航

## 快速开始

- [首页](README.md) - 项目说明与导航
- [01_Overview.md](01_Overview.md) - 项目概览

## API 参考

- [02_API_Reference.md](02_API_Reference.md) - Native C API 完整文档

## 架构与设计

- [03_Architecture.md](03_Architecture.md) - 详细架构设计

## 构建与部署

- [04_Build.md](04_Build.md) - 构建配置与编译产物

## 安全

- [05_Security.md](05_Security.md) - 安全风险评审

---

## 新人阅读顺序

```
┌─────────────────────────────────────────────────────────────────┐
│  1. README.md                                                   │
│     ↓                                                           │
│  2. 01_Overview.md                                              │
│     了解: 项目定位、核心能力、运行环境                            │
│     ↓                                                           │
│  3. 02_API_Reference.md                                         │
│     学习: 8 个 API 的使用方式                                    │
│     ↓                                                           │
│  4. 03_Architecture.md                                         │
│     理解: 客户端-服务器架构、数据流                               │
│     ↓                                                           │
│  5. 04_Build.md                                                │
│     掌握: 编译配置与产物                                         │
└─────────────────────────────────────────────────────────────────┘
```

## 按角色导航

### 应用开发者

1. [01_Overview.md](01_Overview.md) - 了解能力边界
2. [02_API_Reference.md](02_API_Reference.md) - 查询 API 用法
3. [README.md](README.md) - 示例代码位置

### 框架开发者

1. [03_Architecture.md](03_Architecture.md) - 理解内部架构
2. [04_Build.md](04_Build.md) - 了解构建配置
3. [05_Security.md](05_Security.md) - 安全注意事项

### 安全工程师

1. [05_Security.md](05_Security.md) - 安全风险评审
2. [03_Architecture.md](03_Architecture.md) - 攻击面分析

---

## 文档变更日志

| 日期 | 版本 | 变更说明 |
|-----|------|---------|
| 2026-02-06 | 1.0 | 初始化版本 |

---

## 相关链接

- [OpenHarmony 泛Sensor子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/%E6%B3%9BSensor%E5%AD%90%E7%B3%BB%E7%BB%9F.md)
- [传感器驱动 HDI](https://gitee.com/openharmony/drivers_peripheral_sensor)
- [SAMGR_lite 框架](https://gitee.com/openharmony/foundation/systemabilitymgr/samgr_lite)

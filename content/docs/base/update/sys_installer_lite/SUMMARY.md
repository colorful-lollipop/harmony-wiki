# 工程 Wiki 导航

## 新人阅读顺序

1. **[README](README.md)** - 文档概览与更新方式
2. **[00_Overview](00_Overview.md)** - 项目定位、核心能力、运行环境
3. **[01_Architecture](01_Architecture.md)** - 架构设计、组件关系、数据流
4. **[02_Public_API](02_Public_API.md)** - 对外 C API 完整说明
5. **[03_Inner_API](03_Inner_API.md)** - 内部模块与 HAL 接口
6. **[04_GN_Build](04_GN_Build.md)** - 构建系统与产物
7. **[05_Security](05_Security.md)** - 安全风险评审
8. **[06_Troubleshooting](06_Troubleshooting.md)** - 常见问题与调试

---

## 快速索引

### 按主题

| 主题 | 文档 |
|------|------|
| 想快速了解项目 | [00_Overview](00_Overview.md) |
| 想集成 OTA 功能 | [02_Public_API](02_Public_API.md) |
| 想适配新芯片 | [03_Inner_API](03_Inner_API.md) + [04_GN_Build](04_GN_Build.md) |
| 关注安全性 | [05_Security](05_Security.md) |
| 遇到问题 | [06_Troubleshooting](06_Troubleshooting.md) |

### API 速查

| API | 文档位置 |
|-----|----------|
| `HotaInit` | [02_Public_API.md#HotaInit](02_Public_API.md) |
| `HotaWrite` | [02_Public_API.md#HotaWrite](02_Public_API.md) |
| `HotaRestart` | [02_Public_API.md#HotaRestart](02_Public_API.md) |
| `HotaHalWrite` | [03_Inner_API.md#HAL接口](03_Inner_API.md) |

### 关键代码路径

| 组件 | 路径 |
|------|------|
| 对外头文件 | `interfaces/kits/` |
| 升级核心逻辑 | `frameworks/source/updater/` |
| 签名验证 | `frameworks/source/verify/` |
| HAL 接口定义 | `hals/` |
| 构建配置 | `frameworks/BUILD.gn` |

---

**提示**: 所有文档中的代码引用均包含文件路径和行号，可直接在仓库中定位验证。

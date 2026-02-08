# Audio Framework Wiki - 目录

## 📖 新人阅读路线

建议阅读顺序（新手友好）：

1. **[00_Overview.md](00_Overview.md)** → 项目定位、核心能力、关键概念
2. **[01_Directory_Structure.md](01_Directory_Structure.md)** → 目录结构与模块职责
3. **[02_Architecture.md](02_Architecture.md)** → 整体架构、IPC/System Ability
4. **[03_NAPI.md](03_NAPI.md)** → JS API 接口清单
5. **[04_Build.md](04_Build.md)** → GN 构建系统与编译产物
6. **[05_Security.md](05_Security.md)** → 权限与安全机制

## 📚 完整目录

### 核心文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [README.md](README.md) | Wiki 说明与更新方式 | ⭐ |
| [SUMMARY.md](SUMMARY.md) | 全站导航与阅读路线 | ⭐ |
| [00_Overview.md](00_Overview.md) | 项目概览、核心能力、关键概念 | ⭐⭐⭐ |
| [01_Directory_Structure.md](01_Directory_Structure.md) | 目录结构与模块职责 | ⭐⭐⭐ |
| [02_Architecture.md](02_Architecture.md) | 架构设计、IPC/System Ability | ⭐⭐⭐ |
| [03_NAPI.md](03_NAPI.md) | N-API 接口清单与调用链 | ⭐⭐⭐ |
| [04_Build.md](04_Build.md) | GN 构建系统、Targets、产物 | ⭐⭐⭐ |
| [05_Security.md](05_Security.md) | 权限模型与安全机制 | ⭐⭐ |

### 附录

| 文档 | 描述 |
|------|------|
| [appendix/Callgraphs.md](appendix/Callgraphs.md) | 关键调用链图 |
| [appendix/Config_Flags.md](appendix/Config_Flags.md) | 特性开关清单 |

## 🔗 快速跳转

**按功能查找**：

- **播放/录音** → [03_NAPI.md](03_NAPI.md) - AudioRenderer/AudioCapturer
- **音量控制** → [03_NAPI.md](03_NAPI.md) - AudioVolumeManager
- **设备路由** → [03_NAPI.md](03_NAPI.md) - AudioRoutingManager
- **音效处理** → [03_NAPI.md](03_NAPI.md) - AudioEffectManager
- **构建编译** → [04_Build.md](04_Build.md)
- **权限问题** → [05_Security.md](05_Security.md)

**按层次查找**：

- **JS API** → [03_NAPI.md](03_NAPI.md)
- **Native API** → [02_Architecture.md](02_Architecture.md) + [04_Build.md](04_Build.md)
- **System Service** → [02_Architecture.md](02_Architecture.md)

---

*最后更新: 2025-02-06*

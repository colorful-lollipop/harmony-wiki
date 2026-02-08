# 阅读路线建议

## 快速了解 (5分钟)
适合：想快速了解 PulseAudio 在 OHOS 中的角色

1. [README.md](README.md) - 本文档
2. [01_Overview.md](01_Overview.md) - 第1、2节 (简介与定位)

## 开发者指南 (20分钟)
适合：需要使用或修改 PulseAudio 的开发者

1. [01_Overview.md](01_Overview.md) - 完整阅读
2. [02_Patches.md](02_Patches.md) - **重点**：了解所有 OHOS 适配点
3. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 了解依赖关系

## 深度技术剖析 (45分钟)
适合：负责升级维护、深度集成的技术专家

1. [01_Overview.md](01_Overview.md) - 完整阅读
2. [02_Patches.md](02_Patches.md) - **核心文档**，逐节精读
3. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 配置细节
4. [05_API_Differences.md](05_API_Differences.md) - API 变更分析
5. [06_Security.md](06_Security.md) - 安全风险评估

## 按角色阅读

### 音频框架开发者
推荐阅读：[01_Overview](01_Overview.md) → [02_Patches](02_Patches.md) → [04_Usage_in_OH](04_Usage_in_OH.md)

重点关注：
- OHOS 专用的协议命令 (`PA_COMMAND_UNDERFLOW_OHOS`)
- HiLog 日志集成方式
- Socket 服务器适配机制

### 构建系统维护者
推荐阅读：[03_Build_Integration.md](03_Build_Integration.md) → [02_Patches.md](02_Patches.md)

重点关注：
- BUILD.gn 的层级结构
- `ohos_paconfig.sh` 配置脚本
- 条件编译宏定义

### 安全工程师
推荐阅读：[06_Security.md](06_Security.md) → [02_Patches.md](02_Patches.md)

重点关注：
- Socket 权限配置
- init 集成安全模型
- 文件系统权限变更

### 升级维护人员
推荐阅读：[02_Patches.md](02_Patches.md) → [05_API_Differences.md](05_API_Differences.md)

重点关注：
- 无 Patch 文件的适配方式
- `HAVE_NO_OHOS` 宏的作用范围
- 上游版本兼容性风险

---

## 文档间关联关系

```
README (入口)
    ├── 01_Overview (基础认知)
    │       └── 04_Usage_in_OH (使用场景)
    ├── 02_Patches (核心适配)
    │       ├── 03_Build_Integration (构建实现)
    │       └── 05_API_Differences (API 影响)
    └── 06_Security (安全视角)
```

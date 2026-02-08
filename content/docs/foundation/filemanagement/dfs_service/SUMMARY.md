# 文档导航

> 新人阅读顺序建议：[概览] → [架构] → [接口] → [构建] → [产物] → [安全]

## 核心文档

| 顺序 | 文档 | 描述 | 关键内容 |
|------|------|------|----------|
| 1 | [首页 / 概览](./00_Overview.md) | 项目定位与核心能力 | 3 大核心模块、功能特性 |
| 2 | [架构设计](./01_Architecture.md) | 组件图与数据流 | 4 个 SA 服务、IPC 通信、线程模型 |
| 3 | [对外 N-API](./02_N-API.md) | JS / NDK / ANI 接口 | 2 个 JS 模块、11 个 NDK API |
| 4 | [内部 Inner API](./03_Inner-API.md) | 系统服务间接口 | 4 个 Inner Kit、依赖关系 |
| 5 | [GN 构建配置](./04_Build.md) | 构建目标与依赖 | targets 列表、编译开关 |
| 6 | [编译产物](./05_Artifacts.md) | 输出文件与安装路径 | .so 库、可执行文件、加载关系 |
| 7 | [安全风险评审](./06_Security.md) | 攻击面与风险点 | 权限校验、数据流、修复建议 |

## 附录

| 文档 | 描述 | 关键内容 |
|------|------|----------|
| [关键调用链](./appendix/Callgraphs.md) | 入口到核心的调用路径 | JS → NAPI → 服务 → IPC |
| [配置说明](./appendix/Config.md) | 配置文件与参数 | .cfg、.json、宏开关 |

## 快速索引

### N-API 模块

| 模块名 | 命名空间 | 导出文件 |
|--------|----------|----------|
| file.cloudSync | `file.cloudSync` | `interfaces/kits/js/cloudfilesync/` |
| cloudSyncManager | `cloudSyncManager` | `interfaces/kits/js/cloudsyncmanager/` |

### SA 服务

| SA ID | 服务名 | 进程 | 配置文件 |
|-------|--------|------|----------|
| 5201 | DistributedFileDaemon | distributedfiledaemon | `services/5201.json` |
| 5204 | CloudSyncService | cloudfileservice | `services/5204.json` |
| 5205 | CloudDaemon | cloudfiledaemon | `services/5205.json` |
| 5207 | CloudDiskService | clouddiskservice | `services/5207.json` |

### 编译产物

| 类型 | 产物 | 路径 |
|------|------|------|
| 共享库 | libdistributedfiledaemon.z.so | `/system/lib/` |
| 共享库 | libcloudsync_sa.z.so | `/system/lib/` |
| 可执行 | sa_main | `/system/bin/` |

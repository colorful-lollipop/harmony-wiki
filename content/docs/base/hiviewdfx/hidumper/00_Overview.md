# 项目概览

> 目的：帮助新人快速理解 HiDumper 是什么、能做什么、如何运行

## 1. 项目定位

**HiDumper** 是 OpenHarmony DFX 子系统中的统一系统信息获取工具，为开发、测试人员和 IDE 工具提供故障定位能力。

### 核心能力
- 系统信息导出（CPU、内存、网络、存储、进程等）
- System Ability 信息查询
- 崩溃日志获取（Faultlog 集成）
- JS Heap/CJ Heap 快照导出
- 内存 smaps 分析

### 适用场景
- 开发者调试时获取系统状态
- 测试人员收集性能数据
- 自动化测试中的状态快照
- 故障分析时的信息采集

## 2. 目录结构

```
hidumper/
├── client/              # 客户端入口
│   └── native/
├── frameworks/         # 框架核心
│   └── native/
│       ├── include/    # 头文件
│       └── src/
│           ├── common/      # 公共参数/配置
│           ├── dump_strategy/ # Dump 策略
│           ├── executor/    # 执行器
│           ├── factory/     # 工厂模式
│           ├── manager/     # DumpManager
│           └── util/        # 工具类
├── services/           # 服务层
│   ├── native/         # 服务实现
│   └── zidl/          # IPC 通讯
├── interfaces/        # 对外接口
│   └── innerkits/      # Native API
├── utils/             # 工具类
│   └── native/
├── sa_profile/        # SA 配置
├── test/             # 测试 (不纳入文档)
├── BUILD.gn          # 根构建
├── hidumper.gni      # GN 配置
└── bundle.json       # Bundle 配置
```

## 3. 关键概念

| 概念 | 说明 |
|-----|------|
| DumpManager | Dump 流程管理器，负责请求分发 |
| Dumper | 各类信息导出器（FileDumper, CmdDumper, CpuDumper, MemDumper 等） |
| Executor | Dump 执行器，具体执行导出逻辑 |
| DumpStrategy | Dump 策略，决定触发哪些 Dumpers |
| System Ability | 系统服务，HiDumper 以 SA 形式运行 |

## 4. 运行条件

### 编译依赖
- OpenHarmony SDK / 构建环境
- GN + Ninja 构建工具
- C++17 编译器

### 运行时依赖
- SystemAbilityManager (SAMgr)
- IPC 框架 (Binder)
- 权限: `ohos.permission.DUMP` (部分操作)

### 目标设备
- OpenHarmony 标准设备
- 支持的架构: arm64, x86_64

## 5. 与其他组件的关系

```
用户命令行 (hidumper)
       │
       ▼
DumpManagerService (SA 1212) ◄─── IPC ───► DumpManagerCpuService (SA 1215)
       │                                      │
       ▼                                      ▼
Dumpers/Executors                      CpuDumper/Usage
       │
       ▼
其他子系统 API (AbilityRuntime, BundleMgr, NetMgr, etc.)
```

## 6. 版本与兼容性

- **最低 OpenHarmony 版本**: 标准系统 (Standard)
- **SA 版本兼容性**: 1212 (主), 1215 (CPU)
- **产物版本**: libhidumperservice.z.so (SA 类型动态库)

## 相关文档

- [系统架构](./01_Architecture.md)
- [API 参考](./02_API_Reference.md)
- [构建系统](./03_Build_System.md)

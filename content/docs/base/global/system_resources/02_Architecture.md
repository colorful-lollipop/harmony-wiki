# 架构说明

## 目的

本文档描述 `system_resources` 模块的架构设计，包括组件关系、数据流、线程模型和关键时序，帮助开发者理解系统资源在 OpenHarmony 中的定位与流转。

## 适用范围

- **模块**: `/base/global/system_resources`
- **关注点**: 静态资源包的架构设计
- **不包含**: 运行时逻辑（由 global_resmgr_standard 实现）

## 架构概览

### 组件图

```mermaid
graph TD
    subgraph "构建时 (Build Time)"
        A[BUILD.gn] -->|定义| B[字体 Targets]
        A -->|定义| C[Hap Targets]
        D[systemres.gni] -->|提供| E[字体列表]
        D -->|提供| F[配置变量]
    end

    subgraph "系统资源 (Static Resources)"
        G[fonts/] -->|包含| H[字体文件 .ttf]
        I[systemres/] -->|包含| J[权限定义]
        I -->|包含| K[多语言字符串]
        I -->|包含| L[分层参数]
    end

    subgraph "构建产物 (Build Outputs)"
        M[SystemResources.hap] -->|安装到| N[系统分区]
        O[字体 .ttf] -->|安装到| P[fonts/ 分区]
    end

    subgraph "运行时 (Runtime)"
        Q[global_resmgr_standard] -->|加载| N
        Q -->|加载| P
        R[应用] -->|查询| Q
    end
```

### 模块职责

| 组件 | 职责 | 类型 |
|------|------|------|
| `fonts/` | 存储系统字体文件 | 静态资源 |
| `systemres/` | 存储系统资源包源文件 | 静态资源 |
| `BUILD.gn` | 定义构建 targets | 构建配置 |
| `systemres.gni` | 定义构建变量 | 构建配置 |
| `bundle.json` | 定义模块元信息 | 模块配置 |

## 数据流

### 构建时数据流

```mermaid
flowchart TD
    A[fonts/ 目录] -->|读取| B[BUILD.gn]
    C[systemres/ 目录] -->|读取| D[systemres/BUILD.gn]
    B -->|处理| E[ohos_prebuilt_etc]
    B -->|处理| F[ohos_shared_headers]
    D -->|处理| G[ohos_resources]
    D -->|处理| H[ohos_hap]
    E -->|输出| I[字体 .ttf 文件]
    F -->|输出| J[头文件集合]
    G -->|输出| K[资源集合]
    H -->|输出| L[SystemResources.hap]
```

### 字体加载数据流

```
fonts/*.ttf
    │
    ▼
BUILD.gn (ohos_prebuilt_etc)
    │
    ▼
out/fonts/*.ttf
    │
    ▼
系统分区 fonts/ 目录
    │
    ▼
global_resmgr_standard (加载并注册)
    │
    ▼
应用通过资源 ID 获取字体
```

## 线程模型

### 关键说明

**本模块为静态资源包，不涉及运行时线程模型。**

资源加载的线程模型由 `global_resmgr_standard` 实现：

| 阶段 | 线程 | 说明 |
|------|------|------|
| 构建时 | 主线程 | GN/Ninja 构建执行 |
| 系统启动 | 主线程 | 资源管理服务初始化 |
| 资源加载 | UI 线程/IO 线程 | 按需加载字体和资源 |

## 关键时序

### 构建时序

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant GN as BUILD.gn
    participant Ninja as Ninja
    participant Out as out/ 目录

    Dev->>GN: 执行 gn gen
    GN->>GN: 解析 systemres.gni
    GN->>GN: 遍历 sys_fonts_list
    GN->>GN: 生成 ohos_prebuilt_etc targets
    GN->>GN: 生成 ohos_hap target
    GN->>Ninja: 生成 build.ninja
    Ninja->>Out: 编译 SystemResources.hap
    Ninja->>Out: 编译字体 .ttf
    Out-->>Dev: 构建完成
```

### 资源加载时序 (运行时)

```
Note: 本模块不涉及运行时逻辑
以下由 global_resmgr_standard 处理:

1. 系统启动时
   └── global_resmgr_standard 初始化
       └── 解析 SystemResources.hap
       └── 注册字体资源
       └── 注册权限定义

2. 应用请求时
   └── 应用调用资源 API
       └── global_resmgr_standard 定位资源
       └── 返回资源引用/数据
```

## 依赖关系

### 模块间依赖

```mermaid
graph LR
    A[system_resources] -->|输出资源| B[global_resmgr_standard]
    C[global_i18n] -->|依赖| B
    D[应用框架] -->|依赖| B
    B -->|消费字体| A
    B -->|消费权限| A
```

### 构建依赖

| 依赖项 | 类型 | 说明 |
|--------|------|------|
| `//build/ohos.gni` | 导入 | OpenHarmony GN 模板 | BUILD.gn:13 |
| `//vendor/.../SystemResources.p7b` | 证书 | Hap 签名证书 | systemres.gni:19-20 |
| `//third_party/skia/.../fontconfig.json` | 配置 | 字体配置文件 | systemres.gni:23-26 |

### 被依赖项

| 依赖方 | 依赖内容 | 说明 |
|--------|----------|------|
| global_resmgr_standard | 字体文件、权限定义 | 资源管理服务 |
| 系统框架 | 权限元数据 | 权限校验 |

## 稳定性评估

### 模块稳定性

| 组件 | 稳定性 | 说明 |
|------|--------|------|
| `fonts/` | 高 | 字体文件几乎不变 |
| `systemres/` | 中 | 权限定义可能随版本更新 |
| `BUILD.gn` | 高 | 构建配置稳定 |
| `systemres.gni` | 中 | 字体列表可能更新 |
| `bundle.json` | 高 | 模块元信息稳定 |

### 接口稳定性

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| Hap 包格式 | 高 | 系统级资源包格式稳定 |
| 字体文件格式 | 高 | TTF 格式稳定 |
| 权限定义格式 | 中 | JSON Schema 可能演进 |

## 相关文档

| 文档 | 链接 |
|------|------|
| 全局导航 | [SUMMARY.md](SUMMARY.md) |
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 构建系统 | [03_Build_System.md](03_Build_System.md) |
| 系统资源 | [04_Resources.md](04_Resources.md) |
| 安全评审 | [05_Security.md](05_Security.md) |

## 更新日志

| 日期 | 变更 | 负责人 |
|------|------|--------|
| 2026-02-06 | 初始版本 | Wiki Generator |

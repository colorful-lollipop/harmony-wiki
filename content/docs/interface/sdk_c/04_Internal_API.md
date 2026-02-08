# 内部 API 与模块接口

> **目的**: 梳理内部 API、模块间依赖关系和接口稳定性  
> **适用范围**: 内部开发者、需要了解模块间协作的开发者  
> **生成时间**: 2025-02-06

---

## 1. 内部 API 概述

### 1.1 什么是内部 API

**内部 API** 是指 OpenHarmony 系统内部模块间使用的接口，**不对外暴露给应用开发者**。

### 1.2 与对外 API 的区别

| 特性 | 对外 API (NDK) | 内部 API |
|------|---------------|----------|
| **使用者** | 应用开发者 | 系统模块 |
| **稳定性** | 高度稳定（5 版本兼容） | 可能变更 |
| **文档** | 完整文档 | 内部注释 |
| **符号可见性** | .ndk.json 显式导出 | 头文件直接包含 |

---

## 2. 核心内部接口

### 2.1 IPC 内部接口

**模块**: `IPCKit/`

| 接口 | 功能 | 稳定性 |
|------|------|--------|
| `OH_IPCSkeleton_GetCallingTokenId` | 获取调用者 Token ID | 稳定 |
| `OH_IPCSkeleton_GetCallingPid` | 获取调用者进程 ID | 稳定 |
| `OH_IPCSkeleton_GetCallingUid` | 获取调用者用户 ID | 稳定 |
| `OH_IPCSkeleton_IsLocalCalling` | 检查是否本地调用 | 稳定 |
| `OH_IPCParcel_WriteInterfaceToken` | 写入接口 Token | 稳定 |
| `OH_IPCParcel_ReadInterfaceToken` | 读取接口 Token | 稳定 |

**依赖方向**:
```
系统服务 ──► IPCKit ──► 其他模块
```

### 2.2 日志内部接口

**模块**: `hiviewdfx/hilog/`

| 接口 | 功能 | 稳定性 |
|------|------|--------|
| `OH_LOG_Print` | 打印日志 | 稳定 |
| `OH_LOG_IsLoggable` | 检查日志级别 | 稳定 |
| `OH_LOG_SetLogLevel` | 设置日志级别 | API 21+ |

**依赖方向**:
```
所有模块 ──► hilog ──► 无依赖（基础层）
```

### 2.3 密钥管理内部接口

**模块**: `security/huks/`

| 接口 | 功能 | 稳定性 |
|------|------|--------|
| `OH_Huks_GenerateKeyItem` | 生成密钥 | 稳定 |
| `OH_Huks_ImportKeyItem` | 导入密钥 | 稳定 |
| `OH_Huks_ExportKeyItem` | 导出密钥 | 稳定 |
| `OH_Huks_DeleteKeyItem` | 删除密钥 | 稳定 |
| `OH_Huks_GetKeyItemParamSet` | 获取密钥参数 | 稳定 |
| `OH_Huks_InitSession` | 初始化密钥会话 | 稳定 |
| `OH_Huks_UpdateSession` | 更新密钥会话 | 稳定 |
| `OH_Huks_FinishSession` | 完成密钥会话 | 稳定 |

**依赖方向**:
```
加密服务、安全模块 ──► huks ──► TEE（可选）
```

---

## 3. 模块依赖关系

### 3.1 依赖方向总览

```
应用层
    │
    ▼
框架层 (arkui, ark_runtime)
    │
    ▼
服务层 (multimedia, graphic, security, network)
    │
    ▼
基础层 (hiviewdfx, global, filemanagement)
    │
    ▼
系统层 (IPCKit, drivers, startup)
```

### 3.2 关键依赖链

| 模块 | 依赖的模块 | 说明 |
|------|-----------|------|
| `arkui/ace_engine` | graphic/native_drawing, multimedia/image_framework | UI 渲染依赖图形和图像 |
| `multimedia/av_codec` | multimedia/media_foundation | 编解码依赖媒体基础类型 |
| `multimedia/player_framework` | multimedia/av_codec, multimedia/media_foundation | 播放器依赖编解码 |
| `multimedia/camera_framework` | graphic/native_window, graphic/native_buffer | 相机依赖窗口和缓冲区 |
| `graphic/native_drawing` | graphic/native_buffer | 绘制依赖缓冲区 |
| `web/webview` | graphic/native_window, arkui/ace_engine | WebView 依赖窗口和 UI |
| `security/huks` | 无（除 TEE 可选依赖） | 密钥管理相对独立 |
| `network/netstack` | security/netssl | 网络栈依赖 SSL/TLS |

### 3.3 禁止的依赖

根据 OpenHarmony 架构设计原则：

| 禁止行为 | 说明 |
|----------|------|
| **下层依赖上层** | 基础层不能依赖服务层或框架层 |
| **循环依赖** | A 依赖 B，B 不能依赖 A |
| **跨层级依赖** | 应通过标准接口通信，避免直接依赖 |

---

## 4. 接口稳定性标注

### 4.1 稳定性等级

| 等级 | 含义 | 变更策略 |
|------|------|----------|
| **稳定 (Stable)** | 接口已成熟，广泛使用 | 保持兼容，仅新增 |
| **实验 (Experimental)** | 新接口，可能变更 | 可修改，需文档说明 |
| **废弃 (Deprecated)** | 已标记废弃 | 保留 5 版本后删除 |
| **内部 (Internal)** | 内部使用，不保证兼容 | 可随时变更 |

### 4.2 主要模块稳定性

| 模块 | 整体稳定性 | 说明 |
|------|-----------|------|
| `hiviewdfx/hilog` | 稳定 | 基础日志服务 |
| `arkui/napi` | 稳定 | NAPI 运行时 |
| `security/huks` | 稳定 | 密钥管理服务 |
| `IPCKit` | 稳定 | IPC 基础服务 |
| `multimedia/av_codec` | 稳定 | 编解码接口 |
| `graphic/native_drawing` | 稳定 | 2D 绘制 API |
| `ai/neural_network_runtime` | 实验 | AI 推理接口 |
| `video_processing_engine` | 实验 | 视频处理新接口 |

---

## 5. 可替换点

### 5.1 可替换模块设计

| 模块 | 可替换点 | 替换方式 |
|------|----------|----------|
| `third_party/musl` | C 库实现 | 替换为其他 libc |
| `third_party/zlib` | 压缩库 | 替换为其他压缩库 |
| `graphic/graphic_2d` | 图形后端 | 替换为其他图形驱动 |
| `security/huks` | 密钥存储后端 | 替换为 TEE/SE 实现 |

### 5.2 接口抽象层

```
┌─────────────────────────────────────────┐
│            对外 API 层                   │
│    （稳定接口，不随实现变更）              │
├─────────────────────────────────────────┤
│            接口抽象层                     │
│    （定义抽象接口，屏蔽实现差异）           │
├─────────────────────────────────────────┤
│            具体实现层                     │
│    （可替换的具体实现）                    │
└─────────────────────────────────────────┘
```

---

## 6. 代码证据

| 结论 | 证据文件 | 关键内容 |
|------|----------|----------|
| IPC 接口 | `IPCKit/*.h` | 进程间通信接口 |
| 日志接口 | `hiviewdfx/hilog/include/hilog/log.h` | 日志服务接口 |
| 密钥接口 | `security/huks/include/native_huks_api.h` | 密钥管理接口 |
| 依赖关系 | `ndk_targets.gni` | 模块依赖列表 |

---

## 7. 相关跳转

- **上一章**: [N-API 接口文档](./03_NAPI_Reference.md)
- **下一章**: [GN 构建目标](./05_GN_Build.md)
- **架构说明**: [架构 - 组件关系](./02_Architecture.md#模块依赖关系图)
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**内部 API 文档 - 基于代码生成**

# 项目概览

## 仓库定位

`drivers_interface` 是 OpenHarmony 驱动子系统的**接口定义仓库**，负责管理所有硬件设备接口（HDI, Hardware Device Interface）的定义。

### 核心职责

1. **接口标准化**: 使用 IDL（接口定义语言）统一描述硬件抽象层接口
2. **跨系统适配**: 支持 Standard/Small/Mini 三种系统类型
3. **代码生成**: 提供 GN 构建模板，自动生成 IPC 框架代码
4. **模块化组织**: 按硬件模块分类，每个模块支持多版本迭代

### 在 OpenHarmony 中的位置

```
┌─────────────────────────────────────────────────────┐
│                   应用层 (ArkUI)                     │
├─────────────────────────────────────────────────────┤
│                   框架层 (JS/Native)                 │
├─────────────────────────────────────────────────────┤
│  drivers_peripheral (驱动实现)                       │
│       ↓ 通过 HDI 接口调用                             │
├─────────────────────────────────────────────────────┤
│  drivers_interface (本仓库 - HDI 接口定义)            │
│       ↓ IDL 编译生成                                  │
├─────────────────────────────────────────────────────┤
│  drivers_framework (HDF 框架)                        │
├─────────────────────────────────────────────────────┤
│                   内核层 (Kernel)                    │
└─────────────────────────────────────────────────────┘
```

## 核心能力

### 1. IDL 接口定义
- 定义硬件设备的抽象接口
- 支持同步/异步调用模式
- 支持回调机制（`[callback]` 标记）
- 支持接口继承扩展

### 2. 自动代码生成
- 客户端代理代码（Proxy）
- 服务端存根代码（Stub）
- 序列化/反序列化代码
- 驱动入口模板代码

### 3. 多版本管理
- 语义化版本号 `[major].[minor]`
- Major 版本不兼容
- Minor 版本向后兼容

## 运行环境

### 适配的系统类型

| 系统类型 | 典型设备 | 系统特征 | HDI 模式 |
|---------|---------|---------|---------|
| **Standard** | 手机、平板 | 完整系统功能 | IPC + Passthrough |
| **Small** | 手表、电视 | 中等复杂度 | Passthrough |
| **Mini** | IoT 设备 | 轻量级 | Low (直通) |

### 依赖的组件

根据 `bundle.json` 分析，主要依赖：

```json
{
  "deps": {
    "components": [
      "ipc",           // 进程间通信
      "hdf_core",      // HDF 核心框架
      "hilog"          // 日志系统
    ],
    "third_party": []
  }
}
```

## 关键概念

### HDI (Hardware Device Interface)

硬件设备接口，是 OpenHarmony 驱动框架的核心抽象：

```
┌─────────────────────────────────────┐
│           上层服务 (Service)          │
│   通过 Proxy 调用 HDI 接口访问硬件     │
└─────────────────────────────────────┘
              ↓ IPC
┌─────────────────────────────────────┐
│           HDI 接口层 (本仓库定义)      │
│     IDL → Proxy/Stub 自动生成        │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         驱动实现 (drivers_peripheral │
│      基于生成的接口实现具体功能)          │
└─────────────────────────────────────┘
```

### IDL (Interface Definition Language)

接口定义语言，用于描述跨进程调用的接口规范：

```idl
package ohos.hdi.audio.v1_0;

interface IAudioManager {
    GetAllAdapters([out] struct AudioAdapterDescriptor[] descs);
    LoadAdapter([in] struct AudioAdapterDescriptor desc, [out] IAudioAdapter adapter);
}
```

### HCS (HDF Configuration Source)

HDF 配置源，用于声明驱动服务和安全配置：

```hcs
audioHost :: host {
    hostName = "audioHost";
    priority = 50;
    audioDevice :: device {
        device0 :: deviceNode {
            policy = 2;
            moduleName = "libaudio_driver.z.so";
            serviceName = "audio_service";
        }
    }
}
```

## 版本与演进

### 当前状态
- **仓库版本**: 3.2+ (基于 Audio 模块 bundle.json)
- **模块数量**: 47+ 硬件模块
- **版本迭代**: 多个模块支持 v1_0 → v6_0 演进

### 版本兼容性规则

| 变更类型 | 版本影响 | 兼容性 |
|---------|---------|--------|
| 新增接口 | Minor | 兼容旧版本 |
| 新增参数 | Minor | 兼容旧版本 |
| 删除/修改接口 | Major | 不兼容 |
| 枚举新增值 | Minor | 兼容旧版本 |

## 相关仓库

| 仓库 | 职责 |
|-----|------|
| [drivers_interface](https://gitee.com/openharmony/drivers_interface) | HDI 接口定义（本文档对应仓库）|
| [drivers_framework](https://gitee.com/openharmony/drivers_framework) | HDF 核心框架实现 |
| [drivers_adapter](https://gitee.com/openharmony/drivers_adapter) | HDF 适配层 |
| [drivers_peripheral](https://gitee.com/openharmony/drivers_peripheral) | 驱动具体实现 |

## 快速开始

### 新增 HDI 模块流程

1. **创建目录结构**
   ```
   drivers/interface/<module>/v1_0/
   ```

2. **定义 IDL 文件**
   - `IModuleInterface.idl`: 主接口
   - `IModuleCallback.idl`: 回调接口（可选）
   - `ModuleTypes.idl`: 数据类型

3. **编写 BUILD.gn**
   ```gn
   import("//build/config/components/hdi/hdi.gni")
   
   hdi("module") {
     module_name = "module_service"
     sources = [
       "IModuleInterface.idl",
       "IModuleCallback.idl",
       "ModuleTypes.idl",
     ]
     language = "cpp"
     subsystem_name = "hdf"
     part_name = "drivers_interface_module"
   }
   ```

4. **配置 bundle.json**: 声明组件信息和依赖

详见 [HDI 接口定义规范](./03_HDI_IDL_Specification.md)

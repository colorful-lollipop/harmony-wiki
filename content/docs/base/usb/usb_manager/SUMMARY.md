# USB Manager Wiki 导航

## 新人学习路线（推荐阅读顺序）

**目标**：快速理解项目、学习 API 使用、掌握架构设计

```
1. [项目概览](01_Overview.md) (30分钟)
   ↓
   理解：项目定位、核心能力、运行环境

2. [对外接口文档](04_Interface.md) (1小时)
   ↓
   学习：JS API 使用方法、参数说明、示例代码

3. [架构与数据流](02_Architecture.md) (1小时)
   ↓
   理解：三层架构、数据流向、线程模型

4. [目录结构与代码地图](03_CodeMap.md) (30分钟)
   ↓
   掌握：快速定位代码位置、模块职责

5. [构建与产物](07_Build.md) (可选，30分钟)
   ↓
   了解：编译配置、产物安装路径、Feature 开关

6. [内部实现细节](08_Internals.md) (可选，1小时)
   ↓
   深入：核心类设计、资源生命周期、内部 API
```

**预期效果**：
- ✅ 5 分钟理解项目定位
- ✅ 15 分钟找到核心代码位置
- ✅ 30 分钟理解基本架构
- ✅ 2 小时完成 API 学习和代码定位

---

## 安全研究路线（推荐阅读顺序）

**目标**：快速识别攻击面、分析安全风险、定位漏洞点

```
1. [项目概览](01_Overview.md) (15分钟)
   ↓
   了解：功能边界、权限要求、运行环境

2. [攻击面分析](05_AttackSurface.md) (2小时) ⭐ 优先级最高
   ↓
   识别：所有外部输入点、敏感操作、信任边界

3. [架构与数据流](02_Architecture.md) (1小时)
   ↓
   理解：信任域划分、权限检查机制、IPC 流程

4. [安全风险评估](06_SecurityReview.md) (2小时)
   ↓
   分析：具体漏洞点、触发路径、修复建议

5. [对外接口文档](04_Interface.md) (1小时)
   ↓
   研究：API 参数、输入验证逻辑、错误处理

6. [内部实现细节](08_Internals.md) (1小时)
   ↓
   深入：关键函数实现、并发控制、资源管理

7. [构建与产物](07_Build.md) (可选，30分钟)
   ↓
   了解：Feature 开关、安全编译选项、依赖关系
```

**预期效果**：
- ✅ 30 分钟识别所有攻击面
- ✅ 1 小时理解信任边界
- ✅ 3 小时完成风险评估
- ✅ 5 小时定位关键漏洞点

---

## 快速索引

### N-API 接口

| 模块 | JS API | 功能 |
|------|--------|------|
| usb | `getDevices()` | 获取 USB 设备列表 |
| usb | `requestRight()` | 请求设备权限 |
| usb | `hasRight()` | 检查设备权限 |
| usb | `bulkTransfer()` | 批量数据传输 |
| usb | `controlTransfer()` | 控制命令传输 |
| usb | `claimInterface()` | 声明接口 |
| usb | `releaseInterface()` | 释放接口 |
| usbmanager | `setCurrentFunctions()` | 设置 USB Device 功能 |
| usbmanager | `getCurrentFunctions()` | 获取当前 USB 功能 |
| usbmanager | `setPortRoles()` | 设置 USB Port 角色 |
| serial | `getPortList()` | 获取串口列表 |
| serial | `open()` | 打开串口 |
| serial | `read()/write()` | 串口读写 |

### 构建产物

| 产物 | 类型 | 安装路径 |
|------|------|---------|
| libusbservice.z.so | System Ability | system/lib64/ |
| libusb.z.so | N-API | system/lib64/module/ |
| libusbmanager.z.so | N-API | system/lib64/module/ |
| libserial.z.so | N-API | system/lib64/module/usbmanager/ |
| libusbsrv_client.z.so | IPC 客户端 | system/lib64/ |

### 系统配置

| 配置项 | 值 | 说明 |
|--------|---|------|
| SA ID | 4201 | USB Service 系统能力 ID |
| 进程名 | usb_service | USB Service 运行进程 |
| 自动重启 | 是 | 服务崩溃自动重启 |

---

## 全部文档

### P0 文档（必须）

#### 项目概览
- [项目概览](01_Overview.md)
  - 一句话定义
  - 能力边界
  - 运行环境
  - 快速开始示例

#### 攻击面分析
- [攻击面分析](05_AttackSurface.md) ⭐ 安全研究必读
  - 外部输入清单（N-API、IPC、配置、USB 设备）
  - 敏感操作清单（系统调用、特权接口、数据库）
  - 信任边界图（可信域划分）
  - 高风险攻击面总结

#### 架构与数据流
- [架构与数据流](02_Architecture.md)
  - 三层架构图（API 层 → Service 层 → HAL 层）
  - 数据流图（应用 → N-API → IPC → Service → HAL）
  - 线程模型（主线程、工作线程、回调线程）
  - 关键时序（设备枚举、权限申请、数据传输）

### P1 文档（重要）

#### 对外接口文档
- [对外接口文档](04_Interface.md)
  - N-API 清单表（Host/Device/Port/Serial 功能模块）
  - IPC 接口定义（IUsbServer.idl 95 个方法）
  - 配置文件格式（SA Profile、init 配置）
  - API 使用示例

#### 目录结构与代码地图
- [目录结构与代码地图](03_CodeMap.md)
  - 顶层目录职责（interfaces、services、utils、frameworks）
  - 核心文件定位（入口、配置、实现）
  - 代码导航图（功能 → 文件映射）
  - 快速查找指南

#### 安全风险评估
- [安全风险评估](06_SecurityReview.md) ⭐ 安全研究必读
  - 输入验证缺陷（类型/长度/范围/null）
  - 内存安全问题（缓冲区溢出、Use-After-Free）
  - 权限与鉴权（Token 验证、权限检查绕过）
  - 并发安全（竞态条件、TOCTOU）
  - 逻辑漏洞（错误处理、资源耗尽）
  - 每条风险包含：证据、触发路径、影响、修复建议

### P2 文档（可选）

#### 构建与产物
- [构建与产物](07_Build.md)
  - GN targets 清单（N-API、内部库、服务）
  - 编译产物和安装路径（.so、.hap）
  - Feature 开关配置（Host/Device/Port 功能）
  - 安全编译选项（Sanitizer、CFI）

#### 内部实现细节
- [内部实现细节](08_Internals.md)
  - 核心类/结构体职责（UsbService、UsbHostManager 等）
  - 内部 API 契约（稳定接口 vs 内部实现）
  - 资源生命周期管理（创建、使用、释放）
  - 关键函数实现细节

### 附录

- [关键调用链](appendix/callgraphs.md)
  - 设备枚举调用链
  - 权限请求调用链
  - 数据传输调用链

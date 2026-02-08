# Wiki 导航与阅读指南

本文档提供 OpenHarmony 指纹认证组件的完整技术文档导航。

---

## 文档结构

### 基础文档

#### 1. [01_Overview.md](./01_Overview.md)
**目的**：介绍指纹认证组件的全貌和核心概念
**适用范围**：所有读者
**关键结论**：
- 本组件是 UserIAM 框架下的执行器实现，通过 HDI 与硬件驱动通信
- 提供 System Ability 943 服务，运行在 useriam 进程
- 核心能力：指纹录入、认证、识别、删除
- **重要**：本组件无 N-API 绑定，JS API 在上层 `user_auth_framework` 中

**相关跳转**：[02_Directory_Structure.md](./02_Directory_Structure.md) | [03_Architecture.md](./03_Architecture.md)

---

#### 2. [02_Directory_Structure.md](./02_Directory_Structure.md)
**目的**：说明代码库的目录组织和模块职责
**适用范围**：需要理解代码组织的开发者
**关键结论**：
- `common/`：公共基础设施（日志、工具类、错误码定义）
- `services/`：核心 System Ability 服务（SA 943）
- `services_ex/`：扩展服务（传感器照明 UI）
- `sa_profile/`：System Ability 配置文件

**相关跳转**：[01_Overview.md](./01_Overview.md) | [03_Architecture.md](./03_Architecture.md)

---

#### 3. [03_Architecture.md](./03_Architecture.md)
**目的**：详细说明组件架构、通信模式、线程模型
**适用范围**：架构师、开发者
**关键结论**：
- 分层架构：UserAuth Framework → FingerprintAuthService → HDI → Hardware Driver
- 通信模式：
  - 下行：Framework → HDI Proxy → Driver（调用操作）
  - 上行：Driver → Callback → Framework（返回结果）
  - 命令：Driver → SA Command → Manager（驱动发起命令）
- 线程模型：主要运行在 SA 主线程，HDI 回调通过 HDF 线程池

**相关跳转**：[04_HDI_Interfaces.md](./04_HDI_Interfaces.md) | [05_Internal_APIs.md](./05_Internal_APIs.md)

---

#### 4. [04_HDI_Interfaces.md](./04_HDI_Interfaces.md)
**目的**：列出所有硬件驱动接口（HDI）定义
**适用范围**：驱动开发者、框架对接开发者
**关键结论**：
- HDI 版本：V2.0
- 主要接口：
  - `IFingerprintAuthInterface`：获取执行器列表
  - `IAllInOneExecutor`：执行器操作（Enroll/Authenticate/Delete 等）
  - `IExecutorCallback`：执行器回调
  - `ISaCommandCallback`：SA 命令回调
- 关键数据类型：`ExecutorInfo`、`Property`、`SaCommand`

**相关跳转**：[03_Architecture.md](./03_Architecture.md) | [06_GN_Targets.md](./06_GN_Targets.md)

---

#### 5. [05_Internal_APIs.md](./05_Internal_APIs.md)
**目的**：说明模块内部接口和依赖关系
**适用范围**：组件维护者、开发者
**关键结论**：
- `FingerprintAuthService`：主服务类，单例模式
- `FingerprintAuthDriverHdi`：HDI 驱动适配器
- `FingerprintAllInOneExecutorHdi`：All-in-One 执行器实现
- `SaCommandManager`：SA 命令管理器（支持传感器照明等扩展命令）
- `SensorIlluminationManager`：传感器照明协调器

**相关跳转**：[03_Architecture.md](./03_Architecture.md) | [02_Directory_Structure.md](./02_Directory_Structure.md)

---

### 构建文档

#### 6. [06_GN_Targets.md](./06_GN_Targets.md)
**目的**：说明 GN 构建系统配置
**适用范围**：构建系统维护者、开发者
**关键结论**：
- 主要产物：
  - `libfingerprintauthservice.z.so`：核心服务库
  - `libfingerprintauthservice_ex.z.so`：扩展服务库
- 特性开关：
  - `use_display_manager_component`：显示管理器集成
  - `use_power_manager_component`：电源管理器集成
- 安全特性：启用 CFI、UBSAN、边界检查、分支保护

**相关跳转**：[07_Build_Artifacts.md](./07_Build_Artifacts.md) | [02_Directory_Structure.md](./02_Directory_Structure.md)

---

#### 7. [07_Build_Artifacts.md](./07_Build_Artifacts.md)
**目的**：列出编译产物和运行时加载关系
**适用范围**：系统集成者、构建工程师
**关键结论**：
- 产物清单：
  - `libfingerprintauthservice.z.so` → `/usr/lib/`（系统库目录）
  - `libfingerprintauthservice_ex.z.so` → `/usr/lib/`
  - `943.json`（SA 配置）→ 系统 SA 配置目录
- 加载流程：
  1. SA 943 配置文件被 samgr 读取
  2. `libfingerprintauthservice.z.so` 被加载到 useriam 进程
  3. `FingerprintAuthService::OnStart()` 执行
  4. HDI 接口初始化

**相关跳转**：[06_GN_Targets.md](./06_GN_Targets.md) | [03_Architecture.md](./03_Architecture.md)

---

### 安全与运维文档

#### 8. [08_Security_Analysis.md](./08_Security_Analysis.md)
**目的**：安全风险分析和评审
**适用范围**：安全审核员、架构师
**关键结论**：
- **重要**：本组件不执行权限检查，权限校验在 `user_auth_framework` 层
- 攻击面：HDI 接口、SA 命令、传感器照明参数
- 已识别风险：
  - 传感器照明参数未充分验证（坐标、半径）
  - 动态库加载（service_ex）存在路径遍历风险
  - 缺少调用者身份验证（依赖上层）
- 信任边界：UserAuth Framework ↔ FingerprintAuthService ↔ HDI Driver

**相关跳转**：[04_HDI_Interfaces.md](./04_HDI_Interfaces.md) | [03_Architecture.md](./03_Architecture.md)

---

#### 9. [09_Troubleshooting.md](./09_Troubleshooting.md)
**目的**：常见问题排查指南
**适用范围**：运维、开发者、QA
**关键结论**：
- 构建问题：依赖缺失、特性开关配置错误
- 运行时问题：HDI 驱动未加载、SA 注册失败、超时
- 调试工具：Hilog 日志、SA dump、HDI 接口测试
- 定位路径：基于日志标签和错误码定位问题

**相关跳转**：[07_Build_Artifacts.md](./07_Build_Artifacts.md) | [06_GN_Targets.md](./06_GN_Targets.md)

---

## 阅读路径推荐

### 新手入门（1-2 小时）
1. [01_Overview.md](./01_Overview.md) - 了解组件全貌（15 分钟）
2. [02_Directory_Structure.md](./02_Directory_Structure.md) - 熟悉代码组织（15 分钟）
3. [03_Architecture.md](./03_Architecture.md) - 理解架构设计（30 分钟）
4. [09_Troubleshooting.md](./09_Troubleshooting.md) - 了解常见问题（15 分钟）

### 开发者深入（2-3 小时）
1. 完成新手入门路径
2. [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) - 掌握 HDI 接口（45 分钟）
3. [05_Internal_APIs.md](./05_Internal_APIs.md) - 理解模块接口（30 分钟）
4. [06_GN_Targets.md](./06_GN_Targets.md) - 了解构建系统（30 分钟）
5. 阅读源代码，对照文档验证理解（1 小时）

### 安全审核（1-2 小时）
1. [01_Overview.md](./01_Overview.md) - 快速了解组件（15 分钟）
2. [03_Architecture.md](./03_Architecture.md) - 理解信任边界（30 分钟）
3. [04_HDI_Interfaces.md](./04_HDI_Interfaces.md) - 识别攻击面（15 分钟）
4. [08_Security_Analysis.md](./08_Security_Analysis.md) - 详细安全分析（30 分钟）
5. [09_Troubleshooting.md](./09_Troubleshooting.md) - 了解安全相关问题（15 分钟）

---

## 重要提示

1. **无 N-API**：本组件是原生 C++ System Ability，不提供 JavaScript 接口。JS API 请参考 `useriam_user_auth_framework`。

2. **权限检查在上层**：本组件不执行权限验证，所有权限校验在 `user_auth_framework` 完成。

3. **HDI 依赖**：本组件依赖 `drivers_interface_fingerprint_auth` 定义的 HDI 接口，南向驱动需实现这些接口。

4. **扩展机制**：通过 `SaCommandManager` 和动态库加载机制支持扩展功能（如传感器照明）。

5. **安全编码实践**：启用了大量安全特性（CFI、UBSAN、边界检查），开发时需注意不要禁用这些保护。

---

## 术语表

| 术语 | 英文 | 说明 |
|--------|------|------|
| 系统能力 | System Ability (SA) | OpenHarmony 的系统服务机制，每个 SA 有唯一 ID |
| 硬件驱动接口 | Hardware Driver Interface (HDI) | OpenHarmony 的硬件抽象层接口 |
| 执行器 | Executor | 用户认证框架中的认证执行单元 |
| 用户身份认证 | User Identity and Access Management (UserIAM) | OpenHarmony 用户身份认证子系统 |
| 控制流完整性 | Control Flow Integrity (CFI) | 防止代码重定向攻击的安全机制 |
| 未定义行为检测器 | Undefined Behavior Sanitizer (UBSAN) | 检测 C++ 未定义行为的安全工具 |

---

## 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| 1.0 | 2026-02-06 | 初始版本，基于代码库生成完整文档 |
| 1.1 | 2026-02-07 | 补充安全风险分析、构建产物、故障排除文档 |

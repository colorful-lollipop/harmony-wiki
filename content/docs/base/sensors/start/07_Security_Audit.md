# 安全风险评审

**适用范围**: 本文档适用于所有需要了解安全风险和攻击面的人员
**目的**: 分析攻击面、信任边界、可被利用点、修复建议
**关键结论**: msdp 服务拥有大量敏感权限，存在潜在的安全风险

---

## 执行摘要

### 检查范围

本安全评审覆盖:
- ✅ INIT 系统配置文件 (`.cfg`)
- ✅ 服务运行时权限配置
- ✅ 目录创建和所有者设置
- ✅ 服务启动路径和参数

### 评审结论

**总体评估**: ⚠️ **中等风险**

**主要风险点**:
1. msdp 服务拥有大量敏感权限（26 个权限，含 8 个 ACL 权限）
2. msdp 可访问系统关键资源（输入、相机、位置、屏幕等）
3. 配置文件可能被篡改（需系统完整性保护）

**缓解措施**:
- 依赖 OpenHarmony 权限系统和安全机制
- 配置文件由系统镜像保护
- 进程隔离（UID/GID 分离）

---

## 攻击面分析

### 攻击面清单

| 攻击面 | 描述 | 风险等级 | 证据 |
|--------|------|----------|------|
| msdp 服务权限 | msdp 拥有 26 个敏感权限，可访问输入、相机、位置、屏幕等 | 🔴 高 | [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16) |
| msdp ACL 权限 | 8 个敏感 ACL 权限，包括输入监听、屏幕截图等 | 🔴 高 | [etc/init/msdp.cfg:44-52](../etc/init/msdp.cfg:44) |
| sensors 服务权限 | sensors 拥有 2 个权限，相对较少 | 🟢 低 | [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15) |
| 配置文件篡改 | /etc/init/*.rc 可能被篡改（需系统完整性保护） | 🟡 中 | [06_Build_Artifacts.md](./06_Build_Artifacts.md) |
| 目录创建权限 | 创建数据目录时可能被利用 | 🟢 低 | [etc/init/sensors.cfg:5](../etc/init/sensors.cfg:5) |

### 攻击面优先级

**高优先级** (🔴):
- msdp 服务权限滥用
- msdp ACL 权限滥用

**中优先级** (🟡):
- 配置文件篡改

**低优先级** (🟢):
- sensors 服务权限
- 目录创建权限

---

## 权限配置详情

### msdp 服务权限清单

#### 普通权限 (18 个)

| 权限名称 | 风险等级 | 说明 |
|----------|----------|------|
| `ohos.permission.ACCELEROMETER` | 🟡 中 | 加速度传感器数据 |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 🟡 中 | 分布式数据同步 |
| `ohos.permission.MICROPHONE` | 🔴 高 | 麦克风录音 |
| `ohos.permission.LOCATION` | 🔴 高 | 位置信息 |
| `ohos.permission.APPROXIMATELY_LOCATION` | 🟡 中 | 大致位置 |
| `ohos.permission.ACCESS_SERVICE_DM` | 🟡 中 | 访问设备管理服务 |
| `ohos.permission.CAMERA` | 🔴 高 | 摄像头 |
| `ohos.permission.READ_HEALTH_DATA` | 🟡 中 | 读取健康数据 |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 🟡 中 | 访问分布式硬件 |
| `ohos.permission.RUNNING_STATE_OBSERVER` | 🟡 中 | 运行状态观察者 |
| `ohos.permission.VIBRATE` | 🟢 低 | 震动器 |
| `ohos.permission.GET_BUNDLE_INFO` | 🟢 低 | 获取应用信息 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 🟡 中 | 获取应用信息(特权) |
| `ohos.permission.GET_RUNNING_INFO` | 🟡 中 | 获取运行信息 |
| `ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS` | 🟡 中 | 管理分布式账户 |
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 🟡 中 | 管理本地账户 |
| `ohos.permission.MANAGE_SETTINGS` | 🟡 中 | 管理系统设置 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 🟡 中 | 后台启动能力 |

**证据**: [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16)

#### ACL 权限 (8 个)

**ACL (Access Control List) 权限** 是需要用户显式授权的敏感权限，风险等级更高。

| ACL 权限名称 | 风险等级 | 潜在危害 |
|-------------|----------|----------|
| `ohos.permission.INPUT_MONITORING` | 🔴 高 | 监听所有输入事件（键盘、触摸等） |
| `ohos.permission.INJECT_INPUT_EVENT` | 🔴 高 | 注入虚假输入事件 |
| `ohos.permission.INTERCEPT_INPUT_EVENT` | 🔴 高 | 拦截和阻止输入事件 |
| `ohos.permission.FILTER_INPUT_EVENT` | 🟡 中 | 过滤输入事件 |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 🟡 中 | 访问分布式硬件设备 |
| `ohos.permission.MONITOR_DEVICE_NETWORK_STATE` | 🟡 中 | 监控设备网络状态 |
| `ohos.permission.MANAGE_MOUSE_CURSOR` | 🟡 中 | 管理鼠标光标 |
| `ohos.permission.CAPTURE_SCREEN` | 🔴 高 | 屏幕截图 |

**证据**: [etc/init/msdp.cfg:44-52](../etc/init/msdp.cfg:44)

**攻击场景示例**:
1. **INPUT_MONITORING + INJECT_INPUT_EVENT**: 监听用户输入并注入恶意操作
2. **CAPTURE_SCREEN + INPUT_MONITORING**: 记录用户操作和屏幕内容（类似键盘记录器）
3. **INTERCEPT_INPUT_EVENT**: 拦截并阻止用户的解锁操作
4. **CAMERA + MICROPHONE + LOCATION**: 监控用户行为和环境

### sensors 服务权限清单

| 权限名称 | 风险等级 | 说明 |
|----------|----------|------|
| `ohos.permission.PERMISSION_USED_STATS` | 🟢 低 | 权限使用统计 |
| `ohos.permission.GET_SENSITIVE_PERMISSIONS` | 🟡 中 | 获取敏感权限信息 |

**证据**: [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15)

**评估**: sensors 服务权限相对较少，风险较低

---

## 特别关注点: musl 版本权限差异

### 🔴 重要发现: musl 版本权限更宽松

**对比分析**:

| 服务 | 非 musl 权限 | musl 权限 | 增长率 |
|------|-------------|----------|--------|
| **sensors** | 2 个 | 9 个 | +350% |
| **msdp** | 26 个 | 36 个 | +38% |
| **msdp ACL** | 8 个 | 12 个 | +50% |

### musl 新增的高风险权限

#### sensors musl 新增权限

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `MANAGE_SECURE_SETTINGS` | 🔴 高 | 管理安全设置 |
| `ACCESS_SECURITY_PRIVACY_CENTER` | 🔴 高 | 访问安全隐私中心 |
| `RECEIVER_STARTUP_COMPLETED` | 🟡 中 | 接收启动完成广播 |

**证据**: 
- 非 musl: [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15) (2 个权限)
- musl: [etc/init/sensors_musl.cfg:16-26](../etc/init/sensors_musl.cfg:16) (9 个权限)

#### msdp musl 新增权限

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ACCESSIBILITY_EXTENSION_ABILITY` | 🔴 高 | 无障碍扩展能力（可监控所有界面） |
| `QUERY_ACCESSIBILITY_ELEMENT` | 🔴 高 | 查询无障碍元素（ACL 权限） |
| `POWER_OPTIMIZATION` | 🟡 中 | 电源优化管理 |
| `ACTIVITY_MOTION` | 🟡 中 | 活动运动检测 |

**证据**:
- 非 musl: [etc/init/msdp.cfg:16-53](../etc/init/msdp.cfg:16) (26 个权限)
- musl: [etc/init/msdp_musl.cfg:17-56](../etc/init/msdp_musl.cfg:17) (36 个权限)

### SELinux 上下文 (musl 特有)

musl 版本增加了 SELinux 安全上下文:

| 服务 | SELinux 上下文 |
|------|---------------|
| sensors | `u:r:sensors:s0` |
| msdp | `u:r:msdp_sa:s0` |

**证据**:
- [etc/init/sensors_musl.cfg:15](../etc/init/sensors_musl.cfg:15)
- [etc/init/msdp_musl.cfg:16](../etc/init/msdp_musl.cfg:16)

### 安全建议

1. **统一权限策略**: 评估 musl 版本额外权限是否必要，考虑统一权限配置
2. **风险评估**: musl 版本增加的无障碍权限 (`ACCESSIBILITY_EXTENSION_ABILITY`) 可监控所有界面，风险极高
3. **使用场景审查**: 确认 musl 版本的使用场景，避免在非必要情况下使用高权限配置

---

## 可被利用点

### 🔴 高风险可被利用点

#### 1. msdp 输入监听和注入

**证据**: [etc/init/msdp.cfg:45-46, 48](../etc/init/msdp.cfg:45)

**风险**: msdp 服务同时拥有 `INPUT_MONITORING` 和 `INJECT_INPUT_EVENT` 权限

**触发条件**: msdp 服务代码存在漏洞或被恶意修改

**影响**:
- 监听用户所有输入（密码、PIN、触摸、键盘）
- 注入虚假输入（模拟用户操作）
- 可能绕过安全锁、执行恶意操作

**攻击链**:
```
攻击者控制 msdp 进程
    ↓
监听用户输入事件 (INPUT_MONITORING)
    ↓
获取用户密码或 PIN
    ↓
注入虚假输入事件 (INJECT_INPUT_EVENT)
    ↓
解锁设备 / 执行恶意操作
```

**修复建议**:
1. ✅ **权限分离**: 不要将 `INPUT_MONITORING` 和 `INJECT_INPUT_EVENT` 同时授予同一服务
2. ✅ **最小权限原则**: 仅授予 msdp 服务实际需要的权限
3. ✅ **审计日志**: 记录所有输入事件监听和注入操作
4. ✅ **用户授权**: 关键操作需要用户显式确认
5. ✅ **代码审查**: 严格审计 msdp 服务实现代码

#### 2. msdp 屏幕截图

**证据**: [etc/init/msdp.cfg:52](../etc/init/msdp.cfg:52)

**风险**: msdp 服务拥有 `CAPTURE_SCREEN` 权限

**触发条件**: msdp 服务代码存在漏洞或被恶意修改

**影响**:
- 截取当前屏幕内容
- 可能泄露用户隐私信息
- 配合输入监听实现完整的键盘记录器

**攻击链**:
```
攻击者控制 msdp 进程
    ↓
截取屏幕 (CAPTURE_SCREEN)
    ↓
结合输入监听 (INPUT_MONITORING)
    ↓
记录用户操作和屏幕内容
    ↓
泄露敏感信息
```

**修复建议**:
1. ✅ **用户授权**: 屏幕截图需要用户显式授权
2. ✅ **使用限制**: 限制截图频率和场景
3. ✅ **水印提示**: 截图时显示水印或提示
4. ✅ **审计日志**: 记录所有截图操作

#### 3. msdp 摄像头和麦克风

**证据**: [etc/init/msdp.cfg:25, 19](../etc/init/msdp.cfg:25)

**风险**: msdp 服务同时拥有 `CAMERA` 和 `MICROPHONE` 权限

**触发条件**: msdp 服务代码存在漏洞或被恶意修改

**影响**:
- 静默录制音频和视频
- 侵犯用户隐私
- 监控用户行为

**修复建议**:
1. ✅ **用户授权**: 摄像头和麦克风使用需要用户显式授权
2. ✅ **硬件指示灯**: 确保摄像头和麦克风使用时有硬件指示灯或系统提示
3. ✅ **限制后台使用**: 禁止后台录制
4. ✅ **审计日志**: 记录所有摄像头和麦克风使用

### 🟡 中风险可被利用点

#### 4. 配置文件篡改

**证据**: [06_Build_Artifacts.md](./06_Build_Artifacts.md)

**风险**: `/etc/init/sensors.rc` 和 `/etc/init/msdp.rc` 可能被篡改

**触发条件**: 系统完整性保护被绕过或系统被 root

**影响**:
- 修改服务启动路径
- 修改服务 UID/GID
- 修改服务权限
- 导致服务启动失败或执行恶意代码

**修复建议**:
1. ✅ **系统完整性保护**: 确保 /etc/init/ 目录受系统完整性保护
2. ✅ **只读挂载**: /etc/init/ 目录应挂载为只读（除启动时）
3. ✅ **签名验证**: 配置文件应验证数字签名
4. ✅ **安全启动**: 启用安全启动（Secure Boot）
5. ✅ **日志监控**: 监控配置文件的修改尝试

#### 5. msdp 权限过多

**证据**: [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16)

**风险**: msdp 服务拥有 26 个权限，权限范围过大

**触发条件**: msdp 服务代码存在漏洞或被恶意修改

**影响**:
- 攻击面过大
- 任何漏洞都可能被利用获取大量权限
- 增加攻击成功概率

**修复建议**:
1. ✅ **权限拆分**: 将 msdp 服务拆分为多个服务，每个服务仅授予必要权限
2. ✅ **最小权限原则**: 仅授予 msdp 服务实际需要的权限
3. ✅ **定期审计**: 定期审查 msdp 服务的权限使用情况

---

## 信任边界

### 进程隔离

```
┌─────────────────────────────────────────────┐
│          root 用户空间                        │
│  ├─ init 进程 (PID: 1)                      │
│  ├─ /etc/init/sensors.rc (只读配置)         │
│  └─ /etc/init/msdp.rc (只读配置)            │
└─────────────────────────────────────────────┘
              ↓ 启动服务
┌─────────────────────────────────────────────┐
│          sensor 用户空间                     │
│  └─ sensors 进程 (UID: sensor)              │
│      ├─ SA 3601 (传感器服务)                │
│      └─ SA 3602 (震动器服务)                │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│          msdp 用户空间                       │
│  └─ msdp 进程 (UID: msdp)                   │
│      └─ msdp SA                             │
└─────────────────────────────────────────────┘
```

**信任边界**:
- root → sensor/msdp: UID 隔离，权限降级
- sensor ↔ msdp: 不同 UID，互相隔离
- 服务进程之间: 通过 SA 框架通信，受权限控制

### 数据流信任边界

```
用户输入 (键盘/触摸)
    ↓
[传感器数据流]
    ↓
sensors 进程 (受权限控制)
    ↓
[应用通过 JS API 访问]
    ↓
应用 (受权限系统控制)

[输入事件流]
    ↓
msdp 进程 (拥有 INPUT_MONITORING)
    ↓
[可能泄露输入数据] ← 🔴 风险点
```

---

## 数据流安全分析

### sensors 服务数据流

```
传感器硬件
    ↓
sensors 进程 (UID: sensor)
    ↓
SA 框架
    ↓
权限系统检查
    ↓
应用 (JS API)
```

**安全评估**:
- ✅ 进程隔离：sensors 进程以 sensor UID 运行
- ✅ 权限控制：应用需要声明传感器权限
- ✅ SA 框架：通过 SA 框架通信，受权限控制
- 🟢 低风险

### msdp 服务数据流

```
输入事件 (键盘/触摸)
    ↓
msdp 进程 (UID: msdp)
    ↓ [INPUT_MONITORING]
    ↓ [可能监听/注入/拦截] ← 🔴 风险点
    ↓
SA 框架
    ↓
应用或其他组件
```

**安全评估**:
- ⚠️ 权限过大：msdp 拥有大量敏感权限
- ⚠️ 输入监听：可监听所有输入事件
- ⚠️ 输入注入：可注入虚假输入
- 🔴 高风险

---

## 缓解措施

### 已有缓解措施

1. **进程隔离**: 服务以非 root UID 运行
   - sensors: UID: sensor
   - msdp: UID: msdp
   - 证据: [etc/init/sensors.cfg:13](../etc/init/sensors.cfg:13), [etc/init/msdp.cfg:14](../etc/init/msdp.cfg:14)

2. **权限系统**: OpenHarmony 权限系统控制服务能力
   - 证据: [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15)

3. **系统完整性保护**: /etc/init/ 目录应受系统完整性保护

### 建议缓解措施

#### 短期措施（1-3 个月）

1. **审计 msdp 权限**:
   - 审查 msdp 服务实际需要的权限
   - 移除不必要的权限
   - 证据: [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16)

2. **添加审计日志**:
   - 记录 msdp 的输入监听、注入、截图操作
   - 记录摄像头和麦克风使用
   - 审计异常权限使用

3. **用户授权提示**:
   - 屏幕截图时显示提示
   - 摄像头和麦克风使用时显示硬件指示灯或系统提示

#### 中期措施（3-6 个月）

1. **权限拆分**:
   - 将 msdp 服务拆分为多个服务
   - 每个服务仅授予必要权限
   - 例如：输入服务独立、摄像头服务独立

2. **配置文件签名**:
   - 对 /etc/init/*.rc 进行数字签名
   - 启动时验证签名
   - 防止配置文件篡改

3. **增强隔离**:
   - 考虑使用沙箱或容器进一步隔离 msdp 服务
   - 限制 msdp 服务的系统访问

#### 长期措施（6-12 个月）

1. **重新设计架构**:
   - 重新评估 msdp 服务的架构设计
   - 考虑使用更细粒度的权限模型
   - 减少对超级权限的依赖

2. **安全启动**:
   - 启用安全启动（Secure Boot）
   - 确保系统镜像完整性
   - 防止篡改

3. **形式化验证**:
   - 对关键服务进行形式化验证
   - 确保服务行为符合预期

---

## 未发现的攻击面

### 检查范围

✅ **已检查**:
- INIT 配置文件 (.cfg)
- 服务权限配置
- 目录创建和所有者设置
- 服务启动路径和参数

### 局限性

❌ **未检查** (超出本仓库范围):
- msdp 服务实现代码（在外部仓库）
- sensors 服务实现代码（在外部仓库）
- SA 配置文件（sensors.json, msdp.json）
- 动态库实现（libsensor_service.z.so, libmiscdevice_service.z.so）
- OpenHarmony 权限系统和 SA 框架实现
- 系统完整性保护机制

### 说明

本安全评审**仅覆盖 sensors_start 组件的配置文件**，不涉及:
- 服务实现代码的漏洞
- SA 框架的漏洞
- OpenHarmony 系统级别的漏洞

完整的安全评估需要结合以下内容:
- sensors_sensor 仓库代码审计
- sensors_miscdevice 仓库代码审计
- msdp 相关仓库代码审计
- OpenHarmony 系统安全评估

---

## 合规性检查

### 权限最小化原则

**sensors 服务**: ✅ 符合
- 权限数量少（2 个）
- 权限与功能匹配

**msdp 服务**: ⚠️ 不符合
- 权限数量过多（26 个）
- 存在不必要权限
- 权限范围过大

### 职责分离原则

**sensors 服务**: ✅ 符合
- 单一职责：传感器数据处理

**msdp 服务**: ⚠️ 部分不符合
- 职责过多：输入、传感器、相机、麦克风、位置等多模态处理
- 建议拆分为多个服务

### 默认拒绝原则

**sensors 服务**: ✅ 符合
- 仅授予明确需要的权限

**msdp 服务**: ⚠️ 不符合
- 授予过多权限，不符合默认拒绝原则

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位
- [架构说明](./02_Architecture.md) - 服务启动流程
- [编译产物](./06_Build_Artifacts.md) - 运行时加载
- [配置文件详解](./appendix/Config_Files.md) - 配置参数完整说明

---

**最后更新**: 2026-02-06

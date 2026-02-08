# 配置文件详解

**适用范围**: 本文档适用于所有需要详细理解配置文件参数的人员
**目的**: 说明配置文件参数、含义、影响范围
**关键结论**: 两个配置文件定义了 sensors 和 msdp 服务的启动参数和权限

---

## sensors.cfg 详解

### 文件位置

**路径**: `/etc/init/sensors.cfg` 或 `/etc/init/sensors_musl.cfg`
**类型**: JSON 格式
**大小**: ~1 KB

### 完整配置

```json
{
    "jobs" : [{
            "name" : "boot",
            "cmds" : [
                "mkdir /data/service/el1/public/sensor",
                "chown sensor sensor /data/service/el1/public/sensor"
            ]
        }
    ],
    "services" : [{
            "name" : "sensors",
            "path" : ["/system/bin/sa_main", "/system/profile/sensors.json"],
            "uid" : "sensor",
            "gid" : ["sensor", "shell"],
            "permission" : [
                "ohos.permission.PERMISSION_USED_STATS",
                "ohos.permission.GET_SENSITIVE_PERMISSIONS"
            ],
            "permission_acls" : [
                "ohos.permission.GET_SENSITIVE_PERMISSIONS"
            ]
        }
    ]
}
```

**完整代码**: [etc/init/sensors.cfg:1-24](../etc/init/sensors.cfg:1)

### 配置参数详解

#### Jobs (任务)

**作用**: 定义在特定时机执行的任务

| 参数 | 值 | 说明 |
|------|-----|------|
| name | boot | 任务名称，在 boot 阶段执行 |
| cmds | 命令列表 | 要执行的命令列表 |

**命令详解**:

##### mkdir

**命令**: `mkdir /data/service/el1/public/sensor`
**作用**: 创建传感器服务的数据目录
**时机**: boot 阶段
**证据**: [etc/init/sensors.cfg:5](../etc/init/sensors.cfg:5)

**路径说明**:
- `/data`: 用户数据目录
- `/service`: 服务数据
- `/el1`: 加密级别 1 (Encryption Level 1)
- `/public`: 公共可访问
- `/sensor`: 传感器服务专用目录

##### chown

**命令**: `chown sensor sensor /data/service/el1/public/sensor`
**作用**: 设置目录所有者和组
**时机**: boot 阶段，在 mkdir 之后
**证据**: [etc/init/sensors.cfg:6](../etc/init/sensors.cfg:6)

**所有者**: sensor:sensor

#### Services (服务)

**作用**: 定义系统服务

##### name

**值**: `sensors`
**说明**: 服务名称，用于标识和管理服务
**证据**: [etc/init/sensors.cfg:11](../etc/init/sensors.cfg:11)

##### path

**值**: `["/system/bin/sa_main", "/system/profile/sensors.json"]`
**说明**: 服务启动路径和参数
**证据**: [etc/init/sensors.cfg:12](../etc/init/sensors.cfg:12)

**参数说明**:
1. `/system/bin/sa_main`: SA 进程管理器可执行文件
2. `/system/profile/sensors.json`: SA 配置文件

**启动方式**:
```bash
/system/bin/sa_main /system/profile/sensors.json
```

##### uid

**值**: `sensor`
**说明**: 服务运行的用户 ID
**证据**: [etc/init/sensors.cfg:13](../etc/init/sensors.cfg:13)

**说明**:
- 服务以 `sensor` 用户身份运行
- 非 root 用户，权限受限
- 需要系统预先创建 sensor 用户

##### gid

**值**: `["sensor", "shell"]`
**说明**: 服务运行的组 ID 列表
**证据**: [etc/init/sensors.cfg:14](../etc/init/sensors.cfg:14)

**组列表**:
1. `sensor`: 传感器服务主组
2. `shell`: Shell 组（用于访问某些 shell 功能）

##### permission

**值**: 服务权限列表
**证据**: [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.PERMISSION_USED_STATS` | 🟢 低 | 权限使用统计 |
| `ohos.permission.GET_SENSITIVE_PERMISSIONS` | 🟡 中 | 获取敏感权限信息 |

**说明**: sensors 服务权限相对较少，风险较低

##### permission_acls

**值**: ACL 权限列表
**证据**: [etc/init/sensors.cfg:19-21](../etc/init/sensors.cfg:19)

| ACL 权限 | 风险等级 | 说明 |
|----------|----------|------|
| `ohos.permission.GET_SENSITIVE_PERMISSIONS` | 🟡 中 | 获取敏感权限信息 |

**说明**: ACL 权限是敏感权限，需要用户授权

---

## msdp.cfg 详解

### 文件位置

**路径**: `/etc/init/msdp.rc` 或 `/etc/init/msdp_musl.cfg`
**类型**: JSON 格式
**大小**: ~2.5 KB

### 完整配置

```json
{
    "jobs" : [{
            "name" : "boot",
            "cmds" : [
                "mkdir /data/service/el1/public/msdp",
                "chown msdp msdp /data/service/el1/public/msdp",
                "start msdp"
            ]
        }
    ],
    "services" : [{
            "name" : "msdp",
            "path" : ["/system/bin/sa_main", "/system/profile/msdp.json"],
            "uid" : "msdp",
            "gid" : ["msdp", "shell", "input", "access_token"],
            "permission" : [
                "ohos.permission.ACCELEROMETER",
                "ohos.permission.DISTRIBUTED_DATASYNC",
                "ohos.permission.MICROPHONE",
                "ohos.permission.INPUT_MONITORING",
                "ohos.permission.INJECT_INPUT_EVENT",
                "ohos.permission.LOCATION",
                "ohos.permission.APPROXIMATELY_LOCATION",
                "ohos.permission.ACCESS_SERVICE_DM",
                "ohos.permission.CAMERA",
                "ohos.permission.READ_HEALTH_DATA",
                "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
                "ohos.permission.INTERCEPT_INPUT_EVENT",
                "ohos.permission.MONITOR_DEVICE_NETWORK_STATE",
                "ohos.permission.RUNNING_STATE_OBSERVER",
                "ohos.permission.FILTER_INPUT_EVENT",
                "ohos.permission.MANAGE_MOUSE_CURSOR",
                "ohos.permission.GET_RUNNING_INFO",
                "ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS",
                "ohos.permission.GET_TELEPHONY_STATE",
                "ohos.permission.GET_BUNDLE_INFO",
                "ohos.permission.MANAGE_LOCAL_ACCOUNTS",
                "ohos.permission.CAPTURE_SCREEN",
                "ohos.permission.VIBRATE",
                "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
                "ohos.permission.MANAGE_SETTINGS",
                "ohos.permission.START_ABILITIES_FROM_BACKGROUND"
            ],
            "permission_acls" : [
                "ohos.permission.INPUT_MONITORING",
                "ohos.permission.INJECT_INPUT_EVENT",
                "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE",
                "ohos.permission.INTERCEPT_INPUT_EVENT",
                "ohos.permission.MONITOR_DEVICE_NETWORK_STATE",
                "ohos.permission.FILTER_INPUT_EVENT",
                "ohos.permission.MANAGE_MOUSE_CURSOR",
                "ohos.permission.CAPTURE_SCREEN"
            ]
        }
    ]
}
```

**完整代码**: [etc/init/msdp.cfg:1-57](../etc/init/msdp.cfg:1)

### 配置参数详解

#### Jobs (任务)

**作用**: 定义在特定时机执行的任务

##### mkdir

**命令**: `mkdir /data/service/el1/public/msdp`
**作用**: 创建 msdp 服务的数据目录
**时机**: boot 阶段
**证据**: [etc/init/msdp.cfg:5](../etc/init/msdp.cfg:5)

##### chown

**命令**: `chown msdp msdp /data/service/el1/public/msdp`
**作用**: 设置目录所有者和组
**时机**: boot 阶段，在 mkdir 之后
**证据**: [etc/init/msdp.cfg:6](../etc/init/msdp.cfg:6)

**所有者**: msdp:msdp

##### start msdp

**命令**: `start msdp`
**作用**: 显式启动 msdp 服务
**时机**: boot 阶段，在目录创建之后
**证据**: [etc/init/msdp.cfg:7](../etc/init/msdp.cfg:7)

**说明**: 与 sensors 服务不同，msdp 服务在 boot job 中显式启动

#### Services (服务)

##### name

**值**: `msdp`
**说明**: 服务名称
**证据**: [etc/init/msdp.cfg:12](../etc/init/msdp.cfg:12)

##### path

**值**: `["/system/bin/sa_main", "/system/profile/msdp.json"]`
**说明**: 服务启动路径和参数
**证据**: [etc/init/msdp.cfg:13](../etc/init/msdp.cfg:13)

##### uid

**值**: `msdp`
**说明**: 服务运行的用户 ID
**证据**: [etc/init/msdp.cfg:14](../etc/init/msdp.cfg:14)

##### gid

**值**: `["msdp", "shell", "input", "access_token"]`
**说明**: 服务运行的组 ID 列表
**证据**: [etc/init/msdp.cfg:15](../etc/init/msdp.cfg:15)

**组列表**:
1. `msdp`: msdp 服务主组
2. `shell`: Shell 组
3. `input`: 输入设备组（访问输入事件）
4. `access_token`: 访问令牌组（访问令牌管理）

##### permission (26 个)

**值**: 服务权限列表
**证据**: [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16)

**权限分类**:

#### 传感器和硬件相关 (3 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.ACCELEROMETER` | 🟡 中 | 加速度传感器 |
| `ohos.permission.CAMERA` | 🔴 高 | 摄像头 |
| `ohos.permission.VIBRATE` | 🟢 低 | 震动器 |

#### 输入相关 (4 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.INPUT_MONITORING` | 🔴 高 | 输入监听 |
| `ohos.permission.INJECT_INPUT_EVENT` | 🔴 高 | 注入输入事件 |
| `ohos.permission.INTERCEPT_INPUT_EVENT` | 🔴 高 | 拦截输入事件 |
| `ohos.permission.FILTER_INPUT_EVENT` | 🟡 中 | 过滤输入事件 |

#### 位置和网络 (3 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.LOCATION` | 🔴 高 | 精确位置 |
| `ohos.permission.APPROXIMATELY_LOCATION` | 🟡 中 | 大致位置 |
| `ohos.permission.MONITOR_DEVICE_NETWORK_STATE` | 🟡 中 | 监控网络状态 |

#### 音频和健康 (2 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.MICROPHONE` | 🔴 高 | 麦克风 |
| `ohos.permission.READ_HEALTH_DATA` | 🟡 中 | 读取健康数据 |

#### 屏幕相关 (1 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.CAPTURE_SCREEN` | 🔴 高 | 屏幕截图 |

#### 分布式相关 (2 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.DISTRIBUTED_DATASYNC` | 🟡 中 | 分布式数据同步 |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 🟡 中 | 访问分布式硬件 |

#### 账户和应用管理 (4 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.MANAGE_DISTRIBUTED_ACCOUNTS` | 🟡 中 | 管理分布式账户 |
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 🟡 中 | 管理本地账户 |
| `ohos.permission.GET_BUNDLE_INFO` | 🟢 低 | 获取应用信息 |
| `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED` | 🟡 中 | 获取应用信息(特权) |

#### 系统和运行时管理 (7 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.ACCESS_SERVICE_DM` | 🟡 中 | 访问设备管理服务 |
| `ohos.permission.RUNNING_STATE_OBSERVER` | 🟡 中 | 运行状态观察者 |
| `ohos.permission.GET_RUNNING_INFO` | 🟡 中 | 获取运行信息 |
| `ohos.permission.GET_TELEPHONY_STATE` | 🟡 中 | 获取电话状态 |
| `ohos.permission.MANAGE_SETTINGS` | 🟡 中 | 管理系统设置 |
| `ohos.permission.START_ABILITIES_FROM_BACKGROUND` | 🟡 中 | 后台启动能力 |
| `ohos.permission.MANAGE_MOUSE_CURSOR` | 🟡 中 | 管理鼠标光标 |

##### permission_acls (8 个)

**值**: ACL 权限列表
**证据**: [etc/init/msdp.cfg:44-52](../etc/init/msdp.cfg:44)

| ACL 权限 | 风险等级 | 说明 |
|----------|----------|------|
| `ohos.permission.INPUT_MONITORING` | 🔴 高 | 输入监听 |
| `ohos.permission.INJECT_INPUT_EVENT` | 🔴 高 | 注入输入事件 |
| `ohos.permission.INTERCEPT_INPUT_EVENT` | 🔴 高 | 拦截输入事件 |
| `ohos.permission.ACCESS_DISTRIBUTED_HARDWARE` | 🟡 中 | 访问分布式硬件 |
| `ohos.permission.MONITOR_DEVICE_NETWORK_STATE` | 🟡 中 | 监控网络状态 |
| `ohos.permission.FILTER_INPUT_EVENT` | 🟡 中 | 过滤输入事件 |
| `ohos.permission.MANAGE_MOUSE_CURSOR` | 🟡 中 | 管理鼠标光标 |
| `ohos.permission.CAPTURE_SCREEN` | 🔴 高 | 屏幕截图 |

---

## musl 与 非 musl 配置差异

### 🔴 重要发现: musl 版本权限更加宽松

**总体对比**:

| 服务 | 非 musl 权限 | musl 权限 | 增长率 | ACL 权限增长 |
|------|-------------|----------|--------|-------------|
| **sensors** | 2 个 | 9 个 | +350% | 1→1 (不变) |
| **msdp** | 26 个 | 36 个 | +38% | 8→12 (+50%) |

### sensors 配置差异

| 配置项 | sensors.cfg | sensors_musl.cfg | 差异说明 |
|--------|-------------|------------------|----------|
| **secon** | 无 | `u:r:sensors:s0` | musl 增加 SELinux 上下文 |
| **权限数量** | 2 个 | 9 个 | musl 增加 7 个权限 |

#### musl 新增权限 (7 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.GET_RUNNING_INFO` | 🟡 中 | 获取运行信息 |
| `ohos.permission.GET_BUNDLE_INFO` | 🟢 低 | 获取应用信息 |
| `ohos.permission.MANAGE_SECURE_SETTINGS` | 🔴 高 | 管理安全设置 |
| `ohos.permission.MANAGE_LOCAL_ACCOUNTS` | 🟡 中 | 管理本地账户 |
| `ohos.permission.RECEIVE_UPDATE_MESSAGE` | 🟡 中 | 接收更新消息 |
| `ohos.permission.ACCESS_SECURITY_PRIVACY_CENTER` | 🔴 高 | 访问安全隐私中心 |
| `ohos.permission.RECEIVER_STARTUP_COMPLETED` | 🟡 中 | 接收启动完成广播 |

**证据**:
- 非 musl: [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15)
- musl: [etc/init/sensors_musl.cfg:16-29](../etc/init/sensors_musl.cfg:16)

### msdp 配置差异

| 配置项 | msdp.cfg | msdp_musl.cfg | 差异说明 |
|--------|----------|---------------|----------|
| **secon** | 无 | `u:r:msdp_sa:s0` | musl 增加 SELinux 上下文 |
| **GID** | 4 个 | 5 个 | musl 增加 `dev_dma_heap` |
| **权限数量** | 26 个 | 36 个 | musl 增加 10 个权限 |
| **ACL 权限** | 8 个 | 12 个 | musl 增加 4 个 ACL 权限 |

#### musl 新增权限 (10 个)

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| `ohos.permission.DISTRIBUTED_SOFTBUS_CENTER` | 🟡 中 | 分布式软总线中心 |
| `ohos.permission.ACCESS_SERVICE_DP` | 🟡 中 | 访问设备配置服务 |
| `ohos.permission.GYROSCOPE` | 🟡 中 | 陀螺仪 |
| `ohos.permission.POWER_OPTIMIZATION` | 🟡 中 | 电源优化 |
| `ohos.permission.ACTIVITY_MOTION` | 🟡 中 | 活动运动检测 |
| `ohos.permission.ACCESSIBILITY_EXTENSION_ABILITY` | 🔴 高 | 无障碍扩展能力 |
| `ohos.permission.QUERY_ACCESSIBILITY_ELEMENT` | 🔴 高 | 查询无障碍元素 |
| `ohos.permission.ACCESS_EXT_SYSTEM_ABILITY` | 🟡 中 | 访问扩展系统能力 |
| `ohos.permission.GET_PAGE_INFO` | 🟡 中 | 获取页面信息 |
| `ohos.permission.PERCEIVE_SMART_POWER_SCENARIO` | 🟡 中 | 感知智能电源场景 |

#### musl 新增 ACL 权限 (4 个)

| ACL 权限 | 风险等级 | 说明 |
|----------|----------|------|
| `ohos.permission.QUERY_ACCESSIBILITY_ELEMENT` | 🔴 高 | 查询无障碍元素 |
| `ohos.permission.ACCESS_EXT_SYSTEM_ABILITY` | 🟡 中 | 访问扩展系统能力 |
| `ohos.permission.GET_PAGE_INFO` | 🟡 中 | 获取页面信息 |

**证据**:
- 非 musl: [etc/init/msdp.cfg:16-53](../etc/init/msdp.cfg:16)
- musl: [etc/init/msdp_musl.cfg:17-69](../etc/init/msdp_musl.cfg:17)

### SELinux 上下文 (musl 特有)

musl 版本增加了 SELinux 安全上下文配置:

| 服务 | SELinux 上下文 | 说明 |
|------|---------------|------|
| sensors | `u:r:sensors:s0` | sensors 域 |
| msdp | `u:r:msdp_sa:s0` | msdp_sa 域 |

**说明**: SELinux 上下文定义了服务的安全域，用于强制访问控制。

### 条件编译

**GN 变量**: `use_musl`
**代码位置**: [etc/init/BUILD.gn:19-23, 30-34](../etc/init/BUILD.gn:19)

```gn
if (use_musl) {
  source = "sensors_musl.cfg"
} else {
  source = "sensors.cfg"
}
```

### 安全建议

🔴 **警告**: musl 版本配置权限更加宽松

1. **sensors**: 权限从 2 个增加到 9 个，增加了 `MANAGE_SECURE_SETTINGS` 等敏感权限
2. **msdp**: 权限从 26 个增加到 36 个，增加了无障碍相关的高风险权限
3. **使用场景**: 需要确认 musl 版本的使用场景，避免在非必要情况下使用高权限配置
4. **权限审查**: 建议审查 musl 版本额外权限的必要性

---

## 配置文件对比

### sensors vs msdp 对比

| 配置项 | sensors | msdp | 差异 |
|--------|---------|------|------|
| 服务名称 | sensors | msdp | 不同 |
| 启动路径 | sa_main + sensors.json | sa_main + msdp.json | SA 配置不同 |
| UID | sensor | msdp | 不同 |
| GID | sensor, shell | msdp, shell, input, access_token | msdp 有更多组 |
| 权限数量 | 2 | 26 | msdp 权限多得多 |
| ACL 权限 | 1 | 8 | msdp ACL 权限多 |
| Boot job | 创建目录 | 创建目录 + 启动服务 | msdp 显式启动 |

---

## 配置文件验证

### JSON 格式验证

```bash
# 验证 JSON 格式
cat /etc/init/sensors.rc | jq .
cat /etc/init/msdp.rc | jq .
```

### 配置文件完整性检查

```bash
# 检查必需字段
cat /etc/init/sensors.rc | jq '.jobs[].name'
cat /etc/init/sensors.rc | jq '.services[].name'

cat /etc/init/msdp.rc | jq '.jobs[].name'
cat /etc/init/msdp.rc | jq '.services[].name'
```

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位
- [目录结构](./01_Directory_Structure.md) - 文件组织
- [架构说明](./02_Architecture.md) - 启动流程
- [GN Targets](./05_GN_Targets.md) - 构建配置
- [安全评审](./07_Security_Audit.md) - 权限和安全分析

---

**最后更新**: 2026-02-06

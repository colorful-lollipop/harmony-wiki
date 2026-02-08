# 编译产物

**适用范围**: 本文档适用于所有需要了解编译产物和运行时加载的人员
**目的**: 说明编译产物清单、安装路径、运行时加载关系
**关键结论**: 本仓库编译产物为两个配置文件，由 INIT 系统在启动时读取

---

## 编译产物清单

### 产物汇总表

| GN Target | 源文件 (非 musl) | 源文件 (musl) | 输出文件 | 安装路径 | 用途 |
|-----------|------------------|---------------|----------|----------|------|
| sensors.rc | sensors.cfg | sensors_musl.cfg | sensors.rc | /etc/init/sensors.rc | sensors 服务启动配置 |
| msdp.rc | msdp.cfg | msdp_musl.cfg | msdp.rc | /etc/init/msdp.rc | msdp 服务启动配置 |

**总计**: 2 个编译产物

### 产物说明

#### sensors.rc

**类型**: 配置文件 (JSON 格式)
**大小**: ~1 KB
**内容**: sensors 服务的启动配置

**主要配置项**:
- 服务名称: `sensors`
- 启动路径: `/system/bin/sa_main /system/profile/sensors.json`
- UID: `sensor`
- GID: `[sensor, shell]`
- 权限: 2 个权限
- Boot Job: 创建数据目录

**详细内容**: [appendix/Config_Files.md](./appendix/Config_Files.md#sensorscfg-详解)

#### msdp.rc

**类型**: 配置文件 (JSON 格式)
**大小**: ~2.5 KB
**内容**: msdp 服务的启动配置

**主要配置项**:
- 服务名称: `msdp`
- 启动路径: `/system/bin/sa_main /system/profile/msdp.json`
- UID: `msdp`
- GID: `[msdp, shell, input, access_token]`
- 权限: 26 个权限（含 8 个敏感 ACL 权限）
- Boot Job: 创建数据目录并启动服务

**详细内容**: [appendix/Config_Files.md](./appendix/Config_Files.md#msdpcfg-详解)

---

## 安装路径

### 系统镜像安装位置

**根目录**: `/`
**配置目录**: `/etc/init/`

```
系统镜像
└── /etc/init/
    ├── sensors.rc    ← sensors 服务启动配置
    └── msdp.rc       ← msdp 服务启动配置
```

### 安装路径详解

#### /etc/init/sensors.rc

**完整路径**: `/etc/init/sensors.rc`
**文件权限**: 644 (rw-r--r--) 或由构建系统决定
**所有者**: root (默认)
**安装时机**: 系统镜像构建时

**用途**: INIT 进程在启动时读取此文件，根据配置启动 sensors 服务

#### /etc/init/msdp.rc

**完整路径**: `/etc/init/msdp.rc`
**文件权限**: 644 (rw-r--r--) 或由构建系统决定
**所有者**: root (默认)
**安装时机**: 系统镜像构建时

**用途**: INIT 进程在启动时读取此文件，根据配置启动 msdp 服务

---

## 运行时加载关系

### 启动流程概览

```mermaid
graph TB
    INIT[/system/bin/init<br/>INIT 进程]
    SENSORS_RC[/etc/init/sensors.rc]
    MSDP_RC[/etc/init/msdp.rc]
    SA_MAIN[/system/bin/sa_main<br/>SA 进程管理器]
    SENSORS_JSON[/system/profile/sensors.json]
    MSDP_JSON[/system/profile/msdp.json]
    SENSORS_PROC[sensors 进程]
    MSDP_PROC[msdp 进程]

    INIT -->|读取| SENSORS_RC
    INIT -->|读取| MSDP_RC
    SENSORS_RC -->|启动路径| SA_MAIN
    MSDP_RC -->|启动路径| SA_MAIN
    SA_MAIN -->|加载配置| SENSORS_JSON
    SA_MAIN -->|加载配置| MSDP_JSON
    SENSORS_JSON -->|创建| SENSORS_PROC
    MSDP_JSON -->|创建| MSDP_PROC

    style SENSORS_RC fill:#e1f5ff
    style MSDP_RC fill:#e1f5ff
    style SENSORS_PROC fill:#fff4e1
    style MSDP_PROC fill:#fff4e1
```

### sensors 服务加载关系

```
1. INIT 进程启动 (/system/bin/init)
    ↓
2. 读取 /etc/init/sensors.rc
    ↓
3. 执行 boot job:
   - mkdir /data/service/el1/public/sensor
   - chown sensor sensor /data/service/el1/public/sensor
    ↓
4. 启动服务:
   execvp("/system/bin/sa_main", ["/system/bin/sa_main", "/system/profile/sensors.json"])
    ↓
5. sa_main 读取 /system/profile/sensors.json (外部文件)
    ↓
6. 创建 sensors 进程
   - UID: sensor
   - GID: sensor, shell
    ↓
7. 加载动态库:
   - libsensor_service.z.so (SA 3601)
   - libmiscdevice_service.z.so (SA 3602)
    ↓
8. 服务就绪
```

**证据**: [etc/init/sensors.cfg:10-22](../etc/init/sensors.cfg:10)

### msdp 服务加载关系

```
1. INIT 进程启动 (/system/bin/init)
    ↓
2. 读取 /etc/init/msdp.rc
    ↓
3. 执行 boot job:
   - mkdir /data/service/el1/public/msdp
   - chown msdp msdp /data/service/el1/public/msdp
   - start msdp (显式启动)
    ↓
4. 启动服务:
   execvp("/system/bin/sa_main", ["/system/bin/sa_main", "/system/profile/msdp.json"])
    ↓
5. sa_main 读取 /system/profile/msdp.json (外部文件)
    ↓
6. 创建 msdp 进程
   - UID: msdp
   - GID: msdp, shell, input, access_token
    ↓
7. 加载 msdp 动态库
    ↓
8. 服务就绪
```

**证据**: [etc/init/msdp.cfg:11-54](../etc/init/msdp.cfg:11)

---

## 运行时依赖

### INIT 系统依赖

**路径**: `/system/bin/init`
**版本**: OpenHarmony INIT 系统
**用途**: 读取并执行 /etc/init/*.rc 配置文件

### sa_main 依赖

**路径**: `/system/bin/sa_main`
**版本**: OpenHarmony SA 框架
**用途**: 根据配置文件加载和启动 System Ability

### SA 配置文件依赖（外部）

| 配置文件 | 位置 | 生成方 | 用途 |
|----------|------|--------|------|
| sensors.json | /system/profile/sensors.json | sensors_sensor 组件 | SA 3601 / 3602 配置 |
| msdp.json | /system/profile/msdp.json | msdp 相关组件 | msdp SA 配置 |

**说明**: 这些文件不在本仓库，由其他组件生成

### 动态库依赖（外部）

| 动态库 | 位置 | 生成方 | SA ID | 用途 |
|--------|------|--------|-------|------|
| libsensor_service.z.so | /system/lib/ | sensors_sensor 组件 | 3601 | 传感器服务 |
| libmiscdevice_service.z.so | /system/lib/ | sensors_miscdevice 组件 | 3602 | 震动器等服务 |
| msdp 动态库 | /system/lib/ | msdp 相关组件 | - | msdp 服务实现 |

**说明**: 这些动态库不在本仓库，由其他组件生成

---

## 运行时进程

### INIT 进程

**进程名**: init
**PID**: 1
**路径**: `/system/bin/init`
**所有者**: root
**职责**: 系统初始化和服务启动

### sensors 进程

**进程名**: sensors
**路径**: `/system/bin/sa_main`
**UID**: sensor
**GID**: sensor, shell
**父进程**: init
**子进程**: 无（但包含多个线程）

**加载的 SA**:
- SA 3601: 传感器服务
- SA 3602: 震动器等服务

**证据**: [etc/init/sensors.cfg:13-14](../etc/init/sensors.cfg:13)

### msdp 进程

**进程名**: msdp
**路径**: `/system/bin/sa_main`
**UID**: msdp
**GID**: msdp, shell, input, access_token
**父进程**: init
**子进程**: 无（但包含多个线程）

**加载的 SA**: msdp SA (具体 SA ID 待确认)

**证据**: [etc/init/msdp.cfg:14-15](../etc/init/msdp.cfg:14)

---

## 运行时数据目录

### sensors 数据目录

**路径**: `/data/service/el1/public/sensor`
**创建时机**: boot job (系统启动时)
**创建者**: INIT 进程
**所有者**: sensor:sensor
**权限**: 755 (drwxr-xr-x) 或由 mkdir 默认决定
**用途**: 存储传感器服务运行时数据

**创建命令**: [等价: etc/init/sensors.cfg:5-6](../etc/init/sensors.cfg:5)
```json
"mkdir /data/service/el1/public/sensor",
"chown sensor sensor /data/service/el1/public/sensor"
```

### msdp 数据目录

**路径**: `/data/service/el1/public/msdp`
**创建时机**: boot job (系统启动时)
**创建者**: INIT 进程
**所有者**: msdp:msdp
**权限**: 755 (drwxr-xr-x) 或由 mkdir 默认决定
**用途**: 存储 msdp 服务运行时数据

**创建命令**: [等价: etc/init/msdp.cfg:5-6](../etc/init/msdp.cfg:5)
```json
"mkdir /data/service/el1/public/msdp",
"chown msdp msdp /data/service/el1/public/msdp"
```

---

## 运行时日志

### INIT 日志

**查看方式**:
```bash
# 查看 INIT 进程日志
hilog -T Init

# 或使用 dmesg (如果支持)
dmesg | grep init
```

**常见日志内容**:
- "Read config file: /etc/init/sensors.rc"
- "Start service: sensors"
- "Read config file: /etc/init/msdp.rc"
- "Start service: msdp"

### 服务日志

**sensors 服务日志**:
```bash
hilog -T sensors
```

**msdp 服务日志**:
```bash
hilog -T msdp
```

---

## 运行时调试

### 检查服务状态

```bash
# 检查 sensors 进程是否运行
ps -ef | grep sensors

# 检查 msdp 进程是否运行
ps -ef | grep msdp

# 查看 SA 状态
hidumper -s 3601  # 传感器服务
hidumper -s 3602  # 震动器服务
```

### 手动重启服务

```bash
# 重启 sensors 服务
systemctl restart sensors
# 或
svc control restart sensors

# 重启 msdp 服务
systemctl restart msdp
# 或
svc control restart msdp
```

### 查看配置文件

```bash
# 查看 sensors 服务启动配置
cat /etc/init/sensors.rc

# 查看 msdp 服务启动配置
cat /etc/init/msdp.rc
```

---

## 性能影响

### 资源占用

**编译产物大小**:
- sensors.rc: ~1 KB
- msdp.rc: ~2.5 KB
- 总计: ~3.5 KB

**运行时资源**:
- 配置文件读取时间: 微秒级
- 服务启动时间: 取决于服务实现（不在本仓库）

**影响评估**: 本仓库的编译产物对系统性能影响可忽略不计

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位
- [目录结构](./01_Directory_Structure.md) - 文件组织
- [架构说明](./02_Architecture.md) - 启动流程
- [GN Targets](./05_GN_Targets.md) - 构建配置
- [安全评审](./07_Security_Audit.md) - 权限和安全
- [配置文件详解](./appendix/Config_Files.md) - 配置参数完整说明

---

**最后更新**: 2026-02-06

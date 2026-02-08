# 配置说明

## 配置文件清单

| 配置文件 | 路径 | 用途 |
|---------|------|------|
| `socperf_resource_config.xml` | `profile/` | 资源定义（CPU/GPU/DDR/NPU） |
| `socperf_boost_config.xml` | `profile/` | 性能提频配置 |
| `1906.json` | `sa_profile/` | System Ability 配置 |

---

## SA 配置 (1906.json)

**文件**：`sa_profile/1906.json`

```json
{
    "process": "resource_schedule_service",
    "systemability": [
        {
            "name": 1906,
            "libpath": "libsocperf_server.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1
        }
    ]
}
```

| 字段 | 说明 |
|------|------|
| `process` | 运行进程名 |
| `name` | SA ID (1906) |
| `libpath` | 服务库路径 |
| `run-on-create` | 启动时加载 |
| `distributed` | 是否支持分布式 |
| `dump_level` | 调试 dump 级别 |

---

## 资源配置文件 (socperf_resource_config.xml)

**文件**：`profile/socperf_resource_config.xml`

### 结构定义

```xml
<?xml version='1.0' encoding="utf-8"?>
<Configs>
    <!-- 资源定义 -->
    <Resources>
        <!-- CPU 频率资源 -->
        <ResNode>
            <ResId>xxx</ResId>
            <Path>...</Path>        <!-- 设备路径 -->
            <Name>...</Name>        <!-- 资源名称 -->
            <Min>0</Min>           <!-- 最小值 -->
            <Max>100</Max>         <!-- 最大值 -->
        </ResNode>
    </Resources>

    <!-- 治理资源 -->
    <GovResources>
        <GovResNode>
            <ResId>xxx</ResId>
            <Name>...</Name>
            <PersistMode>0</PersistMode>  <!-- 持久化模式 -->
            <Available>...</Available>    <!-- 可用性配置 -->
        </GovResNode>
    </GovResources>

    <!-- 场景资源 -->
    <SceneResources>
        <SceneResNode>
            <Name>...</Name>
            <ResItems>
                <ResItem>
                    <ResId>xxx</ResId>
                </ResItem>
            </ResItems>
        </SceneResNode>
    </SceneResources>
</Configs>
```

### 元素说明

| 元素 | 说明 | 必需 |
|------|------|------|
| `ResId` | 资源唯一标识 | 是 |
| `Path` | 内核设备路径 | 是 |
| `Name` | 资源名称 | 否 |
| `Min`/`Max` | 有效值范围 | 否 |
| `PersistMode` | 持久化模式 | 否 |

---

## 提频配置文件 (socperf_boost_config.xml)

**文件**：`profile/socperf_boost_config.xml`

### 结构定义

```xml
<?xml version='1.0' encoding="utf-8"?>
<Configs>
    <!-- 提频场景 -->
    <Scenes>
        <Scene>
            <Name>scene_name</Name>        <!-- 场景名称 -->
            <CmdId>xxx</CmdId>            <!-- 命令 ID -->
            <Actions>
                <Action>
                    <ResId>xxx</ResId>    <!-- 资源 ID -->
                    <Value>xxx</Value>    <!-- 目标值 -->
                    <Duration>xxx</Duration>  <!-- 持续时间(ms) -->
                    <Path>...</Path>      <!-- 可选路径覆盖 -->
                </Action>
            </Actions>
        </Scene>
    </Scenes>

    <!-- 设备模式 -->
    <DeviceMode>
        <Mode name="mode_name">
            <CmdId>xxx</CmdId>
            <Tags>
                <Tag>on</Tag>             <!-- 开事件 -->
                <Tag>off</Tag>            <!-- 关事件 -->
            </Tags>
        </Mode>
    </DeviceMode>
</Configs>
```

### 元素说明

| 元素 | 说明 |
|------|------|
| `CmdId` | 场景命令 ID，用于匹配请求 |
| `ResId` | 关联的资源 ID |
| `Value` | 调频目标值 |
| `Duration` | 生效持续时间(ms) |
| `Mode.name` | 设备模式名称 |

---

## 配置加载流程

```
1. Init()
   ↓
2. GetRealConfigPath("socperf_resource_config.xml")
   ↓
3. LoadAllConfigXmlFile()
   ├─ LoadConfigXmlFile("socperf_resource_config.xml")
   │   ├─ ParseResourceXmlFile()
   │   └─ ParseBoostXmlFile()
   └─ LoadConfigXmlFile("socperf_boost_config.xml")
       └─ ParseBoostXmlFile()
   ↓
4. InitPerfFunc() [加载性能回调]
```

---

## 关键 API

### SocPerfConfig

| 方法 | 说明 |
|------|------|
| `IsGovResId(int32_t resId)` | 检查是否为治理资源 |
| `IsValidResId(int32_t resId)` | 检查资源 ID 是否有效 |
| `GetInstance()` | 获取单例 |

---

## 配置验证

### CheckPairResIdValid

检查配对资源 ID 的有效性。

### CheckDefValid

检查默认值的有效性。

### CheckActionResIdAndValueValid

检查动作中资源 ID 和值的有效性。

**证据**：`services/core/include/socperf_config.h:74-81`

```cpp
bool CheckPairResIdValid() const;
bool CheckDefValid() const;
bool CheckActionResIdAndValueValid(const std::string& configFile);
```

---

## 常见问题

### Q1: 配置修改后不生效

**原因**：配置只会在 `OnStart()` 时加载一次。

**解决**：重启服务或触发重新加载。

### Q2: CmdId 冲突

**原因**：多个场景使用相同的 CmdId。

**解决**：确保所有 CmdId 唯一。

### Q3: ResId 无效

**原因**：Action 中引用的 ResId 未在 Resources 中定义。

**解决**：确保 ResId 已正确定义。

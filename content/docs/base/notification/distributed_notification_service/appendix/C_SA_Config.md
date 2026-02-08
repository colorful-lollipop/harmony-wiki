# 附录C: SA配置说明

## SA 配置文件格式

### ANS Service (3203)

**文件**: `sa_profile/3203.json`

```json
{
    "process": "foundation",
    "systemability": [{
        "name": 3203,
        "libpath": "libans.z.so",
        "run-on-create": true,
        "depend": [3299],
        "extension": ["backup", "restore"],
        "depend_time_out": 60000,
        "distributed": false,
        "dump_level": 1
    }]
}
```

### Reminder Service (3204)

**文件**: `services/reminder/sa_profile/3204.json`

```json
{
    "process": "foundation",
    "systemability": [{
        "name": 3204,
        "libpath": "libreminder.z.so",
        "run-on-create": false,
        "distributed": false,
        "dump_level": 1
    }]
}
```

## 配置字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `process` | string | 运行进程名 |
| `name` | int | SA ID |
| `libpath` | string | 库路径 |
| `run-on-create` | boolean | 是否自动启动 |
| `depend` | array | 依赖的SA ID |
| `extension` | array | 扩展能力 |
| `depend_time_out` | int | 依赖超时(ms) |
| `distributed` | boolean | 是否分布式 |
| `dump_level` | int | dump级别 |

## SA 构建配置

**文件**: `sa_profile/BUILD.gn`

```gn
ohos_sa_profile("ans_sa_profile") {
    sources = [ "3203.json" ]
    part_name = "distributed_notification_service"
}
```

## 客户端连接

```cpp
// 获取SA代理
sptr<ISystemAbilityManager> samgr =
    SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();

sptr<IRemoteObject> remote =
    samgr->GetSystemAbility(ADVANCED_NOTIFICATION_SERVICE_ABILITY_ID);

sptr<IAnsManager> proxy = iface_cast<IAnsManager>(remote);
```

**证据**: `frameworks/core/src/ans_notification.cpp`

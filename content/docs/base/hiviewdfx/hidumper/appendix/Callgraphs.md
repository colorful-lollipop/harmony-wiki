# 关键调用链图谱

> 目的：记录 HiDumper 中关键功能的入口→核心逻辑→系统调用的完整调用链

## 1. Dump 请求调用链

### 1.1 CLI 入口 → Dump 输出

```
dump_client_main.cpp
     │
     ├── ParseArgv(int argc, char* argv[])
     │       │
     │       └── DumperOpts::Parse() ──▶ 解析 -c, -p, --mem 等选项
     │
     ├── DumpManagerClient::Request(int argc, const char* argv[], ...)
     │       │
     │       └── IPCSkeleton::GetSystemAbilityManager()
     │               │
     │               └── GetSystemAbility(1212)
     │                       │
     │                       ▼
     │               IRemoteObject (Stub)
     │
     ▼
DumpManagerService (SA 1212)
     │
     ├── DumpManagerService::OnRemoteRequest()
     │       │
     │       └── IDumpBroker::Request(fd, args)
     │               │
     │               ▼
     │       DumpManager::Request(fd, args)
     │               │
     │               ├── DumpController::PreDump()
     │               │       │
     │               │       └── DumpStrategyFactory::Create()
     │               │               │
     │               │               ▼
     │               │       SystemInfoDumpStrategy / NetDumpStrategy / etc.
     │               │
     │               └── DumpImplement::Dump()
     │                       │
     │                       ▼
     │               创建具体 Dumper:
     │               - FileDumper
     │               - CmdDumper  
     │               - CpuDumper
     │               - MemDumper
     │               - ProcessInfoDumper
     │               - NetDumper
     │               - SystemAbilityDumper
     │                       │
     │                       ▼
     │               Dumper::Dump()
     │                       │
     │                       ▼
     │               Output (FdOutput / ZipOutput)
     │                       │
     │                       ▼
     │               返回结果到 fd
     │
     ▼
DumpManagerClient 接收 Response
     │
     └── 输出到 stdout
```

### 1.2 关键文件清单

| 阶段 | 文件 | 功能 |
|-----|------|------|
| CLI 入口 | `client/native/dump_client_main.cpp` | 参数解析 |
| 客户端封装 | `services/native/src/dump_manager_client.cpp` | IPC 客户端 |
| 服务 Stub | `services/zidl/src/dump_broker_stub.cpp` | 消息分发 |
| 管理器 | `frameworks/native/src/manager/dump_manager.cpp` | 流程编排 |
| 控制器 | `frameworks/native/src/manager/dump_controller.cpp` | 控制逻辑 |
| 策略工厂 | `frameworks/native/src/factory/dump_strategy_factory.cpp` | 策略创建 |
| Dumper 基类 | `frameworks/native/src/executor/*_dumper.cpp` | 各功能执行 |
| 输出 | `frameworks/native/src/util/dump_controller.cpp` | 结果输出 |

## 2. CPU 服务调用链

```
DumpManagerCpuClient::Request()
        │
        ▼
DumpManagerCpuService (SA 1215)
        │
        ├── IHidumperCpuService::Request()
        │       │
        │       └── CpuUsageCollector::Collect()
        │               │
        │               ├── ReadCpuStat()
        │               │       │
        │               │       └── ReadFile("/proc/stat")
        │               │
        │               └── CalculateCpuUsage()
        │                       │
        │                       ▼
        │               返回 DumpCpuData
        │
        ▼
DumpCpuData 结构返回
```

## 3. 内存信息获取调用链

```
MemDumper::Dump()
        │
        ├── GetProcessInfo::GetPidInfos()
        │       │
        │       └── ReadDir("/proc")
        │               │
        │               ▼
        │       解析 /proc/<pid>/status
        │
        ├── GetHeapInfo::GetHeapInfoByPid(pid)
        │       │
        │       └── ParseSmaps(pid)
        │               │
        │               ├── ReadFile("/proc/<pid>/smaps")
        │               │
        │               └── ParseSmapsInfo()
        │                       │
        │                       ▼
        │               SmapsMemoryInfo
        │
        └── Output 结果
```

## 4. System Ability 查询调用链

```
SystemAbilityDumper::Dump()
        │
        ├── ISystemAbilityManager::ListSystemAbilities()
        │       │
        │       └── IPCSkeleton::GetSystemAbilityManager()
        │               │
        │               ▼
        │       SAMgr 返回 SA 列表
        │
        └── 遍历 SA ID 并查询每个 SA:
                │
                ├── GetSystemAbilityInfo(saId)
                │       │
                │       └── IPC 调用 SA
                │
                └── Output SA 信息
```

## 5. 权限校验调用链

```
DumpManagerService::Request()
        │
        ├── IPCSkeleton::GetCallingUid()
        │       │
        │       └── 验证 UID 是否为 root 或 system
        │
        ├── IPCSkeleton::GetCallingTokenID()
        │       │
        │       └── AccessTokenKit::VerifyAccessToken()
        │               │
        │               └── 检查 ohos.permission.DUMP
        │
        └── 通过后继续处理请求
```

## 6. 工厂模式创建 Dumper

```
DumpStrategyFactory::Create(std::string option)
        │
        ├── Switch(option):
        │   ├── "-c"  → SystemInfoDumpStrategy
        │   ├── "-p"  → ProcessInfoDumpStrategy  
        │   ├── "--net" → NetDumpStrategy
        │   ├── "--mem" → MemDumpStrategy
        │   ├── "--cpuusage" → CpuDumpStrategy
        │   └── ...
        │
        └── 返回具体 Strategy
                │
                ▼
        Strategy::PreDump()
                │
                └── 创建对应 Dumper:
                    - FileDumper (SystemInfoDumpStrategy)
                    - CmdDumper (SystemInfoDumpStrategy)
                    - CpuDumper (CpuDumpStrategy)
                    - MemDumper (MemDumpStrategy)
                    - NetDumper (NetDumpStrategy)
                    - StorageDumper (StorageDumpStrategy)
                    - SystemAbilityDumper (SystemAbilityDumpStrategy)
                    - EventListDumper (EventDumpStrategy)
```

## 7. 异常处理调用链

```
try {
    Dumper::Dump()
} catch (const std::exception& e) {
    // dump_implement.cpp
    HILOGE("Dump failed: %{public}s", e.what());
    return ERR_DUMP_FAILED;
}

// FD 泄漏检测
if (ScanPidOverLimit()) {
    HILOGW("FD leak detected");
}

// 返回错误码
return result;
```

## 相关文档

- [系统架构](./01_Architecture.md)
- [API 参考](./02_API_Reference.md)
- [构建系统](./03_Build_System.md)

# 编译产物与安装

> **目的**: 完整的编译产物清单,安装路径和运行时加载关系
> **适用范围**: 部署工程师、平台工程师
> **阅读时间**: 25分钟

---

## 概述

powermgr_lite 根据系统类型生成不同的编译产物:
- **mini 系统 (LiteOS-M)**: 静态库 (.a)
- **small 系统 (LiteOS-A)**: 共享库 (.so)

---

## 产物清单

### Mini 系统产物

| 产物名称 | 类型 | 大小 | 说明 |
|----------|------|------|------|
| `libpowermgr.a` | static_library | ~8KB | Frameworks 静态库 |
| `libpowermgrservice.a` | static_library | ~14KB | Services 静态库 |
| `libpowermgr_utils.a` | static_library | ~2KB | 工具静态库 |
| `libnativeapi_battery_simulator.a` | static_library | ~4KB | JS 模块静态库 |

### Small 系统产物

| 产物名称 | 类型 | 大小 | 说明 |
|----------|------|------|------|
| `libpowermgr.so` | shared_library | ~8KB | Frameworks 共享库 |
| `libpowermgrservice.so` | shared_library | ~14KB | Services 共享库 |
| `libnativeapi_battery_simulator.a` | static_library | ~4KB | JS 模块静态库 (链接到 ACE 引擎) |

**注意**: `libpowermgr_utils.a` 在 small 系统中被链接到 `libpowermgrservice.so`,不独立产出。

---

## 安装路径

### Mini 系统安装

**特点**: 静态链接,无独立安装路径

```
[系统构建时]
  └── 链接阶段
      ├── libpowermgr.a → 链接到系统可执行文件
      ├── libpowermgrservice.a → 链接到系统可执行文件
      └── libpowermgr_utils.a → 链接到系统可执行文件

[运行时]
  └── 代码在系统可执行文件中,无独立 .so
```

### Small 系统安装

**特点**: 动态链接,共享库安装到标准路径

```
[系统构建时]
  └── 链接阶段
      ├── libpowermgr.so → 共享库
      ├── libpowermgrservice.so → 共享库
      └── libnativeapi_battery_simulator.a → 链接到 JS 引擎

[安装到文件系统]
  /usr/lib/
      ├── libpowermgr.so
      ├── libpowermgrservice.so
      └── libnativeapi_battery_simulator.a (可选)
```

---

## 运行时加载关系

### Mini 系统加载

```
[系统启动]
  └── 内核初始化
      ├── 初始化 LiteOS-M 电源子系统
      └── 注册系统服务
            ↓
      [powermgrservice 初始化]
      └── SYS_SERVICE_INIT(Init)  // power_manage_service.c:77-82
            ↓
      [服务注册]
      └── SAMGR_GetInstance()->RegisterService(&g_service)
            ↓
      [Feature 注册]
      ├── SYS_FEATURE_INIT(Init)  // power_manage_feature.c
      └── RegisterFeature("powermanage", &g_feature)
            ↓
      [应用启动]
      └── 静态链接 powermgr 代码
            ├── 调用 CreateRunningLock()
            ├── 直接调用服务函数
            └── 无动态加载
```

**特点**:
- 无动态加载
- 所有代码在应用地址空间
- 服务和应用在同一地址空间

### Small 系统加载

```
[系统启动]
  └── 内核初始化
      ├── 初始化电源子系统
      ├── 挂载 proc 文件系统 (/proc/power/*)
      └── 初始化 binder 驱动
            ↓
      [powermgrservice 进程启动]
      └── 独立用户态进程
            ↓
      [服务注册]
      └── SAMGR_GetInstance()->RegisterService(&g_service)
                  ↓
      [应用进程启动]
      ├── libpowermgr.so 通过 dlopen() 加载
      │   └── 符号解析
      │       └── CreateRunningLock, SuspendDevice, etc.
      │
      ├── [IPC 连接]
      │   └── SAMGR_GetInstance()->GetFeatureApi("powermgr", "powermanage")
      │
      └── [JS 引擎加载]
          └── libnativeapi_battery_simulator.a 静态链接
              └── InitBatteryModule(exports)
                  └── 注册 battery.getStatus
```

**特点**:
- 独立进程空间
- 通过 IPC 通信
- 动态符号解析
- JS 引擎静态链接电池模块

---

## 依赖加载顺序

### Mini 系统

```
[链接顺序]
  1. libpowermgr_utils.a (工具)
  2. libpowermgr.a (frameworks)
  3. libpowermgrservice.a (services)
  4. LiteOS-M 内核
  5. samgr_lite
  6. hilog_lite (静态)
```

### Small 系统

```
[加载顺序]
  1. libpowermgr.so (依赖 samgr_lite, hilog_lite)
  2. libpowermgrservice.so (依赖 libpowermgr.so, samgr_lite, hilog_lite, ipc_single)
  3. JS 引擎 (链接 libnativeapi_battery_simulator.a)
  4. samgr_lite (共享)
  5. hilog_lite (共享)
  6. ipc_single (共享)
```

---

## 运行时资源占用

### 内存占用 (Small 系统)

| 组件 | 代码段 (.text) | 数据段 (.data) | BSS (.bss) | 总计 |
|------|----------------|----------------|-------------|------|
| libpowermgr.so | ~4KB | ~1KB | ~0.5KB | ~5.5KB |
| libpowermgrservice.so | ~8KB | ~2KB | ~1KB | ~11KB |
| JS 模块 | ~2KB | ~0.5KB | ~0KB | ~2.5KB |
| **总计** | **~14KB** | **~3.5KB** | **~1.5KB** | **~19KB** |

**运行时 RAM** (估算):
- 代码段: 14KB (可能被换出)
- 数据段: 3.5KB
- BSS: 1.5KB
- **总运行时 RAM**: ~5KB (不含栈/堆)

### 进程占用 (Small 系统)

```
[powermgrservice 进程]
  └── 基础内存
      ├── 栈: 2KB (配置在 power_manage_service.c:27)
      ├── 堆: 动态分配 (运行锁、向量等)
      ├── 全局变量: ~2KB
      └── 线程: 主服务线程 + Suspend 线程

[应用进程]
  └── libpowermgr.so 加载
      ├── 符号表: ~1KB
      ├── 代理对象: 动态分配
      └── 运行锁对象: 应用创建
```

---

## 运行时验证

### 检查共享库加载
```bash
# 检查 libpowermgr.so 加载
ldd /path/to/your_app | grep powermgr

# 应显示:
# libpowermgr.so => /usr/lib/libpowermgr.so
#     libsamgr.so => /usr/lib/libsamgr.so
#     libhilog_lite.so => /usr/lib/libhilog_lite.so
```

### 检查符号导出
```bash
# 查看 libpowermgr.so 导出符号
nm -D /usr/lib/libpowermgr.so

# 应显示:
# 00000000 T CreateRunningLock
# 00000000 T DestroyRunningLock
# 00000000 T AcquireRunningLock
# 00000000 T ReleaseRunningLock
# 00000000 T SuspendDevice
# 00000000 T WakeupDevice
```

### 检查 JS 模块
```bash
# 查看 JS 引擎链接的 battery 模块
strings /path/to/ace_engine | grep battery

# 应显示:
# battery.getStatus
# BatteryModule
```

---

## 故障排查

### 问题 1: 共享库未找到

**症状**:
```
error while loading shared libraries: libpowermgr.so: cannot open shared object file
```

**原因**: 库未安装到正确路径

**解决方案**:
```bash
# 检查库是否存在
ls -l /usr/lib/libpowermgr.so

# 如不存在,重新安装
# hb install
```

### 问题 2: IPC 连接失败

**症状**:
```
Failed to get feature api: powermgr:powermanage
```

**原因**: 服务未启动或注册失败

**解决方案**:
```bash
# 检查服务运行状态
# 查看 hilog 日志
hdc shell hilog -x

# 应看到:
# [powermgr] Succeed to init power manager service
```

### 问题 3: 运行锁获取超时

**症状**:
```
AcquireRunningLock() 返回 FALSE
```

**原因**: 平台操作超时或系统忙

**解决方案**:
1. 检查内核日志: `/proc/kmsg` (Mini) 或 `dmesg` (Small)
2. 检查是否有其他进程持有锁
3. 检查 `/sys/power/wakeup_count` (Small) 是否正常

### 问题 4: 屏保未激活

**症状** (Small 系统 + enable_screensaver=true):
```
SetScreenSaverState(TRUE) 成功,但屏保未出现
```

**原因**: AMS 服务未启动或 Ability 配置错误

**解决方案**:
```bash
# 检查 AMS 服务运行状态
hdc shell "sa_list | grep AbilityManagerService"

# 检查屏保 Ability 是否安装
hdc shell "bm dump | grep screensaver"
```

---

## 相关文档

- [06_GN_Targets.md](06_GN_Targets.md) - GN 构建配置
- [03_Architecture.md](03_Architecture.md) - 架构与组件
- [09_Troubleshooting.md](09_Troubleshooting.md) - 更多故障排查

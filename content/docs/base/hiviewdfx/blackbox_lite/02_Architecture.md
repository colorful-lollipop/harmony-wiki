# 02_Architecture - 架构说明

## 2.1 组件图

### 2.1.1 模块关系

```
┌─────────────────────────────────────────────────────────────────┐
│                        blackbox_lite                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │ blackbox_core.c │  │blackbox_adapter │  │blackbox_detector│ │
│  │   (核心逻辑)     │  │    (适配层)      │  │   (事件上报)     │ │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘ │
├───────────┼─────────────────────┼─────────────────────┼──────────┤
│           │        WEAK         │                     │          │
│           │        symbols       │                     │          │
│           └─────────────────────┘                     │          │
│                                                       │          │
└───────────────────────────────────────────────────────┼──────────┘
                                                        │
                              ┌─────────────────────────┼──────────┐
                              │                         │          │
                              ▼                         ▼          │
                    ┌─────────────────┐    ┌─────────────────┐     │
                    │  平台实现层      │    │  hiview_lite    │     │
                    │blackbox_adapter │    │  (事件上报)      │     │
                    │  _impl.c        │    │UploadEventByFile│     │
                    └─────────────────┘    └─────────────────┘     │
                              │                                       
                              ▼                                       
                    ┌─────────────────┐                              
                    │  LiteOS 内核    │                              
                    │ LOS_Sem*/LOS_*  │                              
                    └─────────────────┘                              
```

### 2.1.2 依赖方向

```
blackbox_core.c
    │
    ├──► blackbox_adapter.h (定义 WEAK 接口)
    │         │
    │         └──► 平台实现层 (重写 WEAK 符号)
    │
    ├──► blackbox.h (ErrorInfo, ModuleOps)
    │
    ├──► blackbox_detector.h (UploadEventByFile)
    │         │
    │         └──► hiview_lite
    │
    ├──► hilog_lite (日志打印)
    │
    └──► liteos_m (LOS_* 信号量/线程 API)
```

## 2.2 数据流

### 2.2.1 故障处理数据流

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  故障源  │───►│BBoxNotify│───►│ ModuleOps │───►│ 适配层   │
│ (中断/异常)│    │  Error   │    │  查找     │    │Dump/Reset│
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
                                                     │
              ┌──────────────────────────────────────┘
              ▼
    ┌─────────────────┐         ┌─────────────────┐
    │ SystemModuleDump │         │SystemModuleReset│
    │ (可选)           │         │ (可选)           │
    └────────┬────────┘         └────────┬────────┘
             │                            │
             ▼                            ▼
    ┌─────────────────┐         ┌─────────────────┐
    │  内存/存储       │         │  系统重启       │
    │  预处理          │         │  (可选)         │
    └─────────────────┘         └─────────────────┘
```

### 2.2.2 日志保存数据流（SaveErrorLog 线程）

```
开机时
    │
    ▼
┌───────────────────┐
│ SaveErrorLog 线程 │ ◄── 启动时机：内核初始化完成后
│    (blackbox_     │
│     core.c:143)   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ 遍历 g_opsList    │
│ 检查每个模块的    │
│ GetLastLogInfo()  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐      ┌───────────────────┐
│ GetLastLogInfo    │      │ SaveLastLog       │
│ 返回非0           │      │ 保存日志到文件    │
│ (有待保存日志)    │─────►│ (适配层实现)       │
└───────────────────┘      └─────────┬─────────┘
                                      │
                                      ▼
                            ┌───────────────────┐
                            │ UploadEventByFile │
                            │ 上报故障事件       │
                            │ (hiview_lite)     │
                            └───────────────────┘
```

## 2.3 线程模型

### 2.3.1 线程与信号量

| 线程/对象 | 用途 | 创建位置 | 同步机制 |
|-----------|------|----------|----------|
| SaveErrorLog | 日志保存工作线程 | `blackbox_core.c:328` | `g_opsListSem` |
| g_opsList | 模块 ops 链表 | `blackbox_core.c:46` | 信号量保护 |
| g_opsListSem | 链表访问信号量 | `blackbox_core.c:47` | 二值信号量 |

**证据**：`blackbox_core.c:318-332`
```c
static void BBoxInit(void)
{
    int ret = -1;
    pthread_t taskId = 0;

    // 创建二值信号量保护链表
    if (LOS_BinarySemCreate(1, &g_opsListSem) != LOS_OK) {
        BBOX_PRINT_ERR("Create binary semaphore failed!\n");
        return;
    }
    UtilsListInit(&g_opsList);

    // 创建 SaveErrorLog 线程
    ret = pthread_create(&taskId, NULL, SaveErrorLog, NULL);
    if (ret != 0) {
        BBOX_PRINT_ERR("Falied to create SaveErrorLog task, ret: %d\n", ret);
    }
}
```

### 2.3.2 线程安全分析

**临界区**：`g_opsList` 链表的增删查操作

| 操作 | 位置 | 信号量使用 |
|------|------|------------|
| 注册模块 ops | `BBoxRegisterModule:206` | LOS_SemPend/Pend + Post |
| 查找模块 ops | `BBoxNotifyError:252` | 条件依赖 needSysReset |
| 保存日志 | `SaveErrorLog:143` | LOS_SemPend/Pend + Post |

**证据**：`blackbox_core.c:158-162`
```c
if (LOS_SemPend(g_opsListSem, LOS_WAIT_FOREVER) != 0) {
    BBOX_PRINT_ERR("Request g_opsListSem failed!\n");
    free(info);
    return NULL;
}
// 遍历链表操作...
(void)LOS_SemPost(g_opsListSem);
```

## 2.4 关键时序

### 2.4.1 系统初始化时序

```
系统启动
    │
    ├─────────────────────────────────────┐
    │ Kernel Init (LiteOS)                │
    │                                      │
    │ CORE_INIT_PRI(BBoxInit, 1)          │───► blackbox_core.c:318
    │  ├─ LOS_BinarySemCreate             │     创建信号量
    │  ├─ UtilsListInit                  │     初始化链表
    │  └─ pthread_create                  │     启动 SaveErrorLog 线程
    │                                      │
    │ CORE_INIT_PRI(BBoxAdapterInit, 2)  │───► blackbox_adapter.c:94
    │  └─ BBoxRegisterModuleOps           │     注册 MODULE_SYSTEM 适配器
    │       └─ SystemModule*             │     (平台实现)          │
    │                                      │
    └─────────────────────────────────────┘
              │
              ▼
    SaveErrorLog 线程开始运行
    (等待 LOG_ROOT_DIR 准备就绪)
```

### 2.4.2 故障处理时序

```
故障发生 (中断/异常上下文)
        │
        ▼
┌───────────────────────┐
│ BBoxNotifyError()     │ ◄── blackbox_core.c:252
│ event/module/Desc +   │     入口参数：事件、模块、描述、重启标志
│ needSysReset          │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ 查找匹配的 ModuleOps  │ ◄── blackbox_core.c:277-302
│ (遍历 g_opsList)      │
└───────────┬───────────┘
            │
    ┌───────┴───────┐
    │               │
    ▼               ▼
┌────────┐    ┌────────────┐
│ Dump   │    │ Reset      │
│ 可选   │    │ 可选       │
└───┬────┘    └─────┬──────┘
    │               │
    │               ▼
    │         ┌────────────┐
    │         │RebootSystem│
    │         │(可选重启)   │ ◄── blackbox_core.c:312
    │         └────────────┘
    │
    ▼
┌───────────────────────┐
│ SaveBasicErrorInfo()  │ ◄── blackbox_core.c:112
│ (无 Dump/Reset 时)    │
│ 保存基本信息到文件    │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ UploadEventByFile()   │ ◄── blackbox_detector.c:18
│ 上报故障事件          │     (目前返回 0，实际由 hiview_lite 实现)
└───────────────────────┘
```

### 2.4.3 日志保存时序（SaveErrorLog）

```
SaveErrorLog 线程 (开机时启动)
        │
        ▼
┌───────────────────────┐
│ 等待 LOG_ROOT_DIR     │ ◄── blackbox_core.c:97-110
│ (1000ms * 1 次)       │     等待文件系统就绪
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ 遍历 g_opsList        │
│ 对每个模块调用:        │
│  ├─ GetLastLogInfo()  │ ◄── blackbox_core.c:169
│  └─ SaveLastLog()     │     blackbox_core.c:175
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ UploadEventByFile()  │ ◄── blackbox_core.c:180
│ 上报已保存日志事件    │
└───────────────────────┘
```

## 2.5 关键代码路径

### 2.5.1 初始化路径

| 步骤 | 函数 | 文件:行 | 说明 |
|------|------|---------|------|
| 1 | `BBoxInit()` | `blackbox_core.c:318` | 初始化函数 |
| 2 | `LOS_BinarySemCreate()` | `blackbox_core.c:323` | 创建信号量 |
| 3 | `UtilsListInit()` | `blackbox_core.c:327` | 初始化链表 |
| 4 | `pthread_create()` | `blackbox_core.c:328` | 启动线程 |
| 5 | `BBoxAdapterInit()` | `blackbox_adapter.c:94` | 适配器初始化 |
| 6 | `BBoxRegisterModuleOps()` | `blackbox_adapter.c:104` | 注册适配器 |

### 2.5.2 故障处理路径

| 步骤 | 函数 | 文件:行 | 说明 |
|------|------|---------|------|
| 1 | `BBoxNotifyError()` | `blackbox_core.c:252` | 故障通知入口 |
| 2 | `FormatErrorInfo()` | `blackbox_core.c:71` | 格式化错误信息 |
| 3 | 遍历 `g_opsList` | `blackbox_core.c:277` | 查找匹配模块 |
| 4 | `ops->Dump()` | `blackbox_core.c:292` | 调用 Dump（可选） |
| 5 | `ops->Reset()` | `blackbox_core.c:297` | 调用 Reset（可选） |
| 6 | `RebootSystem()` | `blackbox_core.c:312` | 系统重启（可选） |

---

## 参考文档

- [01_Overview](01_Overview.md) - 项目概览
- [03_API_Reference](03_API_Reference.md) - API 参考
- [05_Security_Review](05_Security_Review.md) - 安全评审

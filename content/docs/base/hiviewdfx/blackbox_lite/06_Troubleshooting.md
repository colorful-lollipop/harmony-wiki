# 06_Troubleshooting - 常见问题

## 6.1 构建问题

### 6.1.1 编译错误：头文件找不到

**错误信息**：
```
fatal error: 'blackbox.h' file not found
```

**原因**：未正确包含 blackbox_lite 的头文件路径

**解决方案**：

在平台的 `BUILD.gn` 中添加 include_dirs：

```gn
include_dirs += [
    "//base/hiviewdfx/blackbox_lite/interfaces/native/kits",
]
```

**证据**：`BUILD.gn:27`
> `//base/hiviewdfx/blackbox_lite/interfaces/native/kits`

---

### 6.1.2 链接错误：未定义引用

**错误信息**：
```
undefined reference to `BBoxNotifyError'
```

**原因**：未链接 blackbox_lite 静态库

**解决方案**：

在 `BUILD.gn` 中添加 deps：

```gn
deps = [
    "//base/hiviewdfx/blackbox_lite:blackbox_lite",
]
```

---

### 6.1.3 编译警告：函数未实现

**错误信息**：
```
warning: undefined reference to 'SystemModuleDump'
```

**原因**：平台未实现 WEAK 函数

**解决方案**：

创建 `blackbox_adapter_impl.c`，实现所有 WEAK 函数：

```c
#include "blackbox_adapter.h"

void SystemModuleDump(const char *logDir, struct ErrorInfo *info)
{
    // 平台实现
}

void SystemModuleReset(struct ErrorInfo *info)
{
    // 平台实现
}
// ... 其他函数
```

**证据**：`blackbox_adapter.c:25-73`

---

### 6.1.4 依赖组件缺失

**错误信息**：
```
error: component 'liteos_m' not found
```

**原因**：`bundle.json` 中声明的依赖未在构建系统中配置

**解决方案**：

确保在 `build/lite/components/hiviewdfx.json` 中添加：

```json
"blackbox_lite": {
    "optional": false,
    "deps": {
        "components": ["utils_lite", "liteos_m"]
    }
}
```

**证据**：`bundle.json:21-27`

---

## 6.2 运行问题

### 6.2.1 SaveErrorLog 线程未启动

**现象**：开机后日志保存线程未运行

**排查步骤**：

1. 检查 `BBoxInit()` 是否被调用
   ```c
   // blackbox_core.c:318
   static void BBoxInit(void)
   {
       // ...
   }
   CORE_INIT_PRI(BBoxInit, 1);
   ```

2. 检查信号量是否创建成功
   ```c
   if (LOS_BinarySemCreate(1, &g_opsListSem) != LOS_OK) {
       BBOX_PRINT_ERR("Create binary semaphore failed!\n");
       return;
   }
   ```

3. 检查线程创建返回值
   ```c
   ret = pthread_create(&taskId, NULL, SaveErrorLog, NULL);
   if (ret != 0) {
       BBOX_PRINT_ERR("Falied to create SaveErrorLog task, ret: %d\n", ret);
   }
   ```

**可能原因**：
- 内核初始化顺序问题
- 内存不足导致信号量创建失败
- 线程创建失败（资源不足）

---

### 6.2.2 故障通知无响应

**现象**：调用 `BBoxNotifyError()` 后无任何处理

**排查步骤**：

1. 检查模块是否已注册
   ```c
   #ifdef BLACKBOX_DEBUG
   PrintModuleOps();
   #endif
   ```

2. 检查 `g_opsList` 是否为空
   ```c
   if (UtilsListEmpty(&g_opsList)) {
       BBOX_PRINT_ERR("No module registered!\n");
       return;
   }
   ```

3. 检查 `MODULE` 名称是否匹配
   ```c
   if (strcmp(ops->ops.module, module) != 0) {
       continue;  // 名称不匹配，跳过
   }
   ```

**可能原因**：
- `BBoxAdapterInit()` 未调用（优先级问题）
- 模块名称不匹配
- 平台未实现适配函数

---

### 6.2.3 日志未保存到文件

**现象**：SaveErrorLog 线程运行，但文件未生成

**排查步骤**：

1. 检查 `LOG_ROOT_DIR` 是否存在
   ```c
   WaitForLogRootDir(dirName);
   // 等待 1000ms
   ```

2. 检查平台实现
   ```c
   // 需实现以下函数
   int SystemModuleGetLastLogInfo(struct ErrorInfo *info);
   int SystemModuleSaveLastLog(const char *logDir, struct ErrorInfo *info);
   int FullWriteFile(const char *filePath, const char *buf,
       unsigned int bufSize, int isAppend);
   ```

3. 检查文件系统权限

---

## 6.3 调试方法

### 6.3.1 启用调试日志

**方法 1**：编译时添加宏定义

```gn
# BUILD.gn
defines += [ "BLACKBOX_DEBUG" ]
```

**方法 2**：在代码中启用

```c
// 使用 BLACKBOX_DEBUG 宏控制
#ifdef BLACKBOX_DEBUG
PrintModuleOps();
#endif
```

**证据**：`blackbox_core.c:191`

---

### 6.3.2 使用 GDB 调试

**连接设备**：
```bash
arm-none-eabi-gdb
(gdb) target remote localhost:3333
```

**设置断点**：
```gdb
(gdb) break BBoxNotifyError
(gdb) break SaveErrorLog
(gdb) break BBoxRegisterModuleOps
```

**查看变量**：
```gdb
(gdb) print info->event
(gdb) print info->module
(gdb) print info->errorDesc
(gdb) print g_opsList
```

---

### 6.3.3 使用 SystemView 调试

**配置 RTOS Trace**：

1. 在 `SaveErrorLog` 入口添加 trace：
   ```c
   #include "trace.h"
   
   static void* SaveErrorLog(void *param)
   {
       (void)param;
       TRACE_START("SaveErrorLog");
       // ...
   }
   ```

2. 启动 SystemView 捕获任务切换

3. 分析事件流和时序

---

### 6.3.4 内存分析

**检查内存占用**：

```c
// 在关键路径添加内存日志
BBOX_PRINT_INFO("malloc: %p, size: %zu\n", ptr, size);

// 检查堆使用
extern unsigned int LOS_MemGetUsedSpace(void *pHeapMem);
```

---

## 6.4 平台适配检查清单

| 检查项 | 说明 | 状态 |
|--------|------|------|
| `SystemModuleDump` | 系统 Dump 接口 | □ |
| `SystemModuleReset` | 系统复位接口 | □ |
| `SystemModuleGetLastLogInfo` | 获取日志信息 | □ |
| `SystemModuleSaveLastLog` | 保存日志接口 | □ |
| `FullWriteFile` | 文件写操作 | □ |
| `GetFaultLogPath` | 获取日志路径 | □ |
| `RebootSystem` | 系统重启 | □ |
| 链接适配层 | 在 BUILD.gn 中链接 | □ |

---

## 6.5 常见错误码

| 错误码 | 含义 | 可能原因 |
|--------|------|----------|
| -1 | 参数错误 | NULL 指针或空缓冲区 |
| -1 | 内存分配失败 | 系统内存不足 |
| -1 | 模块已注册 | 重复调用 `BBoxRegisterModuleOps` |
| -1 | 信号量获取失败 | 系统负载高 |
| -1 | 文件操作失败 | 路径错误或权限不足 |

**证据**：`blackbox_core.c:213, 220, 225, 272`

---

## 参考文档

- [01_Overview](01_Overview.md) - 项目概览
- [02_Architecture](02_Architecture.md) - 架构说明
- [03_API_Reference](03_API_Reference.md) - API 参考
- [04_Build_Configuration](04_Build_Configuration.md) - 构建配置
- [05_Security_Review](05_Security_Review.md) - 安全评审

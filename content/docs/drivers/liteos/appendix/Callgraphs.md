# 附录：关键调用链

## 概述

本文档描述 hievent 模块的完整调用链路，便于理解数据流向和故障排查。

## 初始化调用链

### HieventInit 初始化

```
LOS_MODULE_INIT()
    │
    └──► HieventInit() [hievent_driver.c:377]
            │
            ├──► HieventDeviceInit() [hievent_driver.c:359]
            │       │
            │       ├──► LOS_MemAlloc(OS_SYS_MEM_ADDR, 1024)
            │       │       └──► 分配环形缓冲区
            │       │
            │       ├──► init_waitqueue_head(&wq)
            │       │       └──► 初始化等待队列
            │       │
            │       └──► LOS_MuxInit(&mtx)
            │               └──► 初始化互斥锁
            │
            └──► register_driver("/dev/hwlog_exception", &g_hieventFops, 0666, &g_hieventDev)
                    └──► 注册字符设备节点
```

**证据**: `hievent_driver.c:359-387`

---

## 事件写入调用链

### 方式 1：用户态 write()

```
用户态 write()
    │
    └──► HieventWrite() [hievent_driver.c:323]
            │
            └──► HieventWriteInternal(buffer, len) [hievent_driver.c:268]
                    │
                    ├──► 参数校验
                    │       ├──► buflen >= sizeof(int)? [line 273]
                    │       ├──► buflen <= 1004? [line 274]
                    │       └──► buffer 非用户地址? [line 281]
                    │
                    ├──► 校验 CHECK_CODE [line 286-290]
                    │       └──► *(int*)buffer == 0x7BCDABCD?
                    │
                    ├──► HieventCoverOldLog(len) [line 293]
                    │       └──► 覆盖旧日志 (环形缓冲区满时)
                    │
                    ├──► HieventHeadInit(&header, len) [line 295]
                    │       ├──► 获取当前时间 [clock_gettime()]
                    │       ├──► 获取 PID [LOS_GetCurrProcessID()]
                    │       └──► 设置 header
                    │
                    ├──► HieventWriteRingBuffer(&header, sizeof(header)) [line 297]
                    │       └──► 写入头部
                    │
                    ├──► HieventWriteRingBuffer(buffer+4, header.len) [line 304]
                    │       └──► 写入负载
                    │
                    └──► wake_up_interruptible(&wq) [line 318]
                            └──► 唤醒等待的读取者
```

**证据**: `hievent_driver.c:268-321`

### 方式 2：HiviewHieventReport()

```
HiviewHieventReport(event)
    │
    ├──► HiviewHieventConvertString(event, &str) [hiview_hievent.c:512]
    │       │
    │       ├──► 填充 eventid [line 447]
    │       ├──► 填充 filePath [line 451-458]
    │       ├──► 填充 time [line 461-464]
    │       └──► HiviewHieventFillPayload() [line 467]
    │               └──► 遍历 Payload 链表，格式化键值对
    │
    └──► HiviewHieventWriteLogException(str, bufLen) [hiview_hievent.c:516]
            │
            └──► LogBufToException(category, level, logType, sn, msg, msglen) [line 484-491]
                    │
                    ├──► 分配缓冲区 [line 356]
                    │       └──► LOS_MemAlloc(OS_SYS_MEM_ADDR, bufLen)
                    │
                    ├──► 写入 CHECK_CODE [line 361-362]
                    │       └──► *(int*)buffer = CHECK_CODE
                    │
                    ├──► 写入 IdapHeader [line 364-368]
                    │       └──► level, category, logType, sn
                    │
                    └──► HieventWriteInternal(buffer, bufLen) [line 376]
                            └──► (进入内核写入流程)
```

**证据**: `hiview_hievent.c:501-521`, `hiview_hievent.c:351-382`

---

## 事件读取调用链

```
用户态 read()
    │
    └──► HieventRead() [hievent_driver.c:160]
            │
            ├──► wait_event_interruptible(wq, size > 0) [line 167]
            │       └──► 阻塞等待数据
            │
            ├──► LOS_MuxAcquire(&mtx) [line 169]
            │       └──► 获取互斥锁
            │
            ├──► HieventReadRingBuffer(&header, sizeof(header)) [line 171]
            │       └──► 读取事件头部
            │
            ├──► 校验缓冲区大小 [line 177-180]
            │       └──► bufLen >= header.len + sizeof(header)?
            │
            ├──► HieventReadRingBuffer(buffer, sizeof(header)) [line 185-186]
            │       └──► 拷贝头部到用户缓冲区
            │
            ├──► HieventReadRingBuffer(buffer+sizeof(header), header.len) [line 192-193]
            │       └──► 拷贝负载到用户缓冲区
            │
            ├──► HieventBufferDec(sizeof(header)) [line 183]
            │       └──► 减少缓冲区占用
            │
            ├──► HieventBufferDec(header.len) [line 199]
            │       └──► 减少缓冲区占用
            │
            └──► LOS_MuxRelease(&mtx) [line 210]
                    └──► 释放互斥锁
```

**证据**: `hievent_driver.c:160-212`

---

## 事件对象生命周期

```
创建
    │
    └──► HiviewHieventCreate(eventid)
            └──► LOS_MemAlloc(sizeof(HiviewHievent))
                    └──► memset_s(event, 0)
                            └──► 返回 event 对象
                                    │
                                    ▼
使用
    │
    ├──► HiviewHieventPutIntegral(event, key, value)
    │       └──► 创建/更新 Payload
    │
    ├──► HiviewHieventPutString(event, key, value)
    │       └──► 创建/更新 Payload
    │
    ├──► HiviewHieventSetTime(event, seconds)
    │       └──► 设置时间戳
    │
    └──► HiviewHieventAddFilePath(event, path)
            └──► 添加路径到数组
                    │
                    ▼
上报
    │
    └──► HiviewHieventReport(event)
            └──► 序列化 → 写入环形缓冲区
                    │
                    ▼
销毁
    │
    └──► HiviewHieventDestroy(event)
            │
            ├──► 遍历释放 Payload 链表
            │
            ├──► 释放 filePath 数组
            │
            └──► LOS_MemFree(event)
```

**证据**: `hiview_hievent.c:153-544`

---

## 关键数据结构流转

```
┌─────────────────────────────────────────────────────────────┐
│                    数据结构转换                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  HiviewHievent (用户态结构体)                               │
│      │                                                       │
│      │ HiviewHieventConvertString()                         │
│      ▼                                                       │
│  "eventid 123 --extra key1:value1;key2:value2;"  (字符串)    │
│      │                                                       │
│      │ LogBufToException()                                   │
│      ▼                                                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ buffer:                                               │   │
│  │   [CHECK_CODE][IdapHeader][msg string]              │   │
│  └─────────────────────────────────────────────────────┘   │
│      │                                                       │
│      │ HieventWriteInternal()                              │
│      ▼                                                       │
│  HieventEntry (内核环形缓冲区结构)                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │   [len][hdrSize][pid][tid][sec][nsec][msg...]       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**证据**: `hievent_driver.c:59-67` (HieventEntry), `hiview_hievent.c:430-470` (ConvertString)

---

*最后更新: 2024*
*基于代码版本: 当前 Git HEAD*

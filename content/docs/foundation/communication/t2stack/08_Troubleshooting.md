# T2Stack 故障排查指南

## 目录

- [1. 常见问题](#1-常见问题)
- [2. 日志与调试](#2-日志与调试)
- [3. 崩溃分析](#3-崩溃分析)
- [4. 性能问题](#4-性能问题)
- [5. 网络问题](#5-网络问题)

---

## 1. 常见问题

### 1.1 构建问题

#### Q1: 编译报错 "bounds_checking_function not found"

**问题**：
```
error: cannot find bounds_checking_function:libsec_shared
```

**原因**：未正确配置外部依赖

**解决方案**：
```bash
# 确保在正确的子系统下编译
hb set
hb build -f

# 或检查依赖配置
cat bundle.json | grep "bounds_checking_function"
```

**代码证据**：`bundle.json:31` - `bounds_checking_function` 在 deps.components 中

---

#### Q2: 链接报错 "cannot find -lcoap"

**问题**：
```
error: cannot find -lcoap
```

**原因**：libcoap 依赖未配置

**解决方案**：
```bash
# 确认 libcoap 子系统已包含
hb set
# 选择包含 libcoap 的产品

# 或检查产品配置
cat device.json | grep "libcoap"
```

**代码证据**：`nstackx_ctrl/BUILD.gn:179` - external_deps 包含 libcoap

---

### 1.2 运行时问题

#### Q3: Fillp 连接失败 "Connection refused"

**问题**：
```
FtConnect() 返回 -1，errno = ECONNREFUSED
```

**排查步骤**：
1. 检查服务端是否已调用 `FtListen()`
2. 检查 IP 地址和端口是否正确
3. 检查防火墙规则

**代码证据**：`fillpinc.h:131-142` - FtConnect 错误处理

**示例代码**：
```c
// 服务端
FtSocket(AF_INET, SOCK_STREAM, IPPROTO_FILLP);
FtBind(fd, (struct sockaddr *)&addr, sizeof(addr));
FtListen(fd, 10);  // 确保已监听

// 客户端
int ret = FtConnect(fd, (struct sockaddr *)&serverAddr, sizeof(serverAddr));
if (ret < 0) {
    int err = FtGetErrno();  // 获取错误码
    printf("Connect failed: %d\n", err);
}
```

---

#### Q4: DFile 文件传输失败 "Session invalid"

**问题**：
```
DFILE_ON_FATAL_ERROR: session invalid
```

**原因**：
- 会话已关闭
- 会话 ID 无效
- 超时导致会话失效

**解决方案**：
```c
// 1. 检查会话 ID 有效性
if (sessionId <= 0) {
    return NSTACKX_EPARAM_INVALID;
}

// 2. 重连机制
void MyMsgReceiver(int32_t sessionId, DFileMsgType msgType, const DFileMsg *msg) {
    if (msgType == DFILE_ON_FATAL_ERROR) {
        // 重建会话
        ReconnectSession();
    }
}
```

---

## 2. 日志与调试

### 2.1 日志开关

**Fillp 日志**：
```c
// 启用调试日志
FillpDebugControl(FILLP_DBGCMD_SET_PRINT_LEVEL, FILLP_DBG_LVL_DEBUG);
```

**DFile 日志**：
```c
// 注册自定义日志回调
NSTACKX_DFileRegisterLogCallback(MyLogCallback);

// 或使用默认日志
NSTACKX_DFileRegisterDefaultLog();
```

**NStackX 日志**：
```c
// 启用用户日志
#ifdef ENABLE_USER_LOG
NSTACKX_DFinderRegisterLog(MyDfinderLog);
#endif
```

### 2.2 日志级别

| 级别 | Fillp | DFile | NStackX |
|------|-------|-------|----------|
| OFF | `DFILE_LOG_LEVEL_OFF` | - | - |
| FATAL | `DFILE_LOG_LEVEL_FATAL` | - | - |
| ERROR | `DFILE_LOG_LEVEL_ERROR` | - | - |
| WARNING | `DFILE_LOG_LEVEL_WARNING` | - | - |
| INFO | `DFILE_LOG_LEVEL_INFO` | - | - |
| DEBUG | `DFILE_LOG_LEVEL_DEBUG` | - | - |

**代码证据**：`nstackx_dfile.h:268-275` - 日志级别枚举

### 2.3 调试命令

**Fillp 调试命令**：
```c
// 查看 socket 信息
FillpDebugControl(FILLP_DBGCMD_SHOW_SOCKET_INFO, NULL);

// 查看初始化资源
FillpDebugControl(FILLP_DBGCMD_SHOW_INIT_RESOURCE, NULL);

// 查看全局配置
FillpDebugControl(FILLP_DBGCMD_SHOW_GLOBAL_CONFIG_RESOURCE, NULL);
```

---

## 3. 崩溃分析

### 3.1 常见崩溃原因

| 错误类型 | 典型表现 | 排查方向 |
|----------|----------|----------|
| **空指针解引用** | SIGSEGV, addr=0x0 | 参数验证 |
| **缓冲区溢出** | SIGABRT/SIGSEGV | 边界检查 |
| **双重释放** | SIGABRT | 内存管理 |
| **栈溢出** | SIGSEGV | 递归调用 |
| **断言失败** | SIGABRT | 逻辑错误 |

### 3.2 使用 hidump 获取崩溃信息

**开启 hidump**：
```c
// Fillp hidump
#define FILLP_ENABLE_DFX_HIDUMPER
// BUILD.gn 中已默认启用
```

**获取 dump**：
```bash
# 查看 hidump 输出
hilog | grep -i "Fillp"
hidump -n t2stack
```

### 3.3 崩溃栈回溯

**使用 addr2line**：
```bash
# 获取崩溃地址
addr2line -e libFillp.so 0x12345678 -f -C
```

---

## 4. 性能问题

### 4.1 性能指标

| 指标 | Fillp | DFile |
|------|-------|-------|
| **吞吐量** | 最高 3.2Gbps | 依赖硬件 |
| **延迟** | < 10ms | < 50ms |
| **CPU 占用** | 1-2 核满载 | 较低 |

### 4.2 性能调优

**Fillp 性能配置**：
```c
// 1. 启用高性能配置
FillpGlobalPreinitExtConfigsSt extConfig = {0};
extConfig.enableDefault10GConfigsForEbackupPdt = 1;
FtConfigSet(FILLP_CONF_INIT_STACK_EXT, &extConfig);

// 2. 调整缓存大小
FtConfigSet(FT_CONF_SEND_CACHE, &(uint32_t){32 * 1024 * 1024});  // 32MB
FtConfigSet(FT_CONF_RECV_CACHE, &(uint32_t){32 * 1024 * 1024});  // 32MB

// 3. 启用 Full CPU 模式
FtConfigSet(FT_CONF_FULL_CPU, &(FILLP_BOOL){FILLP_TRUE});
```

**代码证据**：`fillpinc.h:764-782` - 高性能配置结构

---

## 5. 网络问题

### 5.1 连通性检查

**基本连通性**：
```bash
# Ping 测试
ping 192.168.1.100

# 端口测试
nc -zv 192.168.1.100 8888

# 抓包分析
tcpdump -i any -w capture.pcap port 8888
```

### 5.2 Fillp 特有的网络问题

| 问题 | 症状 | 解决方案 |
|------|------|----------|
| **UDP 包丢失** | 传输速度慢，大量重传 | 检查网络质量 |
| **NAT 穿透失败** | 跨网段无法连接 | 检查 NAT 配置 |
| **防火墙拦截** | 连接建立后立即断开 | 配置防火墙规则 |
| **MTU 不匹配** | 大文件传输失败 | 检查 MTU 设置 |

### 5.3 DFile 特有的网络问题

| 问题 | 症状 | 解决方案 |
|------|------|----------|
| **TCP 连接中断** | DFILE_ON_CONNECT_FAIL | 检查网络稳定性 |
| **多路径失效** | 只有一个路径有流量 | 检查多路径配置 |
| **加密协商失败** | 密钥交换失败 | 检查密钥配置 |

---

## 相关文档

- [API 参考](./03_CAPI_Reference.md) - 错误码说明
- [安全评审](./07_Security_Review.md) - 安全相关问题
- [架构说明](./01_Architecture.md) - 组件交互

---

*文档版本：1.0.0*
*最后更新：2026-02-06*

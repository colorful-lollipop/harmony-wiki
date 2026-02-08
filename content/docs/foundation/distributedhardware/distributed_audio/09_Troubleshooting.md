# 常见问题与故障排除

## 调试方法

### 日志查看

```bash
# 查看分布式音频日志
hilog | grep -i daudio

# 查看特定域日志（DOMAIN=0xD004130）
hilog | grep D004130

# 实时日志
hilog -T
```

### HiDumper 命令

```bash
# 查看分布式音频状态
hidumper -s 4805  # Source 服务
hidumper -s 4806  # Sink 服务

# 查看详细信息
hidumper -s 4805 -a -h
```

### 事件追踪

```bash
# 启用 HiTrace
hitrace -b 10240 -t 10 daudio

# 查看追踪日志
hitrace -p /data/log/hitrace.txt
```

## 常见问题

### 1. 设备无法发现对端音频设备

**症状**: 组网后无法看到对端设备的扬声器/麦克风

**排查步骤**:

1. **检查 SA 服务状态**
```bash
# 检查 SA 是否注册
samgr_tool -l | grep 4805
samgr_tool -l | grep 4806
```

2. **检查设备管理器**
```bash
# 检查设备是否在线
# 查看分布式硬件框架日志
hilog | grep -i distributedhardware
```

3. **检查网络连接**
```bash
# 检查软总线连接
# 查看软总线日志
hilog | grep -i softbus
```

**可能原因**:
- SA 服务未启动
- 设备未建立可信关系
- 软总线连接失败

### 2. 音频播放/录音无声

**症状**: 可以选到分布式设备但无声

**排查步骤**:

1. **检查 HDF 设备注册**
```bash
# 查看系统音频设备
ls /dev/audio/
```

2. **检查传输通道**
```bash
# 查看软总线通道
# 检查是否有数据传输
hilog | grep -i transport
```

3. **检查音频参数**
```bash
# 查看参数协商日志
hilog | grep -i "param\|sample\|channel"
```

**可能原因**:
- 音频参数协商失败
- 传输通道未建立
- HDF 设备未正确注册

### 3. 音频延迟高

**症状**: 音频有显著延迟（>500ms）

**排查步骤**:

1. **检查延迟测试日志**
```bash
# 启用延迟测试（编译时开启）
# 查看延迟统计
hilog | grep -i latency
```

2. **检查网络质量**
```bash
# ping 对端设备
ping <peer_ip>
```

3. **检查抖动缓冲区**
```bash
# 查看队列长度
hilog | grep -i "queue\|jitter"
```

**优化建议**:
- 减小抖动缓冲区大小
- 使用低延迟编解码器
- 优化网络环境

### 4. 服务崩溃

**症状**: daudio 进程崩溃或重启

**排查步骤**:

1. **查看崩溃日志**
```bash
# 查看崩溃信息
hilog | grep -i "crash\|fatal\|abort"
```

2. **查看内存使用**
```bash
# 查看进程内存
dumpsys meminfo daudio
```

3. **使用调试版本**
```bash
# 编译调试版本
# 启用 AddressSanitizer
```

**常见原因**:
- 内存越界访问
- 空指针解引用
- 资源泄漏

### 5. 权限错误

**症状**: 调用 IPC 接口返回权限错误

**排查步骤**:

1. **检查权限配置**
```bash
# 查看进程权限
cat /proc/<pid>/status
```

2. **检查 SA 配置**
```bash
# 查看 SA 配置文件
cat /system/profile/4805.json
cat /system/etc/init/daudio.cfg
```

3. **检查调用者权限**
```bash
# 查看访问令牌
# 应用必须有 ohos.permission.DISTRIBUTED_DATASYNC
```

**修复方法**:
- 确保调用者声明所需权限
- 检查 SELinux 策略
- 验证 SA 配置正确

## 调试代码位置

### 关键日志位置

| 功能 | 文件路径 | 日志级别 |
|------|----------|----------|
| IPC 调用 | services/audiomanager/servicesource/src/daudio_source_stub.cpp | INFO |
| 设备管理 | services/audiomanager/managersource/src/daudio_source_manager.cpp | INFO |
| 数据传输 | services/audiotransport/senderengine/src/av_sender_engine_transport.cpp | DEBUG |
| HDI 回调 | services/audiohdiproxy/src/daudio_hdi_handler.cpp | INFO |
| 错误处理 | common/src/daudio_util.cpp | ERROR |

### 调试宏

```cpp
// common/include/daudio_log.h
#define DHLOGD(fmt, ...) // Debug 日志
#define DHLOGI(fmt, ...) // Info 日志
#define DHLOGW(fmt, ...) // Warning 日志
#define DHLOGE(fmt, ...) // Error 日志
```

### 性能追踪点

```cpp
// common/dfx_utils/include/daudio_hitrace.h
// 启用追踪
HITRACE_METER_NAME(HITRACE_TAG_DISTRIBUTED_AUDIO, "OperationName");

// 常用追踪点
- DAudioSourceManager::EnableDAudio
- DAudioSourceDev::OnWriteData
- AVTransSenderTransport::FeedAudioData
- DSpeakerClient::OnEngineTransDataAvailable
```

## 故障定位流程

```
问题发生
    │
    ├──► 查看日志 ──► hilog | grep -i daudio
    │
    ├──► 确认服务状态 ──► samgr_tool -l | grep 4805/4806
    │
    ├──► 检查设备连接 ──► 软总线/设备管理器日志
    │
    └──► 分析具体模块
            │
            ├──► IPC 问题 ──► 检查 Stub/Proxy 日志
            │
            ├──► 传输问题 ──► 检查 Transport 日志
            │
            ├──► 设备问题 ──► 检查 Manager/Dev 日志
            │
            └──► HDI 问题 ──► 检查 HDF/Proxy 日志
```

## 常用命令速查

```bash
# 1. 查看服务状态
samgr_tool -l | grep daudio

# 2. 查看进程
ps -A | grep daudio

# 3. 查看日志
hilog | grep D004130

# 4. 查看内存
dumpsys meminfo daudio

# 5. 查看线程
ps -T -p <pid>

# 6. 查看打开的文件
ls -la /proc/<pid>/fd

# 7. 查看堆栈
debuggerd <pid>

# 8. 启用详细日志
# 修改代码中 LOG_DOMAIN 级别或编译参数

# 9. HiDumper
hidumper -s 4805 -a -h

# 10. 性能追踪
hitrace -b 10240 -t 10 daudio
```

## 测试验证

### 基础功能测试

```bash
# 1. 启动测试示例（如果有）
/system/bin/audio_distributed_test

# 2. 检查设备列表
# 通过音频框架 API 查询分布式设备
```

### 压力测试

```bash
# 1. 长时间播放/录音测试
# 持续运行 24 小时

# 2. 频繁启停测试
# 循环启用/禁用分布式音频

# 3. 网络波动测试
# 模拟弱网环境
```

## 问题反馈

当遇到无法解决的问题时，请收集以下信息：

1. **日志文件**
   - 完整 hilog 日志
   - 崩溃堆栈（如有）

2. **环境信息**
   - 系统版本
   - 设备型号
   - 组网方式

3. **复现步骤**
   - 具体操作步骤
   - 预期结果
   - 实际结果

4. **配置信息**
   - SA 配置文件
   - 进程配置

---

*文档生成时间: 2025-02-06*

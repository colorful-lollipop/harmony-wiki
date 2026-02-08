# 故障排查指南

## 常见问题

### 1. 播放无声

**可能原因**:
- 音量设置为 0
- 音频输出设备异常
- 文件格式不支持

**排查步骤**:
1. 检查系统音量
2. 检查其他应用是否有声音
3. 检查日志：`hilog | grep -i audio`

**相关日志**:
```
// 文件: media_log.h
MEDIA_LOGE("Audio output failed");
```

### 2. 视频无法渲染

**可能原因**:
- Surface 未正确设置
- 视频格式不支持
- GPU 资源异常

**排查步骤**:
1. 检查 `setDisplaySurface` 是否调用
2. 检查 surface 是否有效
3. 检查日志

### 3. 录制失败

**可能原因**:
- 权限未授予
- 麦克风被占用
- 存储空间不足

**排查步骤**:
1. 检查权限：`hdc shell dumpsys permission ohos.permission.MICROPHONE`
2. 检查麦克风占用：`hdc shell dumpsys audio`
3. 检查存储空间

**权限检查日志**:
```cpp
// 文件: services/utils/media_permission.cpp:39
return Security::AccessToken::AccessTokenKit::VerifyAccessToken(
    tokenCaller, "ohos.permission.MICROPHONE");
```

### 4. IPC 调用失败

**可能原因**:
- Media Service 未启动
- SA 注册异常
- 跨进程通信中断

**排查步骤**:
1. 检查 SA 状态：`hdc shell dumpsys sa media`
2. 检查日志：`hilog | grep -i "OnRemoteRequest"`

### 5. 内存占用过高

**可能原因**:
- 多个播放器实例未释放
- 缓存过大

**排查步骤**:
1. 检查进程内存：`hdc shell dumpsys meminfo <pid>`
2. 检查播放器实例数量

## 日志查看

### 关键日志标签

| 标签 | 模块 | 查看命令 |
|------|-----|---------|
| LOG_CORE | 核心日志 | `hilog | grep "LOG_CORE"` |
| LOG_DOMAIN_PLAYER | 播放器 | `hilog | grep "Player"` |
| LOG_DOMAIN_RECORDER | 录制器 | `hilog | grep "Recorder"` |

### 日志级别

```cpp
// 文件: media_log.h
MEDIA_LOGD("Debug");   // Debug 级别
MEDIA_LOGI("Info");    // Info 级别
MEDIA_LOGW("Warning"); // Warning 级别
MEDIA_LOGE("Error");   // Error 级别
```

## 调试工具

### hdc 命令

```bash
# 查看 Media Service 状态
hdc shell dumpsys sa media

# 查看音频服务
hdc shell dumpsys audio

# 查看权限状态
hdc shell dumpsys permission

# 获取进程 ID
hdc shell pidof media_service
```

## 相关文档

- [安全风险评审](15_Security_Review.md)
- [常见问题](appendix/FAQ.md)

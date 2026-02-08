# 常见问题

## 目的

本文档收集 media_lite 项目开发、构建和运行过程中的常见问题及解决方案。

## 适用范围

- 适用于开发者
- 适用于测试人员
- 适用于运维人员

---

## 构建相关问题

### Q1: 编译失败，提示找不到 `player_impl` 或 `recorder_impl`

**问题描述**：
```
ninja: error: unknown target: player_impl
```

**可能原因**：
- 未正确配置 `enable_media_passthrough_mode`
- 依赖的 media_utils_lite 未编译

**解决方案**：
```bash
# 检查配置
grep "enable_media_passthrough_mode" config.gni

# 如果使用 Passthrough 模式，确保已编译 recorder_impl/player_impl
hb build -f enable_media_passthrough_mode=true media_lite

# 确保 media_utils_lite 已编译
hb build media_utils_lite
```

---

### Q2: 如何切换 Binder 模式和 Passthrough 模式？

**问题描述**：
不确定如何选择运行模式

**解决方案**：
```bash
# Passthrough 模式（性能优先）
hb build -f enable_media_passthrough_mode=true media_lite

# Binder 模式（多进程架构）
hb build -f enable_media_passthrough_mode=false media_lite
```

**说明**：
- Passthrough 模式：客户端直接链接实现库，减少 IPC 开销
- Binder 模式：客户端和服务端分离，通过 IPC 通信

---

### Q3: 如何启用相机支持？

**问题描述**：
Recorder 无法使用相机功能

**解决方案**：
```gn
# 编辑 config.gni，设置 enable_multimedia_camera_lite = true
```

---

## 运行时相关问题

### Q4: media_server 服务启动失败

**问题描述**：
```
Media server initialize failed.
```

**可能原因**：
- 依赖的服务未启动（samgr, camera_server）
- 权限不足
- 配置文件错误

**解决方案**：
```bash
# 检查服务状态
ps -A | grep media

# 查看日志
hdc shell logcat | grep media_server

# 检查权限
hdc shell pm list permission -ohos.permission.MICROPHONE
```

---

### Q5: Player/Recorder 权限被拒绝

**问题描述**：
```
MEDIA_PERMISSION_DENIED
Process can not access microphone.
```

**可能原因**：
- 应用未声明所需权限
- 权限未授予

**解决方案**：
```json
// 在应用配置文件（module.json）中添加权限
{
  "reqPermissions": [
    {
      "name": "ohos.permission.MICROPHONE",
      "reason": "$string:mic_reason"
    },
    {
      "name": "ohos.permission.WRITE_MEDIA",
      "reason": "$string:write_media_reason"
    }
  ]
}

// 安装应用后授予权限
hdc shell pm grant permission <package_name> ohos.permission.MICROPHONE
```

---

### Q6: 播放器无法播放某些格式

**问题描述**：
播放 MP4 文件失败

**可能原因**：
- 格式编解码器不支持
- 文件损坏
- 内存不足

**解决方案**：
```bash
# 检查播放器支持格式
# 查看 media_utils_lite 中的编解码器列表

# 使用测试程序验证
test_play_file_h265 test_file.mp4

# 查看日志定位问题
hdc shell logcat | grep Player
```

---

## 开发相关问题

### Q7: JSI API 调用无效

**问题描述**：
JS 调用 `play()` 无效果

**可能原因**：
- 未先设置 `src` 属性
- 播放器状态错误
- 异常处理错误

**解决方案**：
```javascript
// 确保先设置 src
import audio from '@system.audio';

audio.src = '/data/test.mp3';
audio.play();

// 监听错误
audio.onerror = (error) => {
    console.error('Play error:', error);
};

// 检查状态
audio.getPlayState({success: (state) => {
    console.log('Status:', state.status);
}});
```

---

### Q8: 如何获取播放器的详细错误信息？

**问题描述**：
只知道播放失败，不知道具体原因

**解决方案**：
```bash
# 启用详细日志
hdc shell hilog -b Player -v

# 或重新编译并启用 MEDIA_DEBUG_LOG
# 修改 frameworks/player_lite/BUILD.gn，添加：
# cflags += [ "-DDEBUG" ]
```

---

### Q9: 如何调试 Player/Recorder 服务？

**问题描述**：
服务端问题难以定位

**解决方案**：
```bash
# 查看服务日志
hdc shell logcat | grep PlayerServer
hdc shell logcat | grep RecorderServer

# 附加调试工具
hdc shell logcat -v
hdc shell logcat --pid=<media_server_pid>

# 查看 IPC 调用统计
hdc shell cat /proc/net/samgr/stats
```

---

## 性能相关问题

### Q10: 播放性能差，卡顿

**问题描述**：
播放高码率视频卡顿

**可能原因**：
- 使用 Binder 模式（IPC 开销）
- 系统资源不足
- 解码器性能差

**解决方案**：
```bash
# 切换到 Passthrough 模式
hb build -f enable_media_passthrough_mode=true media_lite

# 降低码率测试
# 检查系统负载
hdc shell top

# 优化 Surface 配置
```

---

### Q11: 录制性能差，丢帧

**问题描述**：
录制时视频帧率不足

**可能原因**：
- CPU 占用过高
- 内存不足
- 存储 I/O 瓶颈

**解决方案**：
```bash
# 检查系统资源
hdc shell free
hdc shell df -h

# 调整编码器参数
# 降低视频分辨率
# 降低帧率
# 降低码率
```

---

## 集成相关问题

### Q12: 如何将 media_lite 集成到自定义系统？

**问题描述**：
需要修改或扩展功能

**解决方案**：

**方案 1：修改源码**
```bash
# 1. 克隆代码
git clone https://gitee.com/openharmony/multimedia_media_lite

# 2. 创建自定义分支
git checkout -b custom_feature

# 3. 修改代码
# 4. 重新编译
hb build -f enable_media_passthrough_mode=true media_lite
```

**方案 2：使用 NDK 接口**
```cpp
// 使用 libmedia_ndk.so 中的接口
#include "player.h"
#include "recorder.h"

// 直接链接 NDK 库
// 无需修改 media_lite 源码
```

---

## 安全相关问题

### Q13: 权限检查失败，返回 MEDIA_PERMISSION_DENIED

**问题描述**：
即使授予权限，仍返回拒绝

**可能原因**：
- 权限名称拼写错误
- 权限未在配置中声明
- permission_lite 服务问题

**解决方案**：
```bash
# 检查权限状态
hdc shell pm list permission

# 查看 permission_lite 日志
hdc shell logcat | grep Permission

# 重启服务
hdc shell killall media_server
# 系统会自动重启服务
```

---

## 代码问题定位

### Q14: 如何定位崩溃位置？

**问题描述**：
应用崩溃，不知道在哪里

**解决方案**：
```bash
# 查看崩溃堆栈
hdc shell hilog -b Media -v

# 查看内核崩溃信息
hdc shell dmesg | grep media

# 如果是崩溃，查看 tombstone
hdc shell ls -la /data/tombstones/
hdc shell cat /data/tombstones/tombstone_XX
```

---

## 工具命令

### 常用命令列表

| 命令 | 说明 | 示例 |
|------|------|------|
| `hdc list targets` | 列出所有设备 | `hdc list targets` |
| `hdc shell <command>` | 执行 shell 命令 | `hdc shell ps -A` |
| `hdc logcat [options]` | 查看日志 | `hdc logcat -b Player` |
| `hdc file send <local> <remote>` | 传输文件 | `hdc file send test.mp3 /sdcard/` |
| `hdc file recv <remote> <local>` | 接收文件 | `hdc file recv /sdcard/test.mp4 ./` |
| `hdc install <hap>` | 安装应用 | `hdc install app.hap` |

---

## 相关跳转

- [项目概览](00_Overview.md)
- [目录结构与模块职责](01_Directory_Structure.md)
- [对外 JSI 接口](03_JSI_Interfaces.md)
- [内部 API](04_Inner_API.md)
- [GN Targets](05_GN_Targets.md)
- [编译产物](06_Build_Artifacts.md)
- [安全风险评审](07_Security_Audit.md)

# 常见问题 (FAQ)

## 1. 构建问题

### Q1: 编译时找不到 midi_framework 目标

**A**: 确保在 `vendor/{vendor}/{product}/config.json` 中添加了依赖:

```json
{
    "components": [
        "midi_framework",
        "drivers_peripheral_midi",
        "drivers_interface_midi"
    ]
}
```

---

### Q2: RK3568 上 USB MIDI 设备列表为空

**A**: 检查以下内容:

1. **内核配置** - 确保已开启 ALSA:
```
CONFIG_SND_USB_AUDIO=y
CONFIG_SND_RAWMIDI=y
CONFIG_SND_SEQUENCER=y
```

2. **权限配置** - 检查 `/system/etc/ueventd.config`:
```
/dev/snd/controlC*    0660   system   audio
/dev/snd/midiC*D*     0660   system   audio
```

3. **服务启动** - 确认 midi_server 进程在运行:
```bash
ps -ef | grep midi_server
```

---

## 2. 运行时问题

### Q3: OH_MIDIClientCreate 返回 IPC_FAILURE

**A**: 可能原因:

1. **服务未启动** - MIDI 服务是按需启动，首次调用会自动拉起
2. **权限问题** - 确保应用有访问系统服务的权限
3. **SELinux** - 检查是否有 AVC denied 日志

调试方法:
```bash
# 查看服务日志
hilog | grep -i midi
```

---

### Q4: OH_MIDISend 返回 WOULD_BLOCK

**A**: 这是正常行为，表示共享内存缓冲区已满:

```cpp
uint32_t written = 0;
OH_MIDIStatusCode ret = OH_MIDISend(device, port, events, count, &written);

if (ret == MIDI_STATUS_WOULD_BLOCK) {
    // 部分事件已写入，等待后继续发送
    usleep(1000);  // 等待 1ms
    // 重新发送未写入的事件
    OH_MIDISend(device, port, events + written, count - written, &written);
}
```

---

### Q5: BLE MIDI 连接失败

**A**: 检查以下内容:

1. **权限声明** - AndroidManifest.xml 或 config.json:
```xml
<uses-permission android:name="ohos.permission.ACCESS_BLUETOOTH" />
```

2. **服务 UUID** - 确保设备使用标准 BLE MIDI UUID:
   - Service: `03B80E5A-EDE8-4B33-A751-6CE34EC4C700`
   - Characteristic: `7772E5DB-3868-4112-A1A9-F2669D106BF3`

3. **MAC 地址格式** - 使用大写、冒号分隔:
```cpp
// 正确
"AA:BB:CC:DD:EE:FF"

// 错误
"aa:bb:cc:dd:ee:ff"  // 小写
"AABBCCDDEEFF"       // 无分隔符
```

---

## 3. 开发调试

### Q6: 如何开启详细日志

**A**: 修改日志级别宏:

```cpp
// interfaces/midi_log.h

// 取消注释以启用 DEBUG 日志
#define MIDI_DEBUG_LOG(fmt, ...) HILOG_DEBUG(LOG_CORE, fmt, ##__VA_ARGS__)
```

然后重新编译:
```bash
./build.sh --product-name rk3568 --build-target midi_framework
```

---

### Q7: 如何查看 MIDI 数据

**A**: 使用 hilog 查看服务端数据转储:

```bash
# 查看 MIDI 数据日志
hilog | grep "DumpMidiEvents"

# 查看共享内存操作
hilog | grep -E "(TryWriteEvents|DrainToBatch)"
```

---

### Q8: 服务自动退出如何排查

**A**: 服务在无客户端连接 60 秒后会自动退出，这是正常行为。

如需调试，可临时修改卸载延迟:

```cpp
// services/server/src/midi_service_controller.cpp:35
constexpr uint32_t UNLOAD_DELAY_DEFAULT_TIME_IN_MS = 60 * 60 * 1000;  // 1小时
```

或在运行时通过测试接口设置:
```cpp
MidiServiceController::GetInstance()->SetUnloadDelay(3600000);  // 1小时
```

---

## 4. 性能优化

### Q9: 如何降低 MIDI 传输延迟

**A**: 优化建议:

1. **使用较小的共享内存** - 默认 8KB，可通过代码调整
2. **避免频繁小数据包** - 批量发送事件
3. **提高线程优先级** - 接收线程使用高优先级
4. **减少日志输出** - 生产环境关闭 DEBUG 日志

---

### Q10: UMP 和 MIDI 1.0 字节流如何转换

**A**: 框架自动处理转换:

- **发送**: 应用始终使用 UMP 格式调用 `OH_MIDISend()`
- **协议适配**: 服务端根据设备能力自动转换
- **接收**: 回调始终收到 UMP 格式数据

如需手动转换，参考:
```cpp
// services/common/src/ump_processor.cpp
UmpProcessor::ConvertMidi1ToUmp(...)   // MIDI 1.0 → UMP
UmpProcessor::ConvertUmpToMidi1(...)   // UMP → MIDI 1.0
```

---

## 5. 其他问题

### Q11: 支持哪些 MIDI 设备

**A**: 支持标准 USB MIDI 和 BLE MIDI 设备:

**USB MIDI**:
- 标准 USB MIDI 接口 (USB Audio Class)
- 需要内核 ALSA 支持

**BLE MIDI**:
- 支持 MIDI over BLE 规范的设备
- 需要蓝牙 4.0+ (BLE)

---

### Q12: 能否播放 MIDI 文件

**A**: **不能**。midi_framework 仅处理实时 MIDI 数据流，不支持:
- MIDI 文件 (.mid) 解析
- 音源合成
- 序列器/播放控制

如需播放 MIDI 文件，需要额外的音源合成器。

---

## 6. 获取帮助

如以上问题无法解决，请:

1. 查看系统日志: `hilog | grep -i midi`
2. 检查版本兼容性
3. 在代码仓库提交 Issue

---

*最后更新: 2026-02-07*

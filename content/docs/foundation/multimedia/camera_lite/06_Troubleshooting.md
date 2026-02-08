# 06 - 常见问题

> Camera_Lite 构建/运行/调试常见问题

---

## 构建问题

### Q1: 编译报错 `undefined reference to 'HalCameraInit'`

**现象**:
```
undefined reference to `HalCameraInit'
undefined reference to `HalCameraGetDeviceNum'
```

**原因**:
HAL层库未正确链接。

**解决**:
1. 确认已编译对应的HAL层代码
2. 检查 `board_name` 是否配置正确
3. 对于 hispark 系列开发板，确保已编译设备相关HAL:
```bash
hb build --product hispark_taurus
```

**代码位置**: `frameworks/BUILD.gn:76-81` 板级特定配置

---

### Q2: 找不到 `camera_lite` target

**现象**:
```
ERROR: target not found: camera_lite
```

**原因**:
hb工具未正确识别组件。

**解决**:
1. 确认在OpenHarmony源码根目录
2. 重新执行 `hb set` 选择产品
3. 检查 `bundle.json` 是否存在且格式正确

**验证**:
```bash
cat foundation/multimedia/camera_lite/bundle.json | grep "name"
# 应输出: "name": "@ohos/camera_lite",
```

---

### Q3: 编译警告 `-Wreorder`

**现象**:
大量初始化顺序警告。

**解决**:
这些警告通常不影响功能，但建议按C++规范修复成员初始化顺序。

---

## 运行问题

### Q4: CameraKit::GetInstance() 返回 nullptr

**现象**:
```cpp
CameraKit *kit = CameraKit::GetInstance();
// kit == nullptr
```

**原因**:
1. 应用缺少 `ohos.permission.CAMERA` 权限
2. 权限管理服务未启动

**解决**:
1. 在 `config.json` 中声明权限:
```json
"reqPermissions": [
  {
    "name": "ohos.permission.CAMERA"
  }
]
```

2. 检查权限服务状态

**代码证据**: `frameworks/camera_kit.cpp:32`
```cpp
if (CheckSelfPermission("ohos.permission.CAMERA") != GRANTED) {
    MEDIA_WARNING_LOG("Process can not access camera.");
    return nullptr;
}
```

---

### Q5: GetCameraIds() 返回空列表

**现象**:
```cpp
std::list<std::string> ids = kit->GetCameraIds();
// ids.empty() == true
```

**原因**:
1. HAL层未正确初始化
2. 相机驱动未加载
3. 设备树配置错误

**排查步骤**:
1. 检查日志:
```bash
hilog | grep -i camera
```

2. 确认HAL初始化成功:
```
Codec module init succeed.
```

3. 检查设备节点:
```bash
ls /dev/camera*
```

---

### Q6: TriggerLoopingCapture 返回 MEDIA_ERR

**现象**:
```cpp
int32_t ret = camera->TriggerLoopingCapture(fc);
// ret == -1 (MEDIA_ERR)
```

**原因**:
1. 未先调用 `Configure()`
2. FrameConfig类型不支持
3. Surface配置错误

**排查**:
1. 确认已调用 `Configure()`:
```cpp
CameraConfig *config = CameraConfig::CreateCameraConfig();
config->SetFrameStateCallback(&callback, &handler);
camera->Configure(*config);  // 必须先配置
```

2. 检查FrameConfig类型:
```cpp
// 正确的类型
FrameConfig fc(FRAME_CONFIG_PREVIEW);  // 预览
FrameConfig fc(FRAME_CONFIG_RECORD);   // 录像
FrameConfig fc(FRAME_CONFIG_CALLBACK); // 回调

// 错误的类型（Capture不支持循环）
FrameConfig fc(FRAME_CONFIG_CAPTURE);  // ❌ 不支持TriggerLoopingCapture
```

**代码证据**: `frameworks/camera_impl.cpp:96-106`

---

### Q7: 预览无画面

**现象**:
预览启动成功但无画面显示。

**原因**:
1. Surface未正确创建
2. Surface尺寸与相机不匹配
3. 显示层配置错误

**排查步骤**:
1. 检查Surface创建:
```cpp
Surface *surface = Surface::CreateSurface();
if (surface == nullptr) {
    // 创建失败
}
```

2. 检查尺寸匹配:
```cpp
// 获取相机支持的分辨率
const CameraAbility *ability = kit->GetCameraAbility(cameraId);
std::list<CameraPicSize> sizes = ability->GetSupportedSizes(CAM_FORMAT_YUV420);
// 使用支持的分辨率创建Surface
```

3. 检查显示层日志:
```bash
hilog | grep -i display
```

---

### Q8: 录像文件无法播放

**现象**:
录像生成的文件无法播放或损坏。

**原因**:
1. 未正确消费Surface数据
2. 编码参数错误
3. 文件写入问题

**解决**:
1. 确保有消费者读取Surface:
```cpp
// 创建 recorder
Recorder *recorder = new Recorder();
recorder->SetVideoSource(VIDEO_SOURCE_SURFACE_YUV);
recorder->SetOutputPath(path);
recorder->Prepare();
recorder->Start();

// 将recorder的surface添加到FrameConfig
fc.AddSurface(*recorderSurface);
```

2. 检查编码格式支持:
```cpp
// 支持的格式
CAM_FORMAT_H264
CAM_FORMAT_H265
CAM_FORMAT_JPEG
```

---

## 调试方法

### 开启详细日志

在代码中添加日志级别:
```cpp
#include "media_log.h"

// 错误日志
MEDIA_ERR_LOG("Error message: %d", errorCode);

// 警告日志
MEDIA_WARNING_LOG("Warning message");

// 信息日志
MEDIA_INFO_LOG("Info message");

// 调试日志
MEDIA_DEBUG_LOG("Debug message");
```

查看日志:
```bash
# 查看所有相机相关日志
hilog | grep -i camera

# 查看错误日志
hilog | grep -i error

# 实时监控
hilog -g
```

---

### 使用 GDB 调试

1. 编译带调试信息的版本:
```gn
# 在 BUILD.gn 中添加
cflags += [ "-g", "-O0" ]
```

2. 启动调试:
```bash
gdb /system/bin/your_app
gdb> set sysroot /path/to/sysroot
gdb> target remote :1234
```

---

### 关键日志检查点

| 组件 | 关键日志 | 含义 |
|------|----------|------|
| CameraKit | `Process can not access camera` | 权限不足 |
| CameraServiceClient | `Camera client initialize success` | 客户端初始化成功 |
| CameraServer | `Create camera callback` | 相机创建回调 |
| CameraDevice | `Camera device start looping capture` | 开始捕获 |
| Codec | `Codec module init succeed` | 编解码器初始化成功 |

---

## 性能问题

### Q9: 预览帧率低

**可能原因**:
1. 分辨率设置过高
2. FPS参数未设置
3. 显示处理耗时

**优化建议**:
```cpp
// 设置合适的FPS
fc.SetParameter(CAM_FRAME_FPS, 30);

// 使用预览专用配置
FrameConfig fc(FRAME_CONFIG_PREVIEW);
```

---

### Q10: 内存占用过高

**排查**:
1. 检查Surface队列大小
2. 确认释放资源:
```cpp
camera->Release();  // 释放相机
delete config;      // 删除配置
```

3. 使用 valgrind 检测内存泄漏:
```bash
valgrind --leak-check=full /system/bin/your_app
```

---

## 兼容性问题

### Q11: Binder模式 vs Passthrough模式

**切换方法**:
在编译时通过gn参数指定:
```bash
gn gen out/Default --args='enable_media_passthrough_mode=true'
```

或在 `config.gni` 中设置:
```gn
enable_media_passthrough_mode = true
```

**注意**:
- Passthrough模式性能更好，但不支持跨进程
- Binder模式支持跨进程，但有IPC开销

---

## 其他问题

### Q12: 如何获取技术支持

1. 查看官方文档:
   - [OpenHarmony文档中心](https://gitee.com/openharmony/docs)
   - [Camera组件说明](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/相机子系统.md)

2. 提交Issue:
   - https://gitee.com/openharmony/multimedia_camera_lite/issues

3. 参考示例:
   - [camera_sample_lite](https://gitee.com/openharmony/applications_sample_camera)

---

## 快速诊断清单

遇到问题时，按以下顺序检查:

- [ ] 权限是否声明 (`ohos.permission.CAMERA`)
- [ ] CameraKit是否获取成功 (非nullptr)
- [ ] 相机列表是否非空
- [ ] 相机能力是否正确获取
- [ ] Surface是否正确创建
- [ ] 是否先调用了 `Configure()`
- [ ] FrameConfig类型是否正确
- [ ] 查看hilog是否有错误信息
- [ ] 检查HAL层是否初始化成功

---

## 参考

- [API参考](02_API_Reference.md) - API使用说明
- [架构说明](01_Architecture.md) - 理解数据流
- [内部接口](03_Inner_API.md) - 实现细节

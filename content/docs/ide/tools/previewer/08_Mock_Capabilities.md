# Mock 能力扩展指南

**目的**: 说明 Previewer 的 Mock 层设计原理，列举已实现的 Mock 能力，并指导如何扩展新的 Mock 能力

**适用范围**: Previewer 组件的开发者、维护者，以及需要自定义 Mock 能力的开发者

**阅读前提**:
- 已阅读 [02_Architecture.md](./02_Architecture.md)
- 已阅读 [03_Communication_Protocol.md](./03_Communication_Protocol.md)
- 熟悉 C++ 编程和 OpenHarmony API

---

## 一、Mock 层设计原理

### 1.1 设计目标

Previewer 的 Mock 层旨在模拟真实设备的系统能力，让 ArkTS/JS 应用在开发环境中能够正常运行，而无需依赖真实硬件。

**核心目标**:
1. **模拟系统能力**: 提供传感器、网络、存储等系统能力的模拟实现
2. **命令响应**: 接收 IDE 发送的命令，更新模拟状态
3. **状态同步**: 将模拟状态的变化通知给 JS 应用
4. **平台抽象**: 为 Rich 和 Lite 两种模式提供统一的 Mock 接口

### 1.2 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                   DevEco Studio IDE                    │
└────────────────────┬────────────────────────────────────┘
                     │ JSON 命令
                     ▼
┌─────────────────────────────────────────────────────────┐
│              CommandLineInterface (CLI)                  │
│                    命令解析与分发                          │
└────────────────────┬────────────────────────────────────┘
                     │ 命令执行
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Mock Layer (交互层)                          │
│  ┌──────────────┬──────────────┬──────────────┐     │
│  │VirtualScreen  │  KeyInput    │ MouseInput   │     │
│  │(虚拟屏幕)      │ (键盘输入)    │ (鼠标输入)    │     │
│  └──────────────┴──────────────┴──────────────┘     │
│  ┌──────────────┬──────────────┬──────────────┐     │
│  │SensorModule  │BatteryModule │LanguageManager│    │
│  │(传感器模拟)    │ (电池模拟)    │ (语言管理)    │     │
│  └──────────────┴──────────────┴──────────────┘     │
└────────────────────┬────────────────────────────────────┘
                     │ 状态变更通知
                     ▼
┌─────────────────────────────────────────────────────────┐
│                 JS Application                           │
│              (ArkTS/JS 应用运行时)                       │
└─────────────────────────────────────────────────────────┘
```

### 1.3 Rich vs Lite Mock 差异

| 维度 | Rich Mock | Lite Mock |
|------|-----------|-----------|
| **架构** | 独立 Mock 模块 | 继承 SoftEngine |
| **实现位置** | `mock/rich/` | `mock/lite/` |
| **屏幕捕获** | 回调接收 RGBA 帧 | 直接 Flush 渲染 |
| **传感器支持** | 基础（语言/地区） | 丰富（心率、气压、步数等） |
| **异步任务** | 引擎内部管理 | AsyncWorkManager |
| **数据共享** | 基础 SharedData | 丰富 SharedData |

**证据**:
```cpp
// Rich VirtualScreen - mock/rich/VirtualScreenImpl.h
class VirtualScreenImpl : public VirtualScreen {
    void Callback(uchar* data, int32_t width, int32_t height);
};

// Lite VirtualScreen - mock/lite/VirtualScreenImpl.h
class VirtualScreenImpl : public OHOS::Graphic::SoftEngine {
    void Flush() override;
};
```

---

## 二、已实现的 Mock 能力

### 2.1 输入设备模拟

#### 2.1.1 键盘输入 (KeyInput)

**文件位置**:
- Rich: `mock/rich/KeyInputImpl.cpp`
- Lite: `mock/lite/KeyInputImpl.cpp`

**功能**:
- 模拟键盘按键按下和释放
- 支持虚拟按键（如 Back、Home、Menu）
- 支持物理按键（通过键名或键码）

**命令接口**:
```json
{
  "version": "1.0.1",
  "type": "action",
  "command": "KeyPress",
  "args": {
    "keyCode": 4,
    "action": 0
  }
}
```

**代码证据**: `cli/CommandLine.cpp` (KeyPressCommand 类)

#### 2.1.2 鼠标/触摸输入 (MouseInput)

**文件位置**:
- Rich: `mock/rich/MouseInputImpl.cpp`
- Lite: `mock/lite/MouseInputImpl.cpp`

**功能**:
- 模拟鼠标/触摸按下、释放、移动
- 支持多点触控
- 支持拖拽手势

**命令接口**:
```json
{
  "version": "1.0.1",
  "type": "action",
  "command": "MousePress",
  "args": {
    "x": 100.5,
    "y": 200.0,
    "type": 0,
    "button": 0,
    "action": 0
  }
}
```

**代码证据**: `cli/CommandLine.cpp` (MousePressCommand 类)

#### 2.1.3 鼠标滚轮/表冠 (MouseWheel)

**文件位置**:
- Rich: `mock/rich/MouseWheelImpl.cpp`
- Lite: `mock/lite/MouseWheelImpl.cpp`

**功能**:
- 模拟鼠标滚轮滚动
- 模拟手表表冠旋转（Lite）

**命令接口**:
```json
{
  "version": "1.0.1",
  "type": "action",
  "command": "CrownRotate",
  "args": {
    "rotation": 10
  }
}
```

### 2.2 虚拟屏幕 (VirtualScreen)

**文件位置**:
- Rich: `mock/rich/VirtualScreenImpl.cpp`
- Lite: `mock/lite/VirtualScreenImpl.cpp`

**功能**:
- 渲染目标缓冲区管理
- RGBA 帧捕获（Rich）
- JPEG 编码和质量控制
- 帧率统计和动态/静态模式切换

**关键参数**:
```cpp
// 图像质量等级
enum class JpgQualityLevel {
    HIGHLEVEL = 100,    // 高质量
    MIDDLELEVEL = 90,   // 中质量
    LOWLEVEL = 85,      // 低质量
    DEFAULTLEVEL = 75   // 默认质量
};

// 帧率控制
const int32_t sendPeriod = 40;  // 每 40ms 发送一帧 (25fps)
```

### 2.3 传感器模拟（Lite 专用）

#### 2.3.1 电池模拟 (BatteryModule)

**文件位置**: `mock/lite/BatteryModuleImpl.cpp`

**功能**:
- 模拟电池电量（0.0-1.0）
- 模拟充电状态（充电/未充电）
- 模拟充电模式（手动/自动）

**命令接口**:
```json
{
  "version": "1.0.1",
  "type": "set",
  "command": "ChargeMode",
  "args": {
    "mode": "charging"
  }
}
```

**代码证据**: `cli/CommandLine.cpp` (ChargeModeCommand 类)

#### 2.3.2 亮度模拟 (BrightnessModule)

**文件位置**: `mock/lite/BrightnessModuleImpl.cpp`

**功能**:
- 模拟屏幕亮度（1-255）
- 模拟亮度模式（手动/自动）
- 亮度变化通知

**初始化**: `ThinPreviewer.cpp:64-66`
```cpp
SharedData<uint8_t>(SharedDataType::BRIGHTNESS_VALUE, 255, 1, 255);
SharedData<uint8_t>(SharedDataType::BRIGHTNESS_MODE, (uint8_t)BrightnessMode::MANUAL,
                    (uint8_t)BrightnessMode::MANUAL, (uint8_t)BrightnessMode::AUTO);
```

#### 2.3.3 地理位置模拟 (GeoLocation)

**文件位置**: `mock/lite/GeoLocation.cpp`

**功能**:
- 模拟经纬度坐标
- 经度范围: -180° 到 180°
- 纬度范围: -90° 到 90°

**初始化**: `ThinPreviewer.cpp:82-85`
```cpp
SharedData<double>(SharedDataType::LONGITUDE, 0, -180, 180);
SharedData<double>(SharedDataType::LATITUDE, 0, -90, 90);
```

#### 2.3.4 传感器模拟 (SensorModule)

**文件位置**: `mock/lite/SensorModuleImpl.cpp`

**功能**:
- 模拟心率（0-255）
- 模拟气压（0-999900）
- 模拟步数（0-999999）

**初始化**: `ThinPreviewer.cpp:72-77`
```cpp
SharedData<uint8_t>(SharedDataType::HEARTBEAT_VALUE, 80, 0, 255);
SharedData<uint32_t>(SharedDataType::SUMSTEP_VALUE, 0, 0, 999999);
SharedData<uint32_t>(SharedDataType::PRESSURE_VALUE, 101325, 0, 999900);
```

### 2.4 系统能力模拟

#### 2.4.1 语言管理 (LanguageManager)

**文件位置**:
- Rich: `mock/rich/LanguageManagerImpl.cpp`
- Lite: `mock/lite/LanguageManagerImpl.cpp`

**功能**:
- 模拟系统语言设置
- 模拟系统地区设置
- 语言变化通知

**命令接口**:
```json
{
  "version": "1.0.1",
  "type": "set",
  "command": "Language",
  "args": {
    "language": "zh_CN"
  }
}
```

**代码证据**: `cli/CommandLine.cpp` (LanguageCommand 类)

#### 2.4.2 系统参数 (HalSysParam - Lite)

**文件位置**: `mock/lite/HalSysParam.cpp`

**功能**:
- 模拟系统参数读取和设置
- 持久化参数存储

#### 2.4.3 异步任务管理 (AsyncWorkManager - Lite)

**文件位置**: `mock/lite/AsyncWorkManager.cpp`

**功能**:
- 管理 JS 异步任务队列
- 任务执行和回调通知

---

## 三、如何扩展新的 Mock 能力

### 3.1 扩展步骤

#### 步骤 1: 确定 Mock 类型

根据要模拟的系统能力，确定需要创建的 Mock 类型：

| Mock 类型 | 适用场景 | 示例 |
|-----------|----------|------|
| **输入设备** | 需要模拟用户输入 | 摄像头、麦克风 |
| **传感器** | 需要读取传感器数据 | 加速度计、陀螺仪 |
| **系统服务** | 需要模拟系统能力 | 网络状态、通知 |
| **状态数据** | 需要持久化状态 | 存储数据、缓存 |

#### 步骤 2: 创建命令类

在 `cli/CommandLine.h/cpp` 中创建对应的命令类：

```cpp
// CommandLine.h
class CameraCommand : public CommandLine {
public:
    void RunSet(const Json2::Value& args) override;
    void RunGet() override;
    void RunAction(const Json2::Value& args) override;
};
```

```cpp
// CommandLine.cpp
void CameraCommand::RunSet(const Json2::Value& args) {
    // 解析参数
    bool enabled = args.GetBool("enabled", false);
    
    // 更新 Mock 状态
    CameraMock::GetInstance().SetEnabled(enabled);
    
    // 通知应用
    // ...
}
```

#### 步骤 3: 注册命令

在 `cli/CommandLineFactory.cpp` 的 `InitCommandMap()` 中注册新命令：

```cpp
void CommandLineFactory::InitCommandMap() {
    // ... 现有命令
    commandMap_["Camera"] = []() { return std::make_unique<CameraCommand>(); };
}
```

#### 步骤 4: 实现 Mock 类

在相应的 `mock/rich/` 或 `mock/lite/` 目录中创建 Mock 实现类：

```cpp
// mock/lite/CameraMock.h
class CameraMock {
public:
    static CameraMock& GetInstance();
    void SetEnabled(bool enabled);
    bool IsEnabled() const;
    void CaptureFrame();  // 模拟拍照
    
private:
    CameraMock() = default;
    ~CameraMock() = default;
    bool enabled_ = false;
};
```

```cpp
// mock/lite/CameraMock.cpp
CameraMock& CameraMock::GetInstance() {
    static CameraMock instance;
    return instance;
}

void CameraMock::SetEnabled(bool enabled) {
    enabled_ = enabled;
    // 通知状态变更
}

void CameraMock::CaptureFrame() {
    // 生成模拟图像帧
    // 通知应用
}
```

#### 步骤 5: 添加到构建系统

在相应的 `mock/BUILD.gn` 中添加新源文件：

```gn
# Lite Mock BUILD.gn
source_set("mock_lite") {
    sources = [
        # ... 现有源文件
        "lite/CameraMock.cpp",
    ]
    include_dirs = [ ... ]
}
```

#### 步骤 6: 实现状态共享（如需要）

如果需要在主线程和 JS 线程之间共享状态，使用 `SharedData`：

```cpp
// ThinPreviewer.cpp (InitSharedData)
SharedData<bool>(SharedDataType::CAMERA_ENABLED, false);

// Mock 实现
CameraMock::GetInstance().SetEnabled(
    SharedData<bool>::GetData(SharedDataType::CAMERA_ENABLED)
);

// 状态变更通知
SharedData<bool>::SetData(SharedDataType::CAMERA_ENABLED, true);
```

### 3.2 扩展示例：网络状态模拟

假设我们需要模拟网络连接状态，按照上述步骤：

#### 3.2.1 创建命令类

```cpp
// cli/CommandLine.h
class NetworkCommand : public CommandLine {
public:
    void RunSet(const Json2::Value& args) override;
    void RunGet() override;
};

// cli/CommandLine.cpp
void NetworkCommand::RunSet(const Json2::Value& args) {
    std::string state = args.GetString("state", "connected");
    
    if (state == "connected") {
        NetworkMock::GetInstance().SetState(NetworkState::CONNECTED);
    } else if (state == "disconnected") {
        NetworkMock::GetInstance().SetState(NetworkState::DISCONNECTED);
    }
}

void NetworkCommand::RunGet() {
    NetworkState state = NetworkMock::GetInstance().GetState();
    // 返回状态给 IDE
}
```

#### 3.2.2 注册命令

```cpp
// cli/CommandLineFactory.cpp
void CommandLineFactory::InitCommandMap() {
    commandMap_["Network"] = []() { return std::make_unique<NetworkCommand>(); };
}
```

#### 3.2.3 实现 Mock 类

```cpp
// mock/lite/NetworkMock.h
enum class NetworkState { UNKNOWN, CONNECTED, DISCONNECTED };

class NetworkMock {
public:
    static NetworkMock& GetInstance();
    void SetState(NetworkState state);
    NetworkState GetState() const;
    
private:
    NetworkMock() = default;
    std::atomic<NetworkState> state_{NetworkState::UNKNOWN};
};
```

#### 3.2.4 命令示例

```json
{
  "version": "1.0.1",
  "type": "set",
  "command": "Network",
  "args": {
    "state": "disconnected"
  }
}
```

---

## 四、Mock 配置文件

### 4.1 mock-config.json

**用途**: 配置 Mock 层的初始状态和行为

**解析位置**: `jsapp/rich/external/StageContext.cpp:452`

**格式示例**:
```json
{
  "version": "1.0.0",
  "deviceType": "liteWearable",
  "mockCapabilities": {
    "sensor": {
      "heartRate": {
        "enabled": true,
        "defaultValue": 80
      },
      "accelerometer": {
        "enabled": false
      }
    },
    "system": {
      "language": {
        "enabled": true,
        "defaultValue": "zh_CN"
      }
    }
  }
}
```

### 4.2 system_capability.json

**用途**: 描述设备支持的系统能力

**解析位置**: `mock/SystemCapability.cpp:54`

**格式示例**:
```json
{
  "deviceType": "liteWearable",
  "capabilities": [
    "heartRate",
    "battery",
    "brightness",
    "location",
    "language",
    "storage"
  ],
  "apiLevel": 9
}
```

---

## 五、调试和测试

### 5.1 Mock 调试技巧

#### 5.1.1 日志输出

使用 `ILOG` 宏输出 Mock 状态变化：

```cpp
void CameraMock::SetEnabled(bool enabled) {
    ILOG("Camera enabled: %d", enabled);
    enabled_ = enabled;
}
```

#### 5.1.2 单元测试

为 Mock 类创建单元测试：

```cpp
// test/unittest/mock_lite/CameraMockTest.cpp
TEST(CameraMockTest, SetEnabled) {
    CameraMock::GetInstance().SetEnabled(true);
    EXPECT_TRUE(CameraMock::GetInstance().IsEnabled());
    
    CameraMock::GetInstance().SetEnabled(false);
    EXPECT_FALSE(CameraMock::GetInstance().IsEnabled());
}
```

#### 5.1.3 命令测试

通过命令行接口测试 Mock 命令：

```bash
# 发送 JSON 命令测试 Camera Mock
echo '{"version":"1.0.1","type":"set","command":"Camera","args":{"enabled":true}}' > /tmp/./myproject_commandPipe
```

### 5.2 Fuzz 测试

对 Mock 参数解析进行 Fuzz 测试：

```cpp
// test/fuzztest/cameraparams_fuzzer/CameraParamsFuzzer.cpp
extern "C" int LLVMFuzzerTestOneInput(const uint8_t* data, size_t size) {
    std::string jsonStr(reinterpret_cast<const char*>(data), size);
    Json2::Value jsonData = JsonReader::ParseJsonData2(jsonStr);
    
    // 测试命令解析
    CameraCommand command;
    if (jsonData.IsMember("args")) {
        command.RunSet(jsonData["args"]);
    }
    
    return 0;
}
```

---

## 六、常见问题

### Q1: 如何确定是 Rich 还是 Lite Mock？

**A**: 根据 `CommandParser` 的 `deviceType` 参数和 BUILD.gn 的编译目标选择。

```cpp
// CommandParser.cpp
if (deviceType == "liteWearable" || deviceType == "smartVision") {
    // 使用 Lite Mock
} else {
    // 使用 Rich Mock
}
```

### Q2: Mock 状态如何同步到 JS 应用？

**A**: 有两种方式：

1. **SharedData**: 适用于 Rich/Lite 都需要的共享状态
2. **回调通知**: 适用于特定能力的状态变化

```cpp
// SharedData 通知
SharedData<bool>::SetData(SharedDataType::BRIGHTNESS_VALUE, 200);

// 会触发注册的回调
SharedData<uint8_t>::AppendNotify(SharedDataType::BRIGHTNESS_VALUE,
                                  TimerTaskHandler::CheckBrightnessValueChanged, curThreadId);
```

### Q3: Mock 性能如何优化？

**A**: 

1. **减少不必要的状态同步**: 只在状态真正变化时通知
2. **使用原子操作**: 对共享状态使用 `std::atomic`
3. **批量通知**: 将多个状态变更合并为一次通知
4. **延迟通知**: 对于高频变化，使用定时器批量处理

### Q4: 如何模拟异步操作？

**A**: 使用 `AsyncWorkManager` (Lite) 或 `EventHandler` (Rich)：

```cpp
// Lite AsyncWorkManager
AsyncWorkManager::GetInstance().PostTask([]() {
    // 异步执行的任务
    CameraMock::GetInstance().CaptureFrame();
});

// Rich EventHandler
EventHandler::GetInstance().PostTask([]() {
    // 异步执行的任务
});
```

---

## 七、参考资料

### 相关文档
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [03_Communication_Protocol.md](./03_Communication_Protocol.md) - 通信协议
- [07_Security_Review.md](./07_Security_Review.md) - 安全风险

### 代码位置
- Mock 基类: `mock/VirtualScreen.h`, `mock/KeyInput.h`, `mock/MouseInput.h`
- Rich Mock 实现: `mock/rich/`
- Lite Mock 实现: `mock/lite/`
- 命令类: `cli/CommandLine.h`, `cli/CommandLine.cpp`
- 命令工厂: `cli/CommandLineFactory.cpp`

### 外部参考
- [OpenHarmony JS API 文档](https://gitee.com/openharmony/docs/blob/master/en/application-dev/uitool/arkui-overview.md)
- [ArkUI 事件处理](https://gitee.com/openharmony/docs/blob/master/en/application-dev/ui/arkts-event.md)

---

**最后更新**: 2026-02-07
**维护者**: Previewer 团队
**反馈**: 请通过 Issue 或 PR 反馈问题和建议

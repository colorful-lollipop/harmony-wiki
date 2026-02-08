# HDI 接口定义规范

## 概述

本文档描述 OpenHarmony HDI（Hardware Device Interface）的 IDL（Interface Definition Language）语法规范、版本管理规则和最佳实践。

## IDL 语法规范

### 包声明

所有 IDL 文件必须以包声明开头，使用以下格式：

```idl
package ohos.hdi.<module>.<version>;
```

**示例**:
```idl
package ohos.hdi.audio.v1_0;
package ohos.hdi.sensor.v3_0;
package ohos.hdi.camera.v1_5;
```

**规则**:
- 包名必须与目录路径匹配
- 版本号使用 `v<major>_<minor>` 格式
- 所有字母小写

---

### 导入语句

使用其他 IDL 包中定义的类型或接口：

```idl
import ohos.hdi.<module>.<version>.<Interface>;
```

**示例**:
```idl
import ohos.hdi.audio.v1_0.AudioTypes;
import ohos.hdi.audio.v1_0.IAudioAdapter;
```

---

### 接口定义

#### 基本语法

```idl
interface IInterfaceName {
    ReturnType MethodName([in] Type param1, [out] Type param2);
}
```

**规则**:
- 接口名必须以 `I` 开头（大驼峰）
- 每个方法以分号结尾

**示例** (`audio/v1_0/IAudioManager.idl:43-85`):
```idl
interface IAudioManager {
    GetAllAdapters([out] struct AudioAdapterDescriptor[] descs);
    LoadAdapter([in] struct AudioAdapterDescriptor desc, [out] IAudioAdapter adapter);
    UnloadAdapter([in] String adapterName);
    ReleaseAudioManagerObject();
}
```

#### 接口继承

新版本接口可以继承旧版本接口：

```idl
interface INewInterface extends ohos.hdi.module.v1_0.IOldInterface {
    NewMethod();
}
```

**示例** (`camera/v1_5/ICameraHost.idl`):
```idl
interface ICameraHost extends ohos.hdi.camera.v1_3.ICameraHost {
    OpenCamera_V1_5([in] String cameraId, [in] ICameraDeviceCallback callbackObj, [out] ICameraDevice device);
    OpenSecureCamera_V1_5([in] String cameraId, [in] ICameraDeviceCallback callbackObj, [out] ICameraDevice device);
}
```

---

### 回调接口

#### 定义语法

使用 `[callback]` 标记声明回调接口：

```idl
[callback] interface ICallbackName {
    CallbackMethod([in] Type param);
}
```

**示例** (`sensor/v3_0/ISensorCallback.idl`):
```idl
[callback] interface ISensorCallback {
    OnDataEvent([in] struct HdfSensorEvents event);
    [oneway] OnDataEventAsync([in] struct HdfSensorEvents[] events);
}
```

#### 单向调用

使用 `[oneway]` 标记表示单向调用（不等待返回）：

```idl
[oneway] OnDataEventAsync([in] struct HdfSensorEvents[] events);
```

---

### 数据类型定义

#### 枚举

```idl
enum EnumName {
    VALUE1 = 0,
    VALUE2 = 1,
    VALUE3,  // 自动递增为 2
};
```

**示例** (`audio/v1_0/AudioTypes.idl:35-76`):
```idl
enum AudioPortDirection {
    PORT_OUT    = 1,
    PORT_IN     = 2,
    PORT_OUT_IN = 3,
};

enum AudioCategory {
    AUDIO_IN_MEDIA         = 0,
    AUDIO_IN_COMMUNICATION = 1,
    AUDIO_IN_RINGTONE      = 2,
    AUDIO_IN_CALL          = 3,
};
```

#### 枚举继承

新版本枚举可以继承旧版本枚举：

```idl
enum EnumName : ohos.hdi.module.v1_0.EnumName {
    NEW_VALUE = 10,
};
```

**示例** (`camera/v1_5/Types.idl`):
```idl
enum OperationMode : ohos.hdi.camera.v1_3.OperationMode {
    STITCHING_PHOTO = 23,
    CINEMATIC_VIDEO = 24,
};
```

#### 结构体

```idl
struct StructName {
    Type field1;
    Type field2;
    Type[] arrayField;
};
```

**示例** (`sensor/v3_0/SensorTypes.idl`):
```idl
struct HdfSensorInformation {
    String sensorName;
    String vendorName;
    float maxRange;
    float accuracy;
    struct DeviceSensorInfo deviceSensorInfo;
};

struct HdfSensorEvents {
    struct DeviceSensorInfo deviceSensorInfo;
    int version;
    long timestamp;
    unsigned char[] data;
};
```

#### 序列化声明

使用 `sequenceable` 声明可跨进程序列化的类型：

```idl
sequenceable OHOS.HDI.Display.HdifdParcelable;
sequenceable ohos.hdi.camera.v1_0.BufferHandleSequenceable;
```

---

### 参数方向标记

| 标记 | 含义 | 说明 |
|-----|------|------|
| `[in]` | 输入参数 | 从调用方传递给被调用方 |
| `[out]` | 输出参数 | 从被调用方返回给调用方 |
| `[inout]` | 输入输出参数 | 双向传递 |

**示例**:
```idl
// 输入参数: sensorId
// 输出参数: info
GetSensorInfo([in] int sensorId, [out] struct SensorInfo info);
```

---

### 基本类型映射

| IDL 类型 | C 语言 | C++ 语言 | 说明 |
|---------|--------|---------|------|
| `void` | `void` | `void` | 空类型 |
| `boolean` | `bool` | `bool` | 布尔值 |
| `byte` | `int8_t` | `int8_t` | 单字节 |
| `short` | `int16_t` | `int16_t` | 短整型 |
| `int` | `int32_t` | `int32_t` | 整型 |
| `long` | `int64_t` | `int64_t` | 长整型 |
| `float` | `float` | `float` | 单精度浮点 |
| `double` | `double` | `double` | 双精度浮点 |
| `String` | `const char*` | `std::string` | 字符串 |
| `Type[]` | `Type*` | `std::vector<Type>` | 数组 |

---

## 版本管理规范

### 版本号格式

使用语义化版本号 `[major].[minor]`：

```
v<major>_<minor>  →  v1_0, v2_1, v3_0
```

### 版本兼容性规则

| 变更类型 | 版本影响 | 兼容性 |
|---------|---------|--------|
| 新增接口方法 | Minor | 兼容 |
| 新增可选参数 | Minor | 兼容 |
| 新增枚举值 | Minor | 兼容 |
| 删除/修改接口 | Major | 不兼容 |
| 删除/修改参数 | Major | 不兼容 |
| 改变方法语义 | Major | 不兼容 |

### 版本目录结构

```
module/
├── v1_0/           # 初始版本
├── v1_1/           # 向后兼容更新
├── v1_2/           # 向后兼容更新
├── v2_0/           # 不兼容更新
└── v3_0/           # 不兼容更新
```

---

## IDL 文件组织

### 标准文件命名

| 文件名 | 用途 |
|-------|------|
| `I<Module>Interface.idl` | 主接口定义 |
| `I<Module>Callback.idl` | 回调接口定义 |
| `<Module>Types.idl` | 数据类型定义 |

**示例** (Audio 模块):
```
audio/v1_0/
├── IAudioManager.idl      # 主接口
├── IAudioAdapter.idl      # 适配器接口
├── IAudioRender.idl       # 渲染接口
├── IAudioCapture.idl      # 采集接口
├── IAudioCallback.idl     # 回调接口
└── AudioTypes.idl         # 数据类型
```

### 复杂模块示例

**Camera 模块** (`camera/v1_5/`):
```
├── ICameraHost.idl              # 主机管理接口
├── ICameraDevice.idl            # 设备操作接口
├── IStreamOperator.idl         # 流操作接口
├── IStreamOperatorCallback.idl  # 流操作回调
├── IImageProcessService.idl     # 图像处理
└── Types.idl                    # 数据类型
```

---

## 最佳实践

### 1. 注释规范

使用 Javadoc 风格注释：

```idl
/**
 * @brief 接口的简要描述
 *
 * 详细描述...
 *
 * @since 版本号
 * @version 版本号
 */
interface IInterface {
    /**
     * @brief 方法简要描述
     *
     * @param paramName 参数说明
     * @return 返回值说明
     * @since 版本号
     */
    Method([in] Type param);
}
```

### 2. 错误码处理

- 返回 `int32_t` 类型表示错误码
- 0 表示成功，负值表示错误
- 正值可能表示部分成功或状态值

**示例**:
```idl
// 返回 0 成功，负值表示错误
int32_t Enable([in] int sensorId);
int32_t Disable([in] int sensorId);
```

### 3. 异步处理

- 使用回调接口实现异步通知
- 或使用 `[oneway]` 进行单向调用

```idl
[callback] interface ISensorCallback {
    OnData([in] struct SensorEvent event);  // 异步数据回调
};

interface ISensorInterface {
    Register([in] int sensorId, [in] ISensorCallback callback);
};
```

---

## 编译生成产物

IDL 文件通过 `hdi()` 模板编译后生成：

```
.idl 文件
    ↓
┌───────────────────────────────────────┐
│  hdi() 编译模板                        │
│  - interface.gni                      │
│  - hdi.gni / hdi_small.gni / hdi_mini │
└───────────────────────────────────────┘
    ↓
┌───────────────────────────────────────┐
│  生成的代码                             │
│  - Proxy (客户端代理)                   │
│  - Stub (服务端存根)                    │
│  - 接口头文件 (*.h)                     │
│  - Driver 模板代码                      │
└───────────────────────────────────────┘
```

---

## 常见问题

### Q1: 如何添加新接口而不破坏兼容性？

**答**: 在 Minor 版本中新增方法，不修改现有方法：

```idl
// v1_0
interface IFoo {
    Method1();
};

// v1_1 (兼容)
interface IFoo {
    Method1();
    Method2();  // 新增，不影响 Method1
};
```

### Q2: 如何处理大对象传输？

**答**: 使用数组类型或 Parcelable 序列化：

```idl
// 大量数据
ReadData([in] int sensorId, [out] unsigned char[] data);

// 复杂对象
ReadFrame([out] struct VideoFrame frame);
```

### Q3: 回调接口有哪些约束？

**答**:
- 回调接口必须使用 `[callback]` 标记
- 回调对象通常通过 `[in]` 参数传递
- 支持 `[oneway]` 进行异步单向回调

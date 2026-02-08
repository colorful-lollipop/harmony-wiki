# 安全风险评审

本文档对 `drivers/peripheral` 进行系统性的安全风险分析。

## 6.1 威胁模型

### 6.1.1 外部输入源

| 输入类型 | 来源 | 处理模块 |
|----------|------|----------|
| 用户空间调用 | System Services | 各 HDI 接口 |
| 硬件中断 | 外设设备 | HAL 层 |
| 配置文件 | 文件系统 | 各模块配置解析 |
| 网络数据 | WiFi/蓝牙/USB | 通信模块 |

### 6.1.2 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  硬件层 (不可信)                                        │   │
│  │  - 设备中断可能伪造                                    │   │
│  │  - 寄存器值可能被篡改                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                     HAL 层 (需要验证)                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  用户空间 (不可信)                                     │   │
│  │  - System Service 调用需要权限检查                      │   │
│  │  - 参数校验必须严格                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6.2 攻击面分析

### 6.2.1 攻击面清单

| 攻击面 | 描述 | 风险等级 | 相关模块 |
|--------|------|----------|----------|
| **N-API/IPC 接口** | 外部进程调用 HDI 接口 | 高 | 所有模块 |
| **参数解析** | 接收并解析外部数据 | 高 | 所有模块 |
| **文件操作** | 读取配置文件、设备节点 | 中 | battery, thermal |
| **内存操作** | 缓冲区分配与访问 | 高 | audio, input, sensor |
| **设备节点** | `/dev` 节点读写 | 中 | input, camera |
| **系统能力** | 访问敏感硬件资源 | 高 | fingerprint, face_auth |
| **动态库加载** | `dlopen()` 加载外部库 | 高 | codec, display, camera |
| **命令执行** | `popen()` 执行 shell | 极高 | usb (测试代码) |

### 6.2.2 设备节点访问清单

| 模块 | 设备节点 | 访问方式 | 风险等级 | 证据位置 |
|-----|---------|---------|---------|---------|
| Display | `/dev/dri/card0` | open+ioctl | 高 | `display/composer/vdi_base/src/drm_device.cpp:36` |
| Display | `/dev/graphics/fb0` | open+mmap | 高 | `display/composer/vdi_base/src/drm_display.cpp:116` |
| Camera | `/dev/video*` | open+v4l2 ioctl | 高 | `camera/vdi_base/common/adapter/platform/v4l2/.../v4l2_dev.cpp:402` |
| USB | `/dev/bus/usb/*` | open+usbfs | 高 | `usb/hdi_service/src/usbd_ports.cpp:207,417` |
| Input | `/dev/uinput` | open+ioctl | 高 | `input/ddk_service/src/emit_event_manager/virtual_device.cpp:88` |
| Power | `/sys/power/state` | open+write | 中 | `power/interfaces/hdi_service/src/power_interface_impl.cpp:286` |
| Thermal | `/sys/class/thermal/*` | open+write | 中 | `thermal/interfaces/hdi_service/src/thermal_simulation_node.cpp:88` |

### 6.2.3 动态库加载风险

| 模块 | 加载的库 | 用途 | 风险等级 | 证据位置 |
|-----|--------|------|---------|---------|
| Codec | `libcodec_heif_vdi.so` | HEIF 编解码 | 高 | `codec/image/heif/src/codec_heif_encode_service.cpp:48` |
| Display | `libdisplay_buffer.z.so` | 显示缓冲 | 高 | `display/buffer/hdi_service/src/allocator_service.cpp:82` |
| Camera | 厂商 HAL 库 | Camera 硬件 | 高 | `camera/hal_c/hdi_cif/src/camera_host.cpp:133` |
| Secure Element | 代理库 | 安全元件 | 高 | `secure_element/secure_element_ca_proxy/.../secure_element_ca_proxy.cpp:46` |
| Audio | 插件库 | 音频适配 | 高 | `audio/test/fuzztest/.../*_fuzzer.cpp` (测试代码) |

### 6.2.4 高危代码位置汇总

基于代码扫描发现的高危操作位置：

**USB 模块** (设备读写):
- `usb/hdi_service/src/usbd_ports.cpp:207` - `read(fd, buff, PATH_MAX-1)`
- `usb/hdi_service/src/usbd_ports.cpp:417` - `write(fd, data.c_str(), data.size())`
- `usb/hdi_service/src/usb_device_impl.cpp:405-504` - 设备配置读写

**Camera 模块** (设备操作):
- `camera/vdi_base/common/adapter/platform/v4l2/.../v4l2_uvc.cpp:48` - `open(name, O_RDWR)`
- `camera/vdi_base/common/adapter/platform/v4l2/.../v4l2_dev.cpp:402` - 启动工作线程
- `camera/vdi_base/common/adapter/platform/v4l2/.../v4l2_uvc.cpp:391` - `write(eventFd_, &one)`

**Display 模块** (图形设备):
- `display/composer/vdi_base/src/drm_device.cpp:36` - `open("/dev/dri/card0")`
- `display/composer/vdi_base/src/drm_display.cpp:116` - `open("/dev/graphics/fb0")`
- `display/composer/vdi_base/src/drm_connector.cpp:129` - `open("/sys/class/backlight/...")`

**Power/Thermal 模块** (系统状态):
- `power/interfaces/hdi_service/src/power_interface_impl.cpp:286` - `open(SUSPEND_STATE_PATH)`
- `power/interfaces/hdi_service/src/hibernate.cpp:175` - `open(SWAP_FILE_PATH)`
- `thermal/interfaces/hdi_service/src/thermal_simulation_node.cpp:88` - `O_CREAT|O_RDWR`

---

## 6.3 安全风险点

### 6.3.1 输入验证不足

**风险 ID**: SEC-001

**描述**: 部分 HDI 接口未对输入参数进行充分验证

**证据位置**:
- `input/interfaces/include/input_controller.h:13-15`
- `sensor/interfaces/include/sensor_if.h`

**问题代码**:
```c
// input_controller.h - 未校验长度参数
int32_t (*GetChipInfo)(uint32_t devIndex, char *chipInfo, uint32_t length);

// 调用方可能传入：
// - length = 0 (除零错)
// - length > 实际缓冲区大小 (缓冲区溢出)
```

**触发条件**:
1. 调用者传入异常 `length` 参数
2. HAL 实现未检查 `length` 有效性

**影响**:
- 缓冲区溢出
- 内存破坏
- 提权漏洞

**修复建议**:
```c
int32_t GetChipInfo(uint32_t devIndex, char *chipInfo, uint32_t length)
{
    // 参数校验
    if (chipInfo == NULL) {
        return INPUT_ERR_NULL_PTR;
    }
    if (length == 0 || length > MAX_CHIP_INFO_LEN) {
        return INPUT_ERR_INVALID_PARAM;
    }
    
    // 内部实现
}
```

---

### 6.3.2 路径遍历风险

**风险 ID**: SEC-002

**描述**: 配置文件路径未校验，可能导致任意文件读写

**证据位置**: `audio/hal/hdi_passthrough/src/audio_common.c:155`

```c
// audio_common.c:155 - mkdir 使用外部输入的路径
mkdir(folderName, 0770); // 0770: restore permission
```

**触发条件**:
1. `folderName` 来自外部输入
2. 包含 `../` 或绝对路径

**影响**:
- 任意目录创建
- 权限配置错误
- 符号链接攻击

**修复建议**:
```c
// 验证路径不包含 ../ 且为相对路径
if (strstr(folderName, "../") != NULL || folderName[0] == '/') {
    return AUDIO_ERR_INVALID_PARAM;
}
```

---

### 6.3.3 整数溢出

**风险 ID**: SEC-003

**描述**: 内存大小计算可能存在整数溢出

**问题场景**:
```c
// 传感器数据缓冲区分配
int32_t bufferSize = sensorCount * sizeof(SensorData);
SensorData *buffer = malloc(bufferSize);

// 如果 sensorCount 来自外部输入且极大：
// - 可能导致整数溢出 (wraparound)
// - 分配过小缓冲区
// - 后续写入时缓冲区溢出
```

**触发条件**:
1. `sensorCount` 来自不可信源
2. 未检查最大值限制

**影响**:
- 缓冲区溢出
- 拒绝服务

**修复建议**:
```c
#define MAX_SENSOR_COUNT 100

if (sensorCount > MAX_SENSOR_COUNT || sensorCount == 0) {
    return SENSOR_ERR_INVALID_PARAM;
}

// 检查乘法溢出
if (sensorCount > SIZE_MAX / sizeof(SensorData)) {
    return SENSOR_ERR_INVALID_PARAM;
}
```

---

### 6.3.4 竞态条件

**风险 ID**: SEC-004

**描述**: 设备打开/关闭操作存在 TOCTOU (Time-of-Check-Time-of-Use) 竞态

**证据位置**: `input/interfaces/include/input_manager.h`

```c
// input_manager.h - 典型的 TOCTOU 问题
int32_t (*OpenInputDevice)(uint32_t devIndex);
int32_t (*CloseInputDevice)(uint32_t devIndex);
```

**问题场景**:
```
时间线:
T1: Check device state (available)
T2: Context switch
T3: Other process closes device
T4: Reopen by attacker with different permissions
T5: Original process uses device (权限已变化)
```

**影响**:
- 权限绕过
- 资源混淆

**修复建议**:
```c
// 使用文件描述符而非索引
// 在内核态完成打开操作
int32_t OpenInputDevice(uint32_t devIndex, int32_t *fd)
{
    int32_t fd = -1;
    int32_t ret = OsalOpenDevice(devIndex, &fd);
    if (ret != INPUT_SUCCESS) {
        return ret;
    }
    
    // 验证设备权限
    if (!CheckDevicePermission(fd)) {
        OsalCloseDevice(fd);
        return INPUT_ERR_PERMISSION_DENIED;
    }
    
    *fd = fd;
    return INPUT_SUCCESS;
}
```

---

### 6.3.5 回调函数验证不足

**风险 ID**: SEC-005

**描述**: 注册外部回调函数前未验证其合法性

**证据位置**: `input/interfaces/include/input_reporter.h`

```c
// input_reporter.h - 回调函数注册
int32_t (*RegisterReportCallback)(uint32_t devIndex, InputReportEventCb *callback);
```

**问题场景**:
```c
InputReportEventCb maliciousCallback = {
    .ReportEventPkgCallback = attacker_controlled_function
};

// 注册恶意回调
inputInterface->iInputReporter->RegisterReportCallback(devIndex, 
                                                        &maliciousCallback);
```

**触发条件**:
1. 恶意应用注册回调
2. 回调在特权上下文中执行

**影响**:
- 代码执行
- 权限提升

**修复建议**:
```c
// 检查回调函数地址是否在合法范围
bool IsValidCallbackAddress(void *callback)
{
    // 限制回调只能来自系统库
    return IsAddressInTrustedRange(callback);
}

// 注册前验证
int32_t RegisterReportCallback(uint32_t devIndex, InputReportEventCb *callback)
{
    if (!IsValidCallbackAddress(callback->ReportEventPkgCallback)) {
        return INPUT_ERR_INVALID_PARAM;
    }
    // ...
}
```

---

### 6.3.6 信息泄露

**风险 ID**: SEC-006

**描述**: 错误信息中可能包含敏感内容

**问题示例**:
```c
// 错误日志泄露文件路径
HDF_LOGE("Failed to open %s: %s", configPath, strerror(errno));

// configPath 可能包含:
// - 用户名: /home/user/config.json
// - 内部路径: /system/vendor/config.json
```

**影响**:
- 系统信息收集
- 攻击面识别

**修复建议**:
```c
// 通用错误信息，不泄露细节
HDF_LOGE("Failed to open device configuration");

// 或仅记录内部路径
HDF_LOGE("Failed to open configuration: %s", 
         IsInternalPath(configPath) ? "[internal]" : "unknown");
```

---

## 6.4 安全机制

### 6.4.1 现有安全机制

| 机制 | 实现位置 | 有效性 |
|------|----------|--------|
| NULL 指针检查 | 各 HAL 实现 | ✅ 有效 |
| 长度参数校验 | 部分模块 | ⚠️ 不完整 |
| 权限声明 | HDF 框架 | ✅ 有效 |
| 审计日志 | 部分模块 | ⚠️ 不完整 |

### 6.4.2 缺失的安全机制

| 机制 | 建议实现位置 | 优先级 |
|------|-------------|--------|
| 参数边界校验 | 所有 HDI 接口 | 高 |
| 地址空间验证 | 回调注册接口 | 高 |
| 输入长度限制 | 字符串参数接口 | 中 |
| 审计日志 | 敏感操作接口 | 中 |

---

## 6.5 权限模型

### 6.5.1 DAC (自主访问控制)

- **设备节点权限**: `/dev/input/*`, `/dev/video/*` 等
- **调用者身份**: 检查 UID/GID
- **配置文件权限**: `.para.dac` 文件

**证据位置**: `usb/cfg/usb.para.dac`

### 6.5.2 HDF 权限框架

- **服务注册权限**: 谁可以注册 HDI 服务
- **接口访问权限**: 谁可以调用 HDI 接口
- **设备访问权限**: 设备节点的访问控制

---

## 6.6 内存安全

### 6.6.1 常见问题

| 问题类型 | 示例模块 | 风险等级 |
|----------|----------|----------|
| 缓冲区溢出 | audio, input | 高 |
| 栈溢出 | sensor | 中 |
| 释放后使用 | 所有模块 | 高 |
| 双重释放 | input | 中 |

### 6.6.2 缓解措施

**代码审计结果**:
- 本仓库为 C/C++ 代码，存在内存安全风险
- 建议使用 AddressSanitizer (ASan) 进行测试
- 建议使用 Static Analyzer 进行代码检查

---

## 6.7 安全建议优先级

| 优先级 | 风险 ID | 建议 | 影响范围 |
|--------|---------|------|----------|
| P0 | SEC-001 | 完善参数校验 | 所有模块 |
| P0 | SEC-005 | 验证回调地址 | input, sensor |
| P1 | SEC-002 | 路径校验 | audio, battery |
| P1 | SEC-003 | 整数溢出检查 | sensor, codec |
| P2 | SEC-004 | 竞态条件缓解 | input, camera |
| P2 | SEC-006 | 错误信息脱敏 | 所有模块 |

---

## 6.8 检查局限性说明

### 6.8.1 检查范围

- **已检查**: 所有 `interfaces/` 目录下的头文件
- **已检查**: 所有 `hal/src/` 目录下的实现
- **已检查**: 所有 `hdi_service/` 目录下的 IPC 代码
- **已检查**: `BUILD.gn` 构建配置

### 6.8.2 未检查范围

- ❌ 测试代码（`test/` 目录）
- ❌ 模糊测试代码（`fuzztest/` 目录）
- ❌ 第三方库代码
- ❌ 内核态驱动代码（`drivers_adapter_khdf_linux`）

### 6.8.3 方法论

- 静态代码审计（手动审查关键文件）
- 模式匹配（查找常见漏洞模式）
- 架构分析（分析数据流和控制流）

---

## 6.9 结论

### 9.1 总体评估

| 维度 | 评分 | 说明 |
|------|------|------|
| 代码质量 | ⭐⭐⭐⭐ | 较好，有基本安全措施 |
| 输入验证 | ⭐⭐⭐ | 不完整，部分接口缺失 |
| 权限控制 | ⭐⭐⭐⭐ | 依赖 HDF 框架 |
| 内存安全 | ⭐⭐⭐ | C 代码固有风险 |

### 9.2 建议改进

1. **短期**: 完善所有 HDI 接口的参数校验
2. **中期**: 增加地址空间验证和审计日志
3. **长期**: 考虑使用内存安全语言重写关键模块

---

**下一节**: [编译产物](07_Build_Artifacts.md) - 了解构建产物详情

# 目录结构与模块职责

## 顶级目录概览

```
drivers_interface/
├── audio/                    # 音频接口
├── battery/                   # 电池接口
├── bluetooth/                 # 蓝牙接口
├── camera/                    # 相机接口
├── codec/                     # 编解码接口
├── display/                   # 显示接口
├── drm/                       # 数字版权管理
├── face_auth/                 # 人脸认证
├── fingerprint_auth/          # 指纹认证
├── huks/                      # 密钥管理
├── input/                     # 输入设备
├── location/                  # 定位服务
├── nfc/                       # 近场通信
├── pin_auth/                  # PIN 认证
├── power/                     # 电源管理
├── ril/                       # 无线接口层
├── secure_element/            # 安全元素
├── sensor/                    # 传感器接口
├── thermal/                   # 温控管理
├── usb/                       # 通用串行总线
├── user_auth/                 # 用户认证
├── vibrator/                  # 振动器
├── wlan/                      # 无线局域网
├── interface.gni              # HDI 构建模板入口
└── bundle.json               # 组件配置
```

## 模块分类

### 基础 I/O 类

#### Audio
- **路径**: `audio/`
- **版本**: v1_0 → v6_0
- **职责**: 音频设备抽象，包括播放、录音、音效处理
- **关键接口**:
  - `IAudioManager`: 音频适配器管理
  - `IAudioRender`: 音频渲染
  - `IAudioCapture`: 音频采集

**证据**: `audio/v1_0/IAudioManager.idl:43-85`

#### Display
- **路径**: `display/`
- **子模块**: buffer, composer, graphic
- **职责**: 显示渲染、帧缓冲区管理、图形合成
- **关键接口**:
  - `IDisplayComposer`: 显示合成器
  - `IAllocator`: 缓冲区分配
  - `IMapper`: 缓冲区映射

**证据**: `display/composer/v1_0/IDisplayComposer.idl`

#### Input
- **路径**: `input/`
- **职责**: 输入设备管理（键盘、鼠标、触摸等）
- **模式**: 支持 Passthrough 模式（无 IPC 开销）
- **关键接口**: `IInputInterfaces`

**证据**: `input/v1_0/BUILD.gn:26` - `mode = "passthrough"`

#### USB
- **路径**: `usb/`
- **职责**: USB 设备通信
- **子模块**: ddk, gadget, serial, scsi_ddk

#### WLAN
- **路径**: `wlan/`
- **版本**: v1_0 → v1_3
- **职责**: 无线局域网连接
- **关键接口**: `IWlanInterface`

---

### 安全认证类

#### HUKS (Huawei Universal Key Store)
- **路径**: `huks/`
- **版本**: v1_0, v1_1
- **职责**: 硬件级密钥管理
- **关键能力**:
  - 密钥生成/导入
  - 签名/验签
  - 加密/解密
  - 密钥访问控制

**证据**: `huks/v1_1/IHuks.idl:39` - 接口声明

#### User Auth
- **路径**: `user_auth/`
- **版本**: v1_0 → v4_1
- **职责**: 用户身份认证框架
- **认证类型**: PIN、人脸、指纹、恢复密钥

#### Face Auth
- **路径**: `face_auth/`
- **版本**: v1_0 → v2_0
- **职责**: 人脸识别认证

#### Fingerprint Auth
- **路径**: `fingerprint_auth/`
- **版本**: v1_0 → v2_0
- **职责**: 指纹识别认证

#### Pin Auth
- **路径**: `pin_auth/`
- **版本**: v1_0 → v3_0
- **职责**: PIN 码认证
- **安全等级**: ESL0 - ESL3

#### Secure Element
- **路径**: `secure_element/`
- **职责**: 安全元素（SE）通信

---

### 感知类

#### Sensor
- **路径**: `sensor/`
- **版本**: v1_0 → v3_1
- **职责**: 传感器数据获取
- **能力**:
  - 获取传感器列表
  - 启用/禁用传感器
  - 设置采样参数
  - 注册数据回调

**证据**: `sensor/v3_0/ISensorInterface.idl:53-175`

#### Camera
- **路径**: `camera/`
- **版本**: v1_0 → v1_5
- **职责**: 相机设备抽象
- **关键接口**:
  - `ICameraHost`: 相机主机管理
  - `ICameraDevice`: 相机设备操作
  - `IStreamOperator`: 流操作

#### Location
- **路径**: `location/`
- **子模块**: agnss, geofence, gnss, lpfence
- **职责**: 定位服务抽象

---

### 连接类

#### Bluetooth
- **路径**: `bluetooth/`
- **子模块**: a2dp, hci, lp_ble
- **职责**: 蓝牙协议栈接口

#### NFC
- **路径**: `nfc/`
- **版本**: v1_0, v1_1
- **职责**: 近场通信接口

#### RIL (Radio Interface Layer)
- **路径**: `ril/`
- **版本**: v1_0 → v1_5
- **职责**: 无线通信抽象

---

### 多媒体类

#### Codec
- **路径**: `codec/`
- **版本**: v1_0 → v4_0
- **职责**: 编解码器接口
- **安全特性**: 分支保护 (`branch_protector_ret = "pac_ret"`)

**证据**: `codec/v4_0/BUILD.gn:18`

#### DRM
- **路径**: `drm/`
- **职责**: 数字版权管理

---

### 系统服务类

#### Battery
- **路径**: `battery/`
- **版本**: v1_0 → v2_0
- **职责**: 电池状态管理
- **关键接口**: `IBatteryInterface`

**证据**: `battery/v2_0/IBatteryInterface.idl`

#### Power
- **路径**: `power/`
- **版本**: v1_0 → v1_3
- **职责**: 电源管理

#### Thermal
- **路径**: `thermal/`
- **版本**: v1_0, v1_1
- **职责**: 温控管理

#### Light
- **路径**: `light/`
- **职责**: 设备灯光控制

#### Vibrator
- **路径**: `vibrator/`
- **版本**: v1_0 → v2_0
- **职责**: 振动器控制

---

### 其他模块

| 模块 | 职责 |
|-----|------|
| `activity_recognition` | 活动识别 |
| `connected_nfc_tag` | NFC Tag |
| `distributed 连接_audio` | 分布式音频 |
| `distributed_camera` | 分布式相机 |
| `ethernet` | 以太网 |
| `intelligent_voice` | 智能语音 |
| `low_power_player` | 低功耗播放 |
| `memorytracker` | 内存追踪 |
| `midi` | MIDI 接口 |
| `motion` | 运动传感器 |
| `partitionslot` | 分区槽位 |
| `tools/hc-gen` | HCS 配置生成工具 |

---

## 版本分布统计

| 模块类别 | 模块数量 | 版本范围 |
|---------|---------|---------|
| 基础 I/O | 5 | v1_0 - v6_0 |
| 安全认证 | 6 | v1_0 - v4_1 |
| 感知类 | 3 | v1_0 - v3_1 |
| 连接类 | 3 | v1_0 - v1_5 |
| 多媒体 | 2 | v1_0 - v4_0 |
| 系统服务 | 7 | v1_0 - v1_3 |

---

## 子模块命名约定

### 版本目录
```
module/
├── v1_0/          # Major.Minor 版本
├── v1_1/
└── v2_0/
```

### 功能性子目录
```
display/
├── buffer/        # 帧缓冲区
├── composer/      # 显示合成
└── graphic/       # 图形通用
```

```
camera/
├── metadata/      # 元数据
└── sequenceable/  # 可序列化对象
```

---

## 忽略的目录

根据任务约束，以下目录不参与文档分析：
- `test/` - 测试代码
- `*_test.*` - 测试文件
- `.git/` - Git 元数据

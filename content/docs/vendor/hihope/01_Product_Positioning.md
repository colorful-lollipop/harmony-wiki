# 产品定位与边界

## 文档信息

- **目的**：详细说明 vendor_hihope 仓库中各产品线的定位、适用场景、核心能力和边界
- **适用范围**：vendor/hihope 仓库全部 13 个产品线
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 是 HiHope 维护的 OpenHarmony 产品配置和适配仓库
  - 不包含系统框架核心代码，仅包含产品配置、HAL 适配和驱动配置
  - 各产品线针对不同硬件平台和应用场景优化

## 产品分类与定位

### 1. 海王星 Neptune 系列

#### 1.1 Neptune IoT Link Demo

**产品标识**：`neptune_iotlink_demo`

**定位**：IoT 设备快速原型开发和学习平台

**目标设备**：HiHope Neptune100 开发板

**核心能力**：
- ✅ BLE（Bluetooth Low Energy）通信
- ✅ 基础外设控制（GPIO、I2C、SPI、UART、PWM、ADC）
- ✅ LiteOS-M 轻量级系统
- ✅ 基础系统服务（hilog_lite, hievent_lite, samgr_lite）

**适用场景**：
- BLE 外设开发和调试
- IoT 传感器数据采集与传输
- 快速原型验证
- 嵌入式系统学习

**运行环境**：
- **系统类型**：Mini（轻量级）
- **内核**：LiteOS-M 3.0.0
- **SoC/芯片**：WinnerMicro Neptune100
- **内存**：< 512KB（取决于具体板卡）
- **存储**：片上闪存 + 外部存储

**产品边界**：
- ✅ 不支持完整图形界面（仅有基础 UI 或无 GUI）
- ❌ 不支持完整 N-API（JavaScript API）
- ❌ 不支持完整的 Ability 框架
- ⚠️ 仅支持精简版系统服务

**证据**：
- 配置文件：`neptune_iotlink_demo/config.json:4`（type: "mini"）
- 子系统：`config.json:25-87`（kernel, startup, hiviewdfx, systemabilitymgr, communication, commonlibrary）
- HAL 实现：`neptune_iotlink_demo/hals/utils/` 目录

#### 1.2 NearLink DK-3863

**产品标识**：`nearlink_dk_3863`

**定位**：星闪（NearLink/SLE）技术学习和开发平台

**目标设备**：Hisilicon WS63V100 开发套件

**核心能力**：
- ✅ SLE（StarLink Enhanced）短距离通信
- ✅ BLE（Bluetooth Low Energy）通信
- ✅ WiFi 网络通信
- ✅ MQTT 物联网协议支持
- ✅ 丰富外设控制（GPIO、PWM、ADC、I2C、SPI）
- ✅ OLED 显示屏支持
- ✅ 多种传感器接口（温湿度、气体等）
- ✅ 28 个循序渐进示例教程

**适用场景**：
- NearLink/SLE 技术学习和实验
- IoT 设备原型开发
- 学生培训和教学
- 技术方案验证

**运行环境**：
- **系统类型**：Mini（轻量级）
- **内核**：LiteOS-M（预构建内核）
- **SoC/芯片**：HiSilicon WS63V100
- **版本**：OpenHarmony 1.0

**产品边界**：
- ✅ 不支持完整图形界面（教程中有 OLED 显示示例）
- ❌ 不支持完整 N-API（JavaScript API）
- ✅ 包含丰富的教学示例和 MQTT 库

**教程示例**（28 个编号教程）：
**基础教程**（00-05）：
- `00_thread` - 线程使用
- `01_timer` - 定时器使用
- `02_delay` - 延时函数
- `03_mutex` - 互斥锁
- `04_semaphore` - 信号量
- `05_message` - 消息队列

**外设教程**（06-13）：
- `06_gpioled` - GPIO 控制 LED
- `07_gpiobutton` - GPIO 按钮输入
- `08_pwmled` - PWM 控制 LED
- `09_adcbutton` - ADC 按键输入
- `10_adchuman` - ADC 人体感应
- `11_aht20` - AHT20 温湿度传感器
- `12_oled` - OLED 显示屏
- `13_adclight` - ADC 光敏传感器

**网络教程**（14-18）：
- `14_easy_wifi` - WiFi 连接
- `15_tcpclient` - TCP 客户端
- `16_tcpserver` - TCP 服务器
- `17_udpclient` - UDP 客户端
- `18_udpserver` - UDP 服务器

**通信协议教程**（19-27）：
- `19_ble_uart` - BLE UART 通信
- `20_mqtt_demo` - MQTT 基础演示
- `21_mqtt_led` - MQTT 控制 LED
- `22_mqtt_sensor` - MQTT 传感器数据
- `23_sle_uart` - SLE（NearLink）UART
- `24_sle_humi` - SLE 湿度传感器
- `25_sle_led` - SLE 控制 LED
- `26_sle_gas` - SLE 气体传感器
- `27_sle_oled` - SLE 控制 OLED

**证据**：
- 配置文件：`nearlink_dk_3863/config.json:2`（type: "mini"）
- 教程目录：`nearlink_dk_3863/ws63_sample/00_thread/` 到 `27_sle_oled/`
- 构建：`nearlink_dk_3863/ws63_sample/00_thread/BUILD.gn` 到 `27_sle_oled/BUILD.gn`（43 个 BUILD.gn）

### 2. 大禹 DAYU 系列

#### 2.1 RK3568（DAYU200）

**产品标识**：`rk3568`

**定位**：RK3568（DAYU200）开发板完整系统配置，功能最全面

**目标设备**：HiHope DAYU200 开发板

**核心能力**：
- ✅ 完整图形界面（ArkUI）
- ✅ N-API（JavaScript API）支持（但实现不在本仓库）
- ✅ 完整 Ability/ExtensionAbility 框架
- ✅ 丰富多媒体支持（相机、音频、视频、图像）
- ✅ 完整通信能力（WiFi、蓝牙、以太网）
- ✅ 60+ 种传感器支持（加速度、陀螺仪、磁力、光线、距离、温度、湿度、气体等）
- ✅ 完整输入输出（触摸、按钮、显示、振动、LED）
- ✅ AI 能力（MindSpore）
- ✅ 完整安全机制（SELinux、Seccomp、权限管理）
- ✅ 完整 HAL 实现（音频、编解码）

**适用场景**：
- RK3568/DAYU200 开发
- 平板和 2 合 1 设备开发
- 智能家居设备开发
- IPC 摄像头开发
- 中高端设备方案验证

**运行环境**：
- **系统类型**：Standard（标准系统）
- **内核**：Linux
- **CPU 架构**：ARM 64 位（target_cpu: "arm"）
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8
- **内存**：2GB - 8GB
- **存储**：eMMC 闪存

**产品边界**：
- ✅ 支持完整的 OpenHarmony 系统能力
- ✅ 启用 SELinux 和 Seccomp
- ✅ 包含最完整的 HDF 配置
- ✅ 资源占用最高

**证据**：
- 配置文件：`rk3568/config.json:2`（type: "standard", api_version: 8）
- HDF 配置：`rk3568/hdf_config/`（90+ 个 .hcs 文件，658 行 device_info.hcs + 901 行 device_info.hcs）
- 子系统：`config.json:15-426`（包含 22 个子系统）

#### 2.2 RK3568 Mini System

**产品标识**：`rk3568_mini_system`

**定位**：RK3568 精简系统配置

**目标设备**：RK3568/DAYU200 板卡的精简应用场景

**核心能力**：
- ✅ 基础图形界面
- ✅ 精简 Ability 框架
- ✅ 基础多媒体支持（音频、相机）
- ✅ 基础通信能力
- ⚠️ 精简 HAL 实现
- ⚠️ 精简安全配置

**适用场景**：
- 资源受限设备
- 功能精简的系统
- 基础演示系统

**运行环境**：
- **系统类型**：Standard（标准系统）
- **内核**：Linux
- **CPU 架构**：ARM 64 位
- **SoC/芯片**：Rockchip RK3568

**产品边界**：
- ⚠️ 不包含 resourceschedule/ 目录
- ⚠️ 不包含 security_config/ 目录
- ⚠️ 不包含 window_config/ 目录
- ⚠️ HDF 配置精简

**证据**：
- 配置文件：`rk3568_mini_system/config.json`（仅 1 行）
- 缺少目录对比 rk3568：无 resourceschedule, security_config, window_config
- 构建：仅 4 个 BUILD.gn 文件（vs rk3568 的 10 个）

### 3. 核心系统产品

#### 3.1 Default Core System

**产品标识**：`default_core_system`

**定位**：RK3568 默认核心系统参考配置

**目标设备**：通用 RK3568 设备

**核心能力**：
- ✅ 基础图形界面
- ✅ 基础多媒体支持
- ✅ 基础 HAL 实现
- ✅ 完整安全机制

**适用场景**：
- 新产品开发参考模板
- 通用设备配置起点
- 功能裁剪参考

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`default_core_system/config.json:2`（inherit: ["productdefine/common/inherit/chipset_common.json"]）
- 目录结构：与 rk3568 相同但内容更精简

### 4. 场景化产品

#### 4.1 2 合 1 核心系统

**产品标识**：`2in1_core_system`

**定位**：2 合 1（平板/笔记本混合）设备核心系统

**目标设备**：RK3568 2 合 1 设备

**核心能力**：
- ✅ 完整图形界面
- ✅ 完整多媒体支持
- ✅ 完整通信能力
- ✅ 完整输入输出
- ✅ 完整安全机制

**适用场景**：
- 平板电脑开发
- 可拆卸键盘设备
- 2 合 1 混合设备

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`2in1_core_system/config.json:2`
- 子系统：`config.json:14-34`（inherit: ["productdefine/common/inherit/rich.json"]）

#### 4.2 平板核心系统

**产品标识**：`tablet_core_system`

**定位**：平板设备专用核心系统

**目标设备**：RK3568 平板设备

**核心能力**：
- ✅ 完整图形界面
- ✅ 完整多媒体支持
- ✅ 位置服务（location 子系统）
- ✅ 完整通信能力
- ✅ 电话能力（telephony 子系统，包含 core_service, telephony_data, state_registry, cellular_data, sms_mms, call_manager）

**适用场景**：
- 平板设备开发
- 大屏设备优化
- 通话功能集成

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`tablet_core_system/config.json:2`（inherit: ["productdefine/common/inherit/tablet.json"]）
- 子系统：`config.json:16-97`（包含 location 和 telephony 子系统）

#### 4.3 可穿戴设备核心系统

**产品标识**：`wearable`

**定位**：可穿戴设备专用核心系统

**目标设备**：RK3568 可穿戴设备（手表、手环等）

**核心能力**：
- ✅ 精简图形界面（针对小屏优化）
- ✅ 基础多媒体支持
- ✅ 完整通信能力
- ✅ 完整安全机制
- ⚠️ 精简传感器和输入输出

**适用场景**：
- 智能手表开发
- 智能手环开发
- 可穿戴设备方案验证

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`wearable/config.json:2`（inherit: ["productdefine/common/inherit/wearable.json"]）
- 子系统：`config.json:15-34`（inherit: wearable 配置）

#### 4.4 智能电视核心系统

**产品标识**：`tv`

**定位**：智能电视专用核心系统

**目标设备**：RK3568 电视设备

**核心能力**：
- ✅ 完整图形界面（针对大屏优化）
- ✅ 完整多媒体支持
- ✅ 完整输入输出
- ✅ 完整安全机制
- ⚠️ 精简传感器配置

**适用场景**：
- 智能电视开发
- 大屏显示设备开发
- 电视应用集成

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`tv/config.json:2`（inherit: ["productdefine/common/inherit/tablet.json"]）
- 目录结构：与 tablet_core_system 类似但针对电视场景优化

#### 4.5 IPC 摄像头核心系统

**产品标识**：`ipcamera_core_system`

**定位**：IPC 摄像头设备专用核心系统

**目标设备**：RK3568 IPC 摄像头

**核心能力**：
- ✅ 完整图形界面
- ✅ 完整多媒体支持（相机、视频、音频）
- ✅ 完整输入输出
- ✅ 完整通信能力
- ✅ 完整安全机制
- ⚠️ 针对视频采集优化

**适用场景**：
- IPC 摄像头开发
- 视频监控系统
- 智能门铃开发

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`ipcamera_core_system/config.json:2`（type: "standard"）
- 目录结构：与其他标准产品类似

### 5. DAYU 系列开发板

#### 5.1 DAYU200（rk3568）

**产品标识**：`rk3568`

**定位**：DAYU200 开发板完整系统

**目标设备**：HiHope DAYU200 开发板

**核心能力**：
- ✅ 功能最全面（见 rk3568 详细说明）

**适用场景**：
- DAYU200 开发
- 高性能设备方案验证

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568

**证据**：
- 配置文件：`rk3568/config.json`
- HDF 配置：完整且详细

#### 5.2 DAYU210（dayu210）

**产品标识**：`dayu210`

**定位**：DAYU210 开发板专用配置

**目标设备**：HiHope DAYU210 开发板

**核心能力**：
- ✅ 完整图形界面
- ✅ 完整多媒体支持
- ✅ 完整通信能力
- ✅ 完整安全机制
- ⚠️ 包含蓝牙源代码实现（`dayu210/bluetooth/`）

**适用场景**：
- DAYU210 开发
- DAYU210 硬件特性适配

**运行环境**：
- **系统类型**：Standard
- **SoC/芯片**：Rockchip RK3568
- **API Level**：8

**证据**：
- 配置文件：`dayu210/config.json:2`
- 蓝牙源码：`dayu210/bluetooth/src/`（包含实际 C++ 源代码）
- 蓝牙头文件：`dayu210/bluetooth/include/utils/`

### 6. 测试产品

#### 6.1 NearLink DK-3863 XTS

**产品标识**：`nearlink_dk_3863_xts`

**定位**：NearLink XTS（X-TS）测试套件配置

**目标设备**：Hisilicon WS63V100

**核心能力**：
- ✅ XTS 测试框架支持
- ⚠️ 仅配置文件，无实际代码

**适用场景**：
- NearLink XTS 测试执行
- 兼容性验证

**运行环境**：
- **系统类型**：Mini（轻量级）
- **内核**：LiteOS-M
- **SoC/芯片**：HiSilicon WS63V100

**产品边界**：
- ✅ 仅测试配置
- ❌ 不包含功能代码
- ❌ 不包含 HAL 实现

**证据**：
- 配置文件：`nearlink_dk_3863_xts/config.json:1`（type: "mini"）
- 构建：`nearlink_dk_3863_xts/BUILD.gn:1`（仅有 1 个 BUILD.gn 文件）

## 产品能力对比矩阵

| 能力类别 | Mini 系统 | Standard 系统 | 说明 |
|----------|----------|---------------|------|
| **图形界面** | ⚠️ 精简或基础 | ✅ 完整 ArkUI | Mini 系统资源受限 |
| **N-API 支持** | ❌ 不支持 | ✅ 支持 | N-API 在 OpenHarmony 框架层实现 |
| **Ability 框架** | ⚠️ 精简 | ✅ 完整 | 标准 Ability/ExtensionAbility |
| **HAL 实现** | ⚠️ 精简或基础 | ✅ 完整 | 标准 HAL 包含完整接口 |
| **HDF 配置** | ⚠️ 精简 | ✅ 完整 | 标准 HDF 包含所有外设 |
| **传感器支持** | ⚠️ GPIO/PWM/ADC/I2C | ✅ 60+ 种传感器 | 标准系统支持丰富的传感器 |
| **安全机制** | ⚠️ 精简 | ✅ 完整 | 标准系统包含 SELinux/Seccomp |
| **多媒体** | ⚠️ 基础 | ✅ 完整 | 标准：相机/音频/视频/图像 |
| **通信** | ⚠️ 部分 | ✅ 完整 | 标准：WiFi/蓝牙/以太网 |
| **输入输出** | ⚠️ 精简 | ✅ 完整 | 标准：触摸/按钮/显示/振动等 |
| **存储** | ⚠️ 闪存 | ✅ eMMC/SD | 标准：大容量存储 |

## 产品选型建议

### 根据开发目标选择产品

| 开发目标 | 推荐产品 | 原因 |
|---------|----------|------|
| **完整系统开发** | rk3568 | 最完整的配置和 HAL 实现 |
| **DAYU210 硬件特性开发** | dayu210 | 包含 DAYU210 特定蓝牙源码 |
| **平板设备开发** | tablet_core_system | 针对平板场景优化 |
| **可穿戴设备开发** | wearable | 针对小屏和低功耗优化 |
| **智能电视开发** | tv | 针对大屏和电视场景优化 |
| **IPC 摄像头开发** | ipcamera_core_system | 针对视频采集优化 |
| **新项目模板** | default_core_system | 可作为裁剪起点 |
| **2 合 1 设备开发** | 2in1_core_system | 针对混合设备优化 |
| **IoT 原型开发** | neptune_iotlink_demo | BLE/IoT 快速原型 |
| **NearLink 技术学习** | nearlink_dk_3863 | 28 个教程，循序渐进 |
| **LiteOS-M 学习** | neptune_iotlink_demo 或 nearlink_dk_3863 | 轻量级系统学习 |
| **NearLink 测试** | nearlink_dk_3863_xts | XTS 测试套件 |
| **精简系统开发** | rk3568_mini_system | 资源受限场景 |

### 根据硬件选型产品

| SoC/芯片 | 可选产品 | 说明 |
|---------|----------|------|
| **RK3568** | rk3568, rk3568_mini_system, tablet_core_system, tv, ipcamera_core_system, default_core_system, 2in1_core_system, wearable, dayu210 | 所有 RK3568 开发板 |
| **Neptune100** | neptune_iotlink_demo | WinnerMicro Neptune100 |
| **WS63V100** | nearlink_dk_3863, nearlink_dk_3863_xts | HiSilicon WS63V100 |

## 产品间差异

### Mini 系统与 Standard 系统差异

| 差异点 | Mini 系统 | Standard 系统 |
|---------|----------|---------------|
| **系统类型** | Mini | Standard |
| **API Level** | 较低（1.0-3.0） | 较高（8-9） |
| **内核** | LiteOS-M（轻量级 RTOS） | Linux（通用 OS） |
| **图形界面** | 精简或基础 | 完整 ArkUI |
| **N-API** | 不支持 | 完整支持 |
| **应用框架** | 精简 Lite Ability | 完整 Ability/ExtensionAbility |
| **HAL 实现** | 基础 HAL 或无 | 完整 HAL |
| **HDF 配置** | 精简 | 完整 |
| **安全机制** | 精简 HUKS | 完整 SELinux/Seccomp |
| **资源占用** | < 512KB RAM | 2GB+ RAM |
| **适用设备** | MCU、传感器、可穿戴 | 平板、手机、电视等 |

### Standard 系统内部差异

| 差异点 | rk3568 | dayu210 | default_core | tablet_core | wearable | 2in1_core | ipcamera_core |
|---------|---------|----------|---------------|----------|----------|----------|----------|
| **蓝牙源码** | 配置 | ✅ 包含 | 配置 | 配置 | 配置 | 配置 |
| **继承配置** | rich.json | rich.json | chipset_common.json | tablet.json | wearable.json | rich.json | tablet.json |
| **AI 能力** | ✅ 包含 | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **传感器** | ✅ 最丰富 | 基础 | 基础 | 基础 | 基础 | 基础 | 基础 |
| **位置服务** | ❌ | ❌ | ❌ | ✅ 包含 | ❌ | ❌ | ❌ |
| **电话能力** | ✅ 包含 | ❌ | ❌ | ✅ 包含 | ❌ | ❌ | ❌ |
| **窗口管理** | ✅ 包含 | ✅ 包含 | ✅ 包含 | ✅ 包含 | ✅ 包含 | ✅ 包含 |

## 产品开发流程

### 基于 rk3568 开发新产品

1. **复制 rk3568 目录**
   ```bash
   cp -r rk3568 my_new_product
   ```

2. **修改 product_name**
   ```json
   "product_name": "my_new_product"
   ```

3. **修改 product_adapter_dir**
   ```json
   "product_adapter_dir": "//vendor/hihope/my_new_product/hals"
   ```

4. **修改 group 名**
   ```gn
   group("my_new_product") {
   }
   ```

5. **选择性裁剪子系统**
   - 保留必要子系统：kernel, startup, hiviewdfx, distributedschedule, hdf, security
   - 移除不需要的子系统：ai, msdp, castplus（按需）

6. **编译配置**
   ```bash
   hb set
   # 选择 my_new_product
   hb build
   ```

### 基于 neptune_iotlink_demo 开发新产品

1. **复制目录并改名**
   ```bash
   cp -r neptune_iotlink_demo my_iot_product
   ```

2. **修改配置文件**
   - config.json 中的 product_name
   - product_adapter_dir 路径
   - 可选：修改 kernel_type 和 board

3. **配置 HAL 适配**
   - 在 `my_iot_product/hals/` 目录添加 HAL 实现
   - 参考 existing HAL 接口定义

## 相关跳转

- [目录结构详解](02_Directory_Structure.md)
- [HAL 实现说明](03_HAL_Implementation.md)
- [HDF 配置详解](04_HDF_Configuration.md)
- [返回 Wiki 首页](SUMMARY.md)

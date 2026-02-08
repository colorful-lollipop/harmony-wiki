# 目录结构与模块职责

> **目的**: 理解仓库的目录组织方式，快速定位各模块位置  
> **适用范围**: 需要了解模块划分、查找特定 API 的开发者  
> **生成时间**: 2025-02-06

---

## 1. 完整目录结构

```
interface_sdk_c/
│
├── AbilityKit/                    # Ability 框架（10 个头文件）
│   ├── ability_base/              #   Ability 基础类型（Want、Context 等）
│   └── ability_runtime/           #   Ability 运行时（生命周期管理）
│
├── ai/                            # AI 模块
│   └── neural_network_runtime/    #   神经网络运行时（端侧 AI 推理）
│
├── ani/                           # ANI (Ark Native Interface)
│   └── ani.h                      #   ArkTS Native 接口声明
│
├── ark_runtime/                   # Ark 运行时
│   └── jsvm/                      #   JSVM (JavaScript Virtual Machine)
│       ├── jsvm.h                 #     JSVM 核心 API
│       └── libjsvm.ndk.json       #     JSVM 符号定义
│
├── arkui/                         # ArkUI 框架（29 个头文件）
│   ├── ace_engine/native/         #   ArkUI 原生引擎（核心）
│   │   ├── native_interface.h     #     ArkUI Native API 统一入口
│   │   ├── native_node.h          #     原生节点操作
│   │   ├── native_animate.h       #     动画 API
│   │   └── ...                    #     其他 20+ 个头文件
│   ├── display_manager/           #   显示管理（屏幕信息、截图）
│   ├── napi/                      #   NAPI 框架（JS-C 互操作）
│   │   ├── native_api.h           #     OpenHarmony N-API 扩展
│   │   ├── common.h               #     公共类型定义
│   │   └── libnapi.ndk.json       #     N-API 符号定义（306 个）
│   └── window_manager/            #   窗口管理（窗口创建、属性设置）
│
├── backgroundtasks/               # 后台任务
│   └── transient/                 #   瞬态任务（短时后台任务）
│
├── BasicServicesKit/              # 基础服务 Kit（多 API 集合）
│   ├── commonevent/               #   公共事件（订阅/发布）
│   ├── libohbattery_info.ndk.json #   电池信息 API
│   ├── libohprint.ndk.json        #   打印 API
│   ├── libohscan.ndk.json         #   扫描 API
│   └── libtime_service.ndk.json   #   时间服务 API
│
├── bundlemanager/                 # 包管理
│   └── bundle_framework/bundle/   #   Bundle 框架（应用包信息查询）
│
├── commonlibrary/                 # 公共库
│   └── memory_utils/              #   内存工具
│       └── libpurgeablemem/       #     可清除内存（内存紧张时自动释放）
│
├── ConnectivityKit/               # 连接 Kit
│   ├── bluetooth/                 #   蓝牙 API
│   └── wifi/                      #   WiFi API
│
├── CryptoArchitectureKit/         # 加密架构 Kit（11 个头文件）
│   ├── crypto_architecture_kit.h  #   总入口
│   ├── crypto_sym_cipher.h        #   对称加密（AES/SM4）
│   ├── crypto_asym_cipher.h       #   非对称加密（RSA/SM2）
│   ├── crypto_signature.h         #   数字签名
│   ├── crypto_digest.h            #   摘要算法
│   └── ...                        #   其他加密组件
│
├── DataProtectionKit/             # 数据保护 Kit
│   └── dlp_permission_api.h       #   DLP 文件权限管理
│
├── distributeddatamgr/            # 分布式数据管理（20+ 个头文件）
│   ├── pasteboard/                #   剪贴板（跨设备复制粘贴）
│   ├── preferences/               #   轻量级存储（Key-Value）
│   ├── relational_store/          #   关系型数据库（RDB）
│   │   ├── relational_store.h     #     RDB 主 API
│   │   ├── oh_cursor.h            #     查询结果游标
│   │   └── oh_predicates.h        #     查询条件构造
│   └── udmf/                      #   统一数据管理（UDMF）
│
├── distributedhardware/           # 分布式硬件
│   └── device_manager/            #   设备管理（发现、认证、连接）
│
├── drivers/                       # 驱动接口
│   └── external_device_manager/   #   外部设备管理 DDK
│       ├── base/                  #     基础 DDK 类型
│       ├── hid/                   #     HID 设备（键盘/鼠标/手柄）
│       ├── scsi_peripheral/       #     SCSI 外设
│       ├── usb/                   #     USB 设备
│       └── usb_serial/            #     USB 串口
│
├── filemanagement/                # 文件管理（10+ 个头文件）
│   ├── cloud_disk_manager/        #   云盘管理
│   ├── environment/               #   应用环境目录（沙箱路径）
│   ├── fileio/                    #   文件 IO（读写、属性）
│   ├── fileshare/                 #   文件分享（URI 权限管理）
│   └── file_uri/                  #   文件 URI 处理
│
├── GameControllerKit/             # 游戏控制器 Kit
│   ├── game_device.h              #   游戏设备管理
│   └── game_pad.h                 #   手柄输入
│
├── global/                        # 全局模块
│   ├── i18n/                      #   国际化（时区、本地化）
│   └── resource_management/       #   资源管理
│       ├── rawfile/               #     原始资源文件访问
│       └── resourcemanager/       #     资源管理器
│
├── graphic/                       # 图形（60+ 个头文件）
│   └── graphic_2d/                #   2D 图形
│       ├── EGL/                   #     EGL 接口
│       ├── GLES2/                 #     OpenGL ES 2.0
│       ├── GLES3/                 #     OpenGL ES 3.0
│       ├── GL4/                   #     OpenGL 4.x
│       ├── KHR/                   #     Khronos 标准扩展
│       ├── native_buffer/         #     原生缓冲区
│       ├── native_color_space_manager/  # 色彩空间管理
│       ├── native_display_soloist/      # 显示同步
│       ├── native_drawing/        #     原生 2D 绘制（50+ API）
│       ├── native_effect/         #     原生效果
│       ├── native_fence/          #     Fence 同步
│       ├── native_image/          #     原生图像
│       ├── native_vsync/          #     垂直同步
│       ├── native_window/         #     原生窗口
│       └── vulkan/                #     Vulkan 图形 API
│
├── hiviewdfx/                     # HiView DFX（调测诊断，12 个头文件）
│   ├── hiappevent/                #   应用事件（埋点上报）
│   ├── hicollie/                  #   性能监控（卡顿检测）
│   ├── hidebug/                   #   调试工具
│   ├── hilog/                     #   日志系统（核心）
│   │   └── include/hilog/log.h    #     HiLog API
│   └── hitrace/                   #   分布式追踪
│
├── inputmethod/                   # 输入法
│   └── include/                   #   输入法控制 API（9 个头文件）
│
├── IPCKit/                        # IPC Kit
│   ├── ipc_cskeleton.h            #   IPC 骨架（身份校验）
│   ├── ipc_cremote_object.h       #   远程对象管理
│   └── ipc_cparcel.h              #   数据序列化
│
├── LocationKit/                   # 定位 Kit
│   ├── oh_location.h              #   定位服务 API
│   └── oh_location_type.h         #   定位类型定义
│
├── multimedia/                    # 多媒体（120+ 个头文件，最大模块）
│   ├── audio_framework/           #   音频框架
│   │   ├── audio_capturer/        #     音频采集
│   │   ├── audio_manager/         #     音频管理
│   │   ├── audio_renderer/        #     音频渲染
│   │   └── audio_suite_engine/    #     音频套件引擎
│   ├── av_codec/                  #   AV 编解码（8 个子模块）
│   │   ├── audio_codec/           #     音频编解码
│   │   ├── audio_decoder/         #     音频解码
│   │   ├── audio_encoder/         #     音频编码
│   │   ├── avcencinfo/            #     CENC 信息
│   │   ├── avdemuxer/             #     解封装
│   │   ├── avmuxer/               #     封装
│   │   ├── avsource/              #     AV 源
│   │   ├── codec_base/            #     编解码基础
│   │   ├── video_decoder/         #     视频解码
│   │   └── video_encoder/         #     视频编码
│   ├── av_session/                #   AV 会话（媒体控制）
│   ├── camera_framework/          #   相机框架
│   │   └── camera.h               #     相机主 API
│   ├── drm_framework/             #   DRM 框架
│   ├── image_effect/              #   图像效果
│   ├── image_framework/           #   图像框架
│   │   └── include/image/         #     image_*.h, pixelmap_*.h
│   ├── media_foundation/          #   媒体基础（公共类型）
│   ├── media_library/             #   媒体库（媒体资源管理）
│   ├── player_framework/          #   播放器框架（9 个子模块）
│   │   ├── avimage_generator/     #     图像生成器
│   │   ├── avmedia_base/          #     媒体基础
│   │   ├── avmedia_source/        #     媒体源
│   │   ├── avmetadata_extractor/  #     元数据提取
│   │   ├── avplayer/              #     播放器
│   │   ├── avrecorder/            #     录制器
│   │   ├── avscreen_capture/      #     屏幕录制
│   │   ├── avtranscoder/          #     转码器
│   │   └── lowpower_avsink/       #     低功耗输出
│   └── video_processing_engine/   #   视频处理引擎
│       ├── image_processing/      #     图像处理
│       └── video_processing/      #     视频处理
│
├── multimodalinput/               # 多模态输入
│   └── kits/c/input/              #   输入管理（键盘/鼠标/触摸）
│
├── network/                       # 网络
│   ├── netmanager/                #   网络连接管理
│   ├── netssl/                    #   SSL/TLS 安全连接
│   └── netstack/                  #   网络协议栈
│       ├── net_http/              #     HTTP 客户端
│       └── net_websocket/         #     WebSocket
│
├── NotificationKit/               # 通知 Kit
│
├── resourceschedule/              # 资源调度
│   ├── background_process_manager/  # 后台进程管理
│   ├── ffrt/                      #   FFRT 并行运行时
│   └── qos_manager/               #   QoS 服务质量管理
│
├── security/                      # 安全（20+ 个头文件）
│   ├── access_token/              #   访问令牌（权限校验）
│   │   └── ability_access_control.h
│   ├── asset/                     #   安全存储（密码、令牌）
│   ├── device_certificate/        #   设备证书管理
│   └── huks/                      #   通用密钥库（HUKS）
│       └── include/native_huks_api.h
│
├── sensors/                       # 传感器
│   ├── miscdevice/vibrator/       #   振动器
│   └── sensor/                    #   传感器（加速度计、陀螺仪等）
│
├── startup/                       # 启动
│   └── init/syscap/               #   系统能力查询
│
├── TEEKit/                        # TEE Kit（可信执行环境）
│   └── include/
│       ├── tee/                   #   TEE 内部 API
│       └── tee_client/            #   TEE 客户端 API
│
├── telephony/                     # 电话
│   ├── cellular_data/             #   蜂窝数据
│   └── core_service/              #   核心服务（SIM 卡、网络状态）
│
├── third_party/                   # 第三方库
│   ├── egl/                       #   EGL
│   ├── icu4c/                     #   ICU 国际化库
│   ├── libuv/                     #   libuv（异步 IO）
│   ├── mindspore/                 #   MindSpore Lite
│   ├── musl/                      #   musl C 库
│   ├── node/                      #   Node.js N-API 兼容层
│   ├── openGLES/                  #   OpenGL ES 头文件
│   ├── openSLES/                  #   OpenSL ES 音频
│   ├── vulkan-headers/            #   Vulkan 头文件
│   └── zlib/                      #   zlib 压缩库
│
├── web/                           # Web
│   └── webview/interfaces/native/ #   WebView 原生接口
│
├── docs/                          # 官方文档
│   ├── capi_naming.md             #   C API 命名规范
│   ├── howto_add.md               #   C API 构建添加指南
│   └── user_guide.md              #   C API 用户指南
│
├── build-tools/                   # 构建工具
│   └── capi_parser/               #   C API 解析工具（兼容性检查）
│
├── wiki/                          # 📚 本 Wiki 文档
│   ├── README.md                  #   Wiki 简介
│   ├── SUMMARY.md                 #   导航与阅读路线
│   ├── 00_Overview.md             #   项目概览
│   ├── 01_Directory_Structure.md  #   ⬅️ 本文档
│   ├── 02_Architecture.md         #   架构说明
│   ├── 03_NAPI_Reference.md       #   N-API 接口文档
│   ├── 04_Internal_API.md         #   内部 API
│   ├── 05_GN_Build.md             #   GN 构建目标
│   ├── 06_Build_Artifacts.md      #   编译产物
│   ├── 07_Security_Analysis.md    #   安全风险评审
│   ├── 08_Common_Issues.md        #   常见问题
│   └── appendix/                  #   附录
│       ├── Callgraphs.md          #     调用链
│       └── Config_Flags.md        #     配置项
│
├── README.md                      # 项目说明
├── README.en.md                   # 项目说明（英文）
├── ndk_targets.gni                # NDK 构建目标列表（287+ 目标）
├── OAT.xml                        # 开源合规检查配置
└── LICENSE                        # 许可证
```

---

## 2. 模块分类与职责

### 2.1 按功能域分类

| 功能域 | 包含模块 | 主要职责 |
|--------|----------|----------|
| **UI 框架** | arkui, ani | ArkUI 渲染、NAPI 绑定、窗口管理 |
| **多媒体** | multimedia | 音视频编解码、相机、播放、图像处理 |
| **图形** | graphic | 2D 绘制、OpenGL/GLES、Vulkan |
| **安全** | security, CryptoArchitectureKit | 密钥管理、加密、证书、权限 |
| **网络** | network, ConnectivityKit | HTTP、WebSocket、WiFi、蓝牙 |
| **存储** | distributeddatamgr, filemanagement | RDB、Preferences、文件 IO |
| **基础服务** | hiviewdfx, global, resourceschedule | 日志、资源、调度、国际化 |
| **输入/外设** | multimodalinput, GameControllerKit, drivers | 输入事件、游戏手柄、USB/HID |
| **系统能力** | ability, bundlemanager, startup | Ability 生命周期、包管理、启动 |

### 2.2 按层级分类

```
应用层
├── GameControllerKit         # 游戏手柄输入
├── NotificationKit           # 通知
└── BasicServicesKit          # 打印/扫描/时间等

框架层
├── arkui/ace_engine          # ArkUI 渲染引擎
├── ark_runtime/jsvm          # JS 虚拟机
├── ani                       # Ark Native Interface
└── ability                   # Ability 框架

服务层
├── multimedia/*              # 多媒体服务
├── graphic/*                 # 图形服务
├── security/*                # 安全服务
├── network/*                 # 网络服务
└── distributeddatamgr/*      # 数据服务

基础层
├── hiviewdfx/*               # 调测诊断
├── global/*                  # 资源/国际化
├── filemanagement/*          # 文件管理
├── resourceschedule/*        # 资源调度
└── commonlibrary/*           # 公共库

系统层
├── drivers/*                 # 设备驱动接口
├── TEEKit                    # 可信执行环境
├── startup/*                 # 启动相关
└── telephony/*               # 电话服务
```

---

## 3. 头文件组织模式

### 3.1 三种常见模式

| 模式 | 路径示例 | 适用场景 |
|------|----------|----------|
| **include/ 子目录** | `hiviewdfx/hilog/include/hilog/log.h` | 标准模式，头文件多 |
| **直接放置** | `arkui/window_manager/oh_window.h` | 简单模块，头文件少 |
| **多级 include** | `global/resource_management/include/rawfile/*.h` | 复杂模块，按功能细分 |

### 3.2 头文件命名规范

| 前缀 | 含义 | 示例 |
|------|------|------|
| `native_*.h` | 原生 API | `native_window.h` |
| `oh_*.h` | OpenHarmony 标准 API | `oh_window.h` |
| `*.h` | 模块内部 | `log.h` |

---

## 4. 关键文件速查

| 文件 | 路径 | 作用 |
|------|------|------|
| N-API 主头文件 | `arkui/napi/native_api.h` | OpenHarmony N-API 扩展 |
| N-API 符号定义 | `arkui/napi/libnapi.ndk.json` | 306 个 N-API 符号 |
| 构建目标列表 | `ndk_targets.gni` | 287+ 个 NDK target |
| 日志 API | `hiviewdfx/hilog/include/hilog/log.h` | HiLog 日志系统 |
| 音频 API | `multimedia/audio_framework/ohaudio.ndk.json` | 130+ 音频 API |
| 图像 API | `multimedia/image_framework/include/image/*.h` | PixelMap/ImageSource |
| 窗口 API | `graphic/graphic_2d/native_window/native_window.h` | 原生窗口 |
| 绘制 API | `graphic/graphic_2d/native_drawing/*.h` | 50+ 绘制 API |
| 权限 API | `security/access_token/ability_access_control.h` | 权限校验 |
| 密钥 API | `security/huks/include/native_huks_api.h` | HUKS 密钥管理 |

---

## 5. 代码证据

| 结论 | 证据文件 | 位置 |
|------|----------|------|
| 目录结构 | `README.md` | 第 8-47 行 |
| N-API 定义 | `arkui/napi/libnapi.ndk.json` | 全文 |
| 构建目标 | `ndk_targets.gni` | 第 17-287 行 |
| 头文件清单 | `glob` 搜索 | 100+ 个 .h 文件 |

---

## 6. 相关跳转

- **上一章**: [项目概览](./00_Overview.md)
- **下一章**: [架构说明](./02_Architecture.md)
- **API 文档**: [N-API 接口文档](./03_NAPI_Reference.md)
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**目录结构文档 - 基于代码生成**

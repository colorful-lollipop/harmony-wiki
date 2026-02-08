# 安全风险评审

## 评审范围

本评审涵盖 castengine_wifi_display 部件的以下方面：

| 范围 | 覆盖情况 |
|------|----------|
| N-API 接口 | ✅ 已覆盖 |
| SA 配置 | ✅ 已覆盖 |
| 权限配置 | ✅ 已覆盖 |
| 网络通信 | ✅ 已覆盖 |
| 文件操作 | ✅ 已覆盖 |
| 进程间通信 | ✅ 已覆盖 |
| 媒体数据处理 | ✅ 已覆盖 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        信任边界                              │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           sharing_service 进程 (特权进程)           │   │
│  │                                                      │   │
│  │  • 拥有 system_basic 权限级别                       │   │
│  │  • 拥有多个敏感系统权限                            │   │
│  │  • 访问相机、麦克风、WiFi 等敏感资源               │   │
│  │  • 监听 TCP/UDP 端口                               │   │
│  │  • 读写 /data/service/ 下的数据目录                │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   外部输入                           │   │
│  │                                                      │   │
│  │  • N-API 调用 (用户应用)                            │   │
│  │  • 外部 WFD 设备 RTSP/RTP 流                       │   │
│  │  • D-Bus/IPC 消息                                   │   │
│  │  • 配置文件读取                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 攻击面清单

| 攻击面 | 类型 | 描述 |
|--------|------|------|
| N-API 接口 | API | JS 到 Native 的调用入口 |
| WFD 协议 | 网络 | 外部设备的 RTSP/RTP 连接 |
| IPC/RPC | 进程通信 | 与其他系统服务的通信 |
| 文件系统 | 文件 | 配置文件和数据文件读写 |
| WiFi 网络 | 网络 | WiFi 设备发现和连接 |
| Surface/Buffer | 共享内存 | 媒体数据传输 |

## 安全风险分析

### 风险 1：路径遍历风险 ⚠️

**风险等级**: 中

**证据**:
- `services/etc/sharing_service.cfg:5-10` 配置了数据目录创建
- 目录权限设置为 0777

```json
"mkdir /data/service/el1/public/database/sharingcodec 0777 audio audio"
```

**可利用路径**:
```
恶意应用 → 创建符号链接 → 指向敏感目录 → 目录被误创建设置权限
```

**影响**:
- 可能导致目录权限被错误设置
- 敏感数据可能被非授权访问

**修复建议**:
- 使用 `makedirs` 代替 `mkdir` 时验证路径
- 避免使用宽泛的目录权限
- 在创建目录前检查路径是否为符号链接

### 风险 2：WiFi 信息泄露 ⚠️

**风险等级**: 中

**证据**:
- `services/etc/sharing_service.cfg:36-39` 配置了 WiFi 权限

```json
"ohos.permission.GET_WIFI_PEERS_MAC",
"ohos.permission.GET_WIFI_INFO",
"ohos.permission.SET_WIFI_INFO",
"ohos.permission.GET_WIFI_LOCAL_MAC"
```

**可利用路径**:
```
恶意应用 → 获取 WiFi MAC 地址 → 设备追踪/识别
```

**影响**:
- 用户设备可能被跨应用追踪
- WiFi 配置信息泄露

**修复建议**:
- 仅在必要时请求 WiFi 权限
- 对 WiFi 信息进行脱敏处理
- 审计 WiFi 权限使用场景

### 风险 3：屏幕捕获权限滥用 ⚠️

**风险等级**: 高

**证据**:
- `services/etc/sharing_service.cfg:35` 配置了屏幕捕获权限

```json
"ohos.permission.CAPTURE_SCREEN"
```

**可利用路径**:
```
恶意应用 → 获取 CAPTURE_SCREEN 权限 → 捕获用户屏幕内容
```

**影响**:
- 用户隐私信息泄露
- 敏感应用内容被截取

**修复建议**:
- 严格控制 CAPTURE_SCREEN 权限授予
- 添加运行时权限确认
- 对屏幕捕获进行水印标识

### 风险 4：设备绑定机制风险 ⚠️

**风险等级**: 中

**证据**:
- `interfaces/innerkits/native/wfd/include/wfd_sink.h:48-49`

```cpp
virtual int32_t GetBoundDevicesList(std::vector<BoundDeviceInfo> &devices) = 0;
virtual int32_t DeleteBoundDevice(std::string &deviceAddress) = 0;
```

**可利用路径**:
```
恶意设备 → 伪造绑定请求 → 被添加到设备列表 → 接收投屏数据
```

**影响**:
- 恶意设备可能接收投屏内容
- 设备绑定被未授权修改

**修复建议**:
- 绑定操作需要用户确认
- 对设备信息进行严格校验
- 定期验证已绑定设备

### 风险 5：外部协议数据校验不足 ⚠️

**风险等级**: 高

**证据**:
- `services/protocol/` 实现 RTSP/RTP 协议解析
- 无明显边界检查代码

**可利用路径**:
```
恶意 WFD 设备 → 发送畸形 RTSP/RTP 包 → 触发解析漏洞
```

**影响**:
- 缓冲区溢出
- 拒绝服务
- 远程代码执行

**修复建议**:
- 对所有协议字段进行严格校验
- 使用内存安全语言或库
- 启用编译器安全标志 (-D_FORTIFY_SOURCE)

### 风险 6：Surface 共享内存风险 ⚠️

**风险等级**: 中

**证据**:
- `interfaces/innerkits/native/wfd/include/wfd_sink.h:40-41`

```cpp
virtual int32_t AppendSurface(std::string deviceId, uint64_t surfaceId) = 0;
virtual int32_t AppendSurface(std::string deviceId, sptr<IBufferProducer> producer) = 0;
```

**可利用路径**:
```
恶意应用 → 注入恶意 Surface Buffer → 获取其他应用数据
```

**影响**:
- 跨应用数据泄露
- 内存数据被篡改

**修复建议**:
- 对 Surface Buffer 进行来源校验
- 限制可共享的 Surface 范围
- 使用内存保护机制

### 风险 7：IPC 消息注入风险 ⚠️

**风险等级**: 中

**证据**:
- `services/interaction/` 实现 IPC/RPC 通信

**可利用路径**:
```
恶意进程 → 伪造 IPC 消息 → 劫持业务逻辑
```

**影响**:
- 业务逻辑被篡改
- 权限提升

**修复建议**:
- 对 IPC 消息进行签名验证
- 使用安全的 RPC 机制
- 限制可发送 IPC 消息的进程

### 风险 8：日志信息泄露 ⚠️

**风险等级**: 低

**证据**:
- `frameworks/kitsimpl/js/wfd/wfd_napi_sink.cpp:131-132`

```cpp
SHARING_LOGW("get app, bundle: %{public}s, ability: %{public}s.",
            item.ability.GetBundleName().c_str(),
            item.ability.GetAbilityName().c_str());
```

**可利用路径**:
```
攻击者 → 获取日志 → 获取应用包名和能力名 → 社会工程攻击
```

**影响**:
- 应用信息泄露
- 用户行为追踪

**修复建议**:
- 限制日志详细程度
- 避免在日志中输出敏感信息
- 实施日志访问控制

## 已有的安全措施

### 编译器安全标志

**证据**:
- `BUILD.gn:38-40` 配置了 CFI

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
}
```

- `BUILD.gn:37` 配置了 FORTIFY_SOURCE

```gn
"-D_FORTIFY_SOURCE=2"
```

### 可见性控制

**证据**:
- `interfaces/innerkits/native/wfd/include/wfd_sink.h:52`

```cpp
class __attribute__((visibility("default"))) WfdSinkFactory {
```

### SELinux

**证据**:
- `services/etc/sharing_service.cfg:43`

```json
"secon" : "u:r:sharing_service:s0"
```

## 安全建议汇总

| 风险 | 等级 | 建议 |
|------|------|------|
| 路径遍历 | 中 | 验证创建路径，避免宽泛权限 |
| WiFi 信息泄露 | 中 | 脱敏处理，审计使用场景 |
| 屏幕捕获 | 高 | 添加运行时确认 |
| 设备绑定 | 中 | 用户确认，设备校验 |
| 协议数据 | 高 | 严格校验，安全编码 |
| Surface 共享 | 中 | 来源校验，内存保护 |
| IPC 消息 | 中 | 签名验证，访问控制 |
| 日志泄露 | 低 | 限制日志，避免敏感信息 |

## 相关文档

- [N-API 接口](03_N-API_Interface.md)
- [SA 配置](07_SA_Configuration.md)
- [GN 构建配置](05_GN_Build.md)

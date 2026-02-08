# 常见问题与调试

## 目的与适用范围

本文档汇总分布式屏幕的常见问题、调试方法和定位路径。

---

## 问题分类

### 1. 服务启动问题

#### 问题: SA服务无法启动

**现象**: 
- 调用`InitSource`/`InitSink`返回`ERR_DH_SCREEN_SA_GET_SOURCESERVICE_FAIL`
- 日志中出现"Get source service failed"

**定位路径**:
```
1. 检查SA配置文件是否存在
   $ ls /system/profile/4807.json /system/profile/4808.json

2. 检查dscreen进程是否运行
   $ ps -ef | grep dscreen

3. 检查init配置
   $ cat /etc/init/dscreen.cfg

4. 查看系统日志
   $ hilog | grep DSCREEN
```

**常见原因**:
| 原因 | 检查方法 | 修复 |
|------|----------|------|
| SA配置文件缺失 | 检查`/system/profile/` | 重新编译安装 |
| 库文件缺失 | 检查`/system/lib/libdistributed_screen_*.z.so` | 重新编译安装 |
| 权限问题 | 检查`dscreen.cfg`的selabel | 修正sepolicy |
| 依赖服务未启动 | 检查`samgr`和`safwk` | 确保基础服务启动 |

**代码证据**: `common/include/dscreen_errcode.h:26-28`
```cpp
ERR_DH_SCREEN_SA_GET_SOURCESERVICE_FAIL = -50001,
ERR_DH_SCREEN_SA_GET_SINKSERVICE_FAIL = -50006,
```

---

### 2. 权限问题

#### 问题: 权限检查失败

**现象**:
- 调用`RegisterDistributedHardware`返回`ERR_DH_SCREEN_SA_CHECK_ENABLE_PERMISSION_FAIL` (-50039)
- 日志中出现"Check enable permission failed"

**定位路径**:
```
1. 检查应用权限声明
   查看应用的config.json中是否声明了ohos.permission.ENABLE_DISTRIBUTED_HARDWARE

2. 检查权限授予状态
   $ accesstoken_tool -c <tokenId>

3. 查看权限检查日志
   $ hilog | grep -i permission
```

**代码证据**: `services/screenservice/sourceservice/dscreenservice/src/dscreen_source_stub.cpp`

```cpp
bool DScreenSourceStub::HasEnableDHPermission() {
    Security::AccessToken::AccessTokenID callerToken = IPCSkeleton::GetCallingTokenID();
    const std::string permissionName = "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE";
    int32_t result = Security::AccessToken::AccessTokenKit::VerifyAccessToken(callerToken, permissionName);
    return (result == Security::AccessToken::PERMISSION_GRANTED);
}
```

**修复方法**:
在应用`config.json`中声明权限:
```json
"reqPermissions": [
    {
        "name": "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE",
        "reason": "$string:permission_distributed_hardware"
    }
]
```

---

### 3. 设备发现与连接问题

#### 问题: 设备无法发现或连接失败

**现象**:
- `RegisterDistributedHardware`返回`ERR_DH_SCREEN_SA_ENABLE_FAILED` (-50016)
- 软总线连接超时

**定位路径**:
```
1. 检查设备是否在同一网络
   $ ifconfig 查看IP地址

2. 检查软总线服务状态
   $ ps -ef | grep softbus

3. 检查设备管理器状态
   $ hidumper -s 4803  (DeviceManager SA)

4. 查看软总线日志
   $ hilog | grep -i softbus

5. 检查同账号验证
   $ hidumper -s 4807 -a "dump"
```

**代码证据**: `services/softbusadapter/include/softbus_permission_check.h:35-36`

```cpp
static bool CheckSrcPermission(const std::string &sinkNetworkId);
static bool CheckSinkPermission(const AccountInfo &callerAccountInfo);
```

**常见原因**:
| 原因 | 检查方法 | 修复 |
|------|----------|------|
| 设备不在同一网络 | ping测试 | 确保设备同局域网 |
| 软总线未启动 | 检查进程 | 重启设备或软总线服务 |
| 账号不同 | 检查账号设置 | 使用相同华为账号登录 |
| 设备未认证 | 检查设备管理器 | 完成设备配对认证 |

---

### 4. 屏幕传输问题

#### 问题: 屏幕数据传输失败

**现象**:
- 连接成功但屏幕黑屏
- 画面卡顿或花屏
- 传输错误码`ERR_DH_SCREEN_TRANS_*`

**定位路径**:
```
1. 检查编码器状态
   $ hilog | grep -i encoder

2. 检查传输通道状态
   $ hilog | grep -i "data channel\|ScreenSourceTrans\|ScreenSinkTrans"

3. 检查软总线会话
   $ hilog | grep -i "session\|softbus"

4. 查看Dump数据（root版本）
   $ ls /data/data/dscreen/

5. 检查Surface状态
   $ hilog | grep -i surface
```

**错误码参考**: `common/include/dscreen_errcode.h:67-79`

```cpp
ERR_DH_SCREEN_TRANS_ERROR = -51000,
ERR_DH_SCREEN_TRANS_TIMEOUT = -51001,
ERR_DH_SCREEN_TRANS_NULL_VALUE = -51002,
ERR_DH_SCREEN_TRANS_ILLEGAL_PARAM = -51003,
ERR_DH_SCREEN_TRANS_SESSION_CLOSED = -51005,
ERR_DH_SCREEN_TRANS_CREATE_CODEC_FAILED = -51006,
```

**常见原因**:
| 原因 | 检查方法 | 修复 |
|------|----------|------|
| 编解码器失败 | 检查av_codec日志 | 检查编码参数，尝试切换编码类型 |
| 会话断开 | 检查网络稳定性 | 检查网络，重连 |
| Surface异常 | 检查graphic日志 | 重启应用或系统 |
| 分辨率不支持 | 检查VideoParam | 调整分辨率到支持范围 |

---

### 5. 编解码问题

#### 问题: 编码/解码失败

**现象**:
- 错误码`ERR_DH_SCREEN_CODEC_*` (-53000 ~ -53008)
- 编码器无法启动

**定位路径**:
```
1. 检查编解码器支持
   查看设备支持的编码格式

2. 检查编码参数
   $ hilog | grep -i "VideoParam\|codecType"

3. 查看编解码器日志
   $ hilog | grep -i "MediaCodec\|ImageSourceEncoder\|ImageSinkDecoder"
```

**支持的编码格式**: `common/include/dscreen_constants.h:47-51`

```cpp
enum CodecType : uint8_t {
    VIDEO_CODEC_TYPE_VIDEO_H264 = 0,  // 默认
    VIDEO_CODEC_TYPE_VIDEO_H265 = 1,
    VIDEO_CODEC_TYPE_VIDEO_MPEG4 = 2,
};
```

**修复方法**:
- 尝试使用H264编码（兼容性最好）
- 降低分辨率或帧率
- 检查设备编解码器能力

---

## 调试方法

### 1. HiLog日志

**日志Domain**: `0xD004140`

**标签**: 
- `dscreenutil` - 工具库
- `dscreensourcesdk` - Source SDK
- `dscreensinksdk` - Sink SDK
- `dscreensource` - Source服务
- `dscreensink` - Sink服务

**查看日志**:
```bash
# 查看所有分布式屏幕日志
hilog | grep DSCREEN

# 查看特定标签日志
hilog | grep dscreensource

# 查看错误级别日志
hilog -E | grep DSCREEN
```

### 2. HiDumper

**Source服务**:
```bash
# 查看Source服务状态
hidumper -s 4807

# Dump详细信息
hidumper -s 4807 -a "dump"
```

**Sink服务**:
```bash
# 查看Sink服务状态
hidumper -s 4808

# Dump详细信息
hidumper -s 4808 -a "dump"
```

**代码证据**: `services/screenservice/sourceservice/dscreenservice/include/dscreen_source_service.h:44`

```cpp
int32_t Dump(int32_t fd, const std::vector<std::u16string>& args) override;
```

### 3. 事件追踪 (HiTrace)

**开启追踪**:
```cpp
#include "dscreen_hitrace.h"

// 代码中已集成HITRACE宏
HITRACE_NAME("DScreen::Enable");  // 在common/include/dscreen_hitrace.h定义
```

**查看追踪**:
```bash
# 抓取trace
hitrace -t 10 -o /data/log/trace.html

# 分析trace
# 在Chrome中打开trace.html查看
```

### 4. Dump数据（root版本）

**启用Dump**:
在root版本中，以下宏定义启用:
- `DUMP_DSCREEN_FILE` (Source端)
- `DUMP_DSCREENREGION_FILE` (Sink端)

**Dump路径**: `/data/data/dscreen/`

**Dump大小限制**: 295MB

**代码证据**: `common/include/dscreen_constants.h:94,150`

```cpp
const std::string DUMP_FILE_PATH = "/data/data/dscreen";
constexpr uint32_t DUMP_FILE_MAX_SIZE = 295 * 1024 * 1024;
```

---

## 性能调优

### 1. 视频参数优化

**关键参数** (`services/common/utils/include/video_param.h`):

| 参数 | 默认值 | 调优建议 |
|------|--------|----------|
| 分辨率 | 设备相关 | 根据网络带宽调整 |
| 帧率 | 60fps | 网络差时降低至30fps |
| 码率 | 12Mbps | 根据质量需求调整 |
| 编码格式 | H264 | 性能敏感时用硬件编码 |

### 2. 延迟优化

**减少延迟的方法**:
1. 降低编码缓冲区大小
2. 使用低延迟编码配置
3. 减少软总线传输缓冲
4. 优化解码器输出延迟

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目基本信息
- [架构设计](01_Architecture.md) - 架构说明
- [安全风险](07_Security.md) - 安全分析
# 构建与调试指南

## 构建命令

### 完整构建

```bash
# 在 OpenHarmony 根目录下执行
./build.sh --product-name <product> --parts distributed_camera
```

### 单独构建模块

```bash
# 构建整个 distributed_camera 部件
hb build -p distributed_camera

# 构建特定模块
hb build -p distributed_camera -T "//foundation/distributedhardware/distributed_camera/common:distributed_camera_utils"
```

---

## 常见问题

### Q1: 编译报错 "undefined reference to"

**问题**：链接错误，找不到符号定义

**解决方案**：
1. 检查 `BUILD.gn` 中 `external_deps` 是否正确声明
2. 检查 `deps` 依赖是否正确
3. 确认依赖的库是否已构建

**示例**：
```
error: undefined reference to 'OHOS::DistributedHardware::DistributedCameraSourceService::xxx'
```

检查：`services/cameraservice/sourceservice/BUILD.gn` 中是否包含相关源文件。

---

### Q2: 权限校验失败

**问题**：调用 IPC 接口返回 `DCAMERA_BAD_VALUE`

**排查步骤**：
1. 检查调用者是否声明了必要权限
   ```cpp
   // 需要声明的权限
   "ohos.permission.ENABLE_DISTRIBUTED_HARDWARE"
   "ohos.permission.ACCESS_DISTRIBUTED_HARDWARE"
   ```
2. 检查 `dcamera.cfg` 中权限配置
3. 使用 `hdc shell` 查看权限状态

**调试命令**：
```bash
hdc shell
hidumper -s AccessTokenService -a '-p'
```

---

### Q3: SA 启动失败

**问题**：Source/Sink SA 无法启动

**排查步骤**：
1. 检查 SA 配置 (4803.json / 4804.json)
2. 检查 `dcamera.cfg` 配置
3. 查看系统日志

**调试命令**：
```bash
# 查看 SA 状态
hdc shell
sa_ps | grep dcamera

# 查看日志
hilog | grep -E "dcamera|distributed_camera"
```

---

### Q4: 设备无法发现

**问题**：分布式设备列表中看不到目标设备

**排查步骤**：
1. 检查设备是否在同一局域网
2. 检查 `distributed_hardware_fwk` 是否正常
3. 检查 SoftBus 是否正常

**调试命令**：
```bash
# 查看设备发现状态
hdc shell
device_manager find

# 查看 SoftBus 状态
hdc shell
softbus_server
```

---

### Q5: 预览卡顿/延迟高

**问题**：分布式相机预览画面卡顿

**排查步骤**：
1. 检查网络带宽
2. 检查 `distributed_camera_wakeup_enabled` 编译开关
3. 检查数据处理流水线配置

**相关参数**：
```gn
# BUILD.gn 中可调整
distributed_camera_wakeup_enabled = true  # 启用唤醒
distributed_camera_common = true           # 使用 FFmpeg 优化
```

---

### Q6: 内存占用过高

**问题**：预览时内存持续增长

**排查步骤**：
1. 检查 `DCAMERA_MMAP_RESERVE` 是否启用
2. 检查 DataBuffer 释放逻辑
3. 检查 Pipeline 资源释放

**相关代码**：
- `common/src/utils/data_buffer.cpp`
- `services/data_process/src/pipeline/*.cpp`

---

## 调试技巧

### 1. 启用调试日志

```bash
# 重新编译时添加 debug flag
build_variant = "root"  # 在 ohos_var.gni 中设置
```

日志位置：`/data/log/`

### 2. 启用 Dump 功能

```gn
# 在对应模块的 BUILD.gn 中
if (build_variant == "root") {
    defines += [ "DUMP_DCAMERA_FILE" ]
}
```

### 3. 使用 HiDumper

```bash
# 查看服务状态
hdc shell
hidumper -s 4803              # Source SA
hidumper -s 4804              # Sink SA
hidumper -s DistributedCamera # 完整信息
```

### 4. 追踪 IPC 调用

```bash
# 使用 hbps 追踪 binder 调用
hbps -p distributed_camera -s 4803
```

---

## 性能调优

### 网络带宽优化

| 参数 | 调整方式 | 影响 |
|------|----------|------|
| `distributed_camera_wakeup_enabled` | BUILD.gn | 减少延迟 |
| `DCAMERA_WAKEUP` | 条件编译 | 唤醒优化 |

### 视频编码优化

| 参数 | 调整方式 | 影响 |
|------|----------|------|
| `DCAMERA_SUPPORT_FFMPEG` | BUILD.gn | FFmpeg 编解码 |
| `distributed_camera_common` | BUILD.gn | 选择处理路径 |

---

## 相关资源

| 资源 | 链接 |
|------|------|
| OpenHarmony 构建指南 | https://gitee.com/openharmony/docs |
| 分布式硬件框架 | https://gitee.com/openharmony/distributedhardware_distributed_hardware_fwk |
| 相机框架 | https://gitee.com/openharmony/multimedia_camera_framework |
| 错误码参考 | `common/include/constants/distributed_camera_errno.h` |

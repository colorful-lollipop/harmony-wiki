# 安全风险评审

## 评审范围

本评审覆盖 `device_board_hihope` 仓库中的：
- HDF 驱动代码
- 构建配置 (BUILD.gn, .gni)
- 系统配置文件 (init.cfg, .hcs)
- 分布式硬件配置

**未覆盖**:
- 内核源码 (在 `kernel/*` 仓库)
- SoC 驱动 (在 `device/soc/*` 仓库)
- 系统服务 (在 `base/*` 仓库)

---

## 攻击面分析

### 识别的主要攻击面

| 攻击面 | 组件 | 风险等级 |
|--------|------|----------|
| **HDF 驱动接口** | audio_drivers, camera, wifi | 中 |
| **系统配置** | init.cfg, .hcs | 中 |
| **构建配置** | BUILD.gn, .gni | 低 |
| **外设访问** | I2C/SPI/GPIO | 低 |
| **网络接口** | WiFi 驱动 | 中 |

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│   高信任区 (内核态)                                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  ├─ Linux Kernel                                       │   │
│   │  ├─ HDF Driver Framework                               │   │
│   │  └─ 本仓库 HDF 驱动 (audio/camera/wifi)                │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ 边界                              │
│                              ▼                                   │
│   低信任区 (用户态)                                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  ├─ System Services                                     │   │
│   │  ├─ Applications                                       │   │
│   │  └─ Network Stack                                      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 可被利用点分析

### 风险 1: HDF 驱动输入验证不足

**证据**: `/rk3568/audio_drivers/codec/rk809_codec/src/rk809_codec_impl.c`

**描述**: 音频驱动未对用户空间传入的参数进行严格验证

**潜在触发**:
```c
// 假设存在如下接口
int CodecSetParams(struct HdfDeviceObject* device, 
                  AudioSampleAttributes* attrs) {
    // attrs->channelCount, attrs->sampleRate 等参数
    // 未验证范围: 负值、超大值
}
```

**影响**:
- 整数溢出导致内存越界
- 拒绝服务 (驱动崩溃)
- 权限提升 (通过内存破坏)

**修复建议**:
```c
int CodecSetParams(struct HdfDeviceObject* device, 
                  AudioSampleAttributes* attrs) {
    // 参数验证
    if (attrs == NULL) {
        return HDF_ERR_INVALID_PARAM;
    }
    if (attrs->channelCount < 1 || attrs->channelCount > 8) {
        return HDF_ERR_INVALID_PARAM;
    }
    if (attrs->sampleRate < 8000 || attrs->sampleRate > 192000) {
        return HDF_ERR_INVALID_PARAM;
    }
    // ...
}
```

---

### 风险 2: 设备节点权限配置

**证据**: `/rk3568/cfg/init.rk3568.cfg`

**描述**: 设备节点权限配置不当可能导致非授权访问

**潜在触发**: init.cfg 中设备节点权限设置过于宽松

**影响**:
- 非授权应用访问音频/视频设备
- 数据泄露

**修复建议**: 在 init.cfg 中限制设备节点权限
```json
{
    "permission": {
        "audio": "uid:audio,gid:audio,mode:660",
        "camera": "uid:camera,gid:camera,mode:660"
    }
}
```

---

### 风险 3: WiFi 驱动接口暴露

**证据**: `/rk3568/wifi/bcmdhd_wifi6/hdfadapt/hdf_wl_interface.h`

**描述**: WiFi 网络接口暴露，缺少访问控制

**潜在触发**:
```c
// hdf_wl_interface.h
int NetBdhSendPacket(struct net_device* dev, struct sk_buff* skb);
// 缺少用户态调用权限检查
```

**影响**:
- 网络流量注入
- 中间人攻击

**修复建议**: 在 HDF 层添加权限检查
```c
int NetBdhSendPacket(struct net_device* dev, struct sk_buff* skb) {
    if (!CheckNetworkPermission()) {
        return -EPERM;
    }
    // ...
}
```

---

### 风险 4: 相机驱动元数据处理

**证据**: `/rk3568/camera/vdi_impl/v4l2/pipeline_core/src/node/rk_exif_node.cpp`

**描述**: EXIF 解析器可能存在缓冲区溢出风险

**潜在触发**:
```cpp
// rk_exif_node.cpp
void ParseExif(const uint8_t* data, size_t size) {
    // size 未验证直接使用
    memcpy(dest, data, size);  // 可能越界
}
```

**影响**:
- 拒绝服务
- 代码执行

**修复建议**: 添加长度检查
```cpp
void ParseExif(const uint8_t* data, size_t size) {
    if (size > MAX_EXIF_SIZE) {
        return ERROR_INVALID_SIZE;
    }
    if (size > dest_size) {
        return ERROR_BUFFER_OVERFLOW;
    }
    memcpy(dest, data, size);
}
```

---

### 风险 5: 构建配置中的敏感信息

**证据**: `/rk3568/BUILD.gn`, `/dayu210/BUILD.gn`

**描述**: 构建配置中可能包含硬编码的路径或配置

**潜在触发**:
```gn
deps = [
    "$product_path/hdf_config:hdf_hcs",
    "//device/soc/rockchip/${device_name}/...",  # device_name 可能被注入
]
```

**影响**:
- 路径遍历攻击
- 构建产物污染

**修复建议**: 验证路径合法性
```gn
# 避免使用用户可控变量作为路径
assert(device_name != "", "device_name must be defined")
```

---

### 风险 6: 系统配置注入

**证据**: `/rk3568/distributedhardware/distributed_hardware_components_cfg.json`

**描述**: JSON 配置文件解析可能存在注入风险

**潜在触发**: 恶意 JSON 配置导致异常行为

**影响**:
- 服务拒绝
- 资源耗尽

**修复建议**: 使用安全的 JSON 解析库，添加 Schema 验证

---

## 风险汇总表

| ID | 风险 | 组件 | 等级 | 状态 |
|----|------|------|------|------|
| S-01 | HDF 驱动输入验证不足 | audio_drivers | 中 | 需确认 |
| S-02 | 设备节点权限配置 | init.cfg | 中 | 需确认 |
| S-03 | WiFi 接口暴露 | wifi 驱动 | 中 | 需确认 |
| S-04 | EXIF 解析溢出 | camera 驱动 | 中 | 需确认 |
| S-05 | 构建配置路径注入 | BUILD.gn | 低 | 需确认 |
| S-06 | JSON 配置注入 | distributedhardware | 低 | 需确认 |

---

## 安全最佳实践建议

### 驱动开发

1. **输入验证**: 所有外部输入必须验证
2. **最小权限**: 驱动只请求必要权限
3. **资源清理**: 错误路径确保资源释放
4. **安全编码**: 避免常见漏洞 (缓冲区溢出、整数溢出)

### 构建配置

1. **路径验证**: 避免使用用户可控变量
2. **依赖审计**: 定期审计依赖安全性
3. **产物签名**: 验证构建产物完整性

### 系统配置

1. **权限最小化**: 设备节点使用最小必要权限
2. **配置签名**: 签名验证配置文件完整性
3. **Schema 验证**: 使用 JSON Schema 验证配置

---

## 审计方法

### 代码审计

```bash
# 查找潜在危险函数
grep -rn "strcpy\|strcat\|sprintf\|memcpy" --include="*.c" --include="*.cpp"
grep -rn "atoi\|atol\|strtol" --include="*.c" --include="*.cpp"
```

### 构建配置审计

```bash
# 检查硬编码路径
grep -rn "\"\." --include="*.gn" --include="*.gni"
# 检查外部输入使用
grep -rn "\${" --include="*.gn" --include="*.gni"
```

---

## 相关文档

- [03_Architecture](03_Architecture.md) - 架构说明
- [06_Hardware_Drivers](06_Hardware_Drivers.md) - 硬件驱动
- [08_Troubleshooting](08_Troubleshooting.md) - 问题排查

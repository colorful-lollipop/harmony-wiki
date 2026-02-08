# 安全风险评审

## 文档信息

- **目的**: 分析 Rockchip OpenHarmony 仓库的安全风险，识别攻击面和可被利用点
- **适用范围**: 所有芯片平台
- **关键结论**: 本仓库主要安全风险集中在 DRM 权限控制、内核驱动输入验证和 IPC 权限检查

## 安全评估范围

### 评估范围

| 范围 | 内容 |
|------|------|
| **包含** | 内核驱动代码、HDI/VDI 实现、系统调用封装、权限控制 |
| **排除** | 测试代码 (`test/`, `unittest/`, `fuzz/`)、上层框架代码 |

### 评估方法

1. **攻击面分析** - 识别外部输入入口点
2. **信任边界** - 确定安全域边界
3. **代码审计** - 检查输入验证、边界检查、权限控制
4. **威胁建模** - STRIDE 模型分析

## 攻击面分析

### 攻击面清单

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| DRM 设备节点 | 本地 | 中 | `/dev/dri/card0` 访问控制 |
| MPP 设备节点 | 本地 | 中 | `/dev/mpp_service` 访问控制 |
| HDMI 热插拔 | 物理 | 低 | 物理接口攻击 |
| 显示缓冲区 | 本地 | 中 | GPU 内存访问 |
| 编解码数据 | 本地 | 高 | 恶意媒体数据 |
| HDF 驱动接口 | 本地 | 中 | RK2206 HDF 调用 |
| WiFi 固件 | 网络 | 高 | 无线协议攻击 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                     非信任域 (应用层)                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │  应用   │  │  服务   │  │  框架   │                     │
│  └────┬────┘  └────┬────┘  └────┬────┘                     │
└───────┼────────────┼────────────┼───────────────────────────┘
        │            │            │  ← 信任边界 (HDI 接口)
┌───────┼────────────┼────────────┼───────────────────────────┐
│       │        半信任域 (VDI 层)   │                         │
│  ┌────┴────────────┴────────────┴────┐                     │
│  │       Rockchip VDI 实现            │                     │
│  │  (本仓库: device/soc/rockchip)    │                     │
│  └────┬────────────────────────┬─────┘                     │
└───────┼────────────────────────┼────────────────────────────┘
        │                        │  ← 信任边界 (系统调用)
┌───────┼────────────────────────┼────────────────────────────┐
│       │       信任域 (内核层)   │                            │
│  ┌────┴────┐            ┌──────┴──────┐                     │
│  │  DRM    │            │   MPP       │                     │
│  │ Driver  │            │  Driver     │                     │
│  └─────────┘            └─────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点分析

### 1. DRM 权限控制绕过

**证据**: `common/sdk_linux/drivers/gpu/drm/drm_ioctl.c:544`

```c
int drm_ioctl_permit(u32 flags, struct drm_file *file_priv)
{
    // ROOT_ONLY: 仅 CAP_SYS_ADMIN 可访问
    if (unlikely((flags & DRM_ROOT_ONLY) && !capable(CAP_SYS_ADMIN)))
        return -EACCES;
    
    // AUTH: 仅已认证或渲染客户端可访问
    if (unlikely((flags & DRM_AUTH) && !drm_is_render_client(file_priv) 
                 && !file_priv->authenticated))
        return -EACCES;
    
    // MASTER: 仅当前主设备可访问
    if (unlikely((flags & DRM_MASTER) && !drm_is_current_master(file_priv)))
        return -EACCES;
}
```

**触发路径**:
1. 应用打开 `/dev/dri/card0`
2. 调用未授权 DRM ioctl
3. 绕过权限检查（如果存在漏洞）

**影响**: 中 - 可能导致显示控制权限提升

**修复建议**:
```c
// 建议: 增加额外的权限审计日志
if (flags & DRM_MASTER) {
    audit_log("DRM_MASTER ioctl attempted by pid %d", current->pid);
    if (!drm_is_current_master(file_priv)) {
        return -EACCES;
    }
}
```

### 2. 编解码器输入验证不足

**证据**: `rk3568/hardware/codec/src/hdi_mpp.c` (待确认具体行号)

**问题描述**: MPP 编解码器接收外部媒体数据，如果输入验证不充分，可能导致：
- 缓冲区溢出
- 整数溢出
- 格式解析错误

**触发路径**:
1. 应用发送恶意构造的视频帧
2. MPP 驱动解析媒体数据
3. 触发内存损坏

**影响**: 高 - 可能导致内核崩溃或代码执行

**修复建议**:
```c
// 建议: 增加输入验证
static int validate_mpp_packet(MppPacket packet) {
    size_t size = mpp_packet_get_size(packet);
    void *data = mpp_packet_get_data(packet);
    
    if (size == 0 || size > MAX_PACKET_SIZE) {
        return -EINVAL;
    }
    
    if (!data || !access_ok(data, size)) {
        return -EFAULT;
    }
    
    // 验证编码头
    if (!validate_codec_header(data, size)) {
        return -EINVAL;
    }
    
    return 0;
}
```

### 3. IPC 权限检查绕过

**证据**: `common/sdk_linux/ipc/util.c:351`

```c
int ipcperms(struct ipc_namespace *ns, struct kern_ipc_perm *ipcp, short flag)
{
    return security_ipc_permission(ipcp, flag);
}
```

**问题描述**: IPC 权限检查依赖 LSM (Linux Security Modules)，如果 SELinux 未启用，可能绕过权限检查。

**触发路径**:
1. 应用创建共享内存/消息队列
2. 未正确设置权限
3. 其他应用访问敏感数据

**影响**: 中 - 可能导致信息泄露

**修复建议**:
```c
// 建议: 强制启用 SELinux 检查
static inline int ipcperms(struct ipc_namespace *ns, 
                          struct kern_ipc_perm *ipcp, 
                          short flag) {
    int rc = security_ipc_permission(ipcp, flag);
    if (rc != 0) {
        audit_ipc_access(ipcp, flag, rc);
        return rc;
    }
    
    // 额外的权限验证
    if (!ipc_owner_or_capable(ipcp)) {
        return -EPERM;
    }
    
    return 0;
}
```

### 4. 显示缓冲区内存泄露

**证据**: `rk3568/hardware/display/src/display_gralloc/display_buffer_vdi_impl.cpp`

**问题描述**: 显示缓冲区分配后，如果应用异常退出，可能未正确释放缓冲区，导致内存泄露。

**触发路径**:
1. 应用分配显示缓冲区
2. 应用崩溃或未正确释放
3. 缓冲区持续占用内存

**影响**: 中 - 可能导致系统内存耗尽

**修复建议**:
```cpp
// 建议: 增加缓冲区引用计数和自动释放
class DisplayBufferVdiImpl : public IDisplayBufferVdi {
private:
    struct BufferInfo {
        BufferHandle* handle;
        pid_t owner;
        time_t alloc_time;
        std::atomic<int> ref_count;
    };
    
    std::map<uint32_t, BufferInfo> buffer_map_;
    std::mutex buffer_mutex_;
    
    // 定期清理孤儿缓冲区
    void CleanupOrphanBuffers() {
        std::lock_guard<std::mutex> lock(buffer_mutex_);
        time_t now = time(nullptr);
        
        for (auto it = buffer_map_.begin(); it != buffer_map_.end();) {
            if (it->second.ref_count == 0 && 
                (now - it->second.alloc_time) > BUFFER_TIMEOUT) {
                FreeMem(*it->second.handle);
                it = buffer_map_.erase(it);
            } else {
                ++it;
            }
        }
    }
};
```

### 5. WiFi 固件加载路径遍历

**证据**: `rk3568/hardware/wifi/ap6xxx/`

**问题描述**: WiFi 固件加载可能未验证固件路径，导致路径遍历攻击。

**触发路径**:
1. 应用构造恶意固件路径
2. 驱动加载固件
3. 执行恶意代码

**影响**: 高 - 可能导致任意代码执行

**修复建议**:
```c
// 建议: 严格验证固件路径
static int load_wifi_firmware(const char *fw_name) {
    char fw_path[PATH_MAX];
    
    // 只允许预定义目录
    const char *allowed_paths[] = {
        "/vendor/firmware/",
        "/system/etc/firmware/",
    };
    
    // 验证文件名不包含路径遍历
    if (strstr(fw_name, "..") || fw_name[0] == '/') {
        return -EINVAL;
    }
    
    // 构建完整路径
    snprintf(fw_path, sizeof(fw_path), "%s%s", 
             allowed_paths[0], fw_name);
    
    // 验证路径在允许范围内
    if (!path_in_allowed_dirs(fw_path, allowed_paths, 
                              ARRAY_SIZE(allowed_paths))) {
        return -EINVAL;
    }
    
    return request_firmware(&fw, fw_path, dev);
}
```

## 安全加固建议

### 1. 输入验证强化

- 所有外部输入必须进行长度、类型、范围验证
- 使用白名单而非黑名单验证
- 对媒体数据进行格式验证

### 2. 权限最小化

- 服务以最小权限运行
- 使用 capabilities 而非 root
- 启用 SELinux 强制模式

### 3. 内存安全

- 使用安全的字符串/内存操作函数
- 启用 ASLR 和堆栈保护
- 定期进行内存泄露检测

### 4. 日志审计

- 记录所有敏感操作
- 实现安全事件告警
- 定期审计日志

## 检查局限性

### 未覆盖范围

1. **上层框架** - 本仓库仅包含硬件适配层，未评估 `drivers/peripheral`、`foundation` 等上层代码
2. **N-API 接口** - 本仓库无 N-API 实现
3. **网络协议栈** - 未深入分析 WiFi/蓝牙协议实现
4. **加密实现** - 未评估加密算法实现

### 建议的进一步检查

1. 使用静态分析工具 (Coverity, CodeQL) 扫描代码
2. 进行模糊测试 (Fuzzing) 发现未知漏洞
3. 安全渗透测试验证可利用性

## 相关链接

- [项目概览](00_Overview.md) - 项目定位
- [架构说明](01_Architecture.md) - 系统架构
- [HDI/VDI 接口](03_HDI_Interfaces.md) - 接口文档

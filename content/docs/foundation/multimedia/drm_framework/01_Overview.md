# 01 项目概览

## 目的与适用范围

本文档面向 OpenHarmony DRM 框架的**新人学习者**和**安全研究员**，帮助快速理解项目定位、核心能力和使用方法。

## 一句话定义

OpenHarmony DRM 框架是多媒体子系统的**数字版权管理组件**，为音视频内容提供**证书管理、许可证申请、安全解密**能力，支持应用调用系统级 DRM 插件完成受保护内容的播放。

## 能力边界

### 能做什么

| 能力 | 说明 |
|------|------|
| DRM证书管理 | 生成证书请求、处理证书响应，完成设备Provision |
| DRM许可证管理 | 生成许可证请求、处理许可证响应，支持离线密钥管理 |
| 安全解密 | 提供媒体数据解密能力，支持软件/硬件加密 |
| 多方案支持 | 支持多种 DRM 方案（如WisePlay、ClearKey等） |
| 事件通知 | 密钥过期、密钥变更等事件回调机制 |
| SVP支持 | 安全视频通路，支持硬件安全区域解密 |

### 不能做什么

| 限制 | 说明 |
|------|------|
| 非DRM内容 | 不支持非加密内容的特殊处理 |
| 内容获取 | 不直接处理媒体内容的网络下载 |
| DRM插件实现 | 只提供框架，具体插件由厂商通过HDI实现 |
| 硬件抽象 | 依赖HDI层与硬件交互，框架本身不直接操作硬件 |
| 权限管理 | 不处理应用权限申请，依赖系统AccessToken |

## 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| 系统版本 | OpenHarmony 3.1+ |
| 子系统 | multimedia |
| 系统能力 | SystemCapability.Multimedia.Drm.Core |
| 进程权限 | `drmserver` uid |
| SAID | 3012 |

### 依赖服务

| 服务 | 组件名 | 说明 |
|------|--------|------|
| SystemAbilityManager | samgr | SA管理服务，负责服务生命周期 |
| HDI DRM Driver | drivers_interface_drm | 硬件DRM插件驱动接口 |
| NetManager | netmanager_base | 网络服务，用于证书Provision |
| AccessToken | access_token | 权限管理服务 |
| BundleManager | bundle_framework | 包管理服务，获取调用者信息 |

## 快速开始

### JS API 使用示例

```javascript
import drm from '@ohos.multimedia.drm';

// 1. 检查DRM方案支持
const isSupported = drm.isMediaKeySystemSupported('com.wiseplay.drm');
if (!isSupported) {
    console.error('DRM scheme not supported');
    return;
}

// 2. 创建MediaKeySystem实例
const keySystem = drm.createMediaKeySystem('com.wiseplay.drm');

// 3. 证书Provision（首次使用）
// 3.1 生成证书请求
const provisionRequest = keySystem.generateKeySystemRequest();
// 3.2 发送请求到Provision服务器（应用层实现）
const response = await fetchProvisionResponse(provisionRequest);
// 3.3 处理证书响应
keySystem.processKeySystemResponse(response);

// 4. 创建MediaKeySession
const keySession = keySystem.createMediaKeySession(
    drm.ContentProtectionLevel.CONTENT_PROTECTION_LEVEL_SW_CRYPTO
);

// 5. 申请许可证
// 5.1 生成许可证请求
const initData = new Uint8Array([/* PSSH box data */]);
const mediaKeyRequest = keySession.generateMediaKeyRequest(
    'video/mp4',
    initData,
    drm.MediaKeyType.MEDIA_KEY_TYPE_ONLINE
);
// 5.2 发送请求到许可证服务器
const licenseResponse = await fetchLicenseResponse(mediaKeyRequest);
// 5.3 处理许可证响应
keySession.processMediaKeyResponse(licenseResponse);

// 6. 检查是否需要安全解码器（SVP）
const needSecureDecoder = keySession.requireSecureDecoderModule('video/mp4');
console.log(`Need secure decoder: ${needSecureDecoder}`);

// 7. 注册事件监听
keySession.on('keyExpired', (event) => {
    console.log('Key expired, need to renew license');
});

// 8. 使用完成后释放资源
keySession.destroy();
keySystem.destroy();
```

### C API 使用示例

```c
#include "native_mediakeysystem.h"
#include "native_mediakeysession.h"
#include "native_drm_common.h"
#include "native_drm_err.h"
#include <stdio.h>

int main() {
    // 1. 检查DRM方案支持
    bool supported = OH_MediaKeySystem_IsSupported("com.wiseplay.drm");
    if (!supported) {
        printf("DRM scheme not supported\n");
        return -1;
    }

    // 2. 创建MediaKeySystem实例
    MediaKeySystem *system = nullptr;
    Drm_ErrCode ret = OH_MediaKeySystem_Create("com.wiseplay.drm", &system);
    if (ret != DRM_ERR_OK) {
        printf("Create MediaKeySystem failed: %d\n", ret);
        return -1;
    }

    // 3. 创建MediaKeySession
    DRM_ContentProtectionLevel level = CONTENT_PROTECTION_LEVEL_SW_CRYPTO;
    MediaKeySession *session = nullptr;
    ret = OH_MediaKeySystem_CreateMediaKeySession(system, &level, &session);
    if (ret != DRM_ERR_OK) {
        printf("Create MediaKeySession failed: %d\n", ret);
        OH_MediaKeySystem_Destroy(system);
        return -1;
    }

    // 4. 生成许可证请求
    DRM_MediaKeyRequestInfo info;
    info.type = MEDIA_KEY_TYPE_ONLINE;
    info.initDataLen = /* length */;
    memcpy(info.initData, /* pssh data */, info.initDataLen);
    strcpy(info.mimeType, "video/mp4");
    info.optionsCount = 0;

    DRM_MediaKeyRequest request;
    ret = OH_MediaKeySession_GenerateMediaKeyRequest(session, &info, &request);
    if (ret != DRM_ERR_OK) {
        printf("Generate request failed: %d\n", ret);
        OH_MediaKeySession_Destroy(session);
        OH_MediaKeySystem_Destroy(system);
        return -1;
    }

    // 5. 处理许可证响应（应用层获取后传入）
    uint8_t responseData[4096];
    int32_t responseLen = /* length */;
    // ... 填充responseData ...
    
    uint8_t offlineKeyId[64];
    int32_t offlineKeyIdLen = sizeof(offlineKeyId);
    ret = OH_MediaKeySession_ProcessMediaKeyResponse(
        session, responseData, responseLen, offlineKeyId, &offlineKeyIdLen
    );

    // 6. 查询安全解码器需求
    bool needSecure = false;
    ret = OH_MediaKeySession_RequireSecureDecoderModule(session, "video/mp4", &needSecure);

    // 7. 清理资源
    OH_MediaKeySession_Destroy(session);
    OH_MediaKeySystem_Destroy(system);

    return 0;
}
```

## 关键概念

### MediaKeySystem

**定义**: 媒体密钥系统，对应一种 DRM 方案（如Widevine、PlayReady、WisePlay）。

**生命周期**:
```
创建 → 配置 → Provision(可选) → 创建Session → 销毁
```

**限制**: 一个应用可创建多个实例，但系统有总数限制（默认最大64个）。

### MediaKeySession

**定义**: 媒体密钥会话，对应一次内容播放的授权上下文。

**生命周期**:
```
创建 → 生成请求 → 处理响应 → 解密使用 → 清理密钥 → 销毁
```

**特点**:
- 每个Session有独立的许可证和密钥
- 支持离线许可证（可持久化存储）
- 支持密钥过期和续期

### ContentProtectionLevel

**内容保护级别**（安全等级从低到高）：

| 级别 | 枚举值 | 说明 |
|------|--------|------|
| UNKNOWN | 0 | 未知级别 |
| SW_CRYPTO | 1 | 软件加密，密钥在普通内存 |
| HW_CRYPTO | 2 | 硬件加密，密钥在TEE/安全芯片 |
| ENHANCED_HW_CRYPTO | 3 | 增强硬件加密，更高安全级别 |
| MAX | 4 | 最高级别 |

### Provision vs License

| 对比项 | Provision | License |
|--------|-----------|---------|
| **目的** | 设备证书配置 | 内容解密密钥 |
| **粒度** | 设备级别 | 内容级别 |
| **频率** | 通常一次 | 每个内容 |
| **数据** | 设备证书 | 内容密钥 |
| **有效期** | 长期（可能过期） | 短期（可设置） |

**调用时序**:
```
首次使用DRM方案 → Provision → 日常使用 → License申请 → 解密播放
```

## 错误码速查

| 错误码 | 值 | 说明 |
|--------|-----|------|
| DRM_ERR_OK | 0 | 成功 |
| DRM_ERR_NO_MEMORY | 24700501 | 内存不足 |
| DRM_ERR_INVALID_VAL | 24700503 | 无效参数 |
| DRM_ERR_SERVICE_DIED | 24700507 | 服务死亡 |
| DRM_ERR_MAX_SYSTEM_NUM_REACHED | 24700510 | 系统数达上限 |
| DRM_ERR_MAX_SESSION_NUM_REACHED | 24700511 | 会话数达上限 |

## 相关链接

- [02_Architecture.md](02_Architecture.md) - 架构设计与数据流
- [03_CodeMap.md](03_CodeMap.md) - 目录结构与代码地图
- [04_Interface.md](04_Interface.md) - 完整API参考
- [05_AttackSurface.md](05_AttackSurface.md) - 攻击面分析
- [06_SecurityReview.md](06_SecurityReview.md) - 安全风险评估
- [07_Build.md](07_Build.md) - 构建与产物
- [SUMMARY.md](SUMMARY.md) - 文档导航

---

*本文档基于 OpenHarmony DRM Framework 3.1 版本*

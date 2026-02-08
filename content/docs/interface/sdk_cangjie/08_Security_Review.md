# 安全风险评审

## 概述

本文档对 Cangjie SDK 进行安全风险评审，基于代码证据识别潜在的安全风险点，并提供修复建议。

> **评审范围**: `interface/sdk_cangjie` 仓库中的 API 声明文件（`.cj.d`）
> **评审依据**: `@!APILevel` 注解中的权限声明、API 设计模式

## 威胁模型

### 外部输入

| 输入类型 | 来源 | 风险等级 |
|---------|------|----------|
| 用户输入 (Want, Parameters) | 应用层 | 高 |
| 网络数据 (HTTP Response) | NetworkKit | 高 |
| 文件数据 (File I/O) | CoreFileKit | 中 |
| 相机/麦克风 | MediaKit, CameraKit | 高 |
| 位置信息 | LocationKit | 高 |
| 传感器数据 | SensorServiceKit | 中 |

### 敏感操作

| 操作 | 系统能力 | 风险等级 |
|------|----------|----------|
| 权限校验 | AccessToken | 高 |
| 密钥管理 | Keystore | 高 |
| IPC 通信 | IPC Core | 中 |
| 网络访问 | NetManager | 中 |
| 设备标识 | DeviceInfo | 中 |

## 攻击面清单

| 攻击面 | 涉及 Kit | 说明 |
|--------|----------|------|
| API 权限声明 | AbilityKit | API 级别权限校验 |
| 敏感数据访问 | UniversalKeystoreKit | 密钥/证书管理 |
| 网络通信 | NetworkKit | HTTP/Socket |
| 文件系统 | CoreFileKit | 文件读写 |
| 进程间通信 | IPCKit | RPC/共享内存 |
| 设备能力 | BasicServicesKit | 设备信息 |
| 位置服务 | LocationKit | 地理位置 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                    Cangjie Application                       │
│                  (开发者代码，信任域)                         │
├─────────────────────────────────────────────────────────────┤
│                      Kit API                                │
│                 (权限校验点，信任边界)                         │
├─────────────────────────────────────────────────────────────┤
│                   Runtime Layer                              │
│              (互操作层，半信任域)                              │
├─────────────────────────────────────────────────────────────┤
│                   System Services                            │
│                (操作系统内核，高信任域)                        │
└─────────────────────────────────────────────────────────────┘
```

## 安全风险点

### 风险 1: 权限声明不完整

**风险等级**: 中

**证据位置**: `api/AbilityKit/ohos.ability_access_ctrl.cj.d`

**问题描述**: 部分敏感 API 缺少 `permission` 字段声明

```cangjie
// 当前声明
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Security.AccessToken"
]
public class AtManager {
    public func checkAccessToken(
        tokenID: UInt32,
        permissionName: Permissions
    ): GrantStatus
}

// 缺失: 没有在函数级别声明具体权限
```

**触发条件**: 开发者未正确请求权限即可调用

**影响**: 权限绕过风险

**修复建议**:
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.CHECK_MANAGER",
    syscap: "SystemCapability.Security.AccessToken"
]
public func checkAccessToken(...): GrantStatus
```

---

### 风险 2: 缺少输入长度校验声明

**风险等级**: 中

**证据位置**: `api/AbilityKit/ohos.bundle.bundle_manager.cj.d`

**问题描述**: Bundle 名称、路径等参数缺少最大长度声明

```cangjie
public class BundleInfo {
    public let name: String              // 缺少长度限制
    public let bundleName: String         // 缺少长度限制
    public let permissions: Array<String> // 缺少元素数量限制
}
```

**触发条件**: 恶意应用传递超长 Bundle 名称

**影响**: 缓冲区溢出、拒绝服务

**修复建议**: 在 API 文档中明确参数长度限制

---

### 风险 3: Ashmem 内存保护声明不明确

**风险等级**: 低

**证据位置**: `api/IPCKit/ohos.rpc.cj.d`

**问题描述**: `PROT_EXEC` 权限在共享内存中使用存在安全风险

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public class Ashmem {
    public static const PROT_EXEC: UInt32 = 4    // 可执行内存
    public static const PROT_READ: UInt32 = 1
    public static const PROT_WRITE: UInt32 = 2
    public static const PROT_NONE: UInt32 = 0
}
```

**触发条件**: 应用创建可执行共享内存

**影响**: 内存执行攻击（如果被恶意篡改）

**修复建议**: 移除或标记 `PROT_EXEC` 为废弃

---

### 风险 4: 网络数据缺少安全传输声明

**风险等级**: 中

**证据位置**: `api/NetworkKit/ohos.net.http.cj.d`

**问题描述**: HTTP API 未强制要求 TLS/HTTPS

```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.INTERNET"
]
public class HttpRequest {
    public func request(url: String): Promise<HttpResponse>
    // url 参数未强制要求 https://
}
```

**触发条件**: 应用使用 HTTP 而非 HTTPS

**影响**: 中间人攻击、数据泄露

**修复建议**: 提供 HTTPS 强制选项或警告机制

---

### 风险 5: 设备标识暴露风险

**风险等级**: 低

**证据位置**: `api/BasicServicesKit/ohos.device_info.cj.d`

**问题描述**: 设备标识 API 可能被滥用

```cangjie
public func getDeviceID(): String
public func getUDID(): String
```

**触发条件**: 应用获取用户设备标识用于追踪

**影响**: 用户隐私泄露

**修复建议**: 限制敏感标识的获取权限

---

### 风险 6: 位置数据精度控制缺失

**风险等级**: 中

**证据位置**: `api/LocationKit/ohos.geo_location_manager.cj.d`

**问题描述**: 位置精度未明确控制

```cangjie
public func getLocation(): Promise<GeoLocation>
```

**触发条件**: 应用获取精确位置

**影响**: 精确位置追踪、隐私泄露

**修复建议**:
```cangjie
public enum LocationPrecision {
    PRECISION_INDOOR
    PRECISION_CITY
    PRECISION_GPS
}

public func getLocation(precision: LocationPrecision): Promise<GeoLocation>
```

---

### 风险 7: 回调中的敏感数据泄露

**风险等级**: 低

**证据位置**: 多个 Kit 中的异步 API

**问题描述**: 回调可能包含敏感数据

```cangjie
public func requestPermissionsFromUser(
    context: UIAbilityContext,
    permissionList: Array<Permissions>,
    requestCallback: AsyncCallback<PermissionRequestResult>
): Unit
// PermissionRequestResult 可能包含敏感信息
```

**触发条件**: 回调被不信任的第三方处理

**影响**: 权限状态泄露

**修复建议**: 敏感结果仅返回必要信息

---

## 安全设计建议

### 1. 权限最小化

```cangjie
// 敏感 API 必须声明权限
@!APILevel[
    since: "22",
    permission: "ohos.permission.SENSITIVE_PERMISSION"
]
public func sensitiveApi(): ReturnType

// 可选权限使用 Option
public func optionalApi(
    optionalPermission: Option<String> = None
): ReturnType
```

### 2. 输入验证

```cangjie
// 参数校验声明
@!APILevel[
    since: "22",
    constraints: [
        "paramName.length <= 256",
        "arrayParam.size <= 100"
    ]
]
public func inputValidationDemo(
    paramName: String,
    arrayParam: Array<String>
): ReturnType
```

### 3. 敏感数据标记

```cangjie
// 敏感数据字段标记
public class SensitiveData {
    @Sensitive
    public let password: String

    @Sensitive
    public let token: String
}

// 返回值自动脱敏
public func getUserInfo(): UserInfo {
    // 自动脱敏敏感字段
}
```

### 4. 安全默认值

```cangjie
// 网络请求默认使用 HTTPS
@!APILevel[
    since: "22"
]
public class HttpRequestOptions {
    public let url: String
    public let secure: Bool = true  // 默认安全
    public let timeout: Int32 = 30000
}
```

## 安全检查清单

### API 声明检查

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 敏感 API 声明权限 | ⚠️ 部分完成 | 部分 API 缺少权限声明 |
| 参数长度限制 | ❌ 未声明 | 需要在文档中补充 |
| 敏感数据标记 | ❌ 未实现 | 建议添加 |
| 安全默认值 | ⚠️ 部分完成 | HTTP 缺少强制 HTTPS |
| 输入验证声明 | ❌ 未实现 | 建议添加 |

### 运行时安全

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 权限校验 | ✅ 已实现 | AtManager |
| 签名验证 | ✅ 已实现 | BundleManager |
| AccessToken | ✅ 已实现 | 系统级支持 |
| 数据加密 | ⚠️ 部分 | 需应用层配合 |

## 相关安全文档

- [OpenHarmony 安全指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/security/)
- [权限管理 API](api/AbilityKit/ohos.ability_access_ctrl.cj.d)
- [Bundle 管理 API](api/AbilityKit/ohos.bundle.bundle_manager.cj.d)

## 附录：检查范围说明

### 本次评审未覆盖范围

1. **C/C++ 实现代码**: 本仓库仅包含 API 声明，实际实现在 wrapper 仓库
2. **Runtime 安全**: Cangjie Runtime 和 ArkTS Runtime 的内部安全机制
3. **系统服务**: IPC/Binder 底层和 System Ability 的安全实现
4. **编译时安全**: 构建工具链的安全性

### 评审局限性

1. 基于静态声明分析，未运行动态测试
2. 未进行渗透测试
3. 权限模型评估基于 API 声明，未验证运行时行为

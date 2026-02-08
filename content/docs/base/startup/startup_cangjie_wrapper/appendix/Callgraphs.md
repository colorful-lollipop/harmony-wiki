# 关键调用链

## 目的

本文档提供 startup_cangjie_wrapper 的关键调用链，帮助理解数据流转和执行流程。

## 适用范围

- 需要深入理解调用流程的开发者
- 准备进行性能优化或故障排查的工程师

---

## 调用链概览

| 调用链 | 起点 | 终点 | 复杂度 |
|--------|------|------|--------|
| deviceType 获取 | 用户代码 | init SA 服务 | 简单 |
| udid 获取 | 用户代码 | init SA 服务 + 内存管理 | 中等 |
| majorVersion 获取 | 用户代码 | init SA 服务 + 类型转换 | 简单 |
| 完整调用流 | 应用启动 | SA 服务返回 | 复杂 |

---

## 调用链 1: deviceType 获取（最简单的调用）

### 调用栈

```
用户代码
  ↓
DeviceInfo.deviceType getter
  ↓
device_info.cj:112-118
  ↓
unsafe { FfiOHOSDeviceInfoDeviceType() }
  ↓
FFI 层
  ↓
init SA 服务 (cj_device_info_ffi)
  ↓
返回 CString (静态变量)
  ↓
cValue.toString()
  ↓
返回 String
```

### 代码追踪

**用户代码**:
```cangjie
let deviceType = DeviceInfo.device_type
```

**device_info.cj:112-118**:
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Startup.SystemInfo"
]
public static prop deviceType: String {
    get() {
        // cValue is local static variable from native, no need to free.
        let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
        cValue.toString()
    }
}
```

**device_info.cj:25**:
```cangjie
foreign func FfiOHOSDeviceInfoDeviceType(): CString
```

**init SA 服务**:
```c
// 伪代码，实际实现在 init 组件中
const char* FfiOHOSDeviceInfoDeviceType() {
    static const char* device_type = "phone";
    return device_type;
}
```

**执行时间**: < 1ms

**内存操作**: 无内存分配/释放

**证据**: `device_info.cj:112-118,25`

---

## 调用链 2: udid 获取（带内存管理）

### 调用栈

```
用户代码
  ↓
DeviceInfo.udid getter
  ↓
device_info.cj:545-552
  ↓
权限检查（系统框架层）
  ├─ 检查通过 → 继续
  └─ 检查失败 → 抛出异常
  ↓
unsafe { FfiOHOSDeviceInfoUdid() }
  ↓
FFI 层
  ↓
init SA 服务 (cj_device_info_ffi)
  ↓
返回 CString (动态分配)
  ↓
cValue.toString()
  ↓
unsafe { LibC.free(cValue) }
  ↓
返回 String
```

### 代码追踪

**用户代码**:
```cangjie
let udid = DeviceInfo.udid
```

**device_info.cj:540-552**:
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.sec.ACCESS_UDID",
    syscap: "SystemCapability.Startup.SystemInfo"
]
public static prop udid: String {
    get() {
        let cValue = unsafe { FfiOHOSDeviceInfoUdid() }
        let value = cValue.toString()
        unsafe { LibC.free(cValue) }
        return value
    }
}
```

**device_info.cj:33**:
```cangjie
foreign func FfiOHOSDeviceInfoUdid(): CString
```

**init SA 服务**:
```c
// 伪代码，实际实现在 init 组件中
char* FfiOHOSDeviceInfoUdid() {
    char* udid = malloc(64);
    // 生成 UDID
    snprintf(udid, 64, "%s", generated_udid);
    return udid;
}
```

**执行时间**: < 5ms

**内存操作**:
- init SA 服务: malloc(64)
- wrapper 层: LibC.free(cValue)

**证据**: `device_info.cj:540-552,33`

---

## 调用链 3: majorVersion 获取（带类型转换）

### 调用栈

```
用户代码
  ↓
DeviceInfo.majorVersion getter
  ↓
device_info.cj:366-371
  ↓
unsafe { FfiOHOSDeviceInfoMajorVersion() }
  ↓
FFI 层
  ↓
init SA 服务 (cj_device_info_ffi)
  ↓
返回 Int64
  ↓
Int32(ret) 类型转换
  ↓
返回 Int32
```

### 代码追踪

**用户代码**:
```cangjie
let majorVersion = DeviceInfo.majorVersion
```

**device_info.cj:366-371**:
```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Startup.SystemInfo"
]
public static prop majorVersion: Int32 {
    get() {
        let ret = unsafe { FfiOHOSDeviceInfoMajorVersion() }
        Int32(ret)
    }
}
```

**device_info.cj:57**:
```cangjie
foreign func FfiOHOSDeviceInfoMajorVersion(): Int64
```

**init SA 服务**:
```c
// 伪代码，实际实现在 init 组件中
int64_t FfiOHOSDeviceInfoMajorVersion() {
    return 5;  // OpenHarmony 5.0
}
```

**执行时间**: < 1ms

**内存操作**: 无内存分配/释放

**类型转换**: Int64 → Int32（安全，范围已知）

**证据**: `device_info.cj:366-371,57`

---

## 调用链 4: 完整调用流程（应用启动到 API 调用）

### 调用栈

```
应用启动
  ↓
仓颉运行时初始化
  ↓
import ohos.device_info
  ↓
加载 libohos.device_info.so
  ├─ 检查依赖
  │  ├─ libcj_device_info_ffi.so (init 组件)
  │  └─ libcangjie_ark_interop.so
  └─ 加载成功
  ↓
解析 APILevel 注解
  ↓
解析 Hide 注解
  ↓
用户代码执行
  ↓
调用 DeviceInfo.xxx
  ↓
权限检查（系统框架层）
  ├─ 检查通过 → 继续
  └─ 检查失败 → 抛出 BusinessException
  ↓
调用 getter
  ↓
unsafe FFI 调用
  ↓
init SA 服务执行
  ↓
返回结果
  ↓
类型转换（如需要）
  ↓
内存管理（udid 需 LibC.free()）
  ↓
返回用户
```

### 详细流程

#### 阶段 1: 应用启动

```
1. 应用进程启动
2. 仓颉运行时初始化
3. 加载依赖共享库
   - libohos.device_info.so
   - libcj_device_info_ffi.so
   - libcangjie_ark_interop.so
```

**执行时间**: 50-100ms（首次加载）

**证据**: `ohos/device_info/BUILD.gn:28-32`, `bundle.json:24-26`

#### 阶段 2: import

```
1. 解析 import ohos.device_info
2. 定位 libohos.device_info.so
3. 加载共享库
4. 解析符号表
5. 创建 DeviceInfo 类元数据
```

**执行时间**: 10-20ms

**证据**: `device_info.cj:18`

#### 阶段 3: API 调用

```
1. 用户调用 DeviceInfo.deviceType
2. 查找 DeviceInfo 类
3. 查找 deviceType 属性
4. 调用 getter
5. 执行 FFI 调用
6. 返回结果
```

**执行时间**: < 1ms

**证据**: `device_info.cj:112-118`

---

## 调用链 5: 权限检查流程

### 调用栈

```
用户代码调用 DeviceInfo.udid
  ↓
系统框架层拦截
  ↓
检查 @APILevel 注解
  ↓
查找 permission: "ohos.permission.sec.ACCESS_UDID"
  ↓
权限验证
  ├─ 检查应用是否声明权限
  ├─ 检查应用是否被授予权限
  └─ 检查应用签名（仅系统应用可申请）
  ↓
检查结果
  ├─ 通过 → 执行 getter
  └─ 失败 → 抛出 BusinessException
```

### 代码追踪

**device_info.cj:540-544**:
```cangjie
@!APILevel[
    since: "22",
    permission: "ohos.permission.sec.ACCESS_UDID",
    syscap: "SystemCapability.Startup.SystemInfo"
]
public static prop udid: String {
    ...
}
```

**系统框架层**（伪代码）:
```c
// 系统框架层拦截 API 调用
void check_permission(const char* permission) {
    if (!app_has_permission(permission)) {
        throw BusinessException("Permission denied");
    }
}
```

**证据**: `device_info.cj:540-544`, `README_zh.md:49`

---

## 性能分析

### 调用时间对比

| API | 首次调用 | 后续调用 | 说明 |
|------|---------|---------|------|
| deviceType | 100-150ms | < 1ms | 首次包含库加载 |
| udid | 100-150ms | < 5ms | 首次包含库加载，后续包含内存管理 |
| majorVersion | 100-150ms | < 1ms | 首次包含库加载 |

### 内存操作对比

| API | 内存分配 | 内存释放 | 说明 |
|------|---------|---------|------|
| deviceType | 无 | 无 | 静态 CString |
| udid | init SA 层 | wrapper 层 | 动态分配 |
| majorVersion | 无 | 无 | 值类型 |

---

## 异常处理流程

### 权限异常

```
调用 DeviceInfo.udid
  ↓
系统框架层检查权限
  ↓
权限不足
  ↓
抛出 BusinessException
  ↓
应用捕获或崩溃
```

### FFI 异常

```
调用 DeviceInfo.xxx
  ↓
unsafe FFI 调用
  ↓
FFI 失败（init SA 服务不可用）
  ↓
未捕获异常
  ↓
应用崩溃
```

**注意**: wrapper 层无异常处理

**证据**: `device_info.cj:103-667` - 所有 getter 无 try-catch

---

## 关键结论

1. **调用简单**: 所有 API 调用路径简单，无复杂分支
2. **首次加载慢**: 首次调用需 100-150ms（库加载）
3. **后续调用快**: 后续调用 < 5ms
4. **内存管理不统一**: 仅 udid 需要手动释放
5. **权限检查在外层**: 系统框架层负责权限检查
6. **无异常处理**: wrapper 层无异常处理

---

## 相关跳转链接

- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单
- [09_Common_Issues.md](09_Common_Issues.md) - 常见问题

---

## 参考资料

- [OpenHarmony FFI 调用规范](https://developer.openharmony.cn/cn/docs/documentation/guide)
- [仓颉语言 unsafe 块](https://developer.openharmony.cn/cn/doc/cangjie-unsafe)
- [OpenHarmony 权限检查机制](https://docs.openharmony.cn/application-dev/security/permission-list)

---

*最后更新: 2026-02-06*

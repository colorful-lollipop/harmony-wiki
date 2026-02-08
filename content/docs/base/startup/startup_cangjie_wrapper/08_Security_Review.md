# 安全风险评审

## 目的

本文档对 startup_cangjie_wrapper 进行安全风险评审，包括攻击面、信任边界、可利用点和修复建议。

## 适用范围

- 关注组件安全性的开发者
- 进行安全审计的工程师
- 负责组件维护的团队

---

## 威胁模型

### 数据流向

```
外部输入（仓颉应用）
  ↓
DeviceInfo.xxx 静态属性调用
  ↓
FFI 层（unsafe 调用）
  ↓
init SA 服务（系统层）
  ↓
返回设备信息
```

### 威胁分析

| 威胁类型 | 是否存在 | 说明 |
|---------|---------|------|
| 输入验证绕过 | 不存在 | 所有 API 为属性，无外部输入 |
| 路径遍历 | 不存在 | 无文件操作 |
| 权限绕过 | 存在 | 依赖系统框架层权限检查 |
| 内存安全风险 | 存在 | FFI 层未进行异常处理 |
| 信息泄露 | 存在 | 敏感信息（UDID、序列号）可能被非法访问 |
| 动态加载风险 | 不存在 | 无动态代码加载 |
| 竞态条件 | 不存在 | 无共享状态 |

---

## 攻击面清单

### 1. 仓颉 API 接口

**攻击面**: 32 个公开 API

**攻击者**: 恶意应用

**攻击方式**:
- 无权限调用敏感 API（udid、serial）
- 绕过权限检查

**证据**: `device_info.cj:103-667`

---

### 2. FFI 层

**攻击面**: 34 个 FFI 函数调用

**攻击者**: 恶意应用（通过 API 间接调用）

**攻击方式**:
- 触发 FFI 层内存错误
- 利用 FFI 函数的未检查行为

**证据**: `device_info.cj:22-94`

---

### 3. 权限控制

**攻击面**: `ohos.permission.sec.ACCESS_UDID` 权限

**攻击者**: 恶意应用

**攻击方式**:
- 通过虚假签名获取权限
- 利用系统权限检查漏洞

**证据**: `device_info.cj:240,542`, `README_zh.md:49`

---

### 4. 信息泄露

**攻击面**: 敏感设备信息

**攻击者**: 恶意应用、追踪者

**攻击方式**:
- 收集 UDID 进行设备指纹追踪
- 收集序列号进行设备识别

**证据**: `device_info.cj:545-552,243-249`

---

## 信任边界

### 边界 1: 应用层 → Wrapper 层

**边界类型**: 仓颉 API 调用

**信任关系**: 不信任应用

**保护措施**:
- 系统框架层权限检查
- @APILevel 注解声明权限要求

**证据**: `device_info.cj:99-103`, `240,542`

---

### 边界 2: Wrapper 层 → FFI 层

**边界类型**: unsafe FFI 调用

**信任关系**: 信任 FFI 实现（由 init 组件提供）

**保护措施**: 无（信任底层实现）

**证据**: `device_info.cj:22-94,115`

---

### 边界 3: FFI 层 → init SA 服务

**边界类型**: SA 服务调用

**信任关系**: 信任 init 组件

**保护措施**: SA 服务权限控制

**证据**: `ohos/device_info/BUILD.gn:28`

---

## 可利用点分析

### 1. 权限绕过风险

**风险等级**: 中

**证据**: `device_info.cj:540-552`

**描述**:

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

**可利用路径**:
```
1. 恶意应用声明 ohos.permission.sec.ACCESS_UDID
2. 系统权限检查（系统框架层）
   - 检查通过: 允许访问
   - 检查失败: 抛出异常
3. 如果存在系统权限检查漏洞，恶意应用可获取 UDID
```

**影响**: 设备唯一标识泄露，可用于设备指纹追踪

**修复建议**:
- 在 wrapper 层添加额外的权限检查（双验证）
- 记录敏感 API 的访问日志
- 对频繁访问进行限流

---

### 2. 信息泄露风险

**风险等级**: 中

**证据**: `device_info.cj:545-552,243-249`

**描述**: `udid` 和 `serial` 属性返回设备唯一标识符

**可利用路径**:
```
1. 恶意应用获取权限（或利用漏洞绕过权限）
2. 调用 DeviceInfo.udid 或 DeviceInfo.serial
3. 获取设备唯一标识
4. 用于用户追踪、广告投放等
```

**影响**: 用户隐私泄露，跨应用追踪

**修复建议**:
- 考虑使用随机化的标识符代替真实 UDID
- 提供用户可控的隐私设置
- 在 SDK 文档中明确说明隐私风险

---

### 3. 内存安全风险

**风险等级**: 低

**证据**: `device_info.cj:547-550`

**描述**: udid 属性手动调用 `LibC.free()`，其他属性不调用

**问题分析**:
```cangjie
// udid: 手动释放
let cValue = unsafe { FfiOHOSDeviceInfoUdid() }
let value = cValue.toString()
unsafe { LibC.free(cValue) }
return value

// 其他属性: 不释放
let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
cValue.toString()
```

**可利用路径**:
```
1. 如果 FfiOHOSDeviceInfoUdid() 返回静态指针（而非动态分配）
2. 调用 LibC.free() 会导致 double free 或非法释放
3. 可能导致崩溃或内存损坏
```

**影响**: 应用崩溃，可能被用于 DoS 攻击

**修复建议**:
- 统一内存管理策略（全部手动释放或全部不释放）
- 添加注释说明哪些 FFI 函数返回动态分配的内存
- 考虑使用 RAII 模式自动管理内存

---

### 4. 异常处理缺失风险

**风险等级**: 低

**证据**: `device_info.cj:103-667`

**描述**: 所有 getter 无异常处理

**问题分析**:
```cangjie
public static prop deviceType: String {
    get() {
        let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
        cValue.toString()  // 无 try-catch
    }
}
```

**可利用路径**:
```
1. FFI 调用失败（init SA 服务不可用）
2. 无异常处理，错误直接向上传播
3. 应用可能崩溃或行为异常
```

**影响**: 应用稳定性问题

**修复建议**:
- 添加异常处理逻辑
- 提供默认值或错误信息
- 记录异常日志

---

### 5. 权限声明不一致风险

**风险等级**: 低

**证据**: `device_info.cj:240,542`, `mock/ohos.device_info.cj:71,226`

**描述**: udid 和 serial 属性标注了 `permission: "ohos.permission.sec.ACCESS_UDID"`，但 wrapper 层无实际权限检查

**问题分析**:
- 权限检查在系统框架层
- wrapper 层仅通过注解声明
- 如果系统框架层有漏洞，权限检查可能失效

**影响**: 权限绕过

**修复建议**:
- 在 wrapper 层添加双验证
- 记录权限检查结果

---

## 检查范围

### 已检查

- [x] 输入验证（无外部输入）
- [x] 权限控制（udid、serial）
- [x] 内存管理（udid 的 LibC.free）
- [x] 异常处理（无）
- [x] 信息泄露（敏感设备信息）
- [x] 动态加载（无）
- [x] 竞态条件（无共享状态）
- [x] 路径遍历（无文件操作）

### 局限性

- **未检查**: init 组件的 FFI 实现细节（外部依赖）
- **未检查**: 系统框架层的权限检查实现（外部组件）
- **未检查**: 仓颉运行时的安全机制（外部组件）
- **未验证**: 实际编译输出的二进制文件

---

## 修复建议优先级

### 高优先级

1. **统一内存管理策略**
   - 统一处理 FFI 返回的 CString
   - 明确注释哪些需要手动释放

### 中优先级

2. **添加异常处理**
   - 捕获 FFI 调用异常
   - 提供友好的错误信息

3. **加强权限控制**
   - wrapper 层添加双验证
   - 记录敏感 API 访问日志

### 低优先级

4. **隐私保护**
   - 提供隐私设置
   - 考虑使用随机化标识符

---

## 关键结论

1. **权限依赖系统**: 权限检查依赖系统框架层，wrapper 层无验证
2. **敏感信息暴露**: udid 和 serial 可能被非法访问
3. **内存管理不统一**: 仅 udid 手动释放内存
4. **无异常处理**: FFI 调用失败时无保护
5. **无输入验证**: 本身无外部输入，无需验证
6. **无文件操作**: 无路径遍历风险
7. **无共享状态**: 无竞态条件
8. **依赖外部安全**: 依赖 init 组件和系统框架层的安全机制

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单

---

## 参考资料

- [OpenHarmony 安全指南](https://docs.openharmony.cn/application-dev/security)
- [OpenHarmony 权限管理](https://docs.openharmony.cn/application-dev/security/permission-list)
- [仓颉语言安全特性](https://developer.openharmony.cn/cn/doc/cangjie-security)
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/)

---

*最后更新: 2026-02-06*

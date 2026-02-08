# 常见问题

> global_cangjie_wrapper 构建、运行、调试问题与定位路径

## 构建问题

### Q1: 编译报错 "cannot find module 'cangjie_ark_interop'"

**问题描述**:
```
error: cannot find module 'cangjie_ark_interop:ohos.ffi'
```

**原因**: 依赖组件未构建或路径配置错误

**解决方案**:
```bash
# 1. 先构建 cangjie_ark_interop 子系统
hb build -p arkcompiler_cangjie_ark_interop

# 2. 确保全局依赖完整
hb set
hb build
```

**证据位置**: `ohos/i18n/BUILD.gn:30-35` - `cj_external_deps` 依赖声明

---

### Q2: 编译报错 "i18n:cj_i18n_ffi not found"

**问题描述**:
```
error: dependency 'i18n:cj_i18n_ffi' not found
```

**原因**: `global_i18n` 子系统未构建

**解决方案**:
```bash
# 构建 global_i18n 子系统
hb build -p global_i18n
```

**证据位置**: `ohos/i18n/BUILD.gn:37` - `external_deps` 依赖 `i18n:cj_i18n_ffi`

---

### Q3: Windows/macOS 构建使用 mock 文件而非源码

**问题描述**: 在 Windows/macOS 上构建时，实际使用的是 mock 实现而非真实代码

**原因**: BUILD.gn 中的平台条件编译

**证据位置**: `ohos/i18n/BUILD.gn:20-28`

```gn
if (is_mingw || is_mac) {
    sources = ["../../mock/ohos.i18n.cj"]  # 使用 mock
} else {
    sources = ["calendar.cj", "i18n_common.cj", "system.cj"]  # 真实源码
}
```

**说明**: 这是预期行为，用于支持跨平台开发。真实功能测试需在 Linux 目标设备上进行。

---

### Q4: 编译产物大小与预期不符

**问题描述**: 编译出的 `.so` 文件比预期大/小

**解决方案**:
1. 检查完整构建而非增量构建
2. 验证依赖是否完整链接
3. 检查是否有未使用的源码被包含

**定位命令**:
```bash
# 查看产物大小
ls -lh out/.../libohos.i18n.so

# 检查链接依赖
readelf -d out/.../libohos.i18n.so

# 检查符号表
nm -C out/.../libohos.i18n.so | head -100
```

---

## 运行问题

### Q5: 调用 API 返回错误码 9001001 (Invalid resource ID)

**问题描述**: `getString(resId)` 返回错误

**可能原因**:
1. 资源 ID 不存在于当前应用的资源表中
2. 资源 ID 格式错误
3. 跨应用访问未授权资源

**解决方案**:
```cangjie
let rm = ResourceManager.getResourceManager("com.example.app", "entry")
try {
    let str = rm.getString(0x1000000)  // 确保资源 ID 正确
} catch (e: BusinessException) {
    console.log("Error:", e.message)
    // 检查 e.code 是否为 9001001
}
```

**证据位置**: `resource_manager_errors.cj:28` - 错误码定义

---

### Q6: getRawFd 获取的文件描述符无法读取

**问题描述**: `getRawFd()` 返回的描述符无法 `read()` 或 `lseek()`

**可能原因**:
1. 原始文件不存在
2. 权限问题
3. 文件描述符已过期

**解决方案**:
```cangjie
let rm = ResourceManager.getResourceManager("com.example.app", "entry")
let fd = rm.getRawFd("test.raw")
if (fd.fd >= 0) {
    // 使用文件描述符
    rm.closeRawFd(fd)  // 使用后必须关闭
} else {
    console.log("Failed to open raw file")
}
```

**证据位置**: `resource_manager.cj:78-100` - `getRawFd` 实现

---

### Q7: Calendar 操作无效果或崩溃

**问题描述**: 调用 `Calendar.setTime()` 或其他方法时崩溃

**可能原因**:
1. Calendar 实例已失效（原生句柄已释放）
2. FFI 调用参数类型不匹配
3. 原生日历服务异常

**解决方案**:
```cangjie
// 1. 检查 Calendar 实例有效性
let cal = Calendar.getCalendar("zh-CN", CalendarType.Gregory)
if (cal != null) {
    cal.setTime(Date.now())
} else {
    console.log("Failed to create calendar")
}

// 2. 检查错误日志
hilog -v D -T CJ-I18n
```

**证据位置**: `calendar.cj:99-340` - Calendar 类实现

---

### Q8: 资源对象不支持跨线程传递

**问题描述**: 将 `AppResource` 或 `RawFileDescriptor` 传给 Worker 线程报错

**原因**: 资源对象设计为单线程使用

**证据位置**: `README.md:83-84` - "Cross-thread transmission of resource objects."

**解决方案**:
- 在 Worker 中重新获取资源
- 使用消息传递而非对象共享

---

### Q9: getString 格式化参数不匹配

**问题描述**: `getString(resId, args)` 格式化结果异常

**可能原因**:
1. 格式符数量与 args 数量不匹配
2. args 类型与格式符类型不匹配

**解决方案**:
```cangjie
// 检查资源定义中的格式符
// strings.xml: "Hello %s, you have %d messages"

// 传入正确参数
let args: Array<ArgsValueType> = [
    ArgsValueType("John"),  // 对应 %s
    ArgsValueType(5)        // 对应 %d
]
let str = rm.getString(resId, args)
```

**证据位置**: `resource_manager_common.cj:496-540` - `formatString` 实现

---

## 调试问题

### Q10: 如何调试 FFI 调用

**解决方案**:

1. **启用 FFI 日志**:
```cangjie
let RES_LOG = HilogChannel(0, 0xD001E00, "CJ-ResourceManager")
RES_LOG.info("Calling CJ_GetString with resId: %{public}d", resId)
```

2. **使用 hilog 查看日志**:
```bash
hilog -v D -T "CJ-*"  # 过滤 CJ 相关日志
hilog -v D -T "CJ-I18n"  # I18n 模块日志
hilog -v D -T "CJ-ResourceManager"  # ResourceManager 日志
```

**证据位置**: `resource_manager_common.cj:25` - 日志通道定义

---

### Q11: 如何追踪资源加载路径

**解决方案**:
```bash
# 启用资源管理调试
hilog -v D | grep -i resmgr

# 查看资源查找过程
hdc shell
hdc:0> param set debug.resource.loading 1
```

---

### Q12: 如何验证 API Level 注解是否生效

**问题描述**: API 返回错误，怀疑 API Level 限制

**解决方案**:
```cangjie
// 检查 API 注解
@!APILevel([
    since: "22",
    syscap: "SystemCapability.Global.I18n"
])
public class Calendar { ... }

// 验证系统版本
console.log("API Level:", System.apiVersion)
```

---

### Q13: 定位 Native 服务崩溃

**问题描述**: FFI 调用后 native 服务崩溃

**解决方案**:
1. 检查崩溃日志
```bash
hilog | grep -i "libohos.*so"
hdc shell
hdc:0> hidump -b
```

2. 使用 addr2line 定位
```bash
arm-linux-ohos/addr2line -e libohos.i18n.so 0x偏移地址
```

---

### Q14: 性能问题排查

**解决方案**:
1. **测量 API 调用时间**:
```cangjie
let start = Date.now()
let str = rm.getString(resId)
let end = Date.now()
console.log("getString took:", end - start, "ms")
```

2. **检查是否存在不必要的缓存**:
```bash
# 查看内存占用
hdc shell
hdc:0> cat /proc/meminfo | grep -i slab
```

---

## 问题定位路径

### 日志查看

| 模块 | 日志标签 | 命令 |
|------|----------|------|
| i18n | `CJ-I18n` | `hilog -v D -T CJ-I18n` |
| ResourceManager | `CJ-ResourceManager` | `hilog -v D -T CJ-ResourceManager` |
| 全部 | `CJ-*` | `hilog -v D -T "CJ-*"` |

### 常用调试命令

```bash
# 查看模块加载
hdc shell
hdc:0> cat /proc/modules | grep ohos

# 查看共享库依赖
readelf -d libohos.i18n.so

# 查看系统能力
param get | grep cap

# 查看资源路径
hdc shell
hdc:0> ls /data/app/.../resources/
```

## 相关文档

- [API 参考](./03_NAPI_Reference.md)
- [安全评审](./07_Security_Review.md)
- [内部 API](./04_Inner_API.md)

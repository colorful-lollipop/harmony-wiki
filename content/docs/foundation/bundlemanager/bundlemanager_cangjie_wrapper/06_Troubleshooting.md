# 问题排查

> bundlemanager_cangjie_wrapper 常见构建/运行/调试问题与定位路径

## 构建问题

### 问题 1: 编译报错找不到依赖

**错误信息**:
```
error: cannot find module 'ohos.xxx'
```

**可能原因**:
- 依赖组件未正确编译
- `cj_external_deps` 配置缺失
- `external_deps` 配置缺失

**排查路径**:
1. 检查 `bundle.json` 中 `deps.components` 是否包含所有依赖
2. 检查对应模块 `BUILD.gn` 中的 `cj_external_deps` 和 `external_deps`
3. 确认依赖组件已编译

**证据来源**:
- `bundle.json:21-27` - 依赖组件列表
- `ohos/bundle/bundle_manager/BUILD.gn:40-56` - 依赖配置

---

### 问题 2: Windows/Mac 平台编译失败

**错误信息**:
```
error: undefined reference to 'FfiOHOS*'
```

**可能原因**:
- Windows/Mac 平台缺少原生依赖
- Mock 文件未正确配置

**解决方案**:
```gn
# BUILD.gn 中已配置条件编译
if (is_mingw || is_mac){
    sources = ["../../mock/ohos.bundle.cj"]
} else {
    sources = ["bundle.cj"]
}
```

**证据来源**:
- `ohos/bundle/BUILD.gn:21-25` - 条件编译配置

---

### 问题 3: C 字符串转换失败

**错误信息**:
```
error: cannot convert String to CString
```

**可能原因**:
- 字符串包含非法字符
- 内存分配失败

**排查路径**:
1. 检查 `LibC.mallocCString()` 调用是否在 `unsafe` 块中
2. 检查字符串是否为 null 或空

**正确示例** (`bundle_manager.cj:119-125`):
```cj
unsafe {
    var res: Array<String> = []
    try (
        cModuleName = LibC.mallocCString(moduleName).asResource(),
        cAbilityName = LibC.mallocCString(abilityName).asResource()
    ) {
        // ...
    }
}
```

---

## 运行时问题

### 问题 4: BusinessException 抛出

**常见错误码**:

| 错误码 | 常量名 | 排查方向 |
|--------|--------|----------|
| 17700002 | ERROR_MODULE_NOT_EXIST | 检查 moduleName 是否正确 |
| 17700003 | ERROR_ABILITY_NOT_EXIST | 检查 abilityName 是否正确 |
| 17700024 | ERROR_PROFILE_NOT_EXIST | 检查 HAP 中是否存在 profile |
| 17700029 | ERROR_ABILITY_IS_DISABLED | 检查 Ability 是否已启用 |
| 17700055 | ERROR_INVALID_LINK | 检查链接格式是否正确 |
| 17700056 | ERROR_SCHEME_NOT_IN_QUERYSCHEMES | 检查 scheme 是否已注册 |

**处理示例** (`bundle_manager.cj:128-129`):
```cj
if (cValue.code != 0) {
    throw BusinessException(cValue.code, getErrorMsg(cValue.code))
}
```

---

### 问题 5: 缓存未命中

**现象**:
每次调用 `getBundleInfoForSelf` 都返回新对象

**可能原因**:
- 不同用户调用（缓存按用户隔离）
- BundleInfo 发生变化

**缓存机制** (`bundle_manager.cj:48-58`):
```cj
func checkToCache(uid: Int32, callingUid: Int32, query: BMQuery, bundleInfo: BundleInfo): Unit {
    if (uid != callingUid) {
        return  // 跨用户不缓存
    }
    synchronized(MTX) {
        cache[query] = bundleInfo
    }
}
```

---

### 问题 6: 内存泄漏

**现象**:
长时间运行后内存占用持续增长

**可能原因**:
- FFI 返回的 C 结构体未释放
- 数组资源未释放

**排查路径**:
1. 检查每个 FFI 调用后是否调用 `free()`
2. 检查数组释放函数是否正确调用

**正确示例** (`bundle_manager.cj:91-93`):
```cj
let ret = unsafe { FfiOHOSGetBundleInfoForSelfV2(bundleFlags) }
defer { unsafe { ret.free() } }  // 确保释放
let info = BundleInfo(ret)
```

---

## 调试方法

### 日志打印

使用 `Hilog` 模块打印日志:

```cj
import ohos.hilog.Hilog

// 打印调试信息
Hilog.debug(0, "BundleManager", "Debug message: ${info.name}")

// 打印错误信息
Hilog.error(0, "BundleManager", "Error: ${errorCode}")
```

---

### 错误码查询

使用 `getErrorMsg()` 获取错误码对应信息:

```cj
let errorMsg = getErrorMsg(cValue.code)
Hilog.error(0, "BundleManager", "Error ${cValue.code}: ${errorMsg}")
```

**证据来源**:
- `error_message.cj` - 错误码对应消息

---

### 断点调试

1. **设置断点**: 在 `.cj` 文件中需要调试的行设置断点
2. **启动调试**: 使用 IDE 的调试功能
3. **查看变量**: 检查 `bundleFlags`, `moduleName`, `link` 等参数值
4. **单步执行**: 跟踪 FFI 调用流程

---

## 性能问题

### 问题 7: API 响应慢

**可能原因**:
- 网络延迟（IPC 调用）
- BundleInfo 数据量大
- 未命中缓存

**优化建议**:
1. 使用 `getBundleInfoForSelf` 时尽量复用 `bundleFlags`
2. 对性能敏感场景考虑本地缓存
3. 使用 `workerthread: true` 的 API 在后台线程执行

---

## 兼容性问题

### 问题 8: API Level 不兼容

**错误信息**:
```
API not available before API Level 22
```

**解决方案**:
```cj
// 检查 API Level
if (currentApiLevel < 22) {
    // 使用兼容方案或提示用户
}
```

---

## 相关文档

- [01_Overview.md](../01_Overview.md) - 项目概览
- [02_API_Reference.md](../02_API_Reference.md) - API 参考
- [03_Architecture.md](../03_Architecture.md) - 架构说明
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - 配置开关
- [SUMMARY.md](../SUMMARY.md) - 文档导航

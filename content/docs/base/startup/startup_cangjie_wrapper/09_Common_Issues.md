# 常见问题与定位路径

## 目的

本文档列出了 startup_cangjie_wrapper 的常见问题、原因分析和定位路径。

## 适用范围

- 遇到开发问题的开发者
- 进行故障排查的工程师

---

## 构建问题

### 问题 1: 编译失败 - 找不到 init:cj_device_info_ffi

**错误信息**:
```
ERROR at //base/startup/startup_cangjie_wrapper/ohos/device_info/BUILD.gn:28:11: Unable to resolve "init:cj_device_info_ffi"
```

**原因分析**:
- init 组件未编译
- init 组件版本不兼容
- GN 构建配置问题

**定位路径**:
1. 检查 init 组件是否已编译:
   ```bash
   ls out/产品名/.../init/cj_device_info_ffi
   ```

2. 检查 init 组件版本:
   ```bash
   cat base/startup/init/bundle.json | grep version
   ```

3. 检查构建依赖:
   ```bash
   ./build.sh --product-name=产品名 --build-target=init
   ./build.sh --product-name=产品名 --build-target=startup_cangjie_wrapper
   ```

**证据**: `ohos/device_info/BUILD.gn:28`

---

### 问题 2: 编译失败 - 找不到 cangjie_ark_interop:ohos.labels

**错误信息**:
```
ERROR at //base/startup/startup_cangjie_wrapper/ohos/device_info/BUILD.gn:31:11: Unable to resolve "cangjie_ark_interop:ohos.labels"
```

**原因分析**:
- cangjie_ark_interop 组件未编译
- cangjie_ark_interop 版本不兼容
- 仓颉编译器未正确安装

**定位路径**:
1. 检查 cangjie_ark_interop 组件:
   ```bash
   ls out/产品名/.../cangjie_ark_interop/ohos.labels
   ```

2. 检查仓颉编译器:
   ```bash
   cjc --version
   ```

**证据**: `ohos/device_info/BUILD.gn:30-32`

---

### 问题 3: Windows/Mac 环境编译失败

**错误信息**:
```
error: undefined reference to 'FfiOHOSDeviceInfoDeviceType'
```

**原因分析**:
- 条件编译未正确选择 mock 实现
- mock 实现缺少 FFI 函数声明

**定位路径**:
1. 检查 BUILD.gn 条件:
   ```bash
   grep -n "is_mingw\|is_mac" ohos/device_info/BUILD.gn
   ```

2. 检查 mock 实现:
   ```bash
   grep -n "foreign func" mock/ohos.device_info.cj
   ```

3. 确认平台变量:
   ```bash
   echo $is_mingw $is_mac
   ```

**证据**: `ohos/device_info/BUILD.gn:22-26`

---

## 运行时问题

### 问题 4: 应用崩溃 - 调用 DeviceInfo.udid 时崩溃

**错误信息**:
```
FATAL EXCEPTION: main
Process: com.example.app, PID: 1234
java.lang.UnsatisfiedLinkError: ...
```

**原因分析**:
- init SA 服务不可用
- FFI 函数调用失败
- 内存管理问题（double free）

**定位路径**:
1. 检查 init SA 服务:
   ```bash
   ps -A | grep init
   ```

2. 检查共享库加载:
   ```bash
   cat /proc/<pid>/maps | grep device_info
   ```

3. 检查日志:
   ```bash
   hilog | grep device_info
   ```

**证据**: `device_info.cj:545-552`

---

### 问题 5: 获取 UDID 失败 - 返回空字符串

**错误现象**:
```cangjie
let udid = DeviceInfo.udid
println(udid)  // 输出空字符串
```

**原因分析**:
- 权限未授予
- init SA 服务返回空字符串
- 权限检查失败

**定位路径**:
1. 检查权限声明:
   ```bash
   grep -n "ohos.permission.sec.ACCESS_UDID" entry/src/main/module.json5
   ```

2. 检查权限授予:
   - 系统设置 → 应用 → 权限

3. 检查日志:
   ```bash
   hilog | grep ACCESS_UDID
   ```

**证据**: `README_zh.md:49`, `device_info.cj:540-544`

---

### 问题 6: 调用 DeviceInfo.xxx 抛出异常

**错误信息**:
```
BusinessException: Permission denied
```

**原因分析**:
- 权限检查失败
- SysCap 不可用

**定位路径**:
1. 检查权限:
   ```bash
   # 同问题 5
   ```

2. 检查 SysCap:
   ```bash
   # 检查设备是否支持 SystemCapability.Startup.SystemInfo
   ```

**证据**: `device_info.cj:99-103`

---

## API 问题

### 问题 7: 调用隐藏 API 失败 - 找不到符号

**错误信息**:
```
error: undefined reference to 'DeviceInfo.productModelAlias'
```

**原因分析**:
- 隐藏 API 标记为 @Hide，不对外暴露

**定位路径**:
1. 检查注解:
   ```bash
   grep -A 5 "productModelAlias" ohos/device_info/device_info.cj
   ```

2. 查看文档说明:
   - `README_zh.md:51-55` - 暂不支持

**证据**: `device_info.cj:198`, `README_zh.md:51-55`

---

### 问题 8: 返回值类型不匹配

**错误信息**:
```
error: type mismatch, expected Int32 but found String
```

**原因分析**:
- 混淆了 String 类型和 Int32 类型的 API

**定位路径**:
1. 查阅 API 清单:
   - `wiki/04_Cangjie_API.md`

2. 检查 API 定义:
   ```bash
   grep -A 2 "public static prop" ohos/device_info/device_info.cj
   ```

**证据**: `device_info.cj:103-667`

---

## 性能问题

### 问题 9: 首次调用 DeviceInfo.xxx 卡顿

**错误现象**:
- 首次调用需要 100ms+
- 后续调用正常

**原因分析**:
- 共享库首次加载
- FFI 函数首次调用

**定位路径**:
1. 检查日志:
   ```bash
   hilog | grep "loading\|FFI"
   ```

2. 检查库加载时间:
   ```bash
   time -p <测试应用>
   ```

**建议**: 在应用启动时预热（提前调用一次）

---

## 开发环境问题

### 问题 10: mock 实现始终返回空字符串或 0

**错误现象**:
```cangjie
let deviceType = DeviceInfo.deviceType
println(deviceType)  // 输出空字符串
```

**原因分析**:
- Windows/Mac 环境使用 mock 实现
- mock 实现返回默认值

**定位路径**:
1. 确认当前平台:
   ```bash
   uname -a
   ```

2. 确认使用的是真实实现还是 mock:
   ```bash
   # Windows/Mac 环境会自动使用 mock
   ```

3. 如需真实数据，在真实设备上运行

**证据**: `mock/ohos.device_info.cj:29-44`, `ohos/device_info/BUILD.gn:22-26`

---

## 调试技巧

### 1. 启用详细日志

```bash
# 设置日志级别
hdc shell hdc shell hilog -b D *:I

# 过滤设备信息相关日志
hilog | grep device_info
```

### 2. 检查共享库

```bash
# 查看库符号
readelf -s libohos.device_info.so

# 查看库依赖
ldd libohos.device_info.so

# 查看库是否加载
cat /proc/<pid>/maps | grep device_info
```

### 3. FFI 调用跟踪

在 `device_info.cj` 中添加日志:

```cangjie
public static prop deviceType: String {
    get() {
        println("[DEBUG] Calling FfiOHOSDeviceInfoDeviceType()")
        let cValue = unsafe { FfiOHOSDeviceInfoDeviceType() }
        let result = cValue.toString()
        println("[DEBUG] Result: ${result}")
        return result
    }
}
```

### 4. 权限检查

```bash
# 检查应用权限
hdc shell bm dump -n <应用包名> | grep permission

# 检查运行时权限
hdc shell permission list --uid <uid>
```

---

## 日志位置

### OpenHarmony 日志

| 日志类型 | 命令 | 说明 |
|---------|------|------|
| 系统日志 | `hilog` | 系统和应用日志 |
| 应用日志 | `hilog | grep <应用包名>` | 应用特定日志 |
| 崩溃日志 | `/data/log/faultlog/` | 应用崩溃日志 |

### 日志过滤

```bash
# 过滤设备信息相关日志
hilog | grep -E "device_info|DeviceInfo|FfiOHOS"

# 过滤错误日志
hilog | grep -i error

# 过滤权限相关日志
hilog | grep -i permission
```

---

## 相关文档

### Wiki 文档

- [00_Overview.md](00_Overview.md) - 项目概览
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [04_Cangjie_API.md](04_Cangjie_API.md) - API 清单
- [07_Build_Artifacts.md](07_Build_Artifacts.md) - 编译产物

### 官方文档

- [OpenHarmony 故障排查指南](https://docs.openharmony.cn/application-dev/faq)
- [仓颉语言调试指南](https://developer.openharmony.cn/cn/doc/cangjie-debugging)
- [OpenHarmony 日志系统](https://docs.openharmony.cn/application-dev/execution/log)

---

## 关键结论

1. **构建问题**: 大部分为依赖组件未编译或版本不兼容
2. **运行时问题**: 主要是权限问题和 SA 服务不可用
3. **性能问题**: 首次调用卡顿为正常现象（库加载）
4. **mock 环境**: Windows/Mac 环境使用 mock，返回默认值
5. **调试工具**: hilog、readelf、ldd 是主要调试工具
6. **日志定位**: 过滤日志是快速定位问题的关键

---

*最后更新: 2026-02-06*

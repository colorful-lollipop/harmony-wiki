# 问题排查

**文档目的**: 提供常见问题诊断方法和调试技巧  
**目标读者**: 开发者、测试工程师  
**阅读时间**: 约 10 分钟

---

## 1. 常见问题速查

### 1.1 初始化问题

#### Q: UiTest 初始化失败，提示 "uitest setup failed"

**现象**: 调用 `Driver.create()` 或 UITest 初始化时抛出异常

**原因**:
1. 测试模式未启用
2. uitest 守护进程启动失败
3. 权限不足

**诊断步骤**:

```bash
# 1. 检查测试模式是否启用
hdc shell param get persist.ace.testmode.enabled
# 应返回: 1

# 2. 检查 uitest 进程是否在运行
hdc shell ps -ef | grep uitest

# 3. 手动启动 uitest 守护进程查看错误
hdc shell uitest start-daemon test@1234@1000@1
```

**解决方案**:
```bash
# 启用测试模式
hdc shell param set persist.ace.testmode.enabled 1

# 重启应用后重试
```

**证据**: `ui_test_api.cj:55-71`

---

#### Q: 提示 "systemParameter is not set" 警告

**现象**: 日志中出现警告但不影响执行

```
UiTestKit_exporter: systemParameter "persist.ace.testmode.enabled" is not set!
```

**原因**: 测试模式未启用

**解决方案**:
```bash
hdc shell param set persist.ace.testmode.enabled 1
```

---

### 1.2 组件查找问题

#### Q: `findComponent()` 总是返回 `None`

**现象**: 无法找到已存在的组件

**原因与排查**:

1. **选择器条件太严格**
   ```cangjie
   // 错误示例：多个条件必须同时满足
   let on = On().text("确定").id("btn_ok").enabled(true)
   
   // 建议：从简单条件开始
   let on = On().text("确定")
   ```

2. **组件尚未加载**
   ```cangjie
   // 使用 waitForComponent 等待组件出现
   let component = driver.waitForComponent(On().text("加载完成"), 5000)
   ```

3. **文本不匹配**
   ```cangjie
   // 检查是否需要使用 Contains 模式
   let on = On().text("确定", pattern: MatchPattern.Contains)
   ```

**调试方法**:
```cangjie
// 1. 先查找所有组件查看实际文本
let components = driver.findComponents(On().onType("Button"))
if (let Some(arr) <- components) {
    for (c in arr) {
        TEST_LOG.info("Button text: ${c.getText()}")
    }
}

// 2. 使用 assertComponentExist 验证
// 如果不存在会抛出异常，可捕获异常查看
try {
    driver.assertComponentExist(On().text("目标文本"))
} catch (e: BusinessException) {
    TEST_LOG.error("Component not found: ${e.message}")
}
```

---

### 1.3 操作执行问题

#### Q: 点击操作没有响应

**现象**: `click()` 调用成功但 UI 无变化

**可能原因**:

1. **操作的是不可点击组件**
   ```cangjie
   // 检查组件是否可点击
   if (c.isClickable()) {
       c.click()
   } else {
       // 尝试通过 Driver 直接点击坐标
       let bounds = c.getBounds()
       let center = c.getBoundsCenter()
       driver.click(center.x, center.y)
   }
   ```

2. **UI 还未稳定**
   ```cangjie
   // 等待 UI 空闲
   driver.waitForIdle(1000, 5000)
   ```

3. **操作被系统拦截**
   - 检查是否有弹窗遮挡
   - 检查是否在其他应用上

---

#### Q: 截图失败或保存路径错误

**现象**: `screenCap()` 返回 `false`

**原因与解决方案**:

1. **路径不在沙箱内**
   ```cangjie
   // 错误：使用系统路径
   driver.screenCap("/data/screenshot.png")
   
   // 正确：使用应用沙箱路径
   let context = AbilityDelegatorRegistry.getAbilityDelegator().getAppContext()
   let filesDir = context.filesDir
   driver.screenCap("${filesDir}/screenshot.png")
   ```

2. **路径不存在**
   ```cangjie
   // 确保目录存在
   import std.fs.File
   File.createDirectory("${filesDir}/screenshots")
   driver.screenCap("${filesDir}/screenshots/test.png")
   ```

---

### 1.4 权限问题

#### Q: 系统参数访问失败，错误码 14700103

**现象**: 
```
BusinessException: Systemparameter get failed: System permission operation permission denied.
```

**原因**: 应用没有系统权限

**解决方案**:
1. 在 `module.json5` 中申请系统权限
2. 使用系统签名

**证据**: `systemparameter.cj:33`, `systemparameter.cj:66-68`

---

### 1.5 构建问题

#### Q: 编译失败，提示找不到 arkxtest

**现象**:
```
ERROR: Dependency "arkxtest:cj_ui_test_ffi" not found
```

**原因**: 缺少依赖子系统

**解决方案**:
1. 确保代码仓包含 `testfwk_arkxtest`
2. 检查 `bundle.json` 依赖配置

**证据**: `ohos/ui_test/BUILD.gn:45`, `bundle.json:25`

---

## 2. 错误码速查表

### 2.1 UiTest 错误码

| 错误码 | 含义 | 触发场景 | 解决方案 |
|--------|------|----------|----------|
| 17000001 | Initialization failed | Driver.create() 失败 | 检查测试模式、uitest 进程 |
| 17000003 | Assertion failed | assertComponentExist 失败 | 确认组件存在 |
| 17000004 | obj create return null reference | 对象引用无效 | 检查 findComponent 是否成功 |

**证据**: `const.cj:20`, `ui_test_api.cj:97-101`, `ui_test_api.cj:133-134`

### 2.2 系统参数错误码

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| 14700101 | System parameter can not be found | 检查参数名是否正确 |
| 14700102 | System parameter value is invalid | 检查参数值格式 |
| 14700103 | System permission operation permission denied | 申请系统权限 |
| 14700104 | System internal error (OOM, deadlock) | 重试或检查系统状态 |

**证据**: `systemparameter.cj:29-36`

### 2.3 FFI/通用错误码

| 错误码 | 含义 | 触发场景 |
|--------|------|----------|
| 14700104 | System internal error | ApiCallParams 构造失败（OOM） |

**证据**: `ui_test_ffi.cj:53`, `ui_test_ffi.cj:62`

---

## 3. 调试方法

### 3.1 日志调试

#### 启用详细日志

```cangjie
import ohos.hilog.HilogChannel

let DEBUG_LOG = HilogChannel(3, 0xD003100, "CJ-UITEST-DEBUG")

// 在关键点添加日志
DEBUG_LOG.info("Finding component with text: ${text}")
let component = driver.findComponent(On().text(text))
if (component.isNone()) {
    DEBUG_LOG.warn("Component not found: ${text}")
} else {
    DEBUG_LOG.info("Component found successfully")
}
```

#### 查看设备日志

```bash
# 查看 UiTest 相关日志
hdc shell hilog | grep CJ-UITEST

# 查看 uitest 守护进程日志
hdc shell hilog | grep uitest
```

### 3.2 断点调试

在关键 API 调用处添加断点：

```cangjie
// 在 ui_test_api.cj 中关键位置断点
func getData(params: ApiCallParams, api: String): String {
    // 断点 1：检查 API 名称
    let ret = unsafe { CJ_ApiCall(params) }
    // 断点 2：检查返回码
    params.free()
    // 断点 3：检查结果数据
    if (ret.code != 0) {
        throw BusinessException(ret.code, "${api} failed: ${dataOrMessage}")
    }
    return dataOrMessage
}
```

### 3.3 参数检查

```cangjie
// 在调用前检查参数
func safeClick(driver: Driver, on: On): Unit {
    // 1. 检查 Driver
    if (driver == None) {
        TEST_LOG.error("Driver is null")
        return
    }
    
    // 2. 查找组件
    let component = driver.findComponent(on)
    if (let Some(c) <- component) {
        // 3. 检查是否可点击
        if (c.isClickable()) {
            c.click()
        } else {
            TEST_LOG.warn("Component is not clickable, trying coordinate click")
            let center = c.getBoundsCenter()
            driver.click(center.x, center.y)
        }
    } else {
        TEST_LOG.error("Component not found: ${on}")
    }
}
```

---

## 4. 性能优化

### 4.1 减少查找次数

```cangjie
// 低效：多次查找
for (i in 0..10) {
    driver.findComponent(On().text("Item ${i}"))?.click()
}

// 高效：一次查找多个
let components = driver.findComponents(On().onType("ListItem"))
if (let Some(arr) <- components) {
    for (c in arr) {
        c.click()
    }
}
```

### 4.2 使用等待而非延迟

```cangjie
// 低效：固定延迟
driver.delayMs(5000)  // 可能等待过久
let c = driver.findComponent(On().text("Loaded"))

// 高效：条件等待
let c = driver.waitForComponent(On().text("Loaded"), 5000)
```

### 4.3 批量操作

```cangjie
// 获取一次，多次操作
let btn = driver.findComponent(On().text("按钮"))
if (let Some(b) <- btn) {
    // 执行多个操作
    b.click()
    let text = b.getText()
    let bounds = b.getBounds()
}
```

---

## 5. 平台差异

### 5.1 Windows/Mac 开发

在 Windows/Mac 上开发时，使用 Mock 实现：

```cangjie
// Mock 实现会返回默认值，不会真正操作 UI
let driver = Driver.create()
driver.click(100, 100)  // 实际不执行任何操作，返回 Unit
```

**说明**: 在 Windows/Mac 上只能进行代码编译，实际测试需要在 OpenHarmony 设备上执行。

**证据**: `mock/ohos.ui_test.cj`, `ohos/ui_test/BUILD.gn:19-30`

---

## 6. 获取帮助

### 6.1 官方资源

- **API 文档**: [cj-apis-ui_test.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/API_Reference/source_en/apis/TestKit/cj-apis-ui_test.md)
- **开发指南**: [cj-arkxtest-guidelines.md](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/blob/master/doc/Dev_Guide/source_en/application-test/cj-arkxtest-guidelines.md)

### 6.2 相关项目

| 项目 | 用途 | 链接 |
|------|------|------|
| arkxtest | UI 测试底层实现 | [testfwk_arkxtest](https://gitcode.com/openharmony/testfwk_arkxtest) |
| ability_cangjie_wrapper | Ability 框架绑定 | [ability_ability_cangjie_wrapper](https://gitcode.com/openharmony-sig/ability_ability_cangjie_wrapper) |
| cangjie_ark_interop | Cangjie-ArkTS 互操作 | [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) |

### 6.3 调试信息收集

遇到问题时，收集以下信息：

```bash
# 1. 系统参数
hdc shell param get persist.ace.testmode.enabled

# 2. 进程状态
hdc shell ps -ef | grep uitest

# 3. 日志
hdc shell hilog -g 1000 | grep -E "(CJ-UITEST|uitest)"

# 4. 应用信息
hdc shell bm dump -n <bundle_name>
```

---

## 7. 下一步阅读

- **[API 参考](20_API_Reference.md)** - 完整 API 清单
- **[安全分析](40_Security.md)** - 安全注意事项
- **[构建系统](30_Build_System.md)** - 集成方法

---

*本文档基于代码仓库静态分析和常见问题整理*  
*证据位置: ohos/ui_test/*.cj, mock/*.cj*

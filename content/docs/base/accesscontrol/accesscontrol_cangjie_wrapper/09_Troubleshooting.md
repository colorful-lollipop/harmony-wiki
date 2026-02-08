# 常见问题排查

## 目的

本文档提供 `accesscontrol_cangjie_wrapper` 项目常见构建、运行、调试问题的排查路径和解决方案。

## 适用范围

本文档适用于：
- 遇到构建失败的开发者
- 遇到运行时错误的应用开发者
- 需要调试问题的维护者

## 关键结论

1. **构建问题**: 平台差异、依赖缺失、权限问题
2. **运行时问题**: 库加载失败、FFI 调用失败、权限请求失败
3. **调试技巧**: 日志查看、符号查看、错误码查询
4. **常见错误**: 12100001（参数无效）、12100009（内部错误）、内存错误

## 相关跳转

- [对外 API](04_Public_API.md) - 查看 API 使用
- [构建产物](07_Build_Artifacts.md) - 了解库文件
- [GN Targets](06_GN_Targets.md) - 查看构建配置
- [安全评审](08_Security_Review.md) - 了解安全风险
- [工作笔记](wiki/_work/NOTES.md) - 查看代码证据索引

---

## 构建问题

### 问题 1: 编译失败 - 找不到 Cangjie 编译器

**错误信息**:
```
ERROR: Unknown build target: ohos_cangjie_shared_library
```

**原因**:
- Cangjie 编译器未安装
- 构建模板 `//build/templates/cangjie/cjc.gni` 不存在

**解决方案**:
```bash
# 1. 安装 Cangjie 编译器
# 参考: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/cangjie-overview-V5

# 2. 检查构建模板
ls -l build/templates/cangjie/cjc.gni

# 3. 清理并重新构建
./build.sh --clean
./build.sh --product-name <product>
```

**证据**: `BUILD.gn:14`, `ohos/ability_access_ctrl/BUILD.gn:16`

---

### 问题 2: 编译失败 - 找不到 FFI 接口

**错误信息**:
```
ERROR: Undefined reference: FfiOHOSAbilityAccessCtrlCheckAccessTokenSync
```

**原因**:
- access_token 子系统未编译
- `external_deps` 中的 FFI 库不存在

**解决方案**:
```bash
# 1. 编译 access_token 子系统
./build.sh --product-name <product> --build-target access_token

# 2. 检查 FFI 库是否存在
ls -l out/<product>/lib*/libcj_ability_access_ctrl_ffi.*

# 3. 重新编译本组件
./build.sh --product-name <product> --build-target accesscontrol_cangjie_wrapper
```

**证据**: `ohos/ability_access_ctrl/BUILD.gn:39`

---

### 问题 3: Windows/macOS 编译失败

**错误信息**:
```
ERROR: Mock implementation missing
```

**原因**:
- Mock 文件不存在
- Mock 文件路径不正确

**解决方案**:
```bash
# 1. 检查 Mock 文件是否存在
ls -l mock/ohos.ability_access_ctrl.cj
ls -l mock/ohos.security.permission_request_result.cj

# 2. 验证 BUILD.gn 中的路径
cat ohos/ability_access_ctrl/BUILD.gn | grep mock

# 3. 如果 Mock 文件不存在，创建最小实现
# mock/ohos.ability_access_ctrl.cj
package ohos.ability_access_ctrl

public enum GrantStatus {
    PermissionGranted | PermissionDenied
}

public class AbilityAccessCtrl {
    protected init() {}
    public static func createAtManager(): AtManager = AtManager()
}

public class AtManager {
    protected init() {}
    public func checkAccessToken(tokenID: UInt32, permissionName: String): GrantStatus {
        return GrantStatus.PermissionDenied  // Mock 返回
    }
    public func requestPermissionsFromUser(
        context: UIAbilityContext,
        permissionList: Array<String>,
        requestCallback: AsyncCallback<PermissionRequestResult>
    ): Unit {
        // Mock 实现
    }
}
```

**证据**: `ohos/ability_access_ctrl/BUILD.gn:20-22`

---

### 问题 4: 权限拒绝 - 构建失败

**错误信息**:
```
ERROR: Permission denied: base/accesscontrol/accesscontrol_cangjie_wrapper
```

**原因**:
- 构建用户没有权限访问源码目录
- Git 文件权限设置不当

**解决方案**:
```bash
# 1. 检查目录权限
ls -ld base/accesscontrol/accesscontrol_cangjie_wrapper

# 2. 修复权限
chmod -R 755 base/accesscontrol/accesscontrol_cangjie_wrapper

# 3. 如果使用 Git，修复 Git 权限
git config core.fileMode false
```

---

## 运行时问题

### 问题 1: 库加载失败 - 找不到共享库

**错误信息**:
```
ERROR: Unable to load library: libohos.ability_access_ctrl.so
java.lang.UnsatisfiedLinkError: libohos.ability_access_ctrl.so
```

**原因**:
- 库文件未编译或未安装
- 库路径不正确
- 库文件损坏

**解决方案**:
```bash
# 1. 检查库文件是否存在
find out/<product> -name "libohos.ability_access_ctrl.so"

# 2. 检查库路径
ls -l out/<product>/lib*/libohos.ability_access_ctrl.so

# 3. 检查库的依赖
ldd out/<product>/lib64/libohos.ability_access_ctrl.so

# 4. 重新编译并安装
./build.sh --product-name <product> --build-target copy_sdk_accesscontrol_cangjie_libs

# 5. 验证符号是否存在
nm -D out/<product>/lib64/libohos.ability_access_ctrl.so | grep AtManager
```

**证据**: `07_Build_Artifacts.md` - 编译产物

---

### 问题 2: TokenID = 0 错误

**错误信息**:
```
BusinessException: 12100001 - The parameter is invalid.
```

**原因**:
- 传入的 tokenID 为 0

**解决方案**:
```cangjie
// 错误代码
let tokenID = 0u32
let status = atManager.checkAccessToken(tokenID, permission)  // 抛出 12100001

// 正确代码
let tokenID = getApplicationTokenID()  // 获取真实的 TokenID
let status = atManager.checkAccessToken(tokenID, permission)
```

**证据**: `cj_ability_access_ctrl.cj:159-161`

---

### 问题 3: Context 无效错误

**错误信息**:
```
BusinessException: 12100009 - Common inner error.
```

**原因**:
- 传入的 context 无效或为 null
- context 不属于应用自身

**解决方案**:
```cangjie
// 错误代码
let context = null
atManager.requestPermissionsFromUser(context, permissions, callback)  // 抛出 12100009

// 正确代码
import ohos.app.ability.ui_ability.*
let context = getUIAbilityContext()  // 获取 UIAbilityContext
atManager.requestPermissionsFromUser(context, permissions, callback)
```

**证据**: `cj_ability_access_ctrl.cj:190-193`

---

### 问题 4: 权限名称无效错误

**错误信息**:
```
BusinessException: 12100003 - The specified permission does not exist.
```

**原因**:
- 权限名称拼写错误
- 权限不存在于系统中
- 权限未在 module.json5 中声明

**解决方案**:
```cangjie
// 错误代码
let permission = "ohos.permission.READ_CALENDARS"  // 拼写错误
let status = atManager.checkAccessToken(tokenID, permission)  // 抛出 12100003

// 正确代码
let permission = "ohos.permission.READ_CALENDAR"  // 正确拼写
let status = atManager.checkAccessToken(tokenID, permission)

// 在 module.json5 中声明权限
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.READ_CALENDAR"
      }
    ]
  }
}
```

**证据**: `cj_ability_access_ctrl_error.cj:30`

---

### 问题 5: 内存错误

**错误信息**:
```
Segmentation fault (core dumped)
Bus error
malloc: *** error for object 0x...: pointer being freed was not allocated
```

**原因**:
- FFI 调用中的 C 内存管理问题
- FFI 接口实现异常
- 指针操作错误

**解决方案**:
```bash
# 1. 启用 AddressSanitizer (ASan)
./build.sh --product-name <product> --ccache --asan

# 2. 使用 GDB 调试
gdb --args ./your_application

# 3. 查看 HiLog 日志
hilog -T CJ-AbilityAccessCtrl

# 4. 检查 access_token 子系统日志
hilog -T AccessToken
```

**证据**: `cj_ability_access_ctrl.cj:163-168` - 内存管理代码

---

### 问题 6: 异步回调未触发

**错误信息**:
```
权限请求已发起，但回调未执行
```

**原因**:
- 用户未操作权限请求对话框
- 对话框被系统取消
- 回调函数异常导致后续代码未执行

**解决方案**:
```cangjie
// 添加日志
atManager.requestPermissionsFromUser(
    context,
    permissions,
    { error, result =>
        ACCESS_LOG.info("Callback triggered: error=${error}, result=${result}")

        if (error == None) {
            ACCESS_LOG.info("Permissions granted: ${result.authResults}")
            // 处理结果
        } else {
            ACCESS_LOG.error("Permission request failed: ${error.code}")
            // 处理错误
        }
    }
)

// 添加超时处理
// 注意：当前 API 不支持超时，需要在应用层实现
```

**证据**: `cj_ability_access_ctrl.cj:196-212`

---

## 调试技巧

### 1. 查看 HiLog 日志

```bash
# 查看所有相关日志
hilog -T CJ-AbilityAccessCtrl

# 实时查看日志
hilog -T CJ-AbilityAccessCtrl | grep -E "error|Error|ERROR"

# 保存日志到文件
hilog -T CJ-AbilityAccessCtrl > access_control.log

# 按时间排序
hilog -T CJ-AbilityAccessCtrl -t
```

**日志标签**:
- `CJ-AbilityAccessCtrl` (0xD005A01)

证据：`cj_ability_access_ctrl.cj:70`

---

### 2. 使用 GDB 调试

```bash
# 1. 编译 Debug 版本
./build.sh --product-name <product> --build-type debug

# 2. 启动 GDB
gdb --args ./your_application

# 3. 设置断点
(gdb) break ohos.ability_access_ctrl.AtManager.checkAccessToken

# 4. 运行
(gdb) run

# 5. 查看变量
(gdb) print tokenID
(gdb) print permissionName

# 6. 查看调用栈
(gdb) bt

# 7. 单步执行
(gdb) step
(gdb) next
```

---

### 3. 使用 LLDB 调试 (macOS)

```bash
# 1. 编译 Debug 版本
./build.sh --product-name <product> --build-type debug

# 2. 启动 LLDB
lldb ./your_application

# 3. 设置断点
(lldb) breakpoint set -n checkAccessToken

# 4. 运行
(lldb) run

# 5. 查看变量
(lldb) frame variable tokenID

# 6. 查看调用栈
(lldb) thread backtrace
```

---

### 4. 查看错误码

**完整错误码列表**: 参见 [对外 API](04_Public_API.md) - 错误码部分

**常用错误码**:

| 错误码 | 消息 | 触发条件 | 解决方案 |
|--------|------|---------|---------|
| 12100001 | The parameter is invalid. | tokenID = 0 | 使用有效的 TokenID |
| 12100002 | The specified tokenID does not exist. | TokenID 不存在 | 检查 TokenID 是否正确 |
| 12100003 | The specified permission does not exist. | 权限不存在 | 检查权限名称拼写 |
| 12100007 | Service is abnormal. | access_token 服务异常 | 重启系统或检查服务状态 |
| 12100008 | Out of memory. | 内存不足 | 释放内存或增加设备内存 |
| 12100009 | Common inner error. | Context 无效 | 使用有效的 UIAbilityContext |

证据：`cj_ability_access_ctrl_error.cj:27-42`

---

### 5. 验证库符号

```bash
# Linux - 查看导出的符号
nm -D out/<product>/lib64/libohos.ability_access_ctrl.so | grep -E "AtManager|AbilityAccessCtrl|GrantStatus"

# 查看依赖的库
ldd out/<product>/lib64/libohos.ability_access_ctrl.so

# Windows - 查看 DLL 导出
dumpbin /EXPORTS out\<product>\ohos.ability_access_ctrl.dll

# 查看 DLL 依赖
dumpbin /DEPENDENTS out\<product>\ohos.ability_access_ctrl.dll

# macOS - 查看动态库符号
nm -gU out/<product>/libohos.ability_access_ctrl.dylib | grep AtManager

# 查看依赖的库
otool -L out/<product>/libohos.ability_access_ctrl.dylib
```

---

## 性能问题

### 问题 1: 权限检查缓慢

**症状**:
- `checkAccessToken()` 调用耗时超过 100ms

**原因**:
- FFI 调用开销
- access_token 子系统性能问题
- 频繁调用

**解决方案**:
```cangjie
// 1. 缓存权限检查结果
let permissionCache = HashMap<String, GrantStatus>()

func checkPermissionWithCache(tokenID: UInt32, permissionName: String): GrantStatus {
    let key = "${tokenID}:${permissionName}"
    if (let Some(cached) <- permissionCache.get(key)) {
        return cached
    }
    let status = atManager.checkAccessToken(tokenID, permissionName)
    permissionCache.put(key, status)
    return status
}

// 2. 减少调用频率
// 不要在循环中频繁调用 checkAccessToken
// 应该在应用启动时一次性检查所有需要的权限
```

---

### 问题 2: 权限请求对话框显示慢

**症状**:
- 调用 `requestPermissionsFromUser()` 后，对话框延迟显示

**原因**:
- UI 线程繁忙
- access_token 子系统响应慢
- 系统资源不足

**解决方案**:
```cangjie
// 1. 在主线程调用
// 确保 requestPermissionsFromUser 在主线程调用
@MainActor
func requestPermissions() {
    atManager.requestPermissionsFromUser(context, permissions, callback)
}

// 2. 减少请求数量
// 避免一次性请求过多权限（建议不超过 10 个）
let permissions = [
    "ohos.permission.READ_CALENDAR",
    "ohos.permission.CAMERA"
    // 限制数量
]

// 3. 分批请求
// 如果需要请求很多权限，分批请求
func requestPermissionsInBatches() {
    let batch1 = ["ohos.permission.READ_CALENDAR", "ohos.permission.CAMERA"]
    let batch2 = ["ohos.permission.READ_CONTACTS", "ohos.permission.WRITE_CONTACTS"]
    // ...
}
```

---

## 开发环境问题

### 问题 1: Mock 实现与真实实现不一致

**症状**:
- Windows/macOS 开发环境测试通过
- 设备上运行失败

**原因**:
- Mock 实现不完整
- Mock 实现未包含真实行为的边界检查

**解决方案**:
```bash
# 1. 在真实设备上测试
# 确保在支持 access_token 的真实设备上测试

# 2. 使用模拟器
# 使用支持 access_token 的模拟器进行测试

# 3. 改进 Mock 实现
// Mock 实现应该模拟真实行为，包括：
// - 参数验证（TokenID != 0）
// - 错误码返回
// - 异步回调
```

---

### 问题 2: IDE 无法识别 API

**症状**:
- IDE 显示 "Cannot find symbol"
- 代码提示不工作

**原因**:
- SDK 未正确导入
- IDE 未配置 Cangjie 支持

**解决方案**:
```bash
# 1. 确保 SDK 已安装
ls -l $HOME/.ohos/packages/sdk/cangjie

# 2. 配置 IDE
# DevEco Studio -> File -> Project Structure -> SDK -> Cangjie

# 3. 刷新项目
./hvigorw clean
./hvigorw assembleHap
```

---

## 常见编码问题

### 问题 1: 忽略异步回调

**错误代码**:
```cangjie
// 错误：忽略回调
atManager.requestPermissionsFromUser(context, permissions, { error, result => })

// 假设权限已授予，继续执行
useCamera()  // 可能会失败，因为权限可能未授予
```

**正确代码**:
```cangjie
// 正确：在回调中处理结果
atManager.requestPermissionsFromUser(context, permissions, { error, result =>
    if (error == None) {
        if (result.authResults[0] == 0) {
            useCamera()  // 在回调中执行
        } else {
            showPermissionDeniedDialog()
        }
    }
})

// 不要假设权限已授予
```

---

### 问题 2: 未处理异常

**错误代码**:
```cangjie
// 错误：未处理异常
let status = atManager.checkAccessToken(tokenID, permission)  // 可能抛出异常

// 正确代码
try {
    let status = atManager.checkAccessToken(tokenID, permission)
    if (status == GrantStatus.PermissionGranted) {
        // ...
    }
} catch (e: BusinessException) {
    ACCESS_LOG.error("Check access token failed: ${e.code} - ${e.message}")
}
```

---

## 获取帮助

### 社区资源

- [OpenHarmony 官方文档](https://docs.openharmony.cn/)
- [OpenHarmony 论坛](https://forums.openharmony.cn/)
- [OpenHarmony GitHub](https://github.com/openharmony)
- [access_token 仓库](https://gitcode.com/openharmony/security_access_token)

### 提交 Bug

如果遇到本文档未覆盖的问题，请提交 Bug：

1. 收集日志信息
2. 复现步骤
3. 设备信息
4. 系统版本

提交到: [OpenHarmony Issue Tracker](https://gitee.com/openharmony/issues)

---

## 下一步

1. 查看 [对外 API](04_Public_API.md) 学习正确使用 API
2. 参考 [GN Targets](06_GN_Targets.md) 了解构建配置
3. 阅读 [安全评审](08_Security_Review.md) 了解安全注意事项

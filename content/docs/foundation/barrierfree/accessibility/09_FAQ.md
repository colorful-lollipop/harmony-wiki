# 常见问题

## 目的

本文档收集和解答 Accessibility 子系统的常见问题，包括构建、运行、调试相关的问题和定位路径。

## 适用范围

- 新项目成员
- 遇到问题的开发者
- 构建工程师
- 调试问题的开发者

## 关键结论

### 问题分类

1. **构建相关** - 编译错误、依赖问题
2. **运行时相关** - 服务启动、连接问题
3. **调试相关** - 日志、工具使用
4. **API 使用相关** - 参数、错误码

---

## 详细内容

### 构建相关

#### Q1: 编译失败，提示找不到 accessibility 相关库

**症状**: 编译时链接错误，如 `undefined reference to 'AccessibilityXXX'`

**可能原因**:
1. 未添加正确的依赖
2. BUILD.gn 中 target 路径错误
3. 未包含正确的 include_dirs

**定位路径**:
1. 检查 `bundle.json` 中的 inner_kits 配置
2. 确认目标 target 的 deps 包含需要的 kit
3. 查看 `interfaces/innerkits/*/BUILD.gn` 确认输出名

**证据**: `bundle.json:100-160`

**解决方案**:
```gn
# 添加正确的依赖
deps = [
    "//interfaces/innerkits/common:accessibility_common",
    "//interfaces/innerkits/asacfwk:accessibilityclient",
]

# 添加 include 路径
include_dirs = [
    "//interfaces/innerkits/common/include",
]
```

#### Q2: 编译报错 "undefined reference to AccessibilityEventInfo"

**症状**: C++ 代码中使用 `AccessibilityEventInfo` 等类型时链接失败

**可能原因**: 未依赖 `accessibility_common`

**定位路径**: 查看目标 target 是否包含 `accessibility_common` 依赖

**证据**: `interfaces/innerkits/common/BUILD.gn`

**解决方案**:
```gn
deps = [
    "//interfaces/innerkits/common:accessibility_common",
]
```

### 运行时相关

#### Q3: AccessibilityService 启动失败

**症状**: 系统日志显示 AccessibilityService 启动失败

**可能原因**:
1. SA 801 注册失败
2. 依赖的系统服务未启动
3. 配置文件缺失或错误

**定位路径**:
1. 查看 HiSysEvent 日志
2. 查看 hilog 输出
3. 检查 `/system/profile/801.json` 是否存在

**证据**: `sa_profile/801.json`, `hisysevent.yaml`

**解决方案**:
1. 确认依赖服务已启动（Window Manager, Input Manager 等）
2. 检查 SA 配置文件完整性
3. 查看系统日志确认错误原因

#### Q4: 无障碍扩展应用连接失败

**症状**: `AccessibilityExtensionAbility` 无法连接到 AccessibilityService

**可能原因**:
1. 权限不足（`ACCESSIBILITY_EXTENSION_ABILITY`）
2. 服务未启动
3. bundle.json 配置错误

**定位路径**:
1. 检查应用权限声明
2. 查看 hilog 日志中的连接错误
3. 确认 AccessibilityService 是否运行

**证据**: `interfaces/kits/napi/accessibility_extension/`

**解决方案**:
```json
// module.json5 中添加权限
{
    "module": {
        "reqPermissions": [
            {
                "name": "ohos.permission.ACCESSIBILITY_EXTENSION_ABILITY"
            }
        ]
    }
}
```

#### Q5: 手势注入失败

**症状**: `injectGesture()` 调用失败或无效果

**可能原因**:
1. 权限不足
2. 手势路径参数错误
3. 系统输入拦截未启用

**定位路径**:
1. 检查应用权限
2. 查看 N-API 调用参数
3. 查看 AccessibilityService 日志

**证据**: `interfaces/kits/napi/accessibility_extension_context/` 中的 injectGesture 实现

**解决方案**:
```typescript
// 确保权限
// module.json5:
"reqPermissions": [
    {
        "name": "ohos.permission.ACCESSIBILITY_EXTENSION_ABILITY"
    }
]

// 正确构造手势路径
const gesturePath = new accessibility.GesturePath([
    new accessibility.GesturePoint(100, 200, 0), // x, y, time
    new accessibility.GesturePoint(150, 200, 100),
]);

await context.injectGesture(gesturePath);
```

### 调试相关

#### Q6: 如何查看 AccessibilityService 日志

**方法**: 使用 hilog 命令

**命令**:
```bash
# 查看 accessibility 进程日志
hdc shell hilog -T AccessibilityService

# 查看所有无障碍相关日志
hdc shell hilog -T ACCESSIBILITY

# 查看特定标签
hdc shell hilog -t AAMS
```

**证据**: `common/log/include/hilog_wrapper.h`

#### Q7: 如何调试 N-API 调用

**方法**: 在 N-API 实现中添加日志

**示例**:
```cpp
static napi_value MyFunction(napi_env env, napi_callback_info info)
{
    HILOG_INFO("MyFunction called");
    // ... 函数实现
    HILOG_INFO("MyFunction completed with result: %{public}d", result);
}
```

**证据**: `interfaces/kits/napi/src/napi_accessibility_system_ability_client.cpp`

#### Q8: 如何调试 IPC 通信问题

**方法**: 查看 IPC 接口的实现

**步骤**:
1. 找到对应的 IPC 接口（如 `IAccessibleAbilityChannel`）
2. 查看 Proxy 实现（客户端）
3. 查看 Stub 实现（服务端）
4. 在 Proxy 和 Stub 的关键位置添加日志

**证据**: `common/interface/src/accessible_ability_channel_proxy.cpp`, `accessible_ability_channel_stub.cpp`

### API 使用相关

#### Q9: "Permission denied" 错误

**症状**: 调用 API 时返回权限拒绝错误

**可能原因**:
1. 应用未声明所需权限
2. 应用不是系统应用（某些接口需要）
3. 权限名称拼写错误

**定位路径**:
1. 检查 module.json5 中的权限声明
2. 查看 API 文档确认需要的权限
3. 查看系统日志确认权限验证失败

**证据**: `services/aams/src/accessible_ability_manager_service.cpp:1447` (CheckPermission)

**解决方案**:
```json
// module.json5
{
    "module": {
        "reqPermissions": [
            {
                "name": "ohos.permission.ACCESSIBILITY_EXTENSION_ABILITY"
            },
            {
                "name": "ohos.permission.QUERY_ACCESSIBILITY_ELEMENT"
            }
        ]
    }
}
```

#### Q10: 获取的元素信息为空

**症状**: `getElements()` 返回空数组

**可能原因**:
1. 目标应用当前无窗口
2. 目标应用未集成无障碍
3. 查询条件不匹配

**定位路径**:
1. 检查目标应用是否支持无障碍
2. 确认应用是否在前台
3. 查看 AccessibilityService 日志

**证据**: `interfaces/innerkits/asacfwk/include/accessibility_element_operator.h`

**解决方案**:
1. 确保目标应用已启动并集成无障碍
2. 使用更宽松的查询条件
3. 检查是否有权限访问目标应用

---

## 相关链接

- [项目概览](00_Overview.md)
- [架构说明](03_Architecture.md)
- [N-API 接口](04_N-API.md)

---

最后更新: 2026-02-06

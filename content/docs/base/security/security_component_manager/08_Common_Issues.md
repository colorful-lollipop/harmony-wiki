# 常见构建/运行/调试问题 - Security Component Manager

> 目的：解决 Security Component Manager 的构建、运行与调试问题

---

## 适用范围

本文档适用于：
- 遇到编译错误的开发者
- 遇到运行时问题的开发者
- 需要调试权限问题的开发者

---

## 关键结论

1. **主要问题类型**：构建错误、服务启动失败、权限不授予、组件无法注册
2. **问题定位方法**：日志分析、SA 状态检查、权限验证
3. **常见修复方案**：配置调整、权限申请、库依赖修复

---

## 构建问题

### 问题 1：编译错误 - undefined reference

**症状**：
```
error: undefined reference to 'SecCompKit::RegisterSecurityComponent'
```

**原因**：
- 应用未正确 Link `libsecurity_component_sdk.so`
- 应用 BUILD.gn 缺少 deps

**修复方案**：

**检查 1**：确认应用 BUILD.gn 包含正确的 deps
```gn
# 应用 BUILD.gn
ohos_shared_library("my_app") {
  deps = [
    "//base/security/security_component_manager/frameworks/inner_api/security_component:libsecurity_component_sdk",
  ]
}
```

**检查 2**：确认头文件包含
```cpp
#include "security_component/sec_comp_kit.h"
```

**证据路径**：`frameworks/inner_api/security_component/BUILD.gn:24`

---

### 问题 2：编译错误 - cannot find -lsecurity_component_sdk

**症状**：
```
error: cannot find -lsecurity_component_sdk
```

**原因**：
- 应用使用错误的依赖名称
- 使用 `external_deps` 而非 `deps`

**修复方案**：

使用正确的 Target 路径：
```gn
# 错误写法
external_deps = [
  "security_component_manager:libsecurity_component_sdk",  # ❌ 错误
]

# 正确写法
deps = [
  "//base/security/security_component_manager/frameworks/inner_api/security_component:libsecurity_component_sdk",  # ✅ 正确
]
```

---

### 问题 3：增强功能编译失败

**症状**：
```
error: 'SecCompEnhanceAdapter' was not declared in this scope
```

**原因**：
- 缺少 `security_component_enhance_enable` feature flag
- 编译配置不正确

**修复方案**：

**方案 1**：启用增强功能（如果需要）
```bash
# 编译时添加 feature flag
./build.sh --product-name rk3568 --build-variant root \
  --gn-args security_security_component_enhance=true
```

**方案 2**：禁用增强功能（如果不需要）
```bash
# 编译时不添加 feature flag（默认禁用）
./build.sh --product-name rk3568 --build-variant root
```

**证据路径**：`security_component.gni:16-21`

---

### 问题 4：CFI 链接失败（测试编译）

**症状**：
```
error: undefined reference to vtable for 'SecCompService'
```

**原因**：
- 测试代码链接了 CFI 版本的源集
- CFI 导致 vtable 类型不匹配

**修复方案**：

使用 no_cfi 变体：
```gn
# 测试 BUILD.gn
ohos_unittest("sec_comp_service_test") {
  deps = [
    ":security_component_no_cfi_service_stub",  # ✅ 使用 no_cfi 变体
  ]
}
```

**证据路径**：`services/security_component_service/sa/BUILD.gn:62-80`

---

## 运行问题

### 问题 1：Security Component Service 未启动

**症状**：
```
应用调用 API 时返回错误码 -55 (SC_SERVICE_ERROR_SERVICE_NOT_EXIST)
```

**排查步骤**：

**步骤 1**：检查 SA Profile
```bash
hdc shell cat /system/profile/3506.json
```

**预期输出**：
```json
{
  "services": [
    {
      "name": "3506",
      "path": "/system/lib/libsecurity_component_service.z.so",
      "ondemand": true
    }
  ]
}
```

**步骤 2**：检查服务库
```bash
hdc shell ls -l /system/lib/libsecurity_component_service.z.so
```

**步骤 3**：检查服务权限
```bash
hdc shell cat /system/etc/init/security_component_service.cfg
```

**预期内容**：
```
{
    "name": "security_component_service",
    "type": "sa",
    "required": [
        "GetSystemAbilityManager",
        "GrantRuntimePermission",
        "RevokeRuntimePermission"
    ]
}
```

**步骤 4**：检查服务进程
```bash
hdc shell ps -A | grep security_component
```

**预期输出**：
```
root        1234  ...  ...  security_component_service
```

**步骤 5**：查看服务日志
```bash
hdc shell hilog -T SecurityComponent | grep OnStart
```

**证据路径**：`services/security_component_service/sa/sa_profile/3506.json`

---

### 问题 2：组件注册失败（错误码 -56）

**症状**：
```
应用调用 RegisterSecurityComponent() 返回 -56 (SC_SERVICE_ERROR_COMPONENT_INFO_INVALID)
```

**排查步骤**：

**步骤 1**：查看服务日志
```bash
hdc shell hilog -T SecurityComponent | grep -i "register"
```

**步骤 2**：检查 JSON 格式

应用传递的 JSON 必须符合以下格式：
```json
{
  "type": 2,  // SecCompType (1=Location, 2=Paste, 3=Save)
  "text": "粘贴",
  "icon": "/path/to/icon.png",
  "rect": {
    "x": 100.0,
    "y": 200.0,
    "width": 120.0,
    "height": 40.0
  }
}
```

**步骤 3**：检查图标路径
```bash
hdc shell ls -l /path/to/icon.png
```

**步骤 4**：检查最小值要求

根据 `sec_comp_info.h` 定义：
- 最小字体大小：12.0（无图标）、10.0（有图标）
- 最小图标大小：12.0
- 最小 padding：0.0（有背景）、4.0（无背景）

**步骤 5**：检查组件重叠
```bash
# 查看日志中的重叠错误
hdc shell hilog -T SecurityComponent | grep -i "overlap"
```

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_info.h:28-33`

---

### 问题 3：点击事件验证失败（错误码 -60）

**症状**：
```
应用调用 ReportSecurityComponentClickEvent() 返回 -60 (SC_SERVICE_ERROR_CLICK_EVENT_INVALID)
```

**排查步骤**：

**步骤 1**：查看服务日志
```bash
hdc shell hilog -T SecurityComponent | grep -i "click"
```

**步骤 2**：检查点击坐标

确保点击坐标在组件矩形内：
- `touchX` 在 `[x, x + width]` 范围内
- `touchY` 在 `[y, y + height]` 范围内

**步骤 3**：检查时间戳

确保点击事件时间戳在有效期内（5000ms）：
```cpp
uint64_t currentTime = GetTimestamp();
uint64_t clickTime = clickInfo.point.timestamp;
if (currentTime - clickTime > 5000) {
    // 时间戳过期
}
```

**步骤 4**：检查窗口覆盖

```bash
# 查看日志中的窗口覆盖错误
hdc shell hilog -T SecurityComponent | grep -i "window"
```

**步骤 5**：检查增强数据

如果启用了增强框架，确保增强数据正确：
```cpp
if (clickInfo.extraInfo.data != nullptr && clickInfo.extraInfo.dataSize > 0) {
    // 增强数据验证逻辑
}
```

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_entity.cpp:124-174`

---

### 问题 4：权限未授予

**症状**：
```
点击事件返回成功，但应用访问敏感数据时被拒绝
```

**排查步骤**：

**步骤 1**：验证权限授予
```bash
# 查看服务日志中的权限授予记录
hdc shell hilog -T SecurityComponent | grep -i "grant"
```

**步骤 2**：检查应用状态

```bash
# 确保应用在前台
hdc shell dump window_manager | grep <应用包名>
```

**步骤 3**：检查权限撤销

```bash
# 查看是否权限被立即撤销
hdc shell hilog -T SecurityComponent | grep -i "revoke"
```

**步骤 4**：验证 Token ID

```bash
# 查看日志中的 Token ID
hdc shell hilog -T SecurityComponent | grep -i "token"
```

**步骤 5**：检查权限有效期

Location/Paste 权限有效期为前台 + 10 秒：
- 确保应用在前台
- 确保未超过 10 秒延迟

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:279-323`

---

### 问题 5：权限未撤销

**症状**：
```
应用进入后台后，临时权限仍然有效
```

**排查步骤**：

**步骤 1**：查看应用状态监听
```bash
# 查看服务日志中的前台/后台事件
hdc shell hilog -T SecurityComponent | grep -i "background"
```

**步骤 2**：检查延迟任务
```bash
# 查看延迟撤销任务是否启动
hdc shell hilog -T SecurityComponent | grep -i "delayed"
```

**步骤 3**：检查 ffrt 任务队列

```bash
# 查看事件处理器日志
hdc shell hilog -T SecurityComponent | grep -i "event"
```

**步骤 4**：验证应用状态转换

确保应用正确触发前台/后台转换：
```cpp
// 在 app_state_observer.cpp 中
void AppStateObserver::OnForegroundApplicationChanged(const AppStateData& appStateData) {
    if (appStateData.state == ApplicationState::APP_STATE_FOREGROUND) {
        // 应用进入前台
    } else if (appStateData.state == ApplicationState::APP_STATE_BACKGROUND) {
        // 应用进入后台
    }
}
```

**步骤 5**：检查应用死亡监听

```bash
# 查看应用死亡事件
hdc shell hilog -T SecurityComponent | grep -i "died"
```

**证据路径**：`services/security_component_service/sa/sa_main/app_state_observer.cpp:32-43`

---

## 调试技巧

### 启用详细日志

**方法 1**：HiLog 过滤

```bash
# 查看 Security Component Manager 日志
hdc shell hilog -T SecurityComponent

# 查看错误日志
hdc shell hilog -T SecurityComponent -L ERROR

# 查看特定标签
hdc shell hilog -T SecurityComponent | grep SecCompService
```

**方法 2**：Dump SA 信息

```bash
# Dump 所有 SA 信息
hdc shell dump -a | grep 3506

# Dump Security Component Service
hdc shell serviceCheck 3506 -dump
```

**方法 3**：Dump 组件信息

```bash
# Dump 所有注册的组件
hdc shell serviceCheck 3506 -dump -c

# 示例输出：
# Component List:
# scId: 1, type: PASTE, pid: 1234, tokenId: 5678
# scId: 2, type: SAVE, pid: 1234, tokenId: 5678
```

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_service.h:56`

---

### 性能分析

**问题**：服务响应慢

**排查步骤**：

**步骤 1**：添加性能日志
```bash
# 启用性能统计
hdc shell hilog -T SecurityComponent -L PERF
```

**步骤 2**：使用 HiTrace 跟踪
```bash
# 查看关键路径
hdc shell hitrace --name security_component
```

**步骤 3**：检查锁竞争

```bash
# 查看锁等待日志
hdc shell hilog -T SecurityComponent | grep -i "lock"
```

---

## 常见错误码速查

| 错误码 | 名称 | 常见原因 | 解决方案 |
|--------|------|----------|----------|
| -50 | `SC_SERVICE_ERROR_VALUE_INVALID` | 参数为空或超出范围 | 检查参数格式和范围 |
| -55 | `SC_SERVICE_ERROR_SERVICE_NOT_EXIST` | 服务未启动 | 检查 SA Profile 和服务进程 |
| -56 | `SC_SERVICE_ERROR_COMPONENT_INFO_INVALID` | JSON 解析失败 | 检查 JSON 格式 |
| -57 | `SC_SERVICE_ERROR_COMPONENT_RECT_OVERLAP` | 组件重叠 | 调整组件位置和尺寸 |
| -58 | `SC_SERVICE_ERROR_COMPONENT_NOT_EXIST` | scId 无效 | 检查 scId 是否已注册 |
| -60 | `SC_SERVICE_ERROR_CLICK_EVENT_INVALID` | 点击验证失败 | 检查坐标、时间戳、窗口覆盖 |
| -62 | `SC_SERVICE_ERROR_CALLER_INVALID` | 调用者无效 | 检查 Token 和 UID |
| -63 | `SC_SERVICE_ERROR_WAIT_FOR_DIALOG_CLOSE` | 等待对话框 | 等待对话框关闭或超时 |
| -100 | `SC_ENHANCE_ERROR_NOT_EXIST_ENHANCE` | 无增强库 | 检查增强库是否安装 |
| -110 | `SC_ENHANCE_ERROR_CHALLENGE_CHECK_FAIL` | Challenge 检查失败 | 检查增强数据和配置 |

**证据路径**：`interfaces/inner_api/security_component/include/sec_comp_err.h:23-54`

---

## 相关跳转

- [对外 API](./03_Public_APIs.md) - 查看 API 错误码说明
- [内部 API](./04_Internal_APIs.md) - 查看内部接口
- [编译产物](./06_Build_Artifacts.md) - 查看部署验证步骤

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)

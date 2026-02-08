# 07_Troubleshooting - 常见问题与调试指南

## 概述

本文档收集 advanced_ui_component 使用过程中的常见问题及其解决方案。

---

## 构建问题

### Q1: es2abc 编译失败 - 找不到入口文件

**现象**:
```
error: es2abc: source file not found: componentname.js
```

**原因**: `BUILD.gn` 中 `src_js` 路径配置错误。

**解决方案**:

检查 `interfaces/BUILD.gn`:
```gn
es2abc_gen_abc("gen_componentname_abc") {
  # 确保路径正确
  src_js = rebase_path("componentname.js", "//foundation/arkui/...")
  # 或使用相对路径
  src_js = "componentname.js"
}
```

**证据来源**: `atomicservicenavigation/interfaces/BUILD.gn:19`

---

### Q2: 链接错误 - 找不到 GetABCCode 符号

**现象**:
```
undefined reference to `NAPI_atomicservice_ComponentName_GetABCCode'
```

**原因**: ABC 字节码未正确链接到共享库。

**解决方案**:

1. 检查 `deps` 配置:
```gn
ohos_shared_library("componentname") {
  sources = [ "componentname.cpp" ]
  # 确保 ABC 目标在 deps 中
  deps = [ ":componentname_abc" ]
}
```

2. 检查 `gen_js_obj` 依赖:
```gn
gen_js_obj("componentname_abc") {
  input = ...
  dep = ":gen_componentname_abc"  # 确保 es2abc 目标被依赖
}
```

**证据来源**: `atomicservicenavigation/interfaces/BUILD.gn:26-31`

---

### Q3: 外部依赖缺失

**现象**:
```
error: dependency 'bundle_framework:appexecfwk_base' not found
```

**原因**: 指定的子系统或部件未构建。

**解决方案**:

```bash
# 1. 确保子系统已构建
hb build //bundles/bundle_framework:appexecfwk_base

# 2. 或在全局构建中包含
hb build -f

# 3. 检查依赖配置
hb env | grep product
```

---

### Q4: 预览模式与真机模式差异

**现象**: 预览正常，但真机运行崩溃。

**原因**: 预览使用 `gen_obj_src_*` 目标，真机使用 `gen_js_obj` 目标。

**解决方案**:

确保 `BUILD.gn` 中的条件配置正确:
```gn
if (use_mingw_win || use_mac || use_linux) {
  deps = [ ":gen_obj_src_componentname_abc_preview" ]
} else {
  deps = [ ":componentname_abc" ]
}
```

**证据来源**: `atomicservice_config.gni:20-25`

---

## 运行时问题

### Q5: 模块加载失败

**现象**:
```
Failed to load module: atomicservice.AtomicServiceNavigation
```

**原因**:
1. .so 文件未安装到正确路径
2. ABC 字节码导出函数未找到

**排查步骤**:

```bash
# 1. 检查模块是否存在
ls -la /system/lib64/module/atomicservice/libatomicservicenavigation.so

# 2. 检查符号
nm -D /system/lib64/module/atomicservice/libatomicservicenavigation.so | grep GetABCCode

# 3. 查看系统日志
hilog | grep -i "atomicservice"
```

---

### Q6: checkUrl 返回异常值

**现象**: `checkUrl` 返回非 0 值，但 URL 应该是合法的。

**原因**:
1. 系统 API Policy 库未加载
2. URL 格式不符合策略要求

**排查步骤**:

```bash
# 检查策略库是否存在
ls -la /system/lib64/platformsdk/libapipolicy_client.z.so

# 检查日志
hilog | grep -E "ApiPolicy|checkUrl"
```

**解决方案**:

1. 确认系统版本支持 API Policy
2. 检查 URL 格式:
   - 必须包含协议头 (http/https)
   - 域名类型必须匹配

**证据来源**: `atomicserviceweb/interfaces/api_policy_adapter.cpp:21-26`

---

### Q7: NavPushPathHelper 静默安装失败

**现象**:
```typescript
NavPushPathHelper.silentInstall('module').catch(err => {
  console.error(err);  // Installation failed
});
```

**原因**:
1. 缺少必要权限
2. 模块名不存在
3. HSP 服务不可用

**排查步骤**:

```typescript
// 1. 检查权限
const permissions = bundle.getGrantedPermissions();
console.info('Granted permissions:', permissions);

// 2. 检查模块是否存在
const exists = NavPushPathHelper.isHspExist('moduleName');
console.info('Module exists:', exists);

// 3. 检查 HSP 服务状态
hilog | grep -i hsp
```

**权限要求**:
```json
{
  "name": "ohos.permission.INSTALL_BUNDLE",
  "usedScene": {
    "abilities": ["EntryAbility"],
    "when": "inuse"
  }
}
```

---

### Q8: 菜单栏不显示

**现象**: 调用 `setMenubarVisible(true)` 后菜单栏未显示。

**原因**:
1. 未处于原子化服务上下文
2. 窗口模式不支持

**解决方案**:

```typescript
// 确保在正确的上下文中调用
if (isAtomicServiceContext()) {
  MenubarAPIImplement.setMenubarVisible(true);
}

// 检查窗口模式
const windowClass = window.getLastWindow(this.context);
const windowMode = windowClass.getWindowMode();
console.info('Window mode:', windowMode);
```

---

## 调试技巧

### 日志过滤

```bash
# 过滤所有组件日志
hilog | grep -E "atomicservice|launchcomponent|dialogaction|navpushpathhelper"

# 过滤错误日志
hilog | grep -iE "error|fail|exception"

# 过滤 N-API 相关
hilog | grep -i napi
```

### 组件状态检查

```typescript
// 启用调试模式
const DEBUG = true;

// 打印组件初始化状态
if (DEBUG) {
  console.info('Component initialized:', {
    name: this.componentName,
    state: this.state,
    timestamp: Date.now()
  });
}
```

### 性能分析

```typescript
// 性能监控
const startTime = performance.now();

// 执行操作
this.doSomething();

const endTime = performance.now();
console.info(`Operation took ${endTime - startTime}ms`);
```

---

## 回退策略

### 组件不可用时

```typescript
try {
  // 使用高级组件
  await AtomicServiceWeb.checkUrl(bundleName, domainType, url);
} catch (error) {
  // 回退到基础实现
  console.warn('Advanced component unavailable, using fallback');
  useBasicWebView(url);
}
```

### 功能降级

```typescript
// 检测功能可用性
const isNavPushAvailable = checkFeatureAvailability('NavPushPathHelper');

if (!isNavPushAvailable) {
  // 禁用相关功能
  this.enableHspFeature = false;
  this.uiState = 'degraded';
}
```

---

## 相关资源

| 资源 | 链接 |
|------|------|
| OpenHarmony SDK | https://docs.openharmony.cn |
| ArkUI 组件 | https://docs.openharmony.cn/pages/master/zh-cn/application-dev/reference/apis-arkui/Readme-CN.md |
| N-API 开发 | https://docs.openharmony.cn/pages/master/zh-cn/application-dev/reference/native-lib/Readme-CN.md |
| 问题反馈 | https://gitee.com/openharmony-sig/arkui_advanced_ui_component/issues |
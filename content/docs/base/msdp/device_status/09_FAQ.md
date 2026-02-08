# 常见问题 (FAQ)

## 目的

本文档列出开发、构建、运行和调试 `device_status` 模块时遇到的常见问题及其解决方案。

---

## 开发问题

### 1. N-API 模块开发

#### Q: 如何添加新的 N-API 模块？

**A**: 参考 [04_N-API_Reference.md](04_N-API_Reference.md) 文档，了解现有 N-API 模块结构。

**关键步骤**：
1. 在 `frameworks/js/napi/` 下创建新模块目录
2. 实现 N-API 注册函数：`ModuleInit()` 或 `Export()`
3. 定义 `napi_module` 结构：设置 `nm_modname`、`nm_register_func` 等
4. 实现导出的 JS 方法
5. 在对应的 `BUILD.gn` 中添加 target
6. 使用 `napi_define_properties` 导出方法
7. 使用 `napi_create_threadsafe_function` 如果需要线程安全

**示例**：
```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_modname = "newModule",
    .nm_register_func = NewModuleInit,
};

extern "C" __attribute__((constructor)) void RegisterModule(void)
{
    napi_module_register(&g_module);
}

static napi_value NewModuleInit(napi_env env, napi_value exports)
{
    // 导出方法
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("newMethod", NewMethod),
    };
    return napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
}
```

#### Q: N-API 回调函数如何工作？

**A**: 回调通过 `napi_send_event` 投递到 ArkTS 事件线程。

**关键点**：
- 使用 `napi_eprio_immediate` 设置优先级
- 回调函数签名：`void CallbackFunction(napi_env env, napi_callback_info info, void *data)`
- 在回调中不要阻塞 UI 线程
- 使用 `napi_delete_reference` 清理引用

**示例**：
```cpp
void CallbackFunction(napi_env env, napi_callback_info info, void *data) {
    // 处理回调数据
    auto* callbackData = static_cast<CallbackData*>(data);

    napi_value jsCallback = nullptr;
    napi_status status = napi_create_function(env, callbackData->jsCallback);
    if (status != napi_ok) {
        FI_HILOGE("Failed to create JS callback");
        return;
    }

    // 调用 JS 回调
    napi_value argv[] = { /* 参数 */ };
    napi_value result = nullptr;
    status = napi_call_function(env, callbackData->jsCallback, 1, argv, &result);

    // 清理引用
    napi_delete_reference(env, jsCallback);
}
```

---

## 构建问题

### 2. GN 构建错误

#### Q: 编译时出现 undefined reference 错误

**A**: 检查目标依赖关系，确保所有依赖项正确配置。

**常见原因**：
- `external_deps` 中的组件名称拼写错误
- 条件编译宏未正确定义或未启用
- 目标名称拼写错误

**解决方法**：
```bash
# 检查未定义的符号
gn gen out --root=target_os 2>&1 | grep -i undefined

# 清理并重新编译
rm -r out && gn gen out --root=target_os
```

#### Q: 如何启用 Rust 实现？

**A**: 修改 `device_status.gni` 文件，设置 `device_status_rust_enabled = true`。

**注意**：
- Rust 实现目前是实验性的
- 启用后会生成 `libfusion_ipc_server_ffi.z.so` 而非 `libdevicestatus_service.z.so`
- SA 配置文件会从 `2902.json` 变为 `2902_rust.json`

#### Q: 特性开关不起作用？

**A**: 检查 `bundle.json` 中的 `features` 列表是否正确配置。

**常见原因**：
- 特性名称拼写错误
- 条件判断逻辑错误
- 相关组件未实现

**调试方法**：
```bash
# 检查哪些特性被启用
grep -r "device_status_" device_status.gni | grep "true"

# 查看 predefines
gn gen out --root=target_os --args="device_status_intention_framework=true" --list-targets
```

---

## 运行问题

### 3. 服务启动失败

#### Q: DeviceStatusService 无法启动？

**A**: 检查系统日志（`hilog -b device_status`）。

**常见原因**：
- 依赖的系统服务未启动（如 BundleManager、SAMGR）
- 权限不足
- 算法库加载失败
- Socket 创建失败

**调试步骤**：
```bash
# 查看服务状态
hidumper -s device_status
hilog -b device_status | grep "OnStart failed"

# 查看 IPC 状态
hilog -b device_status | grep "IPC"

# 检查依赖服务
hilog -b device_status | grep "BundleManager\|SAMGR"
```

#### Q: IntentionService 启动失败？

**A**: 同样检查系统日志。

**常见原因**：
- 插件加载失败
- 设备管理器初始化失败
- DSoftBus 连接失败

---

### 4. N-API 问题

#### Q: N-API 模块加载失败？

**A**: 检查应用日志和系统日志。

**常见原因**：
- N-API 模块未编译到系统
- 依赖库加载失败
- 权限不足导致模块禁用

**调试方法**：
```javascript
// 在应用中添加 try-catch
try {
  const result = deviceStatus.on(type, callback);
  console.log('Success:', result);
} catch (error) {
  console.error('Failed:', error.message);
}
```

---

## 调试问题

### 5. HiSysEvent 上报

#### Q: 如何上报自定义事件？

**A**: 参考 `hisysevent.yaml` 和 `hisyseventdrague.yaml` 配置文件。

**关键点**：
- 使用正确的 domain（MSDP 或 DRAG_UE）
- 正确设置事件级别（BEHAVIOR、FAULT、STATISTIC）
- 参数名称和类型与配置一致

**示例**：
```cpp
#include "hiappevent_ndk.h"

int32_t ret = HiAppEventWrite(HiAppEventDomainType::DOMAIN_MSDP,
    eventCode, "param1=value1", "param2=value2");
```

### 6. Dump 调试

#### Q: 如何查看服务状态？

**A**: 使用 `hidumper -s device_status` 命令。

**示例**：
```bash
# 查看服务整体状态
hidumper -s device_status

# 查看拖拽状态
hidumper -s device_status drag

# 查看协同状态
hidumper -s device_status coordinate
```

---

## 权限问题

### 7. 权限申请被拒绝

#### Q: 应用无法调用协同功能？

**A**: 检查应用是否具有 `ohos.permission.COOPERATE_MANAGER` 权限。

**常见原因**：
- 权限未在 `module.json` 中声明
- 应用签名问题
- Token 验证失败

**解决方法**：
1. 检查 `bundle.json` 中的 `syscap` 是否包含所需权限
2. 在应用 `module.json` 中声明所需权限
3. 使用 DevEco 工具检查权限配置

---

## 性能问题

### 8. 拖拽卡顿

#### Q: 拖拽操作不流畅？

**A**: 检查 `device_status_drag_enable_monitor` feature 是否启用，并查看性能检查日志。

**常见原因**：
- 主线程阻塞
- 动画渲染耗时过长
- 拖拽数据量过大
- 频览生成复杂度高

**调试方法**：
```cpp
// 性能检查已内置在代码中
#ifdef ENABLE_PERFORMANCE_CHECK
auto start = std::chrono::high_resolution_clock::now();
// ... 执行操作 ...
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
FI_HILOGI("Operation took: %{public}lld us", duration.count());
#endif
```

**优化建议**：
1. 异步化耗时操作
2. 减少拖拽数据大小
3. 优化预览算法
4. 使用硬件加速

---

## 工具使用

### 9. vdevadm 工具

#### Q: 如何测试虚拟设备？

**A**: 参考 `tools/vdev/` 目录实现。

**示例**：
```bash
# 创建虚拟鼠标
vdevadm create mouse

# 创建虚拟键盘
vdevadm create keyboard

# 列出设备
vdevadm list

# 删除设备
vdevadm delete <device_id>
```

---

## 相关文档

- **[01_Overview](01_Overview.md)** - 项目概览和安全模型
- **[02_Directory_Structure](02_Directory_Structure.md)** - 模块职责和边界
- **[03_Architecture](03_Architecture.md)** - 架构设计
- **[04_N-API_Reference](04_N-API_Reference.md)** - JavaScript API 参考
- **[06_GN_Targets](06_GN_Targets.md)** - GN 构建目标
- **[08_Security_Review](08_Security_Review.md)** - 完整安全评审

---

## 更新记录

- **初始版本**: 2026-02-06
- **代码版本**: HEAD commit of `/base/msdp/device_status`

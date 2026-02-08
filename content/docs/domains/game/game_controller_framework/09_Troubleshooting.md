# 09_Troubleshooting - 常见问题与定位路径

## 目的

本文档提供 GameController Framework 的常见构建、运行和调试问题的解决方案。

## 适用范围

- 开发人员
- 测试人员
- 运维人员

## 构建问题

### 问题 1: 编译失败 - 缺少依赖

**症状**:
```
ninja: error: 'target //domains/game/game_controller_framework/frameworks/native:gamecontroller_client depends on //domains/game/game_controller_framework/frameworks/native:game_controller_interface', which is not found
```

**原因**: IDL 文件未生成 Stub/Proxy 代码

**解决方案**:
1. 确保已安装 OpenHarmony 完整工具链
2. 清理构建输出：`rm -rf out/`
3. 重新编译：`./build.sh --product-name rk3568 --ccache --build-target game_controller_framework --build-variant root`

**证据**: `frameworks/native/BUILD.gn:18-22` - idl_gen_interface 依赖定义

---

### 问题 2: CAPI 找不到符号

**症状**:
```
ld: error: undefined reference to 'OH_GameDevice_RegisterDeviceMonitor'
```

**原因**: 应用链接时未正确链接 `libohgame_controller.z.so`

**解决方案**:
1. 确保在 BUILD.gn 中包含该库：`"//domains/game/game_controller_framework/interfaces/kits/c:ohgame_controller"`
2. 检查应用 CMakeLists.txt 或 Android.bp 中的链接配置
3. 使用 `objdump` 检查符号：`objdump -T libohgame_controller.z.so | grep OH_GameDevice_RegisterDeviceMonitor`

**证据**: `interfaces/kits/c/BUILD.gn:27-68` - ohgame_controller target 定义

---

### 问题 3: IPC 连接超时

**症状**:
```
[ERROR] [GameController] [IPC] Connect to GameControllerSA timeout
```

**原因**:
1. SA 未正常启动
2. SAMGR 服务异常
3. 进程权限问题

**定位步骤**:
```bash
# 1. 检查 SA 是否运行
hdc shell
# 进入设备 Shell
ps -ef | grep gamecontroller_server

# 2. 检查 SAMGR 日志
hdc shell hilog -x | grep GameController

# 3. 检查 SA 状态
hdc shell sa -l | grep 8450
```

**解决方案**:
1. 重启设备
2. 手动拉起 SA：`hdc shell sa start 8450`
3. 检查 SELinux 权限：`hdc shell ls -Z /dev/binder`

**证据**:
- `frameworks/native/sa_client/include/gamecontroller_server_client_proxy.h`: SA 连接逻辑
- `sa_profile/8450.json`: SA 配置

---

### 问题 4: 设备监听不触发

**症状**:
调用 `OH_GameDevice_RegisterDeviceMonitor(callback)` 后，设备连接时回调未触发

**定位步骤**:
```c
// 1. 检查回调注册返回值
GameController_ErrorCode ret = OH_GameDevice_RegisterDeviceMonitor(myCallback);
if (ret != GAME_CONTROLLER_SUCCESS) {
    printf("Register failed: %d\n", ret);
    // 处理错误
}

// 2. 检查 MMI 服务状态
hdc shell hilog -x | grep MultiModalInput

// 3. 检查设备是否被识别
hdc shell cat /data/el2/game_controller_gamepad/device_info
```

**常见原因**:
1. 回调函数签名不正确
2. MMI 服务未正常工作
3. 设备未被正确识别

**解决方案**:
1. 确认回调函数签名：`void (*GameDevice_DeviceMonitorCallback)(GameDevice_DeviceEvent event)`
2. 重启 MMI 服务：`hdc shell param set persist.multimodalinput.restart 1`
3. 检查设备配置文件：`cat /system/etc/game_controller_gamepad/device_config`

**证据**:
- `interfaces/kits/c/game_device.h:68`: 回调签名定义
- `frameworks/native/multi_modal_input/include/multi_modal_input_monitor.h`: MMI 监听器

---

### 问题 5: 输入事件未拦截

**症状**:
调用 `OH_GamePad_ButtonA_RegisterButtonInputMonitor(callback)` 后，按键事件未触发

**定位步骤**:
```bash
# 1. 检查 Window Framework 日志
hdc shell hilog -x | grep GameController

# 2. 检查输入事件流向
hdc shell hilog -x | grep Input

# 3. 验证输入拦截注册
# 检查 WindowInputIntercept 是否正确注册
```

**常见原因**:
1. Window Framework 未加载 `libgamecontroller_event.z.so`
2. 输入事件未正确注册到拦截器
3. 输入事件被其他组件消费

**解决方案**:
1. 检查 Window Framework 配置
2. 重启 Window Manager：`hdc shell killall -9 ohos.window`
3. 验证应用是否有权限监听输入

**证据**:
- `README_zh.md:81-84`: Window Framework dlopen 加载
- `frameworks/native/window/include/window_input_intercept.h`: 输入拦截接口

---

### 问题 6: 输入转触控不工作

**症状**:
按下键盘/鼠标按键时，屏幕上未出现触控事件

**定位步骤**:
```bash
# 1. 检查配置文件
hdc shell cat /system/etc/game_controller_gamepad/default_key_mapping.json
hdc shell cat /system/etc/game_controller_gamepad/game_support_key_mapping.json

# 2. 检查按键映射服务日志
hdc shell hilog -x | grep KeyMapping

# 3. 检查游戏是否支持转触控
hdc shell hilog -x | grep "IsSupportGameKeyMapping"
```

**常见原因**:
1. 游戏未在支持列表中（`game_support_key_mapping.json`）
2. 按键映射配置缺失或错误
3. 游戏窗口 ID 未正确设置

**解决方案**:
1. 添加游戏到支持列表（通过 InnerAPI）
2. 验证按键映射配置格式正确
3. 调用 InnerAPI 启用按键映射：`EnableGameKeyMapping(gameInfo, true)`

**证据**:
- `README_zh.md:52-55`: 转触控功能说明
- `frameworks/native/key_mapping/include/key_mapping_service.h`: IsSupportGameKeyMapping()

---

### 问题 7: SA 自动卸载问题

**症状**:
SA 在使用过程中频繁自动卸载

**原因**:
默认空闲超时为 30 秒，可能对某些场景过短

**解决方案**:
修改 `ability_event_handler.cpp` 中的卸载延迟：

```cpp
// 修改 /service/ipc/src/ability_event_handler.cpp
constexpr int32_t UNLOAD_DELAY_TIME = 60;  // 增加到 60 秒
```

重新编译 SA。

**证据**:
- `sa_profile/8450.json:10`: `"recycle-strategy": "low-memory"`
- `service/ipc/src/ability_event_handler.cpp`: 延迟卸载实现

---

### 问题 8: 内存泄漏

**症状**:
长期运行后内存持续增长

**定位步骤**:
```bash
# 1. 使用 Valgrind 检测泄漏
hdc shell valgrind --leak-check=full --show-leak-kinds=all /system/bin/gamecontroller_server

# 2. 使用 AddressSanitizer
./build.sh --product-name rk3568 --build-target game_controller_framework --variant root --sanitize=address

# 3. 检查代码中的手动内存管理
# 搜索 malloc、free、new/delete（不带智能指针）
```

**常见泄漏点**:
1. 回调函数未正确释放
2. 事件队列中对象未释放
3. JSON 解析结果未清理

**解决方案**:
1. 使用智能指针替代原始指针
2. 添加 RAII 资源管理
3. 使用内存分析工具定期检查

**证据**:
- `service/common/src/json_utils.cpp`: malloc/free 使用
- `frameworks/capi/src/game_device_proxy.cpp:29`: 内存分配检查

---

### 问题 9: 权限被拒绝

**症状**:
```
[ERROR] [GameController] [Permission] No system permissions for this operation
```

**原因**:
应用或服务没有足够的权限调用 InnerAPI

**定位步骤**:
```bash
# 1. 检查调用者 UID
hdc shell ps -ef | grep <app_name>

# 2. 检查权限声明
hdc shell bm dump -n <bundle_name> | grep requestPermissions

# 3. 检查访问令牌
hdc shell atm dump -a <pid>
```

**解决方案**:
1. 在应用的 `module.json5` 中添加必要的权限
2. 确保应用使用系统签名
3. 重新安装应用

**需要的权限**（根据权限检查代码）:
- `ohos.permission.GET_BUNDLE_INFO_PRIVILEGED`
- `ohos.permission.PUBLISH_SYSTEM_COMMON_EVENT`
- `ohos.permission.INTERACT_ACROSS_USERS_ACCOUNT`（如需要）

**证据**:
- `service/common/src/permission_utils.cpp`: IsSACall() 和 IsSystemAppCall()
- `etc/init/gamecontroller_server.cfg`: SA 权限配置

---

### 问题 10: JSON 配置文件损坏

**症状**:
```
[ERROR] [GameController] [Config] Failed to parse JSON file
```

**原因**:
配置文件格式错误或内容损坏

**解决方案**:
```bash
# 1. 备份当前配置
hdc shell cp /system/etc/game_controller_gamepad/default_key_mapping.json /sdcard/backup_default.json

# 2. 重置为默认配置
hdc shell rm /system/etc/game_controller_gamepad/custom_key_mapping.json

# 3. 使用 InnerAPI 恢复默认配置
# 通过终端厂商服务调用 SetDefaultGameKeyMappingConfig()
```

**预防措施**:
1. 定期备份配置文件
2. 使用 JSON Schema 验证配置
3. 添加配置版本管理

**证据**:
- `service/common/src/json_utils.cpp`: JSON 解析实现
- `README_zh.md:13-17`: 配置文件说明

---

### 问题 11: 插件加载失败

**症状**:
```
[ERROR] [GameController] [Plugin] Failed to load plugin library
```

**原因**:
插件库路径不正确或库未找到

**定位步骤**:
```bash
# 1. 检查插件库是否存在
hdc shell ls -l /system/lib/plugin_*.so

# 2. 检查加载路径
hdc shell cat /etc/plugin.conf

# 3. 检查插件库符号
hdc shell nm -D /system/lib/<plugin_name>.so
```

**解决方案**:
1. 确保插件库正确安装到系统
2. 检查加载路径配置（`PLUGIN_LIB_PATH`）
3. 验证插件库与当前架构兼容（ARM/ARM64）

**证据**:
- `frameworks/native/plugin/include/plugin_manager.h`: 插件管理器
- `frameworks/native/key_mapping/src/input_to_touch_client.cpp`: dlopen 加载插件

---

## 调试工具

### 日志系统

#### HiLog 日志查询

```bash
# 实时查看日志
hdc shell hilog -T

# 过滤特定模块
hdc shell hilog -x | grep GameController

# 保存日志到文件
hdc shell hilog -x > /sdcard/gamecontroller.log &

# 清空日志缓冲区
hdc shell hilog -r
```

#### 日志域和标签

| 日志域 | 十六进制值 | 说明 | 证据 |
|---------|-----------|------|--------|
| GameController | 0xD004732 | 游戏控制器日志 | `frameworks/native/BUILD.gn:20` |
| Input | 0xD003200 | 输入日志 | - |
| MMI | 0xD002800 | 多模态输入 | - |

**证据**:
- `frameworks/native/BUILD.gn:20-21`: 日志域定义

### 性能分析

#### 使用 Perfetto

```bash
# 1. 记录性能数据
hdc shell "cd /data/local/tmp && perfetto -o gamecontroller_trace.perfetto-trace -t 10s -b gamecontroller"

# 2. 导出跟踪文件
hdc file recv /data/local/tmp/gamecontroller_trace.perfetto-trace
```

#### CPU 和内存分析

```bash
# CPU 性能
hdc shell top -d 1 | grep gamecontroller

# 内存使用
hdc shell cat /proc/<pid>/status | grep -E 'VmRSS|VmSize'

# 进程树
hdc shell pstree -p <pid>
```

### 网络调试

#### Binder 通信调试

```bash
# 1. 检查 Binder 状态
hdc shell service list | find gamecontroller

# 2. Binder 调试日志
hdc shell hilog -x | grep -E "Binder|IPC"
```

## 常见错误码

| 错误码 | 十六进制 | 说明 | 触发场景 | 文件:行号 |
|---------|---------|------|---------|-----------|
| GAME_CONTROLLER_SUCCESS | 0 | 成功 | - | interfaces/kits/c/game_controller_type.h |
| GAME_CONTROLLER_PARAM_ERROR | 401 | 参数错误（null、超出范围）| API 调用时参数无效 | interfaces/kits/c/game_controller_type.h |
| GAME_CONTROLLER_MULTIMODAL_INPUT_ERROR | 32200001 | 多模态输入服务异常 | MMI 服务异常 | interfaces/kits/c/game_controller_type.h |
| GAME_CONTROLLER_NO_MEMORY | 32200002 | 内存不足 | 内存分配失败 | interfaces/kits/c/game_controller_type.h |
| GAME_ERR_NO_SYS_PERMISSIONS | 32200010 | 无系统权限 | InnerAPI 调用未授权 | frameworks/native/common/include/gamecontroller_errors.h: |
| GAME_ERR_BUNDLE_NAME_INVALID | 32200008 | Bundle 名称无效 | Bundle 名称验证失败 | frameworks/native/common/include/gamecontroller_errors.h |
| GAME_ERR_ARGUMENT_INVALID | 32200011 | 参数无效 | 参数校验失败 | frameworks/native/common/include/gamecontroller_errors.h |
| GAME_ERR_ARGUMENT_NULL | 32200002 | 参数为 null | Null 检查失败 | frameworks/native/common/include/gamecontroller_errors.h |
| GAME_ERR_ARRAY_MAXSIZE | 32200009 | 数组大小超限 | 数组越界 | frameworks/native/common/include/gamecontroller_errors.h |
| GAME_ERR_IPC_CONNECT_STUB_FAIL | 32200003 | IPC 连接失败 | SA 连接异常 | frameworks/native/common/include/gamecontroller_errors.h |
| GAME_ERR_TIMEOUT | 32200004 | 超时 | IPC 或操作超时 | frameworks/native/common/include/gamecontroller_errors.h |

**证据**:
- `interfaces/kits/c/game_controller_type.h`: 错误码枚举定义
- `frameworks/native/common/include/gamecontroller_errors.h`: 服务错误码定义

## 关键结论

1. **日志优先**: 使用 HiLog 和标签过滤快速定位问题
2. **权限检查**: 确保应用和 SA 有足够权限
3. **配置验证**: 使用工具验证 JSON 配置格式
4. **内存调试**: 使用 Valgrind/AddressSanitizer 检测泄漏
5. **性能分析**: 使用 Perfetto 和 top/ps 分析性能

## 相关文档

- [00_Overview.md](./00_Overview.md) - 快速概览
- [04_External_CAPI.md](./04_External_CAPI.md) - CAPI 错误码
- [05_Inner_API.md](./05_Inner_API.md) - InnerAPI 错误码

---

**版本**: 1.0 | **更新时间**: 2026-02-06

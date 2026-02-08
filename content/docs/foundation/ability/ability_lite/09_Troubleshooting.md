# 常见问题排查

## 目的

本文档提供 ability_lite 常见构建、运行和调试问题的排查方法。

## 适用范围

- 遇到问题的开发者
- 需要调试 ability_lite 的维护人员

## 构建问题

### 问题 1: 编译失败，找不到头文件

**症状**:
```
fatal error: 'ability.h' file not found
```

**排查路径**:
1. 检查 `ability_lite.gni` 中路径定义是否正确
2. 确认依赖组件已编译
3. 检查 BUILD.gn 中 `include_dirs` 配置

**代码检查点**:
- `ability_lite.gni:18` - 检查 `ability_lite_path` 定义
- `frameworks/ability_lite/BUILD.gn:101` - 检查 include_dirs

**解决方案**:
```bash
# 确保依赖组件已编译
ninja -C out bundle_framework_lite hilog_lite samgr_lite

# 重新编译 ability_lite
ninja -C out aafwk_abilitykit_lite
```

### 问题 2: 链接错误，未定义符号

**症状**:
```
undefined reference to `StartAbility'
```

**排查路径**:
1. 检查是否正确链接 `libabilitymanager.so`
2. 检查函数签名是否匹配
3. 确认 C/C++ 混用时的 extern "C"

**代码检查点**:
- `interfaces/kits/ability_lite/ability_manager.h:46-50` - 检查 extern "C"
- 链接命令中的 `-labilitymanager`

**解决方案**:
```gn
# 在 BUILD.gn 中添加依赖
deps += [
    "${aafwk_lite_path}/frameworks/abilitymgr_lite:abilitymanager",
]
```

### 问题 3: Feature 标志不生效

**症状**:
Page Ability 相关代码未编译

**排查路径**:
1. 检查 `ability_lite_enable_ohos_appexecfwk_feature_ability` 设置
2. 检查 bundle.json 中 features 定义
3. 确认 args.gn 中配置

**代码检查点**:
- `bundle.json:16` - features 定义
- `frameworks/ability_lite/BUILD.gn:80` - 条件编译检查

**解决方案**:
```bash
# 在 args.gn 中启用
echo "ability_lite_enable_ohos_appexecfwk_feature_ability = true" >> out/args.gn

# 重新生成
gn gen out
ninja -C out
```

## 运行时问题

### 问题 4: Ability 启动失败

**症状**:
```
StartAbility failed, ret = -1
```

**排查路径**:
1. 检查 AMS 服务是否运行
2. 检查 Want 参数是否正确
3. 查看日志获取详细错误

**调试代码**:
```cpp
// 在 ability_mgr_feature.cpp:147 添加日志
PRINTI("AbilityMgrFeature", "StartAbilityInvoke called, uid=%d", uid);

// 在 app_manager.cpp 添加日志
PRINTI("AppManager", "StartAbility bundleName=%s", want->element->bundleName);
```

**日志检查**:
```bash
# 查看 AMS 日志
hilog | grep AbilityManagerService
hilog | grep AbilityMgrFeature

# 查看应用日志
hilog | grep Ability
```

**常见原因**:
| 错误码 | 原因 | 解决 |
|--------|------|------|
| PARAM_NULL_ERROR (1) | 参数为空 | 检查 Want 初始化 |
| PARAM_CHECK_ERROR (9) | 参数校验失败 | 检查 bundleName/abilityName |
| MEMORY_MALLOC_ERROR (2) | 内存不足 | 检查系统内存 |

### 问题 5: Service Ability 连接失败

**症状**:
```
ConnectAbility failed
```

**排查路径**:
1. 检查 Service Ability 是否已注册
2. 检查 IAbilityConnection 回调是否设置
3. 检查权限

**调试代码**:
```cpp
// 在 ability_mgr_feature.cpp:361 添加日志
PRINTI("AbilityMgrFeature", "ConnectAbilityInvoke uid=%d", uid);

// 检查回调设置
if (conn == nullptr || conn->OnAbilityConnectDone == nullptr) {
    PRINTE("AbilityMgrFeature", "Invalid connection callback");
}
```

### 问题 6: JS 调用 N-API 失败

**症状**:
```
Error: Cannot find module '@ohos.aafwk'
```

**排查路径**:
1. 检查 `libaafwk.so` 是否安装到 `/system/lib/module/`
2. 检查 N-API 模块是否正确注册
3. 检查 JS 导入路径

**代码检查点**:
- `interfaces/kits/js/napi/js_aafwk.cpp:176` - 模块名是否为 "aafwk"
- `interfaces/kits/js/napi/BUILD.gn:40` - relative_install_dir 是否为 "module"

**解决方案**:
```bash
# 确认模块已安装
ls /system/lib/module/libaafwk.so

# 重新安装
ninja -C out //foundation/ability/ability_lite/interfaces/kits/js/napi:aafwk
```

### 问题 7: 权限加载失败

**症状**:
```
load application permission ret = X
```

**排查路径**:
1. 检查 permission_lite 服务是否运行
2. 检查应用权限配置文件
3. 检查 UID 是否正确分配

**代码检查点**:
- `services/abilitymgr_lite/src/app_record.cpp:60` - LoadPermissions 调用

**调试代码**:
```cpp
// 在 app_record.cpp:58 添加日志
AbilityMsStatus AppRecord::LoadPermission() const
{
    PRINTI("AppRecord", "Loading permission for %s, uid=%d", 
           bundleInfo_.bundleName, bundleInfo_.uid);
    int ret = LoadPermissions(bundleInfo_.bundleName, bundleInfo_.uid);
    PRINTI("AppRecord", "LoadPermissions ret=%d", ret);
    // ...
}
```

## 调试方法

### 启用详细日志

**代码位置**: 各模块日志宏

```cpp
// 在 abilityms_log.h 中定义日志级别
#define PRINTI(tag, fmt, ...) HILOG_INFO(HILOG_MODULE_APP, fmt, ##__VA_ARGS__)
#define PRINTE(tag, fmt, ...) HILOG_ERROR(HILOG_MODULE_APP, fmt, ##__VA_ARGS__)
```

**运行时启用**:
```bash
# 设置日志级别
hilog -b I  # Info 级别
hilog -b D  # Debug 级别
```

### 使用 aa 工具调试

**工具位置**: `services/abilitymgr_lite/tools/`

```bash
# 启动 Ability
aa start -p com.example.app -n MainAbility

# 停止 Ability
aa stop -p com.example.app -n ServiceAbility

# Dump AMS 状态
aa dump
```

### GDB 调试

```bash
# 附加到 AMS 进程
gdb-pid $(pidof abilityms)

# 设置断点
break AbilityMgrFeature::StartAbilityInvoke
break AppManager::StartAbility

# 运行
continue
```

### 跟踪 IPC 调用

```bash
# 使用 strace 跟踪 IPC
strace -e trace=sendmsg,recvmsg -p $(pidof abilityms)

# 查看 IPC 日志
hilog | grep -E "(SendRequest|Invoke)"
```

## 性能问题

### 问题 8: Ability 启动慢

**排查路径**:
1. 检查 AppSpawn 响应时间
2. 检查 Bundle 信息查询时间
3. 检查权限加载时间

**性能分析点**:
```cpp
// 在关键路径添加时间统计
#include <sys/time.h>

struct timeval start, end;
gettimeofday(&start, nullptr);
// ... 操作 ...
gettimeofday(&end, nullptr);
long elapsed = (end.tv_sec - start.tv_sec) * 1000 + 
               (end.tv_usec - start.tv_usec) / 1000;
PRINTI("Performance", "Operation took %ld ms", elapsed);
```

**常见瓶颈**:
| 阶段 | 可能原因 | 优化建议 |
|------|----------|----------|
| AppSpawn | 进程创建慢 | 预加载常用库 |
| Bundle 查询 | 磁盘 IO | 缓存 Bundle 信息 |
| 权限加载 | PMS 响应慢 | 异步加载权限 |

## 内存问题

### 问题 9: 内存泄漏

**排查工具**:
```bash
# 使用 valgrind
valgrind --leak-check=full --show-leak-kinds=all ./abilityms

# 查看内存使用
cat /proc/$(pidof abilityms)/status | grep VmRSS
```

**常见泄漏点**:
- `js_aafwk.cpp:146` - malloc 未释放
- `ability_record.cpp` - AbilityRecord 未正确销毁
- `want.cpp` - Want 数据未 ClearWant

## 相关链接

- [架构说明](02_Architecture.md)
- [对外 Native API](03_Native_API.md)
- [安全评估](08_Security_Assessment.md)

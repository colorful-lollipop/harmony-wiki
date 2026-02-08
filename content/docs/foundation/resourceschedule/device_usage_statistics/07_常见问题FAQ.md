# 常见问题 FAQ

## 目的

本文档提供 device_usage_statistics 组件的常见构建、运行、调试问题和定位路径。

## 适用范围

- 构建相关问题
- 运行时问题
- 调试技巧
- 性能问题

---

## 构建问题

### Q1: 编译时提示找不到 IDL 生成的头文件

**现象**:
```
error: 'ibundle_active_service_ipc_interface_code.h' file not found
```

**原因**: IDL 接口生成代码位于 out 目录，未正确引用

**解决方法**:
1. 确保 `BUILD.gn` 中正确配置 `idl_gen_interface` target
2. 检查 `include_dirs` 是否包含 `${target_gen_dir}`
3. 重新编译：`gn gen out/default` && `ninja -C out/default`

**证据**:
- BUILD.gn:35-44: usagestats_public_config 定义

---

### Q2: 编译时提示权限相关头文件未找到

**现象**:
```
error: 'accesstoken_kit.h' file not found
```

**原因**: `access_token` 组件未启用

**解决方法**:
1. 检查 `device_usage_statistics.gni` 中的依赖配置
2. 确认 bundle.json 中包含 `access_token` 依赖
3. 如果确实不需要 access_token，使用 Feature Flag 关闭

**证据**:
- bundle.json:31: access_token 依赖
- device_usage_statistics.gni: Feature Flag 定义

---

### Q3: 编译产物体积过大

**现象**:
生成的 `.so` 文件体积超过预期

**原因**:
1. 包含调试符号
2. 未启用优化选项
3. 包含不必要的依赖

**解决方法**:
1. 检查 `cflags` 是否包含 `-Os` 优化选项
2. 确认 `-fvisibility=hidden` 隐藏内部符号
3. 使用 `strip` 工具去除调试符号

**证据**:
- BUILD.gn:124-128, 170-175: cflags 配置

---

## 运行时问题

### Q4: SA 1907 启动失败

**现象**:
系统日志提示 "SA 1907 start failed"

**原因**:
1. 数据库初始化失败
2. 依赖的 SystemAbility 未就绪
3. 配置文件加载失败

**定位方法**:
```bash
# 查看系统日志
hdc shell hilog -t BUNDLE_ACTIVE

# 查看 SA 状态
hdc shell sa list
```

**解决方法**:
1. 检查数据库路径权限
2. 确认依赖的 SA 已启动
3. 检查配置文件格式

**证据**:
- services/common/src/bundle_active_service.cpp:122-127: OnStart 实现

---

### Q5: N-API 调用返回权限被拒绝

**现象**:
```typescript
queryBundleEvents(begin, end).catch(err => {
    console.error(err.code); // 201: ERR_PERMISSION_DENIED
})
```

**原因**:
1. 应用未授予 BUNDLE_ACTIVE_INFO 权限
2. 非系统应用尝试调用敏感接口
3. Token ID 验证失败

**解决方法**:
1. 在 config.json 中添加权限声明：
```json
{
  "module": {
    "requestPermissions": [
      {"name": "ohos.permission.BUNDLE_ACTIVE_INFO"}
    ]
  }
}
```
2. 确认应用签名正确
3. 对于 setAppGroup 等敏感接口，需要系统应用

**证据**:
- services/common/src/bundle_active_service.cpp:717-766: 权限检查实现
- 03_N-API接口文档.md: 权限要求章节

---

### Q6: 数据库查询超时

**现象**:
N-API 调用长时间无响应

**原因**:
1. 数据库文件损坏
2. 大量数据导致查询缓慢
3. 事务阻塞

**定位方法**:
```bash
# 查看数据库日志
hdc shell hilog -t BUNDLE_ACTIVE | grep database

# 检查数据库文件
hdc shell ls -l /data/service/el2/database/bundle_active/
```

**解决方法**:
1. 删除或重建数据库文件
2. 检查磁盘空间是否充足
3. 优化查询条件，减少查询范围

**证据**:
- services/common/src/bundle_active_usage_database.cpp: 数据库操作实现

---

## 调试技巧

### Q7: 如何启用调试日志

**方法**:
1. 在 `bundle_active_debug_mode.h` 中设置调试宏
2. 重新编译并部署
3. 使用 `hilog -t BUNDLE_ACTIVE` 查看日志

**日志等级**:
```cpp
BUNDLE_ACTIVE_LOGI()  // INFO
BUNDLE_ACTIVE_LOGD()  // DEBUG
BUNDLE_ACTIVE_LOGW()  // WARNING
BUNDLE_ACTIVE_LOGE()  // ERROR
```

**证据**:
- services/common/src/bundle_active_log.cpp: 日志工具实现

---

### Q8: 如何使用 Dump 接口调试

**方法**:
```bash
# 连接到设备
hdc shell

# 使用 dump 命令
dump -h 1907
dump -h 1907 dump_events 0 999999999999 -1
```

**Dump 参数**:
- `dump_events`: 导出事件
- `dump_groups`: 导出分组
- `dump_usage`: 导出使用统计

**证据**:
- services/common/src/bundle_active_service.cpp:910-1007: Dump 接口实现

---

### Q9: 如何跟踪 N-API 调用链

**方法**:
1. 在 NAPI 实现函数入口添加日志
2. 在 IPC 调用前后添加日志
3. 在服务端接口添加日志

**示例**:
```cpp
// frameworks/src/bundle_state_query_napi.cpp
napi_value QueryBundleEvents(napi_env env, napi_callback_info info) {
    BUNDLE_ACTIVE_LOGI("QueryBundleEvents called");
    // ... 参数解析
    BUNDLE_ACTIVE_LOGI("Calling IPC: begin=%{public}s, end=%{public}s",
        to_string(beginTime).c_str(), to_string(endTime).c_str());
    ErrCode errorCode = BundleActiveClient::GetInstance().QueryBundleEvents(...);
    BUNDLE_ACTIVE_LOGI("IPC returned: errorCode=%{public}d", errorCode);
    // ...
}
```

**证据**:
- frameworks/src/bundle_state_query_napi.cpp: NAPI 实现
- interfaces/innerkits/src/bundle_active_client.cpp: IPC 调用

---

## 性能问题

### Q10: 查询性能慢

**可能原因**:
1. 数据库文件过大
2. 未使用合适的查询条件
3. 索引缺失

**优化建议**:
1. 缩短查询时间范围
2. 使用 `queryBundleStatsInfoByInterval` 按间隔统计
3. 定期清理旧数据

**证据**:
- README_zh.md:120-123: 数据持久化时机

---

### Q11: 内存占用过高

**可能原因**:
1. 缓存过多事件数据
2. 未及时释放资源
3. 数据库连接未关闭

**定位方法**:
```bash
# 查看进程内存占用
hdc shell ps -A | grep device_usage

# 使用 dump 命令查看状态
dump -h 1907 dump_status
```

**解决方法**:
1. 检查事件上报频率
2. 确认数据库连接正确关闭
3. 调整刷新间隔（默认 30 分钟）

**证据**:
- services/common/src/bundle_active_usage_database.cpp:501-576: FlushData 实现

---

## 开发问题

### Q12: 如何添加新的 N-API 接口

**步骤**:
1. 在 IDL 文件中添加接口定义
2. 在 BundleActiveService 中实现接口
3. 在 bundle_active_client 中添加客户端调用
4. 在 N-API 文件中实现参数解析和异步工作
5. 在 usage_statistics_init.cpp 中注册新方法
6. 添加错误码定义（如需要）

**示例**:
```cpp
// 1. IDL 定义
void MyNewInterface([out] MyResult result, [in] MyParam param);

// 2. 服务端实现
ErrCode BundleActiveService::MyNewInterface(MyResult &result, const MyParam &param) {
    // 实现逻辑
}

// 3. N-API 实现
napi_value MyNewInterface(napi_env env, napi_callback_info info) {
    // 参数解析
    // 异步工作
    // 返回 Promise
}

// 4. 注册
static napi_property_descriptor desc[] = {
    // ...
    DECLARE_NAPI_FUNCTION("myNewInterface", MyNewInterface),
};
```

**证据**:
- IBundleActiveService.idl: IDL 接口定义
- frameworks/src/usage_statistics_init.cpp:33-62: 方法注册

---

### Q13: 如何测试 N-API 接口

**方法**:
1. 编写 JS 测试代码
2. 使用 ace-test 或 xvfs-test 运行
3. 使用 hdc shell 直接测试

**示例**:
```javascript
// 测试 queryBundleEvents
import bundleState from '@ohos.resourceschedule.usageStatistics';

try {
    const events = await bundleState.queryBundleEvents(
        Date.now() - 86400000,  // 24小时前
        Date.now()
    );
    console.log('Events:', events);
} catch (err) {
    console.error('Error:', err.code, err.message);
}
```

**证据**:
- interfaces/test/unittest/device_usage_statistics_jsunittest: 单元测试

---

## 相关跳转链接

- [00_项目概览.md](00_项目概览.md) - 了解项目定位
- [03_N-API接口文档.md](03_N-API接口文档.md) - 查看接口详情
- [05_GN构建系统.md](05_GN构建系统.md) - 了解构建配置
- [06_安全风险评审.md](06_安全风险评审.md) - 了解安全机制

---

## 版本信息

- **生成时间**: 2026-02-06
- **文档版本**: v1.0

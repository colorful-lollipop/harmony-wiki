# 常见问题

## Q1: 如何添加新的 N-API 模块？

**步骤**:
1. 在 `frameworks/js/` 下创建新目录
2. 实现 `napi_define_properties` 和 `napi_create_function`
3. 在 `native_module_ohos_media.cpp` 中注册

**参考代码**:
```cpp
// 文件: frameworks/js/media/native_module_ohos_media.cpp
static napi_value Export(napi_env env, napi_value exports)
{
    // 新模块注册
    OHOS::Media::NewModuleNapi::Init(env, exports);
    return exports;
}
```

## Q2: 如何添加新的权限检查？

**步骤**:
1. 在 `services/utils/media_permission.cpp` 中添加检查函数
2. 在服务端调用权限检查
3. 返回错误码

**参考代码**:
```cpp
// 文件: services/utils/media_permission.cpp
int32_t MediaPermission::CheckCustomPermission()
{
    auto callerUid = IPCSkeleton::GetCallingUid();
    Security::AccessToken::AccessTokenID tokenCaller = IPCSkeleton::GetCallingTokenID();
    return Security::AccessToken::AccessTokenKit::VerifyAccessToken(
        tokenCaller, "ohos.permission.CUSTOM_PERMISSION");
}
```

## Q3: 如何调试 IPC 调用？

**步骤**:
1. 开启日志：`hilog` 级别设为 Debug
2. 查看调用链：`hilog | grep "OnRemoteRequest"`
3. 检查参数序列化

**日志位置**:
```cpp
// 文件: services/services/player/ipc/player_service_stub.cpp:314
MEDIA_LOGD("Stub: OnRemoteRequest task: %{public}s is received", taskName);
```

## Q4: 如何处理跨引擎切换？

**说明**:
- HiStreamer 和 LPP 引擎通过工厂模式切换
- 根据场景选择合适引擎

**切换逻辑**:
```cpp
// 文件: services/services/factory/engine_factory_repo.cpp
std::shared_ptr<PlayerEngine> EngineFactoryRepo::CreatePlayerEngine()
{
    if (lowPowerMode) {
        return std::make_shared<LppPlayerEngine>();
    }
    return std::make_shared<HiPlayerEngine>();
}
```

## Q5: 权限被拒如何排查？

**检查项**:
1. `config.json` 中是否声明权限
2. 用户是否授权
3. 权限级别是否匹配

**排查命令**:
```bash
# 查看应用权限
hdc shell dumpsys permission <bundle_name>

# 查看权限定义
hdc shell cat /system/etc/permissions/
```

## 相关文档

- [故障排查指南](appendix/Troubleshooting.md)
- [安全风险评审](15_Security_Review.md)

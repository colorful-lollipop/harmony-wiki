# 常见构建、运行与调试问题

## 目的

本文档收集 DSLM 模块的常见问题、定位路径和解决方案。

## 适用范围

- ✅ 常见构建问题
- ✅ 常见运行时问题
- ✅ 调试方法与工具

## 构建问题

### 问题 1：编译失败 - 找不到头文件

**症状**：
```
error: 'idevice_security_level.h' file not found
```

**原因**：include_dirs 配置不完整。

**定位路径**：检查 BUILD.gn 的 include_dirs 配置。

**解决方案**：
1. 确保所有模块的 BUILD.gn 都包含 `../../common/include` 和 `../../interfaces/inner_api/include`
2. 参考 `services/sa/BUILD.gn:169-175` 的 include_dirs 配置

---

### 问题 2：链接错误 - 未定义符号

**症状**：
```
undefined reference to 'RequestDeviceSecurityInfo'
```

**原因**：external_deps 中缺少 `device_security_level:dslm_sdk`。

**定位路径**：检查调用方的 BUILD.gn。

**解决方案**：
```gn
external_deps += [ "device_security_level:dslm_sdk" ]
```

---

### 问题 3：Standard 版本构建失败 - CFI 相关错误

**症状**：
```
error: CFI violation detected
```

**原因**：代码存在不符合 CFI（Control Flow Integrity）规范的跳转。

**定位路径**：检查报错位置和 cfi_blocklist.txt。

**解决方案**：
1. 查看 `cfi_blocklist.txt` 中的黑名单配置
2. 确保代码不包含被黑名单的跳转
3. 如确实需要，在 cfi_blocklist.txt 中添加例外

---

### 问题 4：Lite 系统编译失败 - SAMGR 相关错误

**症状**：
```
error: 'samgr' not found
```

**原因**：Lite 系统的 dslm_lite_component_path 配置不正确。

**定位路径**：检查 `common/dslm.gni` 中的路径变量。

**解决方案**：
确保 `common/dslm.gni` 中定义了正确的路径：
```gn
dslm_samgr_path = "//foundation/systemabilitymgr"
```

---

### 问题 5：Feature flags 未生效

**症状**：配置的 feature flags 未生效。

**原因**：feature flags 在错误的 BUILD.gn 中声明。

**定位路径**：检查 `services/sa/BUILD.gn`、`services/msg/BUILD.gn` 等文件。

**解决方案**：
1. 确保 feature flags 在正确的 BUILD.gn 的 `declare_args()` 中声明
2. 参考 `bundle.json` 中的 features 列表：
```json
"features": [
    "device_security_level_feature_cred_level",
    "device_security_level_feature_plugin_path",
    "device_security_level_feature_secondary_session_name"
]
```

## 运行时问题

### 问题 1：SA 3511 启动失败

**症状**：
```
DSL service failed to start
```

**原因**：依赖的 SA（4700、3510）未就绪。

**定位路径**：
1. 检查 SA 配置：`profile/dslm_service.xml:21`
2. 使用 hidumper 检查依赖 SA 状态：
```bash
hidumper -s 4700
hidumper -s 3510
```

**解决方案**：
1. 确保 DeviceManager (4700) 和 Huks (3510) 已启动
2. 检查依赖超时配置（60000ms）
3. 查看系统日志：
```bash
hilog -T DSLM | grep "service"
```

---

### 问题 2：查询设备安全等级超时

**症状**：
```
RequestDeviceSecurityInfo() timeout
```

**原因**：目标设备离线或网络问题。

**定位路径**：
1. 检查目标设备在线状态
2. 检查 DSoftBus 连接状态

**解决方案**：
1. 使用 `RequestOption` 的 timeout 参数设置合理的超时时间
2. 实现重试逻辑
3. 使用异步接口 `RequestDeviceSecurityInfoAsync()` 避免阻塞

---

### 问题 3：设备凭据验证失败

**症状**：
```
VerifyDslmCred() failed: ERR_VERIFY_MODE_CRED_ERR
```

**原因**：证书链验证失败或签名验证失败。

**定位路径**：
1. 查看系统日志：
```bash
hilog -T DSLM | grep -i "verify\|cert\|sign"
```

**解决方案**：
1. 检查目标设备的凭据是否正确
2. 检查 Huks 是否正确存储根密钥
3. 确认 Challenge-Nonce 是否匹配

---

### 问题 4：内存泄漏

**症状**：长时间运行后内存占用持续增长。

**原因**：设备列表未正确清理离线设备。

**定位路径**：
1. 使用内存分析工具：ASan、Valgrind
2. 查看 DFX 大数据统计

**解决方案**：
1. 确保 `DelDslmDeviceInfo()` 正确释放内存
2. 使用智能指针（Standard 版本）
3. 添加超时清理机制

---

### 问题 5：回调未被触发

**症状**：
```
RequestDeviceSecurityInfoAsync() callback never called
```

**原因**：SA 卸载或进程崩溃。

**定位路径**：
1. 检查 SA 状态：
```bash
hidumper -s 3511
```

**解决方案**：
1. 检查 SA 是否被自动卸载（10 秒无请求）
2. 查看 DSLM 服务日志：
```bash
hilog -T DSLM | grep "unload\|crash"
```

## 调试方法

### 1. HiDumper

**用途**：查看 DSLM 服务的运行状态和设备列表。

**命令**：
```bash
# 查看服务状态
hidumper -s 3511

# 查看所有设备
hidumper -s 3511 -l

# 查看帮助
hidumper -s 3511 -h
```

**输出示例**：
```
Dslm Dump:
Device List:
  Device 1: [UDID], Level: SL3, Online: true
  Device 2: [UDID], Level: SL2, Online: false
```

---

### 2. HiSysEvent

**用途**：查看 DSLM 的系统事件。

**配置文件**：`hisysevent.yaml`

**查询事件**：
```bash
# 查看所有 DSLM 事件
hilog -T DSLM | grep "EVENT"

# 过滤特定事件类型
hilog -T DSLM | grep "DEVICE_ONLINE\|DEVICE_OFFLINE"
```

---

### 3. HiTrace

**用途**：性能追踪和分析。

**证据**：`services/dfx/dslm_hitrace.cpp`

**使用方法**：
```cpp
// 在代码中添加追踪点
StartTrace("DSLMP", "QueryDeviceLevel");
// ... 业务逻辑
FinishTrace("DSLMP", "QueryDeviceLevel");
```

**查看追踪**：
```bash
# 查看 HiTrace 报告
# 具体命令依赖 HiTrace 工具
```

---

### 4. 系统日志

**用途**：查看详细日志信息。

**命令**：
```bash
# 查看所有 DSLM 日志
hilog -T DSLM

# 过滤特定级别
hilog -T DSLM | grep "ERROR\|WARN"

# 实时监控
hilog -T DSLM -v
```

**日志标签**：
- `DSLMP` - 主要日志标签
- `DSLM` - 服务日志标签

---

### 5. DFX 大数据

**用途**：查看性能指标和统计数据。

**证据**：`services/dfx/dslm_bigdata.cpp`

**查询方法**：
```bash
# 查看 DSLM 性能数据
# 具体命令依赖 DFX 大数据工具
```

---

### 6. 内存泄漏检测

**编译选项**：启用 Address Sanitizer (ASan)

**修改 BUILD.gn**：
```gn
ohos_shared_library("dslm_service") {
    # ... 现有配置
    configs += [ ":use_asan" ]
}

config("use_asan") {
    cflags = [ "-fsanitize=address" ]
}
```

**运行时检测**：ASan 会自动检测内存泄漏、释放后使用等错误。

---

### 7. 线程分析

**工具**：
- **GDB**：调试多线程问题
- **LLDB**：LLVM 调试器
- **ThreadSanitizer (TSan)**：检测数据竞态

**启用 TSan**：
```gn
ohos_shared_library("dslm_service") {
    # ... 现有配置
    configs += [ ":use_tsan" ]
}

config("use_tsan") {
    cflags = [ "-fsanitize=thread" ]
}
```

## 常见错误码

| 错误码 | 值 | 说明 | 可能原因 | 解决方案 |
|---------|-----|------|----------|----------|
| `SUCCESS` | 0 | 成功 | - |
| `ERR_INVALID_PARA` | 1 | 参数无效 | 检查参数 |
| `ERR_INVALID_LEN_PARA` | 2 | 长度无效 | 检查设备标识符长度 |
| `ERR_NO_MEMORY` | 3 | 内存不足 | 检查系统内存 |
| `ERR_IPC_ERR` | 17 | IPC 错误 | 检查 SA 状态 |
| `ERR_PERMISSION_DENIAL` | 30 | 权限拒绝 | 检查调用者权限 |
| `ERR_TIMEOUT` | 8 | 超时 | 增加超时时间 |
| `ERR_NOT_ONLINE` | 14 | 设备不在线 | 检查设备状态 |
| `ERR_VERIFY_MODE_CRED_ERR` | 32 | 凭据验证失败 | 检查证书和签名 |

## 性能优化建议

1. **减少重复查询**：缓存设备安全等级，避免重复验证凭据
2. **异步优先**：使用异步接口 `RequestDeviceSecurityInfoAsync()`，避免阻塞
3. **合理超时**：根据网络情况设置合适的 timeout
4. **及时清理**：定期清理离线设备的缓存

## 关键结论

1. **调试工具**：HiDumper（状态查看）、HiSysEvent（事件）、HiTrace（性能追踪）、系统日志
2. **常见问题**：SA 启动、查询超时、凭据验证失败、内存泄漏、回调未触发
3. **定位方法**：查看日志、使用 HiDumper、启用 Sanitizer、性能追踪

## 相关跳转

- [03_Architecture.md](./03_Architecture.md) - 系统架构
- [08_Security_Review.md](./08_Security_Review.md) - 安全风险评审

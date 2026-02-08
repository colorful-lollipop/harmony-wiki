# 07 - 常见问题与定位

## 目的与适用范围

**本文档目的**：提供 `device_attest` 模块常见问题的定位方法和解决方案。

**适用范围**：
- 应用开发者
- 系统开发者
- 测试人员
- 运维人员

## 错误码速查

### JS 层错误码

| 错误码 | 常量定义 | 说明 | 解决方案 |
|--------|----------|------|----------|
| 202 | `DEVATTEST_ERR_JS_IS_NOT_SYSTEM_APP` | 非系统应用 | 确保应用有系统签名 |
| 401 | `DEVATTEST_ERR_JS_PARAMETER_ERROR` | 参数错误 | 检查参数类型和数量 |
| 20000001 | `DEVATTEST_ERR_JS_SYSTEM_SERVICE_EXCEPTION` | 系统服务异常 | 重启设备或检查日志 |

### 原生层错误码

| 错误码 | 常量定义 | 说明 |
|--------|----------|------|
| -2 | `DEVATTEST_INIT` | 初始化中 |
| -1 | `DEVATTEST_FAIL` | 通用失败 |
| 0 | `DEVATTEST_SUCCESS` | 成功 |
| 0x10001 | `DEVATTEST_SERVICE_FAILED` | 服务失败 |
| 0x10002 | `DEVATTEST_WRITE_FAIL` | 写入失败 |
| 0x10003 | `DEVATTEST_PARAM_NULL` | 参数为空 |
| 0x10004 | `DEVATTEST_SA_NO_INIT` | SA 未初始化 |

## 常见问题

### Q1: 调用 getAttestStatus 返回错误码 202

**现象**:
```javascript
// 错误信息
{"code": 202, "message": "This api is system api, Please use the system application to call this api"}
```

**原因**: 调用应用不是系统应用

**定位路径**:
1. 检查应用签名
   ```bash
   # 查看应用签名信息
   hdc shell bm dump -n <bundle_name>
   ```

2. 检查权限检查日志
   ```bash
   hdc shell hilog | grep -i "not a system"
   ```
   日志位置：`services/devattest_ability/src/devattest_service_stub.cpp:57`

3. 检查 Token 类型
   ```bash
   hdc shell hilog | grep -i "check permission"
   ```
   日志位置：`common/permission/src/permission.cpp:42`

**解决方案**:
- 使用系统签名重新打包应用
- 在 `Install_list_permissions.json` 中配置为系统应用

### Q2: 认证状态一直返回 0（未认证）

**现象**:
```javascript
{
    authResult: 0,  // 未认证
    softwareResult: 0,
    ticket: ""
}
```

**可能原因**:
1. 设备首次启动，认证流程尚未执行
2. 网络未连接
3. 设备信息配置错误
4. Token 读取失败

**定位路径**:

1. 检查服务是否启动
   ```bash
   hdc shell ps -e | grep devattest
   ```

2. 检查服务日志
   ```bash
   hdc shell hilog | grep -i "DevAttest"
   ```

3. 检查网络状态
   ```bash
   hdc shell hilog | grep -i "network"
   ```
   相关代码：`services/devattest_ability/src/devattest_network_manager.cpp`

4. 检查 Token 读取
   ```bash
   hdc shell hilog | grep -i "token"
   ```
   相关代码：`services/core/adapter/attest_adapter_hal.c`

5. 检查设备信息读取
   ```bash
   hdc shell param get | grep -E "(product|ohos)"
   ```

**解决方案**:
- 确保网络连接正常
- 检查 `ohos.para` 和 `vendor.para` 配置正确
- 确认 OEM HAL 接口已实现并返回正确数据

### Q3: 编译产物缺失

**现象**: 运行时提示找不到 so 文件

```
error: cannot find symbol 'deviceAttest' from 'deviceattest.z.so'
```

**定位路径**:

1. 检查编译输出
   ```bash
   find out -name "*devattest*" -type f
   ```

2. 检查构建配置
   ```bash
   # 确认组件已启用
   grep "device_attest" out/<product>/build_configs.json
   ```

3. 检查产物安装
   ```bash
   hdc shell ls -la /system/lib/module/ | grep deviceattest
   hdc shell ls -la /system/lib/ | grep devattest
   ```

**解决方案**:
```bash
# 重新构建
cd <ohos_src>
./build.sh --product-name=<product> --build-target //test/xts/device_attest/build:attest_standard_packages

# 推送产物（开发调试用）
hdc file send out/<product>/system/lib/module/deviceattest.z.so /system/lib/module/
hdc shell chmod 644 /system/lib/module/deviceattest.z.so
hdc shell reboot
```

### Q4: IPC 调用超时

**现象**:
```javascript
// 调用 getAttestStatus 长时间无响应
await deviceAttest.getAttestStatus();  // 超时
```

**原因**: 服务未启动或 IPC 通信异常

**定位路径**:

1. 检查服务状态
   ```bash
   hdc shell samgr -l | grep 5501
   ```

2. 检查 SA 加载日志
   ```bash
   hdc shell hilog | grep -i "LoadSystemAbility"
   ```
   相关代码：`interfaces/innerkits/native_cpp/src/devattest_client.cpp:82`

3. 检查服务启动原因
   ```bash
   hdc shell hilog | grep -i "OnStart"
   ```

**解决方案**:
- 手动触发服务启动
  ```bash
  hdc shell samgr -a 5501
  ```
- 检查 `devattest_service.json` 配置正确

### Q5: 认证失败（authResult = 2）

**现象**:
```javascript
{
    authResult: 2,  // 失败
    softwareResult: 2,
    ticket: ""
}
```

**可能原因**:
1. 设备信息与云端不匹配
2. Token 无效
3. 签名验证失败
4. 网络通信失败

**定位路径**:

1. 检查认证流程日志
   ```bash
   hdc shell hilog | grep -i "ProcAttest\|AuthDevice\|ActiveToken"
   ```
   相关代码：`services/core/attest/attest_service.c`

2. 检查网络通信
   ```bash
   hdc shell hilog | grep -i "SendAuthMsg\|SendActiveMsg"
   ```
   相关代码：`services/core/network/attest_network.c`

3. 检查服务器响应
   ```bash
   hdc shell hilog | grep -i "ParseAuthResult\|ParseActiveResult"
   ```

4. 检查加密操作
   ```bash
   hdc shell hilog | grep -i "security\|encrypt\|decrypt"
   ```

**解决方案**:
- 确认设备信息已正确注册到兼容性平台
- 检查 `manuKey` 和 `productId` 配置正确
- 确认 Token 有效且未过期
- 检查网络连接稳定

### Q6: 内存泄漏

**现象**: 长时间运行后内存持续增长

**定位路径**:

1. 启用内存泄漏检测（debug 版本）
   ```bash
   # 编译时启用
   ./build.sh --product-name=<product> --gn-args="enable_attest_debug_memory_leak=true"
   ```

2. 查看内存日志
   ```bash
   hdc shell hilog | grep -i "mem\|malloc\|free"
   ```
   相关代码：`services/core/utils/attest_utils_memleak.c`

3. 使用 valgrind（如可用）
   ```bash
   valgrind --leak-check=full devattest_service
   ```

**常见泄漏点检查**:
- `QueryAttest()` 返回的 `ticket` 是否释放
- `AttestResultInfo` 中的字符串是否正确管理
- 网络响应缓冲区是否释放

### Q7: 编译错误

**现象**: 编译时出现头文件或依赖错误

**常见错误**:
```
error: 'hks_api.h' file not found
error: undefined reference to `HksEncrypt'
```

**解决方案**:
1. 检查依赖组件是否启用
   ```json
   // bundle.json 依赖检查
   "deps": {
     "components": [
       "huks",
       "mbedtls",
       "openssl"
     ]
   }
   ```

2. 检查头文件路径
   ```gn
   # BUILD.gn 中检查 include_dirs
   include_dirs = [
     "//base/security/huks/interfaces/inner_api",
   ]
   ```

3. 完整构建
   ```bash
   ./build.sh --product-name=<product> --ccache
   ```

## 调试技巧

### 开启调试日志

1. **编译时启用**
   ```bash
   ./build.sh --product-name=<product> --gn-args="attest_build_target=attest_debug"
   ```

2. **运行时过滤**
   ```bash
   hdc shell hilog -b D  # 设置日志级别为 DEBUG
   hdc shell hilog -g    # 查看日志级别
   ```

3. **模块日志标签**
   ```bash
   hdc shell hilog | grep -E "DevAttest|ATTEST"
   ```

### 抓取完整日志

```bash
# 清除日志
hdc shell hilog -r

# 执行操作
# ...

# 抓取日志
hdc shell hilog > attest_log.txt

# 过滤关键日志
grep -E "(ERROR|WARN|FAIL)" attest_log.txt
```

### 动态调试

1. **服务启动调试**
   ```bash
   # 手动启动服务（带调试输出）
   hdc shell devattest_service &
   ```

2. **gdb 调试（如可用）**
   ```bash
   gdb devattest_service
   (gdb) b GetAttestStatus
   (gdb) run
   ```

## 相关链接

- [N-API 接口文档](03_NAPI.md) - 接口使用说明
- [架构说明](02_Architecture.md) - 了解模块架构
- [安全评审](06_Security.md) - 安全注意事项

---

**证据来源**：
- 错误码定义：`common/devattest_errno.h`
- 权限检查：`common/permission/src/permission.cpp`
- 服务实现：`services/devattest_ability/src/devattest_service.cpp`
- 核心逻辑：`services/core/attest/attest_service.c`

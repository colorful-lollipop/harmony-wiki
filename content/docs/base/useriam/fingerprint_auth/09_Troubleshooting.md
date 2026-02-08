# 故障排除

## 目的

本文档提供指纹认证组件的常见构建、运行、调试问题及定位方法。

## 适用范围

- 系统集成人员
- 测试工程师
- 开发者

## 关键结论

1. **常见问题分类**：
   - 构建失败
   - 运行时错误
   - 认证失败
   - 性能问题

2. **日志工具**：
   - `hilog` - 系统日志工具
   - `hdc shell` - 设备调试工具
   - `dumpsys` - 服务调试工具

3. **定位路径**：
   - 服务启动问题 → 检查 SA 配置和日志
   - HDI 通信问题 → 检查驱动加载和接口版本
   - 认证失败 → 检查权限、驱动和传感器状态

---

## 构建问题

### 1. HDI 依赖缺失

**错误信息**：
```
error: undefined reference to 'IFingerprintAuthInterface::Get()'
```

**原因**：`drivers_interface_fingerprint_auth` 组件未编译

**解决方法**：
```bash
# 检查组件配置
./build.sh --product-name <product> --build-target drivers_interface_fingerprint_auth

# 重新编译指纹认证组件
./build.sh --product-name <product> --build-target fingerprint_auth
```

**证据**：`bundle.json:29`（外部依赖定义）

---

### 2. UserAuth Framework 缺失

**错误信息**：
```
error: undefined reference to 'IAuthDriverHdi'
```

**原因**：`user_auth_framework` 组件未编译

**解决方法**：
```bash
# 检查组件配置
./build.sh --product-name <product> --build-target user_auth_framework

# 重新编译指纹认证组件
./build.sh --product-name <product> --build-target fingerprint_auth
```

**证据**：`bundle.json:40`（外部依赖定义）

---

### 3. 图形库依赖缺失

**错误信息**：
```
error: undefined reference to 'RSSurfaceNode'
```

**原因**：`graphic_2d` 或 `graphic_surface` 组件未编译

**解决方法**：
```bash
# 检查组件配置
./build.sh --product-name <product> --build-target graphic_2d
./build.sh --product-name <product> --build-target graphic_surface

# 重新编译扩展库
./build.sh --product-name <product> --build-target fingerprintauthservice_ex
```

**证据**：`services_ex/BUILD.gn:54-67`（图形库依赖）

---

### 4. 符号导出失败

**错误信息**：
```
error: version script error: symbol 'GetInstance' not in export list
```

**原因**：版本脚本配置错误

**解决方法**：
检查 `services/fingerprint_auth_service_map` 和 `services_ex/fingerprint_auth_service_ex_map` 是否正确

**版本脚本模板**：
```
{
    global:
        FingerprintAuthService*;
        GetInstance*;
    local:
        *;
};
```

**证据**：`services/BUILD.gn:93`（版本脚本引用）

---

## 运行时问题

### 1. SA 未启动

**现象**：
```bash
hdc shell dump -l 943
# 输出：Ability not found
```

**原因**：SA 配置文件缺失或错误

**定位步骤**：
```bash
# 1. 检查 SA 配置文件
hdc shell cat /system/profile/943.json

# 2. 检查共享库是否存在
hdc shell ls -l /system/lib64/libfingerprintauthservice.z.so

# 3. 检查 useriam 进程是否运行
hdc shell ps -A | grep useriam

# 4. 检查 SA 日志
hdc shell hilog -T FINGERPRINT_AUTH_SA
```

**预期输出**：
```json
{
    "process": "useriam",
    "systemability": [
        {
            "name": 943,
            "libpath": "libfingerprintauthservice.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": ["libfingerprint_auth_proxy_2.0.z.so"]
        }
    ]
}
```

**证据**：`sa_profile/943.json`

---

### 2. HDI 连接失败

**现象**：
```bash
hdc shell hilog -T FINGERPRINT_AUTH_SA
# 输出：start driver manager failed
```

**原因**：HDI 驱动未加载或接口版本不匹配

**定位步骤**：
```bash
# 1. 检查 HDI Proxy 是否加载
hdc shell ps -A | grep fingerprint_auth

# 2. 检查 HDI 服务是否注册
hdc shell hdilist -i fingerprint_auth

# 3. 检查 HDI Proxy 版本
hdc shell ls -l /system/lib64/libfingerprint_auth_proxy_2.0.z.so

# 4. 检查 HDI 日志
hdc shell hilog -T HDF
```

**预期输出**：
```
HDIServiceName: fingerprint_auth_interface_service
HDIVersion: 2.0
```

**证据**：`sa_profile/943.json:10`（min_hdi_proxy_version）

---

### 3. 扩展库加载失败

**现象**：
```bash
hdc shell hilog -T FINGERPRINT_AUTH_SA
# 输出：Load extension library failed
```

**原因**：`libfingerprintauthservice_ex.z.so` 不存在或符号缺失

**定位步骤**：
```bash
# 1. 检查扩展库是否存在
hdc shell ls -l /system/lib64/libfingerprintauthservice_ex.z.so

# 2. 检查符号是否导出
hdc shell readelf -s /system/lib64/libfingerprintauthservice_ex.z.so | grep GetSensorIlluminationTask

# 3. 检查动态链接依赖
hdc shell ldd /system/lib64/libfingerprintauthservice_ex.z.so

# 4. 检查加载日志
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep dlopen
```

**预期输出**：
```
GetSensorIlluminationTask
```

**证据**：`services/src/service_ex_manager.cpp:30-52`（动态加载代码）

---

### 4. 传感器照明异常

**现象**：
- 认证时传感器区域不亮
- 传感器区域显示异常

**原因**：
- Rosen 渲染服务未启动
- 显示管理器未配置
- 坐标参数错误

**定位步骤**：
```bash
# 1. 检查 Rosen 渲染服务
hdc shell ps -A | grep render_service

# 2. 检查显示管理器
hdc shell ps -A | grep display_manager

# 3. 检查 SA 命令日志
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep SaCommand

# 4. 检查传感器照明日志
hdc shell hilog -T SensorIllumination
```

**预期日志**：
```
[INFO] ProcessSaCommands: command id = 3 (TURN_ON_SENSOR_ILLUMINATION)
[INFO] TurnOnSensorIllumination: executorId = 1
```

**证据**：`services/src/sensor_illumination_manager.cpp:1-206`

---

## 认证问题

### 1. 权限拒绝

**现象**：
```bash
# 应用调用认证 API 失败
# 错误码：PERMISSION_DENIED
```

**原因**：应用未申请或授权 `ohos.permission.USE_USER_IDENTITY` 权限

**定位步骤**：
```bash
# 1. 检查应用权限
hdc shell shell bm dump -n <bundle_name> | grep permissions

# 2. 检查权限授权
hdc shell shell permission list --user 0 | grep USE_USER_IDENTITY

# 3. 检查 UserAuth Framework 日志
hdc shell hilog -T UserAuth
```

**解决方法**：
在应用清单文件中声明权限：
```json
{
  "reqPermissions": [
    {
      "name": "ohos.permission.USE_USER_IDENTITY"
    }
  ]
}
```

**证据**：`README_ZH.md:16`（权限说明）

---

### 2. 指纹未录入

**现象**：
```bash
# 认证失败，错误码：NOT_ENROLLED (10)
```

**原因**：用户未录入指纹

**定位步骤**：
```bash
# 1. 检查录入日志
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep Enroll

# 2. 检查指纹模板存储
hdc shell hilog -T HDF | grep template

# 3. 调用录入 API
hdc shell aa start -a EnrollActivity
```

**解决方法**：
先调用录入 API 录入指纹，再进行认证

**证据**：`common/inc/fingerprint_auth_defines.h:68`（NOT_ENROLLED 错误码）

---

### 3. 认证超时

**现象**：
```bash
# 认证失败，错误码：TIMEOUT (4)
```

**原因**：
- 指纹传感器无响应
- 传感器被遮挡
- 驱动超时配置过短

**定位步骤**：
```bash
# 1. 检查传感器状态
hdc shell hilog -T HDF | grep sensor

# 2. 检查超时配置
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep timeout

# 3. 检查认证日志
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep Authenticate
```

**预期日志**：
```
[INFO] Authenticate: scheduleId = 12345
[ERROR] OnResult: result = TIMEOUT
```

**证据**：`common/inc/fingerprint_auth_defines.h:44`（TIMEOUT 错误码）

---

### 4. 指纹匹配失败

**现象**：
```bash
# 认证失败，错误码：FAIL (1)
```

**原因**：
- 指纹质量不佳
- 录入的指纹与实际指纹不匹配
- 指纹模板损坏

**定位步骤**：
```bash
# 1. 检查指纹质量
hdc shell hilog -T HDF | grep quality

# 2. 检查匹配日志
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep match

# 3. 重新录入指纹
hdc shell aa start -a EnrollActivity
```

**预期日志**：
```
[INFO] OnResult: result = FAIL
[INFO] fingerprint quality: poor
```

**证据**：`common/inc/fingerprint_auth_defines.h:32`（FAIL 错误码）

---

## 性能问题

### 1. 认证响应慢

**现象**：
- 认证响应时间 > 2 秒

**原因**：
- HDI 驱动性能不佳
- 传感器硬件性能限制
- 系统资源紧张

**定位步骤**：
```bash
# 1. 检查认证时间
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep -E "Authenticate|OnResult"

# 2. 检查系统资源
hdc shell top -n 1 | grep useriam

# 3. 检查 HDI 调用时间
hdc shell hilog -T HDF | grep -E "Authenticate|Enroll"
```

**优化建议**：
- 升级 HDI 驱动版本
- 优化传感器硬件
- 增加系统资源

---

### 2. 内存占用高

**现象**：
```
useriam 进程内存占用 > 50MB
```

**原因**：
- 指纹模板缓存过大
- 传感器照明 UI 资源未释放

**定位步骤**：
```bash
# 1. 检查进程内存
hdc shell dumpsys meminfo useriam

# 2. 检查指纹模板缓存
hdc shell hilog -T FINGERPRINT_AUTH_SA | grep template

# 3. 检查 UI 资源释放
hdc shell hilog -T SensorIllumination | grep Release
```

**优化建议**：
- 清理指纹模板缓存
- 及时释放 UI 资源

---

## 调试工具

### 1. hilog（日志工具）

**常用命令**：
```bash
# 查看所有日志
hdc shell hilog

# 按标签过滤
hdc shell hilog -T FINGERPRINT_AUTH_SA

# 清空日志
hdc shell hilog -r

# 保存日志到文件
hdc shell hilog -t FINGERPRINT_AUTH_SA > fingerprint_auth.log
```

**日志标签**：
- `FINGERPRINT_AUTH_SA` - 主服务日志
- `FINGERPRINT_AUTH_EX` - 扩展服务日志
- `HDF` - HDI 框架日志
- `UserAuth` - UserAuth Framework 日志

---

### 2. dumpsys（服务调试工具）

**常用命令**：
```bash
# 查看 SA 状态
hdc shell dump -l 943

# 查看 SA 详细信息
hdc shell dump -s 943 -a

# 查看所有 SA
hdc shell dump -l
```

---

### 3. hdc shell（设备调试工具）

**常用命令**：
```bash
# 连接设备
hdc shell

# 查看进程
hdc shell ps -A

# 查看文件
hdc shell ls -l /system/lib64/libfingerprintauthservice.z.so

# 查看配置
hdc shell cat /system/profile/943.json
```

---

## 日志级别说明

| 日志级别 | 标签 | 说明 |
|---------|------|------|
| DEBUG | IAM_LOGD | 详细调试信息 |
| INFO | IAM_LOGI | 常规信息 |
| WARN | IAM_LOGW | 警告信息 |
| ERROR | IAM_LOGE | 错误信息 |
| FATAL | IAM_LOGF | 致命错误 |

**证据**：`common/logs/iam_logger.h`

---

## 相关跳转

- [01_Overview.md](./01_Overview.md) - 组件概览
- [03_Architecture.md](./03_Architecture.md) - 架构设计
- [07_Build_Artifacts.md](./07_Build_Artifacts.md) - 构建与产物

---

## 参考资料

1. **OpenHarmony 调试文档**：[OpenHarmony 调试指南](https://docs.openharmony.cn/)
2. **HDI 驱动开发**：[HDF 框架文档](https://docs.openharmony.cn/)
3. **用户 IAM 框架**：[UserIAM 文档](https://docs.openharmony.cn/)

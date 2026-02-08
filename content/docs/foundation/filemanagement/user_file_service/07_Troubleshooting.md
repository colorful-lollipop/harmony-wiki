# User File Service - 问题定位

## 概述

本文档收集 user_file_service 常见问题及定位方法，包括构建问题、运行时问题和调试技巧。

---

## 构建问题

### Q1: 编译报错 "napi_module_register" 未定义

**现象**：
```
error: undefined reference to 'napi_module_register'
```

**原因**：
缺少 N-API 框架依赖

**解决方案**：
检查 `external_deps` 是否包含 `napi:ace_napi`

**证据**：`frameworks/js/napi/file_access_module/BUILD.gn:77`

```gn
external_deps = [
    "napi:ace_napi",  # 确保包含此依赖
]
```

---

### Q2: 编译报错 "SystemAbility" 未找到

**现象**：
```
error: 'system_ability.h' file not found
```

**原因**：
缺少 SAFwk 依赖

**解决方案**：
```gn
external_deps = [
    "safwk:system_ability_fwk",  # 添加此依赖
]
```

---

### Q3: CFI 编译报错

**现象**：
```
error: CFI: failed to indirect call
```

**原因**：
控制流完整性检查失败，通常是函数指针类型不匹配

**解决方案**：
```gn
# 临时禁用 CFI 进行调试
sanitize = {
    cfi = false,  # 仅调试时
}
```

**注意**：发布版本必须启用 CFI

---

### Q4: SA ID 冲突

**现象**：
```
error: SA id 5010 already registered
```

**原因**：
SA ID 与其他服务冲突

**解决方案**：
检查 `services/5010.json` 中 SA ID 是否唯一

---

### Q5: HAP 签名失败

**现象**：
```
error: sign failed: certificate not found
```

**原因**：
签名证书配置错误

**解决方案**：
```gn
ohos_hap("external_file_manager_hap") {
    # 使用正确的证书
    certificate_profile = "//vendor/tools/hap_sign_conf/filemanagement/user_file_service/external_file_manager.p7b"
}
```

---

## 运行时问题

### Q6: N-API 模块加载失败

**现象**：
```
Failed to load module 'file.fileAccess'
```

**排查步骤**：
1. 检查库文件是否存在
   ```bash
   ls -la /system/lib/module/file/libfileaccess.z.so
   ```
2. 检查依赖是否完整
   ```bash
   ldd /system/lib/module/file/libfileaccess.z.so
   ```
3. 检查日志
   ```bash
   hidumper -a | grep filemanagement
   ```

**常见原因**：
- 库文件未安装
- 依赖库缺失
- SELinux 策略阻止

---

### Q7: 文件访问权限拒绝

**现象**：
```
E_PERMISSION: Permission denied
```

**排查步骤**：
1. 检查应用是否申请权限
2. 检查权限是否授予
3. 检查权限名称是否正确

**解决方案**：
```typescript
// 检查权限
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';

// 权限名称
const PERMISSION = 'ohos.permission.FILE_ACCESS_MANAGER';
```

---

### Q8: SA 启动失败

**现象**：
```
Failed to start FileAccessService
```

**排查步骤**：
1. 检查 SA 配置
   ```bash
   cat /system/profile/5010.json
   ```
2. 检查 init 配置
   ```bash
   cat /system/etc/init/file_access_service.cfg
   ```
3. 检查日志
   ```bash
   hilog | grep -E "FileAccessService|5010"
   ```

**常见原因**：
- 库文件路径错误
- 依赖服务未启动
- 配置文件格式错误

---

### Q9: 文件变化通知不生效

**现象**：
文件变化后观察者未收到通知

**排查步骤**：
1. 检查观察者是否正确注册
2. 检查 NotifyType 是否匹配
3. 检查 URI 是否精确

**解决方案**：
```cpp
// 正确注册观察者
int32_t RegisterNotify(const Uri &uri, bool notifyForDescendants,
                       const sptr<IFileAccessObserver> &observer);
```

---

### Q10: 云同步功能异常

**现象**：
云同步文件夹状态不正确

**排查步骤**：
1. 检查云盘开关是否启用
   ```bash
   getprop persist.sys.user_file_service_cloud_disk_enable
   ```
2. 检查 dfs_service 是否正常运行
3. 检查网络连接

---

## 调试技巧

### 1. 日志级别调整

```bash
# 设置日志级别为 Debug
hdc shell hilog -v D

# 过滤 filemanagement 日志
hdc shell hilog | grep "filemanagement"
```

**日志域**：`0xD00430A`

```bash
hdc shell hilog | grep "0xD00430A"
```

---

### 2. 服务状态检查

```bash
# 检查 SA 运行状态
hdc shell sa_conn 5010

# 检查 SA 列表
hdc shell dumpsys --ability
```

---

### 3. 文件系统访问检查

```bash
# 检查文件是否存在
hdc shell ls -la /storage/emulated/0/

# 检查权限
hdc shell ls -la /system/lib/module/file/
```

---

### 4. IPC 通信调试

```bash
# 启用 IPC 追踪
hdc shell trace ipc

# 捕获 IPC 消息
hdc shell bdmsg
```

---

### 5. 核心转储分析

```bash
# 启用 core dump
ulimit -c unlimited

# 发生 crash 后
gdb /system/lib64/libfile_access_service.z.so core.xxx
```

---

## 常用调试命令

| 命令 | 用途 |
|------|------|
| `hdc shell dumpsys activity service 5010` | 查看 SA 状态 |
| `hdc shell cat /etc/init/file_access_service.cfg` | 查看配置 |
| `hdc shell hilog \| grep filemanagement` | 查看日志 |
| `hdc shell ldd /system/lib/module/file/libfileaccess.z.so` | 检查依赖 |
| `hdc shell sa_conn 5010` | SA 连接测试 |

---

## 性能问题

### Q11: 文件操作响应慢

**可能原因**：
1. 底层服务响应慢
2. IPC 通信延迟
3. 文件数量过多

**排查方法**：
```bash
# 使用 hitrace 追踪
hdc shell hitrace --trace_categories file_access -b 10240 -t 5
```

---

### Q12: 内存占用过高

**排查方法**：
```bash
# 查看内存使用
hdc shell cat /proc/$(pid)/status | grep VmRSS

# 使用内存分析
hdc shell dump_mem <pid>
```

---

## FAQ

### Q: 如何确认 N-API 版本？

```cpp
// 代码中检查
#include "js_native_api.h"

napi_env env;
// ...
napi_status status = napi_get_version(env, &version);
```

### Q: 支持哪些文件类型？

- **媒体文件**：图片 (jpg, png, gif)、音频 (mp3, aac)、视频 (mp4)
- **文档文件**：txt, pdf, doc, xls, ppt
- **归档文件**：zip, rar

### Q: 最大支持文件大小？

由底层文件系统决定，建议不超过 2GB。

### Q: 支持多用户吗？

**证据**：`file_access_service.h:268`

```cpp
int32_t GetCurrentUserId();
```

支持多用户隔离，通过 `GetCurrentUserId()` 获取当前用户。

---

## 错误码速查

| 错误码 | 常量 | 场景 |
|--------|------|------|
| 0 | E_OK | 成功 |
| -1 | E_PERMISSION | 权限不足 |
| -2 | E_URI | URI 错误 |
| -3 | E_IO | IO 错误 |
| -4 | E_NOENT | 文件不存在 |
| -5 | E_EXIST | 文件已存在 |

**完整错误码列表**：`utils/file_access_framework_errno.h`

---

## 联系支持

如遇未解决问题：

1. 收集日志：`hdc shell bugreport`
2. 记录复现步骤
3. 提交 Issue 到 OpenHarmony

# 常见问题排查

## 目的

本文档汇总 `sys_installer` 常见问题、定位方法和解决方案。

## 适用范围

开发人员、测试人员、运维人员。

## 问题分类

### 1. 构建问题

#### 1.1 编译失败

**症状**: GN 构建报错

**定位路径**:
```bash
# 检查 GN 配置
cat out/standard/build_configs.json | grep sys_installer

# 检查编译日志
cat out/standard/build.log | grep -A 10 "sys_installer"
```

**常见原因**:
- 依赖组件未编译
- 头文件路径错误
- 宏定义冲突

**解决方案**:
```bash
# 1. 清理并重新编译m -rf out/standard/base/update/sys_installer
./build.sh --product standard --target sys_installer

# 2. 检查依赖
./build.sh --product standard --target //base/update/sys_installer:sys_installer --verbose
```

#### 1.2 链接错误

**症状**: 未定义符号

**定位路径**:
```bash
# 检查符号
nm -C out/standard/system/lib/libsys_installer.z.so | grep "T "

# 检查依赖库
readelf -d out/standard/system/lib/libsys_installer.z.so | grep NEEDED
```

**常见原因**:
- 静态库链接顺序错误
- 符号未导出
- 库文件缺失

**解决方案**:
- 检查 `BUILD.gn` 中的 `deps` 顺序
- 确保头文件中有 `__attribute__((visibility("default")))`

### 2. 运行时问题

#### 2.1 SA 启动失败

**症状**: SA 4101/4103 无法启动

**定位路径**:
```bash
# 查看 init 日志
hilog | grep sys_installer

# 查看 SA 状态
aa dump -a | grep -E "4101|4103"

# 查看进程
ps -ef | grep -E "sys_installer|module_update"
```

**常见原因**:
- 配置文件错误
- 库文件缺失
- 权限不足

**解决方案**:
```bash
# 1. 检查配置文件
cat /system/etc/init/sys_installer.cfg

# 2. 检查库文件
ls -la /system/lib/libsys_installer.z.so

# 3. 检查权限
ls -la /system/bin/check_module_update_init
```

#### 2.2 IPC 调用失败

**症状**: 客户端调用返回错误

**定位路径**:
```bash
# 查看 hilog
hilog | grep -E "SysInstaller|ModuleUpdate"

# 查看 hisysevent
hisysevent -q -n SYS_INSTALLER

# 查看 crash 日志
ls -la /data/log/faultlog/temp/
```

**常见原因**:
- SA 未启动
- 权限不足
- 参数错误

**解决方案**:
```cpp
// 检查返回值
int32_t ret = SysInstallerKits::GetInstance().StartUpdatePackageZip(taskId, pkgPath);
if (ret != 0) {
    // 错误处理
    // ERR_INVALID_VALUE (-1): 参数错误
    // ERR_PERMISSION_DENIED (-3): 权限不足
    // ERR_NOT_SUPPORTED (-4): 不支持的操作
}
```

#### 2.3 更新失败

**症状**: 更新过程中断或失败

**定位路径**:
```bash
# 查看更新日志
cat /data/updater/log/sys_installer.log

# 查看系统日志
hilog -p sys_installer

# 检查磁盘空间
df -h /data
```

**常见原因**:
- 磁盘空间不足
- 包损坏
- 签名验证失败

**解决方案**:
```bash
# 1. 检查磁盘空间
df -h /data/module_update_package/

# 2. 验证包完整性
openssl dgst -sha256 /data/module_update_package/update.zip

# 3. 清除缓存
rm -rf /data/module_update_package/*
```

### 3. 模块更新问题

#### 3.1 模块安装失败

**症状**: InstallModulePackage 返回错误

**定位路径**:
```bash
# 查看模块更新日志
cat /data/log/module_update.log

# 查看 HVB 日志 (如启用)
cat /data/log/hvb.log

# 检查模块目录
ls -la /data/module_update/active/
```

**常见原因**:
- 包签名验证失败
- HVB 验证失败
- 文件系统错误

**错误码参考**:
```cpp
// services/module_update/util/include/module_error_code.h
ERR_OK = 0;                    // 成功
ERR_VERIFY_FAIL = 1;           // 验证失败
ERR_EXTRACT_FAIL = 2;          // 解压失败
ERR_MOUNT_FAIL = 3;            // 挂载失败
ERR_DM_CREATE_FAIL = 4;        // DM 创建失败
ERR_LOOP_CREATE_FAIL = 5;      // Loop 设备创建失败
ERR_FILE_OPERATION_FAIL = 6;   // 文件操作失败
ERR_HVB_VERIFY_FAIL = 7;       // HVB 验证失败
ERR_INVALID_PACKAGE = 8;       // 无效包
ERR_NO_SPACE = 9;              // 空间不足
```

#### 3.2 模块卸载失败

**症状**: UninstallModulePackage 返回错误

**定位路径**:
```bash
# 检查模块是否在使用
lsof | grep /data/module_update/active/<module>

# 检查挂载点
mount | grep module_update

# 检查进程
ps -ef | grep <module_name>
```

**常见原因**:
- 模块正在使用
- 挂载点未卸载
- 权限不足

**解决方案**:
```bash
# 1. 强制卸载
umount -f /data/module_update/active/<module>

# 2. 重启后清理
reboot
```

### 4. 性能问题

#### 4.1 更新速度慢

**症状**: 更新进度长时间停滞

**定位路径**:
```bash
# 查看 CPU 使用率
top -p $(pidof sys_installer_sa)

# 查看 IO 情况
iostat -x 1

# 查看日志频率
hilog | grep -c "progress"
```

**常见原因**:
- CPU 占用高
- IO 瓶颈
- 网络延迟 (流式更新)

**解决方案**:
```cpp
// 设置 CPU 亲和性
SysInstallerKits::GetInstance().SetCpuAffinity(taskId, 0xF0);
```

#### 4.2 内存占用高

**症状**: 更新过程中内存不足

**定位路径**:
```bash
# 查看内存使用
procrank | grep sys_installer

# 查看内存详情
cat /proc/$(pidof sys_installer_sa)/status | grep -E "VmRSS|VmSize"
```

**常见原因**:
- 大文件缓存
- 内存泄漏

**解决方案**:
```bash
# 限制缓存大小
echo 1048576 > /proc/sys/vm/dirty_bytes
```

### 5. 安全问题

#### 5.1 权限拒绝

**症状**: 返回 ERR_PERMISSION_DENIED

**定位路径**:
```bash
# 检查调用者 UID
echo $UID

# 检查权限
acm check -p ohos.permission.UPDATE_SYSTEM

# 查看 SELinux 日志
cat /data/log/audit.log | grep denied
```

**常见原因**:
- 缺少 UPDATE_SYSTEM 权限
- UID 不在白名单
- SELinux 拒绝

**解决方案**:
```xml
<!-- 在应用的 config.json 中声明权限 -->
"reqPermissions": [
    {
        "name": "ohos.permission.UPDATE_SYSTEM"
    }
]
```

#### 5.2 签名验证失败

**症状**: 返回 ERR_VERIFY_FAIL

**定位路径**:
```bash
# 检查证书
openssl pkcs7 -in update.zip.sig -inform DER -print_certs

# 检查系统证书
ls -la /system/etc/certs/

# 查看验证日志
hilog | grep -i "verify"
```

**常见原因**:
- 包签名不正确
- 证书过期
- 证书链不完整

**解决方案**:
- 使用正确的签名密钥
- 更新系统证书
- 检查证书链

## 调试技巧

### 启用详细日志

```cpp
// 在代码中设置日志级别
#define LOG_LEVEL LOG_DEBUG

// 或使用环境变量
setenv("LOG_LEVEL", "DEBUG", 1);
```

### 使用 GDB 调试

```bash
# 附加到运行中的进程
gdb -p $(pidof sys_installer_sa)

# 设置断点
(gdb) b SysInstallerServer::StartUpdatePackageZip

# 继续执行
(gdb) c
```

### 使用 strace

```bash
# 跟踪系统调用
strace -p $(pidof sys_installer_sa) -e trace=file,network
```

### 查看 Binder 调用

```bash
# 查看 Binder 事务
hilog | grep -E "BC_|BR_"
```

## 日志位置

| 日志类型 | 路径 | 说明 |
|----------|------|------|
| 系统日志 | /data/log/hilog/ | hilog 日志 |
| 更新日志 | /data/updater/log/sys_installer.log | 更新过程日志 |
| 模块日志 | /data/log/module_update.log | 模块更新日志 |
| 崩溃日志 | /data/log/faultlog/temp/ | crash 日志 |
| 审计日志 | /data/log/audit.log | SELinux 审计 |

## 常用命令速查

```bash
# 查看 SA 状态
aa dump -a | grep -E "4101|4103"

# 查看进程
ps -ef | grep -E "sys_installer|module_update"

# 查看日志
hilog | grep -E "SysInstaller|ModuleUpdate"

# 检查磁盘空间
df -h /data

# 检查内存
free -m

# 查看 Binder 状态
cat /proc/binder/state

# 查看文件句柄
ls -la /proc/$(pidof sys_installer_sa)/fd/ | wc -l
```

## 相关链接

- [对外 API](03_External_APIs.md)
- [安全风险分析](06_Security_Analysis.md)
- [编译产物](07_Build_Products.md)

---

*证据来源*:
- `common/include/sys_installer_common.h`: 日志路径
- `services/module_update/util/include/module_error_code.h`: 错误码
- 实际调试经验

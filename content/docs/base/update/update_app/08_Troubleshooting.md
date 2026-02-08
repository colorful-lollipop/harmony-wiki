# 故障排查指南

> 本文档汇总 update_app 模块的常见构建、运行和调试问题，提供定位路径和解决方案。

## 1 构建问题

### 1.1 GN 配置问题

#### 问题：gn gen 失败

**错误信息**：
```
ERROR at //src/base/update/BUILD.gn:5
A dependency was not found.
```

**排查步骤**：

```bash
# 1. 检查依赖配置
gn desc out/default //src/base/update:update_core deps

# 2. 检查缺失依赖
gn check out/default //src/base/update:update_core

# 3. 检查路径配置
cat build/config.gni | grep root_out_dir
```

**解决方案**：

```bash
# 清理并重新生成
rm -rf out/default
gn gen out/default

# 或指定正确的依赖路径
gn gen out/default --args='
third_party_dir = "//third_party"
'
```

---

#### 问题：ninja 编译失败

**错误信息**：
```
ninja: fatal: mkdir(exists): No such file or directory
```

**排查步骤**：

```bash
# 1. 检查输出目录
ls -la out/

# 2. 检查目录权限
ls -la out/default/

# 3. 手动创建缺失目录
mkdir -p out/default/obj
mkdir -p out/default/libs
```

**解决方案**：

```bash
# 清理并重新构建
rm -rf out/default
gn gen out/default
ninja -C out/default

# 或使用绝对路径
ninja -C $(pwd)/out/default
```

### 1.2 依赖冲突

#### 问题：符号重定义

**错误信息**：
```
multiple definition of `VersionManager::GetInstance()';
```

**排查步骤**：

```bash
# 1. 检查重复源文件
gn desc out/default //src/base/update:update_core sources

# 2. 检查静态库
gn desc out/default //src/base/verify:verify_core sources

# 3. 查看链接顺序
ninja -C out/default -t commands | head -20
```

**解决方案**：

```gn
# BUILD.gn 中添加排除
static_library("update_core") {
  sources = [
    "version_manager.cpp",
    # 排除重复文件
  ]
  
  # 检查重复
  if (defined(invoker.exclude_sources)) {
    sources -= invoker.exclude_sources
  }
}
```

### 1.3 编译选项问题

#### 问题：头文件找不到

**错误信息**：
```
fatal error: 'update/version_manager.h' file not found
```

**排查步骤**：

```bash
# 1. 检查头文件路径
find . -name "version_manager.h"

# 2. 检查 include_dirs 配置
gn desc out/default //src/base/update:update_core include_dirs

# 3. 检查头文件是否存在
ls -la include/update/version_manager.h
```

**解决方案**：

```gn
# BUILD.gn 中添加正确的 include_dirs
static_library("update_core") {
  include_dirs = [
    "//include/update",
    "//include/utils",
    "//src/base/common",
  ]
}
```

## 2 运行问题

### 2.1 服务启动失败

#### 问题：UpdateService 无法启动

**错误信息**：
```
[01/01/1970 00:00:00] E/SAMGR: Start service failed, saId: 4001
```

**排查步骤**：

```bash
# 1. 检查服务状态
hilog | grep -E "UpdateService|SA"

# 2. 检查依赖库
ldd /system/bin/update_service

# 3. 检查配置文件
cat /system/etc/update/update.cfg
```

**日志示例**：

```
# hilog 输出
01-01 00:00:00.123 I/update_service: Starting UpdateService...
01-01 00:00:00.124 E/update_service: Failed to load libupdate_core.z.so
01-01 00:00:00.124 E/SAMGR: Start service failed
```

**解决方案**：

```bash
# 1. 检查库文件
ls -la /system/lib/module/update/

# 2. 修复库路径
# 在 device.mk 中添加
PRODUCT_COPY_FILES += \
    $(LOCAL_PATH)/out/libs/libupdate_core.z.so:$(TARGET_OUT)/lib/module/update/libupdate_core.z.so

# 3. 重启服务
sa_main update_service restart
```

---

#### 问题：权限不足

**错误信息**：
```
E/update_napi: Permission denied: ohos.permission.UPDATE_APP
```

**排查步骤**：

```bash
# 1. 检查权限声明
cat config.json | grep permissions

# 2. 检查 Capability 配置
cat config.json | grep "SystemCapability"

# 3. 手动授权
hidc setperm <package> ohos.permission.UPDATE_APP system
```

**解决方案**：

```json
// config.json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.UPDATE_APP",
        "reason": "Need to update applications",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "system_grant"
        }
      }
    ]
  }
}
```

### 2.2 网络下载问题

#### 问题：下载超时

**错误信息**：
```
E/download: Download timeout after 300 seconds
```

**排查步骤**：

```bash
# 1. 测试网络连通性
ping update.example.com

# 2. 测试 HTTPS 访问
curl -v https://update.example.com

# 3. 检查防火墙规则
iptables -L -n | grep 443
```

**解决方案**：

```cpp
// 增加超时时间
DownloadConfig config;
config.timeout = 600;  // 10 分钟
config.maxRetries = 5;

// 或分片下载
std::vector<std::string> segments = SplitDownload(url, 3);
```

---

#### 问题：SSL 证书错误

**错误信息**：
```
E/download: SSL certificate problem: certificate has expired
```

**排查步骤**：

```bash
# 1. 检查系统时间
date

# 2. 检查证书链
openssl s_client -connect update.example.com:443 -servername update.example.com

# 3. 检查 CA 证书
ls -la /etc/ssl/certs/
```

**解决方案**：

```bash
# 1. 更新系统时间
timedatectl set-time "2026-02-06 08:00:00"

# 2. 更新 CA 证书
update-ca-certificates

# 3. 或更新证书到信任列表
cp new_cert.pem /usr/local/share/ca-certificates/
update-ca-certificates
```

### 2.3 补丁应用问题

#### 问题：补丁验证失败

**错误信息**：
```
E/patch: Signature verification failed
```

**排查步骤**：

```bash
# 1. 检查签名证书
openssl verify -CAfile root_ca.crt patch.zip

# 2. 检查证书链
openssl x509 -in cert.pem -noout -subject -issuer

# 3. 检查签名时间
openssl pkcs7 -in signature.p7 -print_certs
```

**解决方案**：

```cpp
// 启用详细日志
VerifyEngine::GetInstance().SetVerbose(true);

// 或更新信任证书
UpdateTrustedCerts(trusted_certs_path);

// 或处理证书过期
if (cert.expired) {
    // 使用备用证书或更新根证书
}
```

---

#### 问题：磁盘空间不足

**错误信息**：
```
E/storage: Not enough disk space
required: 200MB, available: 50MB
```

**排查步骤**：

```bash
# 1. 检查磁盘空间
df -h /data/

# 2. 清理临时文件
rm -rf /data/update/temp/*

# 3. 清理旧备份
rm -rf /data/update/backup/*
```

**解决方案**：

```cpp
// 检查可用空间
int64_t available = GetAvailableSpace("/data/update/");
int64_t required = CalculateRequiredSpace(patchPath);

if (available < required) {
    // 清理空间或返回错误
    CleanupTempFiles();
    CleanupOldBackups();
}
```

## 3 调试方法

### 3.1 日志调试

#### 启用调试日志

```bash
# 运行时启用调试日志
hdc shell
param set update.debug.log.enable 1
restart update_service

# 或编译时启用
gn gen out/debug --args="enable_debug_log=true"
```

#### 日志过滤

```bash
# 按标签过滤
hilog | grep -E "update|patch|download|verify"

# 按级别过滤
hilog | grep -E "E/|W/"  # 只看错误和警告

# 实时监控
hilog -f update.log &
```

### 3.2 调试断点

#### GDB 调试

```bash
# 附加到进程
gdb /system/bin/update_service
(gdb) attach $(pidof update_service)

# 设置断点
(gdb) break VersionManager::CheckForUpdates
(gdb) continue

# 查看变量
(gdb) print packageName
(gdb) print result
```

#### LLDB 调试

```bash
# 附加到进程
lldb /system/bin/update_service
(lldb) process attach --pid $(pidof update_service)

# 设置断点
(lldb) breakpoint set --name ApplyUpdate

# 单步执行
(lldb) step
(lldb) frame variable
```

### 3.3 性能分析

#### CPU 性能

```bash
# CPU 占用分析
perf record -g -p $(pidof update_service)
perf report

# 或使用 simpleperf
simpleperf record -p $(pidof update_service) --duration 10
simpleperf report
```

#### 内存分析

```bash
# 内存使用
cat /proc/$(pidof update_service)/status | grep -E "VmRSS|VmSize|VmData"

# 内存泄漏检测
valgrind --leak-check=full /system/bin/update_service
```

### 3.4 网络抓包

#### HTTPS 抓包

```bash
# 使用代理抓包
curl -x http://proxy:8080 https://update.example.com

# 或导出 SSL KEY
# 1. 设置环境变量
export SSLKEYLOGFILE=/data/ssl_key.log

# 2. 运行更新
./update_app

# 3. 使用 Wireshark 分析
wireshark -k
```

## 4 常见错误码

### 4.1 N-API 错误码

| 错误码 | 常量名 | 说明 | 解决方案 |
|--------|--------|------|----------|
| 401 | PARAMETER_INVALID | 参数无效 | 检查输入参数格式 |
| 1001 | PACKAGE_NOT_FOUND | 应用不存在 | 检查包名是否正确 |
| 1002 | NETWORK_ERROR | 网络错误 | 检查网络连接 |
| 1003 | SERVER_ERROR | 服务器错误 | 联系服务器管理员 |
| 1004 | DOWNLOAD_FAILED | 下载失败 | 检查存储空间 |
| 1005 | STORAGE_FULL | 存储空间不足 | 清理存储 |
| 1006 | FILE_NOT_FOUND | 文件不存在 | 检查文件路径 |
| 1007 | PATCH_INVALID | 补丁文件无效 | 重新下载 |
| 1008 | SIGNATURE_INVALID | 签名校验失败 | 更新证书 |
| 1009 | APPLY_FAILED | 应用失败 | 查看日志 |
| 1010 | ROLLBACK_REQUIRED | 需要回滚 | 等待回滚完成 |

### 4.2 系统错误码

| 错误码 | 说明 | 可能原因 |
|--------|------|----------|
| EPERM | 操作不允许 | 权限不足 |
| ENOENT | 文件不存在 | 路径错误 |
| EEXIST | 文件已存在 | 路径冲突 |
| ENOSPC | 空间不足 | 存储满 |
| EINVAL | 参数无效 | 参数错误 |
| EIO | I/O 错误 | 磁盘问题 |
| ETIMEDOUT | 超时 | 网络问题 |

## 5 诊断工具

### 5.1 内置诊断命令

```bash
# 检查更新服务状态
update_cli --status

# 检查磁盘空间
update_cli --disk-check

# 检查网络连通性
update_cli --network-test

# 检查证书有效期
update_cli --cert-check

# 清理临时文件
update_cli --cleanup
```

### 5.2 日志分析

```bash
# 导出日志
update_cli --export-logs /data/update_logs.zip

# 分析日志
log_analyzer /data/logs/ -o report.html

# 常见问题检测
log_analyzer /data/logs/ --check-common-issues
```

### 5.3 配置验证

```bash
# 验证配置文件
update_cli --validate-config

# 检查权限配置
update_cli --check-permissions

# 检查证书链
update_cli --verify-chain
```

## 6 升级与回滚

### 6.1 回滚操作

```bash
# 查看可用回滚点
update_cli --list-rollback-points

# 执行回滚
update_cli --rollback <rollback_id>

# 取消回滚（在回滚完成前）
update_cli --cancel-rollback
```

### 6.2 版本恢复

```bash
# 恢复到上一版本
update_cli --restore-previous

# 从备份恢复
update_cli --restore-from-backup /data/backup/backup_20260101

# 强制恢复（清空所有更新）
update_cli --factory-reset
```

## 7 相关文档

| 文档 | 描述 |
|------|------|
| [01_Project_Overview.md](./01_Project_Overview.md) | 项目运行环境 |
| [03_N-API_Reference.md](./03_N-API_Reference.md) | 错误码参考 |
| [07_Security_Review.md](./07_Security_Review.md) | 安全问题排查 |
| [appendix/Config_Flags.md](./appendix/Config_Flags.md) | 配置开关 |

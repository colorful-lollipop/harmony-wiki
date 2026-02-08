# HDF Core 常见问题与调试

## 1. 构建问题

### 1.1 编译错误: 找不到头文件

**现象**:
```
fatal error: 'hdf_remote_service.h' file not found
```

**原因**:
- include_dirs 配置不完整
- 依赖模块未编译

**解决**:
1. 检查 BUILD.gn 中的 `include_dirs`
2. 确保依赖的 `deps` 已正确声明
3. 先编译依赖模块:
```bash
ninja -C out drivers/hdf_core/adapter/uhdf2/ipc:libhdf_ipc_adapter
```

### 1.2 链接错误: 未定义符号

**现象**:
```
undefined reference to `HdfRemoteServiceObtain'
```

**原因**:
- 未链接必要的库
- 库顺序不正确

**解决**:
1. 检查 `deps` 是否包含 `libhdf_ipc_adapter`
2. 确保库顺序: 被依赖的库在后

### 1.3 GN 配置不生效

**现象**:
修改 `uhdf.gni` 后构建行为未改变

**原因**:
- GN 缓存未更新

**解决**:
```bash
# 重新生成构建配置
gn gen out --args='...'

# 或清理后重建
rm -rf out && gn gen out --args='...'
```

## 2. 运行问题

### 2.1 hdf_devmgr 启动失败

**现象**:
```
failed to publish device service manager
```

**排查步骤**:

1. 检查日志:
```bash
hilog | grep -i "hdf"
```

2. 检查 System Ability 注册:
```bash
# 查看 SA 列表
ps -ef | grep hdf
```

3. 检查配置文件:
```bash
# 确认配置文件存在
ls /system/etc/init/hdf_devmgr.cfg
ls /system/etc/hdfconfig/
```

4. 检查权限:
```bash
# 确认 hdf_devmgr 有正确权限
ls -l /system/bin/hdf_devmgr
```

### 2.2 驱动加载失败

**现象**:
```
failed to load driver: xxx
```

**排查步骤**:

1. 检查 HCS 配置:
```bash
# 查看设备配置
cat /system/etc/hdfconfig/hdf.hcs
```

2. 检查驱动 .so 文件:
```bash
# 确认驱动库存在
ls /vendor/lib/*.z.so
ls /system/lib/*.z.so
```

3. 检查日志详情:
```bash
# 查看详细错误
hilog -p hdf -l debug
```

4. 检查符号:
```bash
# 确认驱动导出符号
readelf -s /vendor/lib/libxxx_driver.z.so | grep HdfDriverEntry
```

### 2.3 服务获取失败

**现象**:
```
GetService service xxx not found
```

**排查步骤**:

1. 检查服务是否已注册:
```bash
# 使用 hdf_dbg 工具
hdf_dbg -l
```

2. 检查驱动是否加载成功:
```bash
# 查看已加载设备
hdf_dbg -d
```

3. 检查服务名拼写:
```c
// 确认服务名一致
// 驱动注册
struct HdfDriverEntry g_xxxDriverEntry = {
    .moduleName = "xxx_driver",  // 与获取时一致
};

// 服务获取
HDIServiceManagerGetService(servMgr, "xxx_service", &service);
```

## 3. 调试方法

### 3.1 日志调试

**启用详细日志**:
```c
// 在代码中添加
#define HDF_LOG_TAG my_driver
#include "hdf_log.h"

HDF_LOGD("Debug info: %{public}d", value);  // DEBUG 级别
HDF_LOGI("Info: %{public}s", str);          // INFO 级别
HDF_LOGW("Warning");                         // WARN 级别
HDF_LOGE("Error: %{public}d", errno);       // ERROR 级别
```

**查看日志**:
```bash
# 实时查看 HDF 日志
hilog | grep -E "(hdf|HDF)"

# 查看特定标签
hilog -p my_driver

# 查看错误级别以上
hilog -l error
```

### 3.2 使用 hdf_dbg 工具

**编译**:
```bash
ninja -C out drivers/hdf_core/framework/tools/hdf_dbg:hdf_dbg
```

**常用命令**:
```bash
# 列出所有服务
hdf_dbg -l

# 列出所有设备
hdf_dbg -d

# 列出所有 host
hdf_dbg -h

# 加载设备
hdf_dbg -L service_name

# 卸载设备
hdf_dbg -U service_name

# 查看帮助
hdf_dbg --help
```

### 3.3 GDB 调试

**调试 DevMgr**:
```bash
# 启动调试
gdb /system/bin/hdf_devmgr

# 设置断点
(gdb) b DevSvcManagerStubAddService
(gdb) b DevMgrServiceStubDispatch

# 运行
(gdb) r
```

**调试驱动**:
```bash
# 附加到 DevHost 进程
gdb -p $(pidof devhost)

# 设置断点
(gdb) b HdfDriverEntry
(gdb) b Dispatch
```

### 3.4 堆栈分析

**崩溃时获取堆栈**:
```bash
# 查看 tombstone
cat /data/tombstones/tombstone_*

# 使用 addr2line
addr2line -e /system/lib/libhdf_ipc_adapter.z.so -a 0x1234
```

## 4. 性能问题

### 4.1 IPC 调用慢

**排查**:
```bash
# 查看 IPC 统计
cat /sys/kernel/debug/binder/stats

# 查看调用延迟
# 在代码中添加时间测量
uint64_t start = OsalGetTime();
// IPC 调用
uint64_t end = OsalGetTime();
HDF_LOGI("IPC latency: %{public}llu us", end - start);
```

**优化建议**:
- 减少 IPC 调用次数
- 使用批量操作
- 考虑使用共享内存（SMQ）

### 4.2 内存泄漏

**检测**:
```bash
# 查看进程内存
ps -ef | grep hdf
cat /proc/[pid]/status | grep VmRSS

# 使用 valgrind（调试版本）
valgrind --leak-check=full /system/bin/hdf_devmgr
```

**常见泄漏点**:
- `HdfSBuf` 未回收
- `HdfRemoteService` 未释放
- `OsalMemAlloc` 未释放

## 5. 配置问题

### 5.1 HCS 配置错误

**现象**:
```
failed to parse hcs config
```

**排查**:
```bash
# 验证 HCS 语法
hc-gen -o /tmp/hdf.hcb -i /system/etc/hdfconfig/hdf.hcs

# 查看编译后的配置
hc-gen -d /tmp/hdf.hcb
```

**常见问题**:
- 语法错误（缺少分号、括号不匹配）
- 节点名重复
- 引用不存在的节点

### 5.2 权限配置错误

**现象**:
SELinux 拒绝访问

**排查**:
```bash
# 查看 SELinux 日志
cat /proc/kmsg | grep avc

# 查看进程安全上下文
ps -Z | grep hdf

# 查看文件安全上下文
ls -Z /system/bin/hdf_devmgr
```

## 6. 常见问题速查

| 问题 | 可能原因 | 解决方法 |
|------|----------|----------|
| 服务获取返回 NULL | 服务未注册/名称不匹配 | 检查服务注册和获取的名称 |
| IPC 调用失败 | 进程未启动/权限不足 | 检查进程状态和 SELinux |
| 驱动 Init 失败 | 配置错误/资源不足 | 检查 HCS 配置和资源分配 |
| 内存分配失败 | 内存不足/泄漏 | 检查内存使用和泄漏 |
| 设备打开失败 | 权限不足/设备忙 | 检查权限和设备状态 |

## 7. 调试开关

### 7.1 编译时调试

**开启详细日志**:
```gn
# 在 BUILD.gn 中添加
defines += [ "HDF_LOG_DEBUG" ]
```

**禁用优化**:
```gn
# 方便调试
cflags = [ "-O0", "-g" ]
```

### 7.2 运行时调试

**设置日志级别**:
```bash
# 设置 HDF 日志级别
setprop persist.hdf.log.level 3  # 0=ERROR, 1=WARN, 2=INFO, 3=DEBUG
```

## 8. 相关文档

- [项目概览](./01_Overview.md)
- [架构说明](./03_Architecture.md)
- [GN 构建](./06_GN_Build.md)
- [安全风险](./07_Security.md)

# 常见问题 (FAQ)

> 目的：收集构建、运行、调试中的常见问题与解决方案

## 1. 构建问题

### Q1: 编译报错 "undefined reference to ..."

**问题描述**:
```
ninja: error: //base/hiviewdfx/hidumper/services:hidumperservice.o
undefined reference to 'DumpManager::xxx'
```

**可能原因**:
1. 源文件未添加到 `sources` 列表
2. 依赖的静态库未链接
3. 条件编译宏未定义

**排查步骤**:
```bash
# 1. 检查 BUILD.gn 中是否包含该文件
grep -n "xxx.cpp" services/BUILD.gn

# 2. 检查依赖是否完整
gn desc out/target //base/hiviewdfx/hidumper/services:hidumperservice deps

# 3. 检查编译宏
gn args out/target --list | grep hidumper
```

**解决方案**:
1. 在 `services/BUILD.gn` 的 `sources` 中添加缺失文件
2. 添加对应的 `deps` 依赖
3. 启用相应的特性开关

---

### Q2: SA 服务无法注册

**问题描述**:
```
E/SAMgr: Failed to publish system ability
```

**可能原因**:
1. SA ID 冲突
2. 库文件路径错误
3. 初始化失败

**排查步骤**:
```bash
# 1. 检查 SA ID 是否被占用
hidumper -ls | grep 1212

# 2. 检查配置文件
cat sa_profile/1212.json

# 3. 查看日志
hilog | grep -i "hidumper"
```

**解决方案**:
1. 确认 `dump_service_id.h` 中的 SA ID 不冲突
2. 检查 `1212.json` 中的 `libpath` 是否正确
3. 在 `OnStart()` 中添加详细的错误日志

---

### Q3: 条件编译导致功能缺失

**问题描述**: 某些 dump 功能不可用，如 `--net` 无输出

**可能原因**:
- 对应的特性开关未启用 (`hidumper_netmanager_base_enable`)

**排查步骤**:
```bash
# 1. 检查编译配置
gn args out/target --list | grep netmanager

# 2. 检查 hidumper.gni
cat hidumper.gni | grep netmanager
```

**解决方案**:
```gn
# 在产品配置中启用
hidumper_netmanager_base_enable = true
```

---

## 2. 运行问题

### Q4: 执行 hidumper 无输出

**问题描述**:
```bash
$ hidumper -c
# 无任何输出
```

**排查步骤**:
```bash
# 1. 检查服务是否运行
hidumper -ls | grep 1212

# 2. 手动启动服务
hidumper -s 1212

# 3. 检查权限
id
```

**解决方案**:
1. 确保 `hidumper_service` 进程正在运行
2. 检查 `ohos.permission.DUMP` 权限
3. 查看 `hilog` 中的错误日志

---

### Q5: 特定 PID 信息获取失败

**问题描述**:
```bash
$ hidumper -p 12345
# Failed to get process info
```

**可能原因**:
1. 进程不存在
2. 权限不足
3. 进程已退出

**排查步骤**:
```bash
# 1. 确认进程存在
ps -p 12345

# 2. 检查进程用户
ls -l /proc/12345/

# 3. 尝试获取同类进程
hidumper -p $$
```

**解决方案**:
1. 确认 PID 对应的进程存在
2. 检查是否在同一个用户组
3. 部分系统进程需要 root 权限

---

### Q6: 内存信息导出失败

**问题描述**:
```bash
$ hidumper --mem-smaps 12345
# Failed to read smaps
```

**可能原因**:
1. `/proc/<pid>/smaps` 权限不足
2. 进程已退出
3. smaps 文件损坏

**排查步骤**:
```bash
# 1. 直接读取 smaps
cat /proc/12345/smaps

# 2. 检查进程状态
cat /proc/12345/status | grep State

# 3. 检查权限
ls -l /proc/12345/smaps
```

**解决方案**:
1. 确认进程存在且可访问
2. 避免对已退出进程进行查询
3. 某些内核进程不支持 smaps 读取

---

## 3. 调试问题

### Q7: 如何调试 SA 服务

**调试步骤**:

1. **修改 BUILD.gn 添加调试信息**:
```gn
hidumperservice_shared_library {
  ...
  cflags = [ "-g" ]
  ldflags = [ "-g" ]
}
```

2. **使用 gdb/lldb 附加**:
```bash
# 查找进程 PID
ps -a | grep hidumper_service

# 附加调试器
lldb -p <pid>

# 或启动调试
gdbserver :1234 --attach <pid>
```

3. **添加日志调试**:
```cpp
// 在关键路径添加日志
HILOGI("DumpRequest: fd=%{public}d, argc=%{public}d", fd, argc);
```

---

### Q8: 如何添加新的 Dumper

**实现步骤**:

1. **创建 Dumper 类** (`frameworks/native/src/executor/`):
```cpp
// my_dumper.h
class MyDumper : public BaseDumper {
public:
    int Dump(const std::vector<std::string> &args) override;
};

// my_dumper.cpp
int MyDumper::Dump(const std::vector<std::string> &args) {
    // 实现 dump 逻辑
    return 0;
}
```

2. **注册到工厂** (`factory/dumper_factory.cpp`):
```cpp
#include "my_dumper.h"
RegisterDumper("my", []() { return std::make_shared<MyDumper>(); });
```

3. **添加 BUILD.gn**:
```gn
sources += [ "my_dumper.cpp" ]
```

4. **添加命令行参数** (`common/option_args.cpp`):
```cpp
AddOption("--my", "My custom dump option");
```

---

### Q9: IPC 调用失败如何排查

**排查步骤**:

1. **检查 SA 状态**:
```bash
hidumper -ls | grep -E "1212|1215"
```

2. **检查 IPC 接口码**:
```bash
# 查看接口定义
cat interfaces/native/innerkits/include/hidumper_service_ipc_interface_code.h
```

3. **启用调试日志**:
```cpp
// 在 dump_broker_stub.cpp 添加
HILOGI("OnRemoteRequest: code=%{public}d", code);
```

4. **检查 Binder 状态**:
```bash
# 查看 binder 统计
cat /sys/kernel/debug/binder/stats
```

---

### Q10: 产物未安装到设备

**问题描述**: 构建成功但设备上无对应文件

**排查步骤**:
```bash
# 1. 检查 out 目录产物
ls -la out/{target}/hiviewdfx/hidumper/

# 2. 检查安装配置
cat services/BUILD.gn | grep install

# 3. 确认产品配置
hb env
```

**解决方案**:
1. 确认 `BUILD.gn` 中有正确的 `install` 配置
2. 检查产品 `modules.build` 配置
3. 重新执行 `hb build` 并烧录

---

## 4. 性能问题

### Q11: Dump 操作耗时过长

**可能原因**:
1. 查询进程过多
2. smaps 解析耗时
3. 网络统计阻塞

**优化建议**:
```bash
# 使用精简模式
hidumper --mem --prune

# 指定特定进程
hidumper -p <pid>

# 限制输出详细程度
hidumper -c base  # 仅 base 分类
```

---

### Q12: 服务内存占用过高

**排查步骤**:
```bash
# 查看服务内存
hidumper --mem $(pidof hidumper_service)

# 检查 fd 泄漏
hidumper -s 1212 -a "--fdinfo"
```

**解决方案**:
1. 定期调用 `ScanPidOverLimit()` 检测 FD 泄漏
2. 优化 Dumper 内部缓存
3. 服务空闲 120s 后自动卸载 (`OnIdle()`)

---

## 5. 相关资源

### 5.1 日志查看

```bash
# HiDumper 相关日志
hilog | grep -i "hidumper"

# 所有日志 (可能过多)
hilog

# 实时日志
hilogcat
```

### 5.2 关键文件路径

| 文件 | 用途 |
|-----|------|
| `/system/bin/hidumper` | CLI 工具 |
| `/system/sa/1212/` | SA 库目录 |
| `/data/log/hidumper/` | dump 输出目录 |
| `/proc/<pid>/smaps` | 进程内存映射 |

### 5.3 调试命令速查

```bash
# SA 列表
hidumper -ls

# 系统信息
hidumper -c

# 进程信息
hidumper -p

# 内存信息
hidumper --mem

# 网络信息
hidumper --net

# CPU 信息
hidumper --cpuusage

# 压缩导出
hidumper --zip
```

## 相关文档

- [构建系统](./03_Build_System.md)
- [系统架构](./01_Architecture.md)
- [安全评审](./05_Security_Review.md)

# 常见问题

## 目的

本文档汇总 hiperf 常见的构建、运行和调试问题及解决方案。

## 适用范围

- hiperf 使用者
- 问题排查人员

## 构建问题

### 1. 编译失败：找不到依赖

**现象**:
```
ERROR: //developtools/hiperf:hiperf depends on 
//developtools/hiperf:hiperf_platform_common which is not found
```

**原因**: 依赖组件未编译或路径错误

**解决**:
```bash
# 确保在正确目录执行
cd /path/to/openharmony

# 完整编译
./build.sh --product {product_name} --build-target hiperf_target
```

### 2. Host 工具编译失败

**现象**:
```
ld.lld: error: unable to find library -latomic
```

**原因**: Host 环境缺少库

**解决**:
```bash
# Linux 安装依赖
sudo apt-get install libatomic1

# 或使用静态链接
--gn-args "hiperf_target_static=true"
```

## 运行问题

### 1. 权限拒绝

**现象**:
```
error: not in developermode, exit.
```

**原因**: 未开启开发者模式或非授权 UID

**解决**:
```bash
# 开启开发者模式
param set const.security.developermode true

# 或使用 root 权限
su
hiperf ...
```

### 2. 应用采样失败

**现象**:
```
IsDebugableApp error, err: app is not debuggable
```

**原因**: 应用未标记为 debuggable

**解决**:
- 确保应用 `config.json` 中设置 `"debuggable": true`
- 仅支持调试版本应用

### 3. 找不到 hdc

**现象**:
```
Exception: Can't find hdc_std in PATH environment.
```

**原因**: PATH 环境变量未包含 hdc

**解决**:
```bash
# Windows
set PATH=%PATH%;C:\path\to\hdc

# Linux/macOS
export PATH=$PATH:/path/to/hdc
```

## 采样问题

### 1. 采样数据为空

**现象**: perf.data 文件很小或无采样数据

**原因**:
- 采样频率过低
- 目标进程未运行
- 权限不足

**解决**:
```bash
# 检查目标进程
ps -ef | grep target

# 提高采样频率
hiperf record -p {pid} -f 4000 -d 10

# 检查权限
cat /proc/sys/kernel/perf_event_paranoid
# 应小于 1
```

### 2. 符号解析失败

**现象**: 报告中显示 "[unknown]"

**原因**: 缺少符号表或符号表不匹配

**解决**:
```bash
# 收集符号表
python script/recv_binary_cache.py \
    -i perf.data \
    -l /path/to/lib.unstripped \
    -l /path/to/exe.unstripped

# 确认符号文件存在
ls binary_cache/
```

### 3. 调用链不完整

**现象**: 火焰图显示断开的调用链

**原因**:
- 使用 FP 回溯但编译时未保留帧指针
- DWARF 信息缺失

**解决**:
```bash
# 使用 DWARF 回溯
hiperf record -s dwarf ...

# 或确保目标使用 -fno-omit-frame-pointer 编译
```

## 性能问题

### 1. 采样开销大

**现象**: 系统卡顿，CPU 占用高

**原因**: 采样频率过高

**解决**:
```bash
# 降低采样频率
hiperf record -f 1000 ...  # 默认 4000

# 限制 CPU 使用率
hiperf record --cpu-limit 10 ...  # 默认 25
```

### 2. 内存不足

**现象**: OOM，进程被杀死

**原因**: 采样数据量大

**解决**:
```bash
# 减少 mmap 页数
hiperf record -m 256 ...  # 默认 1024

# 设置数据大小限制
hiperf record --data-limit 100M ...
```

## 调试技巧

### 1. 启用调试日志

```bash
# 启用 DEBUG 级别日志
hiperf --debug record ...

# 启用 VERBOSE 级别日志
hiperf --verbose record ...

# 启用 MUCH 级别日志
hiperf --much record ...

# 混合输出到屏幕
hiperf --mixlog record ...

# 指定模块日志
hiperf --logtag PerfEvents,VirtualRuntime record ...
```

### 2. 检查数据文件

```bash
# 查看数据文件头
hiperf dump --header perf.data

# 查看事件统计
hiperf dump --stat perf.data

# 导出原始数据
hiperf dump --data perf.data > data.txt
```

### 3. 验证符号解析

```bash
# 查看加载的符号文件
hiperf report --dump-symbols perf.data

# 详细报告
hiperf report --verbose perf.data
```

## 相关跳转

- [项目定位](01_Overview.md) - 了解约束条件
- [安全风险](08_Security.md) - 了解权限要求
- [对外 API](04_Public_API.md) - API 使用说明

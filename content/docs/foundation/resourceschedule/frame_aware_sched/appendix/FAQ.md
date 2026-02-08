# 常见问题与排查

## 构建问题

### Q1: 编译时提示找不到头文件

**问题**: `fatal error: 'frame_info_const.h' file not found`

**解决**:
```bash
# 确保在 OpenHarmony 构建环境中编译
source build.sh
hb set
hb build -f
```

### Q2: 链接失败提示 `undefined reference to 'EnableRtg'`

**问题**: 链接 `librtg_interface.z.so` 失败

**解决**:
- 检查 `BUILD.gn` 中是否正确声明 `external_deps`
- 确保链接了正确的库

## 运行问题

### Q3: RTG 初始化失败

**问题**: `rtg Open fail, errno = xxx`

**解决**:
1. 检查 `/proc/self/sched_rtg_ctrl` 是否存在
2. 确认内核是否支持 RTG 功能
3. 检查权限：`ls -l /proc/self/sched_rtg_ctrl`

### Q4: 帧事件不上报

**问题**: 调用 `BeginFlushAnimation()` 无反应

**解决**:
1. 检查 `FrameUiIntf::Init()` 是否调用成功
2. 查看日志：`hilog | grep "ueaClient-FrameUiIntf"`
3. 确认 `inited` 标志位是否为 `true`

### Q5: RTG 组操作失败

**问题**: `add thread to rtg failed`

**解决**:
1. 检查线程 ID 是否有效
2. 查看内核日志：`dmesg | grep sched_rtg`
3. 确认 RTG 功能是否已启用

## 调试方法

### 查看日志

```bash
# 查看所有 RME 相关日志
hilog | grep -E "RME|rtg|FrameUi|FrameMsg"

# 实时查看日志
hilog | grep "rtg_interface"
```

### 追踪调用链

```bash
# 使用 htrace 追踪
hitrace --trace_schedule on
```

### 检查 RTG 状态

```bash
# 查看当前 RTG 组
cat /proc/sched_rtg_ctrl
```

## 性能问题

### Q6: 帧率下降

**问题**: 启用 frame_aware_sched 后帧率降低

**解决**:
1. 检查 `hwrme.xml` 配置是否正确
2. 确认 `<enable>` 是否设置为 `1`
3. 查看是否有过多的 RTG 操作日志

## 已知限制

1. 仅支持 standard 系统类型
2. 需要内核支持 RTG 和 cgroup 功能
3. 不支持 32 位系统

# 关键配置参数

## hwrme.xml 配置项

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `<version>` | 1.0 | 配置版本 |
| `<log_open>` | 0 | 日志开关（0=关闭，1=开启） |
| `<enable>` | 1 | RME 功能使能（0=关闭，1=开启） |
| `<SOC>` | 990 | SoC 型号 |
| `<frame_sched_reset_count>` | 1 | 帧调度重置计数 |
| `<fps_list>` | 60 90 | 支持的帧率列表 |
| `<render_type>` | 0 1 2 | 渲染类型列表 |
| `<framedetect>` | - | 帧检测配置 |
| `<default_util>` | 600 | 默认调度利用率 |

## 代码中的常量

### rtg_interface.h

| 常量 | 值 | 说明 |
|------|-------|------|
| `MAX_TID_NUM` | 5 | 单次添加的最大线程数 |
| `MAX_SUBPROCESS_NUM` | 8 | 最大子进程数 |
| `MULTI_FRAME_NUM` | 5 | 多帧数量 |

### rme_constants.h

| 常量 | 值 | 说明 |
|------|-------|------|
| `RT_PRIO` | 0 | 实时优先级 |
| `RT_NUM` | 4 | 实时线程数 |

## 编译开关

### BUILD.gn 编译选项

| 编译选项 | 说明 |
|----------|------|
| `-fstack-protector-strong` | 栈保护 |
| `-Wno-shift-negative-value` | 禁止负数移位警告 |
| `branch_protector_ret="pac_ret"` | PACRET 分支保护 |

## 日志标签

| 标签 | 模块 | 说明 |
|------|------|------|
| `rtg_interface` | RTG 控制 | RTG 接口日志 |
| `ueaClient-FrameUiIntf` | UI 接口 | UI 帧事件客户端日志 |
| `ueaServer-FrameMsgIntf` | 消息接口 | 帧感知服务端日志 |

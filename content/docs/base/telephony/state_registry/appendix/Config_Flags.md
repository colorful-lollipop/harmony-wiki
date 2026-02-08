# 配置开关与宏定义

## 概述

本文档汇总 State Registry 模块中所有可配置的编译开关和运行时配置，用于定制功能和调试问题。

## 编译时配置

### 全局构建参数

**文件位置**：`BUILD.gn`

#### telephony_state_registry_hicollie_able

| 属性 | 值 |
|------|-----|
| 类型 | boolean |
| 默认值 | true |
| 作用 | 启用 Hicollie 性能追踪功能 |
| 开启效果 | 在日志中输出性能追踪信息 |

```gn
declare_args() {
  telephony_state_registry_hicollie_able = true
}
```

#### telephony_extra_defines

| 属性 | 值 |
|------|-----|
| 类型 | list |
| 默认值 | [] |
| 作用 | 额外的编译宏定义 |

```gn
if (defined(global_parts_info) &&
    defined(global_parts_info.telephony_telephony_enhanced)) {
  telephony_extra_defines += [ "OHOS_BUILD_ENABLE_TELEPHONY_EXT" ]
  telephony_extra_defines += [ "OHOS_BUILD_ENABLE_TELEPHONY_VSIM" ]
}
```

### 条件编译宏

| 宏名 | 条件 | 作用 |
|------|------|------|
| `OHOS_BUILD_ENABLE_TELEPHONY_EXT` | telephony_enhanced | 启用电信扩展功能 |
| `OHOS_BUILD_ENABLE_TELEPHONY_VSIM` | telephony_enhanced | 启用虚拟 SIM 功能 |
| `HICOLLIE_ENABLE` | hicollie_able | 启用性能分析 |

### 编译器标志

```gn
cflags_cc = [
  "-O2",
  "-D_FORTIFY_SOURCE=2",
]
```

| 标志 | 作用 |
|------|------|
| `-O2` | 优化级别 2 |
| `-D_FORTIFY_SOURCE=2` | 增强运行时缓冲区溢出检测 |

### Sanitizer 配置

```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

| 配置项 | 值 | 作用 |
|--------|-----|------|
| cfi | true | 启用控制流完整性检查 |
| cfi_cross_dso | true | 启用跨 DSO 的 CFI 检查 |
| debug | false | 禁用调试模式 |

**证据来源**：`BUILD.gn` 根构建文件。

## 运行时配置

### 日志配置

**日志标签**：
```cpp
#define TELEPHONY_LOG_TAG "StateRegistry"
```

**日志域**：
```cpp
#define LOG_DOMAIN 0xD001F07
```

### SA 配置

**配置文件**：`sa_profile/state_registry_sa_profile.xml`

| 配置项 | 值 | 说明 |
|--------|-----|------|
| name | StateRegistrySA | SA 名称 |
| ondemand | true | 按需启动 |
| process | telephony | 所在进程 |

### 事件类型配置

| 事件名 | 类型值 | 默认启用 |
|--------|--------|----------|
| networkStateChange | 0 | 是 |
| signalInfoChange | 1 | 是 |
| cellInfoChange | 2 | 是 |
| cellularDataConnectionStateChange | 3 | 是 |
| cellularDataFlowChange | 4 | 是 |
| callStateChange | 5 | 是 |
| simStateChange | 6 | 是 |

## 功能开关

### 调试模式

| 开关 | 位置 | 默认值 | 作用 |
|------|------|--------|------|
| LOG_DEBUG | 代码中 | false | 调试日志开关 |
| DUMP_ENABLED | 代码中 | true | Dump 功能开关 |

### 性能监控

| 监控项 | 开关 | 输出位置 |
|--------|------|----------|
| 回调延迟 | HICOLLIE_ENABLE | hilog |
| 内存使用 | HICOLLIE_ENABLE | hilog |
| IPC 耗时 | HICOLLIE_ENABLE | hilog |

## 组件配置

### 依赖组件开关

| 组件 | 开关 | 作用 |
|------|------|------|
| access_token | external_deps | 权限校验 |
| hilog | external_deps | 日志输出 |
| ipc | external_deps | IPC 框架 |
| safwk | external_deps | SA 框架 |
| samgr | external_deps | 服务管理 |

## 配置优先级

```
1. 全局 args (最高优先级)
   └── global_parts_info
   
2. declare_args()
   └── telephony_state_registry_hicollie_able
   
3. 条件判断
   └── telephony_extra_defines
   
4. 运行时配置
   └── SA Profile
```

## 配置验证

### 检查构建配置

```bash
# 生成构建配置
gn gen out/<product> --args="telephony_state_registry_hicollie_able=true"

# 查看配置
gn args out/<product> --list | grep telephony
```

### 检查运行时配置

```bash
# 查看 SA 配置
cat out/<product>/packages/system/etc/sa_config/state_registry_sa_profile.xml

# 查看系统属性
hdc shell
getprop telephony.state_registry.enabled
```

## 相关文档

- [GN 构建](../../05_GN_Build.md)
- [编译产物](../../06_Build_Artifacts.md)
- [故障排查](../../08_Troubleshooting.md)

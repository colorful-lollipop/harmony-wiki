# 07_Troubleshooting - 故障排查

## 1. 构建问题

### 1.1 找不到依赖

**问题**: 编译提示找不到 `grpc` / `protobuf`

**解决方案**:
```bash
# 同步依赖
hb sync -p

# 检查环境
hb env
```

### 1.2 Native Hook 编译失败

**问题**: `native_hook` 编译报错与 ASan 相关

**原因**: ASan 与 Hook 不兼容

**解决方案**:
```bash
# 使用非 ASan 构建
gn gen out/default --args="enable_asan=false"
ninja -C out/default native_hook
```

### 1.3 插件版本脚本错误

**问题**: `version_script` 文件找不到

**解决方案**:
```gn
# 使用相对路径时，确保路径正确
ohos_shared_library("myplugin") {
  # 错误: version_script = "libmyplugin.map"
  # 正确:
  version_script = "//path/to/myplugin:libmyplugin.map"
}
```

---

## 2. 运行时问题

### 2.1 hiprofiler_cmd 无法连接服务

**问题**: 执行 `hiprofiler_cmd` 提示无法连接 Profiler Service

**排查步骤**:

```bash
# 1. 检查服务是否运行
hilog | grep -i profiler

# 2. 检查 SA 是否注册
hidumper --systemability 6105  # DFX_SYS_PROFILER_ABILITY_ID

# 3. 手动启动服务
hilog | grep -i "profiler service started"
```

### 2.2 插件加载失败

**问题**: 插件加载失败，`dlsym` 返回 nullptr

**排查步骤**:

```bash
# 1. 检查插件是否存在
ls -la /system/lib/plugin/

# 2. 检查插件依赖
ldd /system/lib/plugin/cpudataplugin.so

# 3. 检查 SELinux 权限
dmesg | grep -i profiler
```

### 2.3 内存数据采集异常

**问题**: `getPss()` / `getNativeHeapSize()` 返回 0 或异常值

**排查步骤**:

```javascript
// 1. 检查是否在应用进程内调用
// N-API 仅在应用进程内有效

// 2. 检查权限
// 需要 ohos.permission.ENABLE_PROFILER

// 3. 调试日志
hilog | grep -i hidebug
```

---

## 3. 调试方法

### 3.1 日志查看

```bash
# 查看所有 profiler 相关日志
hilog | grep -E "(profiler|hidebug|hiprofiler)"

# 查看 Profiler Service 日志
hilog | grep -E "(profiler_service|PluginManager)"

# 查看插件日志
hilog | grep -E "(cpu_plugin|memory_plugin|ftrace)"
```

### 3.2 服务状态检查

```bash
# 查看 SA 列表
hidumper --sa

# 查看 Profiler SA 详情
hidumper --systemability 6105

# 查看内存使用
hidumper -m hiprofiler
```

### 3.3 数据采集验证

```javascript
// 基础测试代码
import hidebug from '@ohos.hidebug';

try {
    // 测试 CPU 使用率
    const cpu = hidebug.getCpuUsage();
    console.log(`CPU: ${cpu}%`);

    // 测试内存
    const pss = hidebug.getPss();
    console.log(`PSS: ${pss}`);

    // 测试调试状态
    const isDebug = hidebug.isDebugState();
    console.log(`Debug: ${isDebug}`);
} catch (e) {
    console.error(`Error: ${e.code} - ${e.message}`);
}
```

---

## 4. 常见错误码

### 4.1 N-API 错误码

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| 201 | PERMISSION_ERROR | 检查权限声明 |
| 401 | PARAMETER_ERROR | 检查输入参数 |
| 801 | VERSION_ERROR | SDK 版本不匹配 |
| 11400101 | SYSTEM_ABILITY_NOT_FOUND | 检查 SA 是否注册 |
| 11400103 | WITHOUT_WRITE_PERMISSON | 检查文件路径权限 |
| 11400110 | LOW_DISK_SPACE | 清理磁盘空间 |

### 4.2 服务端错误码

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| -1 | 通用失败 | 检查日志 |
| -2 | 插件加载失败 | 检查插件依赖 |
| -3 | 会话不存在 | 检查 sessionId |
| -4 | 缓冲区不足 | 增加 bufferPages |

---

## 5. 性能问题

### 5.1 性能采集影响

**问题**: 开启性能采集后应用变慢

**优化建议**:

| 采集项 | 性能影响 | 优化建议 |
|--------|----------|----------|
| CPU 采样 | 低 | 采样间隔设大 (>100ms) |
| 内存追踪 | 中 | 仅追踪必要进程 |
| Ftrace | 高 | 限制事件类型 |
| Native Hook | 高 | 减少采样深度 |

### 5.2 内存占用

**问题**: 采集数据占用过多内存

**解决方案**:

```javascript
// 1. 减少缓冲区大小
const config = {
    buffers: {
        pages: 1024  // 减少 (默认 16384)
    }
};

// 2. 缩短采样时长
sampleDuration: 10000  // 10秒

// 3. 限制采集插件数量
plugin_configs: [
    // 仅启用必要插件
]
```

---

## 6. 定位路径速查表

| 问题现象 | 查看日志 | 检查配置 | 其他 |
|----------|----------|----------|------|
| 服务无法启动 | `hilog \| grep profiler` | - | 检查 SA ID |
| 插件加载失败 | `hilog \| grep dlopen` | 检查插件路径 | `ldd` 检查依赖 |
| 数据为空 | `hilog \| grep PluginSession` | 检查 bufferPages | 检查采集时长 |
| 权限错误 | - | 检查 permission | `hidumper --sa` |
| 性能差 | `hilog \| grep Perf` | 减少采样频率 | 限制插件数量 |

---

## 7. 相关跳转

| 主题 | 链接 |
|------|------|
| 项目概览 | [00_Overview.md](./00_Overview.md) |
| N-API 接口 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) |
| 安全评审 | [06_Security_Review.md](./06_Security_Review.md) |
| 构建配置 | [05_Build_System.md](./05_Build_System.md) |

---

*最后更新: 2026-02-06*

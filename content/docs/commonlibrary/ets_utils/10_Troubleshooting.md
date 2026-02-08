# 故障排除

> 常见构建、运行、调试问题与解决方案

## 1. 构建问题

### 1.1 GN 找不到依赖

**错误信息**:
```
error: cannot find dependency: hilog:libhilog
```

**解决方案**:
```bash
# 检查依赖是否在系统中
ls /system/lib/module/ | grep hilog

# 重新同步依赖
hb sync

# 或检查 bundle.json 配置
cat bundle.json | grep external_deps
```

### 1.2 编译超时

**错误信息**:
```
ninja: build stopped: subcommand timed out.
```

**解决方案**:
```bash
# 增加超时时间
hb build -T 60m

# 只构建指定模块
hb build --parts ets_utils

# 清理后重新构建
hb clean
hb build --parts ets_utils
```

### 1.3 字节码生成失败

**错误信息**:
```
es2abc: error: failed to translate ts to abc
```

**解决方案**:
```bash
# 检查 TypeScript 语法
tsc --noEmit <module>.ts

# 检查 import 路径
# 确保路径正确且文件存在
```

## 2. 运行问题

### 2.1 模块加载失败

**错误信息**:
```
error: Failed to load native module: libbuffer.so
dlopen failed: library "libnapi.so" not found
```

**解决方案**:
```typescript
// 1. 检查设备架构
adb shell cat /proc/cpuinfo | grep "model name"

// 2. 检查依赖库
adb shell ldd /system/lib/module/libbuffer.so

// 3. 确保系统镜像包含依赖
// 重新刷写系统镜像
```

### 2.2 API 未定义

**错误信息**:
```
Cannot find module '@ohos.buffer' or its corresponding type declarations
```

**解决方案**:
```typescript
// 1. 检查 API 可用性 (系统版本)
let buffer = (globalThis as any).buffer;
if (buffer) {
    // 使用
}

// 2. 更新 SDK
// 下载匹配的 SDK 版本

// 3. 检查权限
// 确保 module.json5 中声明了 syscap
```

### 2.3 Worker 启动失败

**错误信息**:
```
Worker initialization failed: Script file not found
```

**解决方案**:
```typescript
// 1. 检查 Worker 文件路径
// Stage 模型
new Worker('entry/ets/workers/worker.ts')

// FA 模型  
new Worker('workers/worker.js')

// 2. 检查 build-profile.json5 配置
{
  "buildOption": {
    "sourceOption": {
      "workers": [
        "./src/main/ets/workers/worker.ts"
      ]
    }
  }
}

// 3. 检查文件权限
adb shell ls -la path/to/worker.js
```

## 3. 调试问题

### 3.1 日志输出

**启用调试日志**:
```cpp
// 在代码中启用
HILOG_DEBUG("Buffer", "Debug message");

// 运行时过滤
hilog -w D <domain> <process_id>
```

**查看日志**:
```bash
# 过滤 ets_utils 日志
adb logcat | grep -E "Buffer|URL|Worker"

# 保存日志到文件
adb logcat > logcat.txt
```

### 3.2 Native 调试

**使用 lldb 调试**:
```bash
# 连接设备
lldb-server platform --server --listen "*:12345"

# 本地连接
lldb
(lldb) platform connect connect://localhost:12345

# 加载符号
(lldb) target create libbuffer.so
(lldb) settings set symbol-file libbuffer.so
```

### 3.3 性能分析

**使用 hiTraceMeter**:
```typescript
import hiTraceMeter from '@ohos.hiTraceMeter'

// 标记代码段
hiTraceMeter.startTrace('myFunction', 1001)

// 执行代码
processData()

hiTraceMeter.finishTrace('myFunction', 1001)

// 查看结果
adb shell hiprint -t 1001
```

## 4. 常见错误码

### 4.1 参数错误 (401)

**含义**: 参数类型不匹配

**示例**:
```typescript
// 错误
new URL(123)  // number 不是 string

// 正确
new URL('https://example.com')
```

### 4.2 内存错误

**常见错误**:
```
Out of memory
Buffer allocation failed
```

**解决**:
```typescript
// 检查分配大小
let MAX_SIZE = 1024 * 1024;  // 1MB

function safeAlloc(size: number): Buffer | null {
    if (size > MAX_SIZE) {
        return null;
    }
    return buffer.alloc(size);
}
```

### 4.3 进程错误

**常见错误**:
```
Process not found (ESRCH)
Permission denied (EPERM)
```

**解决**:
```typescript
import Process from '@ohos.process'

try {
    Process.kill(pid, 9)
} catch (e) {
    console.error('Kill failed:', e.code)
}
```

## 5. 性能优化

### 5.1 Buffer 优化

```typescript
// ❌ 低效: 多次分配
for (let i = 0; i < 1000; i++) {
    let buf = buffer.alloc(1024)
    // 处理
}

// ✅ 高效: 复用
let buf = buffer.alloc(1024 * 1000)
for (let i = 0; i < 1000; i++) {
    // 复用同一 Buffer
}
```

### 5.2 Worker 优化

```typescript
// ❌ 低效: 频繁创建
for (let i = 0; i < 100; i++) {
    let w = new Worker('task.worker')
    w.postMessage(data)
    w.terminate()
}

// ✅ 高效: 复用
let worker = new Worker('task.worker')
for (let i = 0; i < 100; i++) {
    worker.postMessage(data)
    // 等待结果
}
// 最后统一 terminate
```

### 5.3 容器选择

| 场景 | 推荐容器 | 原因 |
|------|----------|------|
| 随机访问 | ArrayList | O(1) 访问 |
| 头尾操作 | Deque | O(1) 头尾 |
| 键值存储 | HashMap | O(1) 查找 |
| 有序遍历 | TreeMap | 保持顺序 |

## 6. 资源

### 6.1 日志工具

```bash
# hilog
adb shell hilog

# dmesg
adb shell dmesg | grep -i error

#崩溃日志
adb shell crashlog
```

### 6.2 调试命令

```bash
# 查看进程
adb shell ps -A | grep ets

# 查看内存
adb shell cat /proc/<pid>/status

# 查看打开文件
adb shell lsof -p <pid>
```

### 6.3 相关文档

- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [08_Compilation_Artifacts.md](./08_Compilation_Artifacts.md) - 编译产物
- [09_Security_Review.md](./09_Security_Review.md) - 安全分析

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

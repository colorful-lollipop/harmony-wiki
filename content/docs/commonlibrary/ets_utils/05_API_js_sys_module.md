# API 参考: js_sys_module

> 进程管理、定时器、控制台等系统级 API

## 模块概述

js_sys_module 提供系统级操作能力：
- **Process**: 进程管理、信号发送、工作目录
- **Timer**: 定时器 (setTimeout, setInterval)
- **Console**: 控制台日志输出
- **DFX**: 调试与性能追踪

## 1. Process 模块

进程管理 API。

**命名空间**: `Process`

### 1.1 属性

| 属性 | 类型 | 只读 | 说明 |
|------|------|------|------|
| uid | number | 是 | 用户 ID |
| gid | number | 是 | 组 ID |
| euid | number | 是 | 有效用户 ID |
| egid | number | 是 | 有效组 ID |
| groups | number[] | 是 | 附加组 ID |
| pid | number | 是 | 进程 ID |
| ppid | number | 是 | 父进程 ID |
| tid | number | 是 | 线程 ID |

**示例**:
```typescript
import Process from '@ohos.process'

console.log(`PID: ${Process.pid}`)
console.log(`PPID: ${Process.ppid}`)
console.log(`UID: ${Process.uid}`)
console.log(`TID: ${Process.tid}`)
```

### 1.2 方法

#### cwd()

获取当前工作目录。

```typescript
Process.cwd(): string
```

#### chdir(dir)

更改工作目录。

```typescript
Process.chdir(dir: string): void
```

#### uptime()

获取系统运行时间 (秒)。

```typescript
Process.uptime(): number
```

#### kill(pid, signal)

发送信号给进程。

```typescript
Process.kill(pid: number, signal: number): boolean
```

**参数**:
| 参数 | 类型 | 说明 |
|------|------|------|
| pid | number | 目标进程 ID |
| signal | number | 信号值 (如 9=SIGKILL) |

#### abort()

立即终止进程并生成 core dump。

```typescript
Process.abort(): void
```

#### exit(code)

退出进程。

```typescript
Process.exit(code: number): void
```

#### on(type, listener)

注册事件监听。

```typescript
Process.on(type: string, listener: Function): void
```

#### off(type)

取消事件监听。

```typescript
Process.off(type: string): boolean
```

### 1.3 系统查询方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| isIsolatedProcess | 无 | boolean | 是否隔离进程 |
| is64Bit | 无 | boolean | 是否 64 位环境 |
| isAppUid | uid: number | boolean | UID 是否属于应用 |
| getUidForName | name: string | number | 用户名转 UID |
| getThreadPriority | tid: number | number | 获取线程优先级 |
| getSystemConfig | name: number | number | 获取系统配置 |
| getEnvironmentVar | name: string | string | 获取环境变量 |
| getAvailableCores | 无 | number[] | 可用 CPU 核心 |
| getStartRealtime | 无 | number | 启动实时时间 |
| getPastCputime | 无 | number | 已用 CPU 时间 |

**示例**:
```typescript
import Process from '@ohos.process'

// 获取环境变量
let home = Process.getEnvironmentVar('HOME')
console.log(`HOME: ${home}`)

// 检查系统
console.log(`64-bit: ${Process.is64Bit()}`)
console.log(`Cores: ${Process.getAvailableCores()}`)

// 获取 CPU 时间
console.log(`Uptime: ${Process.uptime()} seconds`)
```

### 1.4 ChildProcess 子进程

通过 `Process.runCmd()` 创建子进程。

#### runCmd()

运行 shell 命令。

```typescript
Process.runCmd(command: string, options?: RunCmdOptions): ChildProcess
```

**RunCmdOptions**:
| 属性 | 类型 | 说明 |
|------|------|------|
| timeout | number | 超时时间 (毫秒) |
| killSignal | number \| string | 终止信号 |
| maxBuffer | number | 输出缓冲区上限 |

#### ChildProcess 属性

| 属性 | 类型 | 只读 | 说明 |
|------|------|------|------|
| pid | number | 是 | 子进程 ID |
| ppid | number | 是 | 父进程 ID |
| killed | boolean | 是 | 是否被终止 |
| exitCode | number | 是 | 退出码 |

#### ChildProcess 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| wait() | Promise\<number\> | 等待进程结束 |
| getOutput() | Promise\<Uint8Array\> | 获取标准输出 |
| getErrorOutput() | Promise\<Uint8Array\> | 获取标准错误 |
| kill(signal) | void | 发送信号 |
| close() | void | 关闭进程 |

**完整示例**:
```typescript
import Process from '@ohos.process'

// 运行命令
let child = Process.runCmd('echo "Hello World"', { timeout: 5000 })

// 等待完成
child.wait().then(code => {
    console.log(`Exit code: ${code}`)
})

// 获取输出
child.getOutput().then(output => {
    console.log(`Output: ${new TextDecoder().decode(output)}`)
})

// 带超时
let longRunning = Process.runCmd('sleep 10', { 
    timeout: 1000,
    killSignal: 9  // SIGKILL
})
```

## 2. Timer 模块

定时器 API。

**命名空间**: 全局

### 2.1 函数

#### setTimeout()

延迟执行。

```typescript
setTimeout(handler: Function | string, delay?: number, ...arguments: any[]): number
```

#### setInterval()

周期执行。

```typescript
setInterval(handler: Function | string, delay?: number, ...arguments: any[]): number
```

#### clearTimeout()

清除定时器。

```typescript
clearTimeout(timeoutID?: number): void
```

#### clearInterval()

清除周期定时器。

```typescript
clearInterval(intervalID?: number): void
```

**示例**:
```typescript
// 延迟执行
let id1 = setTimeout(() => {
    console.log('1秒后执行')
}, 1000)

// 周期执行
let id2 = setInterval(() => {
    console.log('每2秒执行一次')
}, 2000)

// 清除
clearTimeout(id1)
clearInterval(id2)
```

## 3. Console 模块

控制台输出。

**命名空间**: `console`

### 3.1 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| log | ...args | 信息日志 |
| info | ...args | 信息 |
| warn | ...args | 警告 |
| error | ...args | 错误 |
| debug | ...args | 调试日志 |
| assert | value, ...args | 断言 |
| count | label? | 计数 |
| countReset | label? | 重置计数 |
| dir | obj, options? | 对象格式化 |
| table | data | 表格输出 |
| time | label? | 开始计时 |
| timeEnd | label? | 结束计时 |
| timeLog | label?, ...args | 计时日志 |
| trace | ...args | 堆栈跟踪 |
| group | ...args | 日志分组 |
| groupCollapsed | ...args | 折叠分组 |
| groupEnd | 无 | 结束分组 |

**示例**:
```typescript
console.log('普通日志')
console.info('信息')
console.warn('警告')
console.error('错误')
console.debug('调试')

// 断言
console.assert(condition, '条件失败时的消息')

// 计数
console.count('click')
console.count('click')
console.countReset('click')

// 表格
console.table([
    { name: 'A', value: 1 },
    { name: 'B', value: 2 }
])

// 计时
console.time('operation')
// ... 执行操作
console.timeLog('operation')  // 输出经过时间
console.timeEnd('operation')  // 结束并输出

// 分组
console.group('Group 1')
console.log('In group')
console.groupCollapsed('Nested')
console.log('Collapsed content')
console.groupEnd()
```

## 4. DFX 模块

调试与性能追踪。

### 4.1 hiTraceMeter

性能追踪。

```typescript
import hiTraceMeter from '@ohos.hiTraceMeter'

// 开始追踪
hiTraceMeter.startTrace('myTask', 1001)

// 执行任务
// ...

// 结束追踪
hiTraceMeter.finishTrace('myTask', 1001)
```

## 相关文档

- [01_Overview.md](./01_Overview.md) - 组件定位
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [06_API_js_concurrent_module.md](./06_API_js_concurrent_module.md) - 并发 API

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

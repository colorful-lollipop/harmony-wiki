# API 参考: js_concurrent_module

> Worker 多线程、Taskpool 任务池等并发编程 API

## 模块概述

js_concurrent_module 提供并发编程能力：
- **Worker**: 多线程通信
- **Taskpool**: 任务调度池
- **Utils**: 并发工具类 (锁、条件变量)

## 1. Worker 模块

### 1.1 Worker 类

Worker 线程用于执行耗时操作，避免阻塞主线程。

**命名空间**: `@ohos.worker`

#### 构造函数

```typescript
new Worker(scriptURL: string, options?: WorkerOptions): Worker
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| scriptURL | string | 是 | Worker 脚本路径 |
| options | WorkerOptions | 否 | 选项 |

**WorkerOptions**:
| 属性 | 类型 | 说明 |
|------|------|------|
| name | string | Worker 名称 |
| sharedBuffer | ArrayBuffer | 共享内存 |

**示例**:
```typescript
import worker from '@ohos.worker'

// Stage 模型
let worker1 = new worker.Worker('entry/ets/workers/worker.ts')

// FA 模型
let worker2 = new worker.Worker('workers/worker.js')

// 带选项
let worker3 = new worker.Worker('workers/worker.ts', {
    name: 'my-worker'
})
```

### 1.2 Worker 方法

#### postMessage()

向 Worker 发送消息。

```typescript
worker.postMessage(message: Object, options?: PostMessageOptions): void
worker.postMessage(message: Object, transfer: ArrayBuffer[]): void
```

#### terminate()

终止 Worker。

```typescript
worker.terminate(): void
```

### 1.3 Worker 事件

| 事件 | 回调参数 | 说明 |
|------|----------|------|
| onmessage | MessageEvent | 收到消息 |
| onerror | ErrorEvent | 发生错误 |
| onmessageerror | MessageEvent | 消息序列化失败 |
| onexit | number | 线程退出 |

**示例**:
```typescript
import worker from '@ohos.worker'

let myWorker = new worker.Worker('workers/worker.ts')

// 监听消息
myWorker.onmessage = (e) => {
    console.log('收到消息:', e.data)
}

// 监听错误
myWorker.onerror = (e) => {
    console.error('Worker 错误:', e.message)
}

// 监听退出
myWorker.onexit = (code) => {
    console.log('Worker 退出，代码:', code)
}

// 发送消息
myWorker.postMessage({ type: 'task', data: [1, 2, 3] })

// 发送 ArrayBuffer (零拷贝)
let buffer = new ArrayBuffer(1024)
myWorker.postMessage(buffer, [buffer])

// 终止
// myWorker.terminate()
```

### 1.4 parentPort 对象

Worker 线程内部用于与主线程通信。

**示例 (worker.ts)**:
```typescript
import worker from '@ohos.worker'

let parentPort = worker.parentPort

// 监听消息
parentPort.onmessage = (e) => {
    let result = e.data * 2
    
    // 发送结果回主线程
    parentPort.postMessage({ result })
}

// 关闭 Worker
parentPort.close()
```

### 1.5 Worker 完整示例

**主线程 (main.ts)**:
```typescript
import worker from '@ohos.worker'

let workerThread = new worker.Worker('workers/compute.ts')

// 发送计算任务
workerThread.postMessage({
    type: 'compute',
    data: { x: 10, y: 20 }
})

// 处理结果
workerThread.onmessage = (e) => {
    console.log('计算结果:', e.data.result)
    
    // 发送更多任务
    workerThread.postMessage({
        type: 'compute',
        data: { x: 100, y: 200 }
    })
}

// 错误处理
workerThread.onerror = (e) => {
    console.error('Worker 错误:', e.message)
}

// 结束
setTimeout(() => {
    workerThread.terminate()
}, 30000)
```

**Worker 线程 (workers/compute.ts)**:
```typescript
import worker from '@ohos.worker'

let parentPort = worker.parentPort

parentPort.onmessage = (e) => {
    let { type, data } = e.data
    
    if (type === 'compute') {
        let result = data.x + data.y
        
        parentPort.postMessage({
            type: 'result',
            result: result
        })
    }
}
```

### 1.6 Worker 文件配置

**Stage 模型 build-profile.json5**:
```json
{
  "buildOption": {
    "sourceOption": {
      "workers": [
        "./src/main/ets/workers/worker.ts"
      ]
    }
  }
}
```

## 2. Taskpool 模块

任务调度池，用于并行执行任务。

### 2.1 Taskpool 类

```typescript
import taskpool from '@ohos.taskpool'
```

#### execute()

执行任务。

```typescript
taskpool.execute(task: Function, ...args: any[]): Promise<any>
```

#### executeDelayed()

延迟执行任务。

```typescript
taskpool.executeDelayed(delay: number, task: Function, ...args: any[]): Task
```

#### cancel()

取消任务。

```typescript
taskpool.cancel(task: Task): boolean
```

### 2.2 TaskGroup 类

任务组。

```typescript
let group = new taskpool.TaskGroup()
group.add(task)
await group.execute()
```

### 2.3 Taskpool 示例

```typescript
import taskpool from '@ohos.taskpool'

// 定义任务函数
function computeHeavyTask(input: number): number {
    let result = 0
    for (let i = 0; i < input * 1000000; i++) {
        result += Math.sqrt(i)
    }
    return result
}

// 执行任务
taskpool.execute(computeHeavyTask, 100)
    .then(result => {
        console.log('计算结果:', result)
    })
    .catch(err => {
        console.error('任务失败:', err)
    })

// 使用任务组
async function runParallel() {
    let group = new taskpool.TaskGroup()
    group.add(() => computeHeavyTask(50))
    group.add(() => computeHeavyTask(80))
    group.add(() => computeHeavyTask(100))
    
    let results = await group.execute()
    console.log('所有结果:', results)
}
```

## 3. 并发工具

### 3.1 AsyncLock 异步锁

```typescript
import { AsyncLock } from '@ohos.util'

let lock = new AsyncLock()

async function criticalSection() {
    // 获取锁
    await lock.lock()
    
    try {
        // 临界区代码
        console.log('执行临界区')
    } finally {
        // 释放锁
        lock.unlock()
    }
}
```

### 3.2 ConditionVariable 条件变量

```typescript
import { ConditionVariable } from '@ohos.util'

let cv = new ConditionVariable()

// 等待条件
await cv.wait()

// 通知一个
cv.notify()

// 通知所有
cv.notifyAll()
```

### 3.3 WorkerPriority 优先级

```typescript
enum WorkerPriority {
    HIGH = 0,
    MEDIUM = 1,
    LOW = 2,
    IDLE = 3,
    DEADLINE = 4,
    VIP = 5
}
```

## 4. Worker vs Taskpool

| 特性 | Worker | Taskpool |
|------|--------|----------|
| 线程模型 | 独立线程 | 线程池 |
| 任务数量 | 1 个长期运行 | 多个短期任务 |
| 消息传递 | postMessage | 函数调用 |
| 适用场景 | 长时间运行 | 短暂并行计算 |
| 资源占用 | 较高 | 可共享 |

## 相关文档

- [01_Overview.md](./01_Overview.md) - 组件定位
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [05_API_js_sys_module.md](./05_API_js_sys_module.md) - 系统 API

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

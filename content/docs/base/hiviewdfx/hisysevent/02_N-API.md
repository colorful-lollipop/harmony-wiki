# N-API 参考

## 模块信息

| 项目 | 值 |
|------|-----|
| 模块名 | `@ohos/hisysevent` |
| N-API 模块 | `hiSysEvent` |
| 实现文件 | `interfaces/js/kits/napi/src/napi_hisysevent_js.cpp` |
| 系统要求 | 仅系统应用可用 |

## API 清单

| JS API | 功能 | 同步/异步 | 系统应用 |
|--------|------|----------|----------|
| [write](#write) | 写入系统事件 | Promise/Callback | 是 |
| [addWatcher](#addwatcher) | 添加事件观察者 | 同步 | 是 |
| [removeWatcher](#removewatcher) | 移除事件观察者 | 同步 | 是 |
| [query](#query) | 查询系统事件 | Callback | 是 |
| [exportSysEvents](#exportsysevents) | 导出系统事件 | Promise | 是 |
| [subscribe](#subscribe) | 订阅系统事件 | Promise | 是 |
| [unsubscribe](#unsubscribe) | 取消订阅 | 同步 | 是 |

## 枚举类型

### EventType - 事件类型

```typescript
enum EventType {
    FAULT = 1,      // 系统故障事件
    STATISTIC = 2,   // 系统统计事件
    SECURITY = 3,   // 系统安全事件
    BEHAVIOR = 4    // 系统行为事件
}
```

### RuleType - 规则类型

```typescript
enum RuleType {
    WHOLE_WORD = 0, // 全词匹配
    PREFIX = 1,     // 前缀匹配
    REGULAR = 2    // 正则匹配
}
```

### ListenerRule - 监听规则

```typescript
interface ListenerRule {
    domain: string;      // 事件领域 (可选)
    event: string;       // 事件名 (可选)
    ruleType: RuleType;  // 匹配规则
}
```

### QueryArg - 查询参数

```typescript
interface QueryArg {
    beginTime?: number;  // 开始时间戳 (毫秒)
    endTime?: number;    // 结束时间戳 (毫秒)
    maxEvents?: number;  // 最大事件数 (默认 1000)
}
```

---

## write

写入一个系统事件。

### 签名

```typescript
function write(
    info: {
        domain: string;
        name: string;
        type: EventType;
        params?: Record<string, any>;
    },
    callback?: AsyncCallback<void>
): Promise<void>;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| info | object | 是 | 事件信息 |
| info.domain | string | 是 | 事件领域，预定义或自定义 |
| info.name | string | 是 | 事件名称 |
| info.type | EventType | 是 | 事件类型 |
| info.params | object | 否 | 事件参数键值对 |
| callback | AsyncCallback | 否 | 异步回调 |

### 返回值

| 类型 | 描述 |
|------|------|
| Promise\<void\> | 无参数的 Promise |
| void | 使用 callback 时的返回值 |

### 示例

```typescript
import hiSysEvent from '@ohos/hisysevent';

// 使用 Promise
hiSysEvent.write({
    domain: 'AAFWK',
    name: 'start_app',
    type: hiSysEvent.EventType.BEHAVIOR,
    params: {
        app_name: 'com.example.demo',
        duration: 1500
    }
}).then(() => {
    console.info('Event written successfully');
}).catch((err) => {
    console.error('Failed to write event:', err.code, err.message);
});

// 使用 Callback
hiSysEvent.write({
    domain: 'HIVIEWDFX',
    name: 'hisysevent_test',
    type: hiSysEvent.EventType.BEHAVIOR,
    params: {
        test_key: 'test_value'
    }
}, (err) => {
    if (err) {
        console.error('Write failed:', err.code, err.message);
    } else {
        console.info('Write succeeded');
    }
});
```

### 错误码

| 错误码 | 常量名 | 描述 |
|--------|--------|------|
| 401 | - | 参数错误 |
| 201 | - | 权限拒绝（非系统应用） |

---

## addWatcher

添加一个事件观察者，用于监听匹配规则的事件。

### 签名

```typescript
function addWatcher(watcher: {
    rules: ListenerRule[];
    onEvent?: (event: HiSysEvent) => void;
}): void;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| watcher | object | 是 | 观察者配置 |
| watcher.rules | ListenerRule[] | 是 | 监听规则数组 |
| watcher.onEvent | function | 否 | 事件回调函数 |

### 示例

```typescript
import hiSysEvent from '@ohos/hisysevent';

hiSysEvent.addWatcher({
    rules: [
        {
            domain: 'AAFWK',
            event: 'start_app',
            ruleType: hiSysEvent.RuleType.WHOLE_WORD
        },
        {
            domain: 'HIVIEWDFX',
            ruleType: hiSysEvent.RuleType.PREFIX  // 匹配所有 HIVIEWDFX 域事件
        }
    ],
    onEvent: (event) => {
        console.info('Received event:', event.domain, event.name);
        console.info('Params:', JSON.stringify(event.params));
    }
});

console.info('Watcher added');
```

---

## removeWatcher

移除已添加的事件观察者。

### 签名

```typescript
function removeWatcher(watcher: {
    rules: ListenerRule[];
    onEvent?: (event: HiSysEvent) => void;
}): void;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| watcher | object | 是 | 要移除的观察者（必须与 addWatcher 参数完全一致） |

### 示例

```typescript
const watcher = {
    rules: [
        {
            domain: 'AAFWK',
            event: 'start_app',
            ruleType: hiSysEvent.RuleType.WHOLE_WORD
        }
    ],
    onEvent: (event) => {
        console.info('Event:', event.name);
    }
};

hiSysEvent.addWatcher(watcher);
// ... 业务逻辑
hiSysEvent.removeWatcher(watcher);
console.info('Watcher removed');
```

---

## query

查询历史系统事件。

### 签名

```typescript
function query(
    queryArg: QueryArg,
    rules: QueryRule[],
    querier: {
        onQueryResult?: (events: HiSysEvent[]) => void;
    }
): void;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| queryArg | QueryArg | 是 | 查询参数 |
| rules | QueryRule[] | 是 | 查询规则 |
| querier | object | 是 | 查询器配置 |
| querier.onQueryResult | function | 否 | 查询结果回调 |

### 示例

```typescript
import hiSysEvent from '@ohos/hisysevent';

hiSysEvent.query(
    {
        beginTime: Date.now() - 3600 * 1000,  // 最近 1 小时
        maxEvents: 100
    },
    [
        {
            domain: 'AAFWK',
            ruleType: hiSysEvent.RuleType.WHOLE_WORD
        }
    ],
    {
        onQueryResult: (events) => {
            console.info('Found', events.length, 'events');
            events.forEach((event) => {
                console.info(`- ${event.domain}:${event.name}`);
            });
        }
    }
);
```

---

## exportSysEvents

导出系统事件到文件。

### 签名

```typescript
function exportSysEvents(
    queryArg: QueryArg,
    rules: QueryRule[]
): Promise<string>;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| queryArg | QueryArg | 是 | 查询参数 |
| rules | QueryRule[] | 是 | 导出规则 |

### 返回值

| 类型 | 描述 |
|------|------|
| Promise\<string\> | 导出文件路径 |

### 示例

```typescript
import hiSysEvent from '@ohos/hisysevent';

hiSysEvent.exportSysEvents(
    {
        beginTime: Date.now() - 24 * 3600 * 1000,  // 最近 24 小时
        maxEvents: 10000
    },
    [
        {
            domain: 'HIVIEWDFX',
            ruleType: hiSysEvent.RuleType.PREFIX
        }
    ]
).then((filePath) => {
    console.info('Events exported to:', filePath);
}).catch((err) => {
    console.error('Export failed:', err.code, err.message);
});
```

---

## subscribe

订阅实时系统事件。

### 签名

```typescript
function subscribe(rules: QueryRule[]): Promise<number>;
```

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| rules | QueryRule[] | 是 | 订阅规则 |

### 返回值

| 类型 | 描述 |
|------|------|
| Promise\<number\> | 订阅 ID，用于后续取消订阅 |

### 示例

```typescript
import hiSysEvent from '@ohos/hisysevent';

hiSysEvent.subscribe([
    {
        domain: 'AAFWK',
        ruleType: hiSysEvent.RuleType.WHOLE_WORD
    }
]).then((subscribeId) => {
    console.info('Subscribed with ID:', subscribeId);
    // 需要配合 addWatcher 使用才能收到回调
}).catch((err) => {
    console.error('Subscribe failed:', err.code, err.message);
});
```

---

## unsubscribe

取消订阅。

### 签名

```typescript
function unsubscribe(): void;
```

### 示例

```typescript
// 先订阅
hiSysEvent.subscribe([
    {
        domain: 'AAFWK',
        ruleType: hiSysEvent.RuleType.WHOLE_WORD
    }
]).then(() => {
    console.info('Subscribed');
});

// 取消订阅
hiSysEvent.unsubscribe();
console.info('Unsubscribed');
```

## 相关链接

- [概览](00_Overview.md) - 项目整体介绍
- [架构说明](01_Architecture.md) - 内部架构
- [C++ API 参考](03_CPP_API.md) - Native 接口

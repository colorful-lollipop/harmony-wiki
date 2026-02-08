# API 参考: js_util_module

> 容器类、JSON、编码工具等实用 API

## 模块概述

js_util_module 提供丰富的实用工具，包括：
- **Container**: 15+ 种容器类 (List, Map, Set 等)
- **Util**: 工具类 (TextEncoder, LruBuffer, Scope)
- **JSON**: JSON 序列化/反序列化
- **Stream**: 流处理

## 1. 容器类概览

### 1.1 线性容器

| 类 | 特点 | 时间复杂度 |
|----|------|----------|
| ArrayList | 动态数组 | O(1) 访问, O(n) 插入 |
| LinkedList | 双向链表 | O(n) 访问, O(1) 插入 |
| List | 单向链表 | O(n) 访问, O(1) 插入 |
| Deque | 双端队列 | O(1) 首尾操作 |
| Queue | 队列 | O(1) 入队出队 |
| Stack | 栈 | O(1) 入栈出栈 |
| Vector | 向量 | O(1) 随机访问 |

### 1.2 关联容器

| 类 | 特点 | 键类型 |
|----|------|--------|
| HashMap | 哈希表 | 任意 |
| TreeMap | 红黑树 | 可比较 |
| PlainArray | 简单数组 | number |
| LightWeightMap | 轻量级 Map | 任意 |

### 1.3 集合容器

| 类 | 特点 | 元素类型 |
|----|------|----------|
| HashSet | 哈希集合 | 任意 |
| TreeSet | 有序集合 | 可比较 |
| LightWeightSet | 轻量级 Set | 任意 |

## 2. 线性容器详解

### 2.1 ArrayList

动态数组实现。

**命名空间**: `@ohos.util`

#### 构造函数

```typescript
new ArrayList<T>(): ArrayList<T>
new ArrayList<T>(array: T[]): ArrayList<T>
```

#### 属性

```typescript
length: number  // 元素数量
```

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| add | element: T | boolean | 尾部添加 |
| insert | element: T, index: number | void | 指定位置插入 |
| has | element: T | boolean | 包含检查 |
| getIndexOf | element: T | number | 查找索引 |
| get | index: number | T | 获取元素 |
| removeByIndex | index: number | T | 按索引删除 |
| remove | element: T | boolean | 按值删除 |
| clear | 无 | void | 清空 |
| forEach | callback: (item, index) => void | void | 遍历 |
| sort | comparator: (a, b) => number | void | 排序 |
| clone | 无 | ArrayList\<T\> | 克隆 |

**示例**:
```typescript
import ArrayList from '@ohos.util.ArrayList'

let list = new ArrayList<number>()

list.add(1)
list.add(2)
list.add(3)

console.log(list.length)  // 3
console.log(list.get(1))  // 2

list.insert(1.5, 1)  // 在索引 1 插入
console.log(list.toString())  // [1, 1.5, 2, 3]

list.removeByIndex(0)  // 删除第一个
console.log(list.toString())  // [1.5, 2, 3]

list.sort((a, b) => b - a)  // 降序
console.log(list.toString())  // [3, 2, 1.5]
```

### 2.2 LinkedList

双向链表实现。

**命名空间**: `@ohos.util`

#### 特性方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| addFirst | element: T | void | 头部插入 |
| addLast | element: T | void | 尾部插入 |
| removeFirst | 无 | T | 删除头部 |
| removeLast | 无 | T | 删除尾部 |
| getFirst | 无 | T | 获取头部 |
| getLast | 无 | T | 获取尾部 |

**示例**:
```typescript
import LinkedList from '@ohos.util.LinkedList'

let list = new LinkedList<string>()

list.addFirst('A')
list.addLast('B')
list.add('C')

console.log(list.getFirst())  // 'A'
console.log(list.getLast())   // 'C'

list.removeFirst()
console.log(list.getFirst())  // 'B'
```

### 2.3 Deque

双端队列。

**命名空间**: `@ohos.util`

#### 方法

| 方法 | 说明 |
|------|------|
| insertFront(element) | 头部插入 |
| insertEnd(element) | 尾部插入 |
| getFirst() | 获取头部 |
| getLast() | 获取尾部 |
| popFirst() | 删除并返回头部 |
| popLast() | 删除并返回尾部 |

### 2.4 Stack

栈结构。

**命名空间**: `@ohos.util`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| push | item: T | T | 入栈 |
| pop | 无 | T | 出栈 |
| peek | 无 | T | 查看栈顶 |
| locate | element: T | number | 查找位置 |
| isEmpty | 无 | boolean | 是否为空 |

### 2.5 Queue

队列结构。

**命名空间**: `@ohos.util`

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| add | element: T | boolean | 入队 |
| getFirst | 无 | T | 获取队头 |
| pop | 无 | T | 出队 |

## 3. 关联容器详解

### 3.1 HashMap

哈希表实现。

**命名空间**: `@ohos.util`

#### 构造函数

```typescript
new HashMap<K, V>(): HashMap<K, V>
```

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| set | key: K, value: V | Object | 设置键值对 |
| get | key: K | V | 获取值 |
| hasKey | key: K | boolean | 键存在检查 |
| hasValue | value: V | boolean | 值存在检查 |
| remove | key: K | V | 删除键值对 |
| clear | 无 | void | 清空 |
| keys | 无 | IterableIterator\<K\> | 键迭代器 |
| values | 无 | IterableIterator\<V\> | 值迭代器 |
| entries | 无 | IterableIterator\<[K, V]\> | 键值对迭代器 |
| forEach | callback: (value, key) => void | void | 遍历 |

**示例**:
```typescript
import HashMap from '@ohos.util.HashMap'

let map = new HashMap<string, number>()

map.set('a', 1)
map.set('b', 2)
map.set('c', 3)

console.log(map.get('a'))  // 1

console.log(map.hasKey('b'))  // true
console.log(map.hasValue(2))  // true

map.forEach((value, key) => {
    console.log(`${key}: ${value}`)
})
// a: 1
// b: 2
// c: 3
```

### 3.2 TreeMap

有序哈希表 (红黑树)。

**命名空间**: `@ohos.util`

#### 特有方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| getFirstKey | 无 | K | 最小键 |
| getLastKey | 无 | K | 最大键 |
| getLowerKey | key: K | K | 前驱键 |
| getHigherKey | key: K | K | 后继键 |

### 3.3 LightWeightMap

轻量级 Map，内存效率更高。

**命名空间**: `@ohos.util`

**特性**: 支持直接通过索引访问键值对

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| getIndexOfKey | key: K | number | 键的索引 |
| getIndexOfValue | value: V | number | 值的索引 |
| getKeyAt | index: number | K | 按索引获取键 |
| getValueAt | index: number | V | 按索引获取值 |
| setValueAt | index: number, value: V | boolean | 按索引设置值 |

## 4. 集合容器

### 4.1 HashSet

哈希集合。

**命名空间**: `@ohos.util`

```typescript
import HashSet from '@ohos.util.HashSet'

let set = new HashSet<string>()

set.add('a')
set.add('b')
set.add('a')  // 重复，不添加

console.log(set.has('a'))  // true
console.log(set.length)    // 2
```

### 4.2 TreeSet

有序集合。

**命名空间**: `@ohos.util`

```typescript
import TreeSet from '@ohos.util.TreeSet'

let set = new TreeSet<number>()

set.add(3)
set.add(1)
set.add(2)

console.log(set.getFirstValue())  // 1 (最小)
console.log(set.getLastValue())   // 3 (最大)
```

### 4.3 LightWeightSet

轻量级集合。

**命名空间**: `@ohos.util`

```typescript
import LightWeightSet from '@ohos.util.LightWeightSet'

let set = new LightWeightSet<string>()

set.add('a')
set.add('b')
set.add('a')

console.log(set.getIndexOf('a'))  // 0
```

## 5. 工具类

### 5.1 LruBuffer

LRU 缓存 (最近最少使用)。

**命名空间**: `@ohos.util`

```typescript
import LruBuffer from '@ohos.util.LruBuffer'

let cache = new LruBuffer<string, number>(3)  // 容量 3

cache.put('a', 1)
cache.put('b', 2)
cache.put('c', 3)

console.log(cache.get('a'))  // 1
cache.put('d', 4)  // 容量已满，淘汰 'b'

console.log(cache.get('b'))  // undefined (已淘汰)
console.log(cache.length)    // 3
```

### 5.2 TextEncoder / TextDecoder

字符编码转换。

**命名空间**: `@ohos.util`

```typescript
import util from '@ohos.util'

let encoder = new util.TextEncoder()
let decoder = new util.TextDecoder('utf-8')

let bytes = encoder.encode('Hello')
console.log(bytes instanceof Uint8Array)  // true

let text = decoder.decode(bytes)
console.log(text)  // 'Hello'
```

### 5.3 Scope

数值范围。

**命名空间**: `@ohos.util`

```typescript
import util from '@ohos.util'

// 创建范围 [10, 100]
let range = new util.Scope(10, 100)

console.log(range.contains(50))    // true
console.log(range.contains(150))   // false

// 交集
let range2 = new util.Scope(50, 150)
let intersect = range.intersect(range2)
console.log(intersect.toString())  // [50, 100]
```

## 6. JSON 模块

### 6.1 JSON 对象

```typescript
import JSON from '@ohos.util.JSON'

// 解析
let obj = JSON.parse('{"name": "John", "age": 30}')
console.log(obj.name)  // 'John'

// 序列化
let str = JSON.stringify(obj)
console.log(str)  // '{"name":"John","age":30}'
```

## 7. PlainArray

键为 number 的数组。

**命名空间**: `@ohos.util`

```typescript
import PlainArray from '@ohos.util.PlainArray'

let arr = new PlainArray<string>()

arr.add(1, 'one')
arr.add(2, 'two')
arr.add(3, 'three')

console.log(arr.get(2))      // 'two'
console.log(arr.getKeyAt(0)) // 1
console.log(arr.getValueAt(0)) // 'one'
```

## 8. Vector

向量容器。

**命名空间**: `@ohos.util`

```typescript
import Vector from '@ohos.util.Vector'

let vec = new Vector<number>()

vec.add(1)
vec.add(2)
vec.add(3)

console.log(vec.get(1))  // 2
```

## 相关文档

- [01_Overview.md](./01_Overview.md) - 组件定位
- [03_API_js_api_module.md](./03_API_js_api_module.md) - 数据处理 API
- [05_API_js_sys_module.md](./05_API_js_sys_module.md) - 系统 API

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

# API 参考: js_api_module

> URL、URI、XML、Buffer 等数据处理 API

## 模块概述

js_api_module 提供基础的数据处理能力，包括：
- **URL/URI**: 网络地址解析与构造
- **XML**: XML 文档解析与序列化
- **Buffer**: 二进制缓冲区操作
- **ConvertXml**: XML 格式转换

## 1. URL 模块

### 1.1 类: URL

网络地址解析与构造。

**命名空间**: `@ohos.url`

#### 构造函数

```typescript
new URL(input: string, base?: string | URL): URL
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| input | string | 是 | URL 字符串或相对路径 |
| base | string \| URL | 否 | 基础 URL |

**示例**:
```typescript
import url from '@ohos.url'

// 绝对 URL
let absUrl = new URL('https://example.com/path')

// 相对 URL + 基础 URL
let relUrl = new URL('/api/users', 'https://example.com')
// 结果: 'https://example.com/api/users'

// 相对路径解析
let pageUrl = new URL('index.html', 'https://example.com/docs/')
// 结果: 'https://example.com/docs/index.html'
```

#### 属性

| 属性 | 类型 | 只读 | 说明 |
|------|------|------|------|
| href | string | 否 | 完整的 URL 字符串 |
| protocol | string | 否 | 协议 (http:, https:, 等) |
| host | string | 否 | 主机名 (含端口) |
| hostname | string | 否 | 主机名 (不含端口) |
| port | string | 否 | 端口号 |
| pathname | string | 否 | 路径 |
| search | string | 否 | 查询字符串 (含 ?) |
| searchParams | URLSearchParams | 否 | 查询参数对象 |
| hash | string | 否 | 片段标识符 (含 #) |
| origin | string | 只读 | 协议+主机+端口 |
| username | string | 否 | 用户名 |
| password | string | 否 | 密码 |

**示例**:
```typescript
let url = new URL('https://user:pass@example.com:8080/path?query=123#hash')

console.log(url.protocol)  // 'https:'
console.log(url.host)       // 'example.com:8080'
console.log(url.hostname)   // 'example.com'
console.log(url.port)       // '8080'
console.log(url.pathname)   // '/path'
console.log(url.search)     // '?query=123'
console.log(url.hash)       // '#hash'
console.log(url.origin)     // 'https://example.com:8080'
```

#### 方法

##### toString()

返回完整的 URL 字符串。

```typescript
url.toString(): string
```

**示例**:
```typescript
let url = new URL('https://example.com')
url.pathname = '/new-path'
console.log(url.toString())
// 'https://example.com/new-path'
```

##### toJSON()

返回 URL 的序列化表示。

```typescript
url.toJSON(): string
```

**示例**:
```typescript
let url = new URL('https://example.com')
console.log(url.toJSON())
// 'https://example.com/'
```

### 1.2 类: URLSearchParams

URL 查询参数处理。

**命名空间**: `@ohos.url`

#### 构造函数

```typescript
new URLSearchParams(): URLSearchParams
new URLSearchParams(string: string): URLSearchParams
new URLSearchParams(obj: Object): URLSearchParams
new URLSearchParams(iterable: Iterable<[string, string]>): URLSearchParams
```

**示例**:
```typescript
import url from '@ohos.url'

// 空参数
let params1 = new url.URLSearchParams()

// 从字符串
let params2 = new url.URLSearchParams('foo=1&bar=2')

// 从对象
let params3 = new url.URLSearchParams({ foo: '1', bar: '2' })

// 从可迭代对象
let params4 = new url.URLSearchParams([
    ['foo', '1'],
    ['bar', '2']
])
```

#### 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| append(name, value) | void | 追加参数 |
| delete(name) | void | 删除所有参数 |
| get(name) | string \| null | 获取第一个值 |
| getAll(name) | string[] | 获取所有值 |
| has(name) | boolean | 是否存在 |
| set(name, value) | void | 设置值 |
| sort() | void | 排序 |
| toString() | string | 转为查询字符串 |
| keys() | IterableIterator\<string\> | 键迭代器 |
| values() | IterableIterator\<string\> | 值迭代器 |
| entries() | IterableIterator\<[string, string]\> | 键值对迭代器 |
| forEach(callback) | void | 遍历 |

**示例**:
```typescript
let params = new url.URLSearchParams('foo=1&bar=2')

params.append('foo', '3')
console.log(params.toString())
// 'foo=1&bar=2&foo=3'

console.log(params.getAll('foo'))
// ['1', '3']

params.set('foo', '0')
console.log(params.toString())
// 'foo=0&bar=2'

params.sort()
console.log(params.toString())
// 'bar=2&foo=0'
```

## 2. URI 模块

### 2.1 类: Uri

URI 标准化处理。

**命名空间**: `@ohos.uri` (通过 `Uri` 对象访问)

#### 构造函数

```typescript
new Uri.Uri(str: string): Uri
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| str | string | 是 | URI 字符串 |

**示例**:
```typescript
import Uri from '@ohos.uri'

let uri = new Uri.Uri('https://user:pass@example.com:8080/path?query=1#frag')
```

#### 属性

| 属性 | 类型 | 只读 | 说明 |
|------|------|------|------|
| scheme | string | 否 | URI 方案 |
| authority | string | 否 | 授权部分 |
| ssp | string | 否 | 方案特定部分 |
| userinfo | string | 否 | 用户信息 |
| host | string | 否 | 主机 |
| port | number | 否 | 端口 |
| path | string | 否 | 路径 |
| query | string | 否 | 查询 |
| fragment | string | 否 | 片段 |

#### 方法

| 方法 | 返回值 | 说明 |
|------|--------|------|
| equals(other: Uri) | boolean | 是否相等 |
| normalize() | Uri | 标准化路径 |
| checkIsAbsolute() | boolean | 是否绝对 URI |
| toString() | string | 转为字符串 |

## 3. XML 模块

### 3.1 类: XmlSerializer

XML 文档生成。

**命名空间**: `@ohos.xml`

#### 构造函数

```typescript
new XmlSerializer(buffer: ArrayBuffer | DataView, encoding?: string): XmlSerializer
```

**参数**:
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| buffer | ArrayBuffer \| DataView | 是 | 输出缓冲区 |
| encoding | string | 否 | 编码格式 (默认 UTF-8) |

**示例**:
```typescript
import xml from '@ohos.xml'

let buffer = new ArrayBuffer(1024)
let serializer = new xml.XmlSerializer(buffer)
```

#### 方法

| 方法 | 参数 | 说明 |
|------|------|------|
| setDeclaration() | 无 | 设置 XML 声明 |
| setNamespace | prefix, namespace | 设置命名空间 |
| startElement | name | 开始元素 |
| endElement | 无 | 结束元素 |
| setAttributes | name, value | 设置属性 |
| addEmptyElement | name | 添加空元素 |
| setText | text | 设置文本内容 |
| setComment | text | 设置注释 |
| setCData | text | 设置 CDATA |
| setDocType | text | 设置 DTD |

**完整示例**:
```typescript
import xml from '@ohos.xml'

let buffer = new ArrayBuffer(1024)
let ser = new xml.XmlSerializer(buffer)

ser.setDeclaration()
ser.startElement('root')
ser.setNamespace('h', 'http://example.com')
ser.setAttributes('id', '1')
ser.startElement('child')
ser.setText('Hello')
ser.endElement()
ser.endElement()
```

### 3.2 类: XmlPullParser

XML 文档解析。

**命名空间**: `@ohos.xml`

#### 构造函数

```typescript
new XmlPullParser(buffer: ArrayBuffer | DataView, encoding?: string): XmlPullParser
```

#### 方法

```typescript
parse(options: ParseOptions): void
```

**ParseOptions**:
| 属性 | 类型 | 说明 |
|------|------|------|
| supportDoctype | boolean | 支持 DTD |
| ignoreNameSpace | boolean | 忽略命名空间 |
| tagValueCallbackFunction | (name: string, value: string) => boolean | 标签值回调 |
| attributeValueCallbackFunction | (name: string, value: string) => boolean | 属性值回调 |
| tokenValueCallbackFunction | (eventType: EventType, value: ParseInfo) => boolean | 令牌回调 |

**示例**:
```typescript
import xml from '@ohos.xml'

let xmlStr = `<?xml version="1.0" encoding="utf-8"?>
<root>
    <item id="1">Content</item>
</root>`

let buffer = new ArrayBuffer(xmlStr.length * 2)
let view = new Uint8Array(buffer)

for (let i = 0; i < xmlStr.length; i++) {
    view[i] = xmlStr.charCodeAt(i)
}

let parser = new xml.XmlPullParser(buffer)

function onTag(name: string, value: string): boolean {
    console.log(`Tag: ${name} = ${value}`)
    return true
}

parser.parse({
    tagValueCallbackFunction: onTag
})
```

## 4. Buffer 模块

### 4.1 类: Buffer

二进制缓冲区操作。

**命名空间**: `@ohos.buffer`

#### 静态方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| alloc | size, fill?, encoding? | Buffer | 分配并初始化 |
| allocUninitialized | size | Buffer | 分配未初始化 |
| allocUninitializedFromPool | size | Buffer | 从池分配 |
| from | array/buffer/string | Buffer | 从数据创建 |
| compare | buf1, buf2 | -1\|0\|1 | 比较缓冲区 |
| concat | list, totalLength? | Buffer | 连接缓冲区 |
| byteLength | string, encoding | number | 获取字节长度 |
| isBuffer | obj | boolean | 是否 Buffer |
| isEncoding | encoding | boolean | 是否有效编码 |
| transcode | src, fromEnc, toEnc | Buffer | 转码 |

#### 实例属性

| 属性 | 类型 | 说明 |
|------|------|------|
| length | number | 缓冲区字节长度 |
| buffer | ArrayBuffer | 底层 ArrayBuffer |
| byteOffset | number | 缓冲区偏移量 |

#### 实例方法

**读写方法**:
| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| write | str, offset?, length?, encoding? | number | 写入字符串 |
| toString | encoding?, start?, end? | string | 转为字符串 |
| fill | value, offset?, end?, encoding? | Buffer | 填充 |
| copy | target, targetStart?, sourceStart?, sourceEnd? | number | 复制 |
| compare | target, ... | -1\|0\|1 | 比较 |
| equals | otherBuffer | boolean | 相等判断 |
| includes | value, ... | boolean | 包含检查 |
| indexOf | value, ... | number | 查找位置 |
| lastIndexOf | value, ... | number | 最后位置 |

**字节序读写**:
| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| readInt8 | offset? | number | 读 Int8 |
| readInt16BE | offset? | number | 读大端 Int16 |
| readInt16LE | offset? | number | 读小端 Int16 |
| readInt32BE | offset? | number | 读大端 Int32 |
| readInt32LE | offset? | number | 读小端 Int32 |
| writeInt8 | value, offset? | number | 写 Int8 |
| writeInt16BE | value, offset? | number | 写大端 Int16 |
| writeInt32BE | value, offset? | number | 写大端 Int32 |

**示例**:
```typescript
import buffer from '@ohos.buffer'

// 创建 Buffer
let buf = buffer.alloc(10)
buf.fill(0)

// 写入字符串
buf.write('hello', 0, 5, 'utf-8')

// 读取
console.log(buf.toString())
// 'hello'

// 整数读写
buf.writeInt32BE(0x12345678, 0)
console.log(buf.readInt32BE(0).toString(16))
// '12345678'
```

### 4.2 类: Blob

二进制大对象。

**命名空间**: `@ohos.buffer`

#### 构造函数

```typescript
new Blob(sources: (string | ArrayBuffer | TypedArray | DataView | Blob)[], options?: BlobOptions): Blob
```

#### 属性

| 属性 | 类型 | 只读 | 说明 |
|------|------|------|------|
| size | number | 是 | Blob 大小 |
| type | string | 是 | MIME 类型 |

#### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| arrayBuffer | 无 | Promise\<ArrayBuffer\> | 转为 ArrayBuffer |
| text | 无 | Promise\<string\> | 转为文本 |
| slice | start?, end?, type? | Blob | 切片 |

## 5. ConvertXml 模块

### 5.1 类: ConvertXml

XML 转 JavaScript 对象。

**命名空间**: `@ohos.convertxml`

#### 构造函数

```typescript
new ConvertXml(): ConvertXml
```

#### 方法

```typescript
convert(xml: string, options?: ConvertOptions): Object
```

**ConvertOptions**:
| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| compact | boolean | false | 紧凑模式 |
| spaces | number | 0 | 缩进空格数 |

**示例**:
```typescript
import convertxml from '@ohos.convertxml'

let converter = new convertxml.ConvertXml()

let result = converter.convert(
    '<root><item>Hello</item></root>',
    { compact: false, spaces: 2 }
)

console.log(JSON.stringify(result, null, 2))
// {
//   "declaration": { "attributes": { "version": "1.0", "encoding": "utf-8" } },
//   "root": {
//     "item": "Hello"
//   }
// }
```

## 相关文档

- [01_Overview.md](./01_Overview.md) - 组件定位
- [02_Architecture.md](./02_Architecture.md) - 架构设计
- [04_API_js_util_module.md](./04_API_js_util_module.md) - 工具模块 API

---

*文档版本: 1.0*
*最后更新: 2026-02-06*

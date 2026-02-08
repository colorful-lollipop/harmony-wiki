# N-API 接口文档

> @OHOS.distributedfile.fileio 与 @system.file 模块 API 清单

## 命名空间概览

| 命名空间 | 用途 | 编程模型 |
|----------|------|----------|
| `@OHOS.distributedfile.fileio` | 基础文件 I/O 操作 | Sync / Callback / Promise |
| `@system.file` | 沙箱文件操作（Legacy） | Legacy Callback |

---

## @OHOS.distributedfile.fileio 模块

### API 分类

#### 1. 基础文件 API

| JS API | 同步/异步 | 功能 | 示例 |
|--------|-----------|------|------|
| `access` / `accessSync` | 两者 | 检查文件是否存在及权限 | `fileio.accessSync("/data/test.txt")` |
| `chown` / `chownSync` | 两者 | 修改文件所有者 | `fileio.chownSync(path, uid, gid)` |
| `chmod` / `chmodSync` | 两者 | 修改文件权限 | `fileio.chmodSync(path, mode)` |

**参数说明**（推断）:
- `path`: string - 文件绝对路径
- `uid`: number - 用户 ID
- `gid`: number - 组 ID
- `mode`: number - 权限模式（如 0o644）

**错误码**（TODO: 待代码确认）:
- `EACCES`: 权限拒绝
- `ENOENT`: 文件不存在
- `EINVAL`: 无效参数

#### 2. 基础目录 API

| JS API | 类型 | 功能 |
|--------|------|------|
| `Dir.openDirSync` | Class Method | 同步打开目录 |
| `Dir.closeSync` | Class Method | 同步关闭目录 |
| `Dir.readSync` | Class Method | 同步读取目录项 |

**Dir 类使用示例**:
```javascript
import fileio from '@OHOS.distributedfile.fileio';

let dir = fileio.openDirSync("/data");
try {
    while (true) {
        let entry = dir.readSync();
        if (!entry) break;
        console.log(entry.name);
    }
} finally {
    dir.closeSync();
}
```

#### 3. 统计 API

| JS API | 类型 | 功能 |
|--------|------|------|
| `Stat.statSync` | Class Method | 获取文件统计信息 |
| `Stat` | Class | 统计信息类（size, mode, atime, mtime 等） |

**Stat 类属性**（推断）:
- `size`: number - 文件大小（字节）
- `mode`: number - 文件权限模式
- `atime`: number - 访问时间
- `mtime`: number - 修改时间
- `ctime`: number - 状态变更时间

#### 4. 流式文件 API

| JS API | 功能 |
|--------|------|
| `Stream.createStreamSync` | 同步创建文件流 |
| `Stream.fdopenStreamSync` | 基于文件描述符创建流 |
| `Stream.read` | 读取数据 |
| `Stream.write` | 写入数据 |
| `Stream.close` | 关闭流 |

**Stream 使用示例**:
```javascript
import fileio from '@OHOS.distributedfile.fileio';

// 同步模式
let stream = fileio.createStreamSync("/data/test.txt", "r");
let buf = new ArrayBuffer(4096);
stream.readSync(buf);
stream.closeSync();

// 异步模式
fileio.createStream("/data/test.txt", "r", (err, stream) => {
    if (!err) {
        stream.read(new ArrayBuffer(4096), {}, (err, buf, len) => {
            if (!err) console.log('read:', len, 'bytes');
            stream.close(() => {});
        });
    }
});
```

---

## @system.file 模块（Legacy API）

> ⚠️ Legacy API，新项目建议使用 @OHOS.distributedfile.fileio

### API 列表

| JS API | 功能 | URI 前缀 |
|--------|------|----------|
| `move` | 移动文件 | internal://* |
| `copy` | 复制文件 | internal://* |
| `list` | 列出目录内容 | internal://* |
| `access` | 检查文件存在性 | internal://* |

### URI 类型

| 目录类型 | URI 前缀 | 访问权限 | 用途 |
|----------|----------|----------|------|
| 临时目录 | `internal://cache/` | 仅当前应用 | 临时下载、缓存 |
| 应用私有目录 | `internal://app/` | 仅当前应用 | 应用数据存储 |
| 外部存储 | `internal://share/` | 所有应用（需授权） | 共享文件 |

### Legacy API 使用示例

```javascript
import file from '@system.file';

// access - Legacy Callback 模式
file.access({
    uri: 'internal://app/test.txt',
    success: () => console.log('exists'),
    fail: (data, code) => console.error('error:', code),
    complete: () => console.log('done')
});
```

---

## 编程模型对比

### 1. 同步编程模型

**特征**: API 名称含 `Sync`

```javascript
import fileio from '@OHOS.distributedfile.fileio';

try {
    let stream = fileio.createStreamSync("/data/test.txt", "r");
    let buf = new ArrayBuffer(4096);
    stream.readSync(buf);
    stream.closeSync();
} catch (e) {
    console.error(e);
}
```

**特点**:
- 阻塞当前执行直到完成
- 简单直观，适合小文件操作
- 主线程阻塞风险

### 2. 异步编程模型 - Callback

**特征**: 无 `Sync` 后缀，参数含回调函数

```javascript
import fileio from '@OHOS.distributedfile.fileio';

fileio.createStream("/data/test.txt", "r", (err, stream) => {
    if (!err) {
        stream.read(new ArrayBuffer(4096), {}, (err, buf, len) => {
            if (!err) console.log('read:', len);
        });
    }
});
```

**特点**:
- 第一个回调参数为 `Error | undefined`
- 非阻塞，适合大文件
- 回调嵌套可能导致复杂

### 3. 异步编程模型 - Legacy

**特征**: `@system.file` 模块，三参数回调

```javascript
import file from '@system.file';

file.access({
    uri: 'internal://app/test.txt',
    success: () => {},
    fail: (data, code) => {},
    complete: () => {}
});
```

---

## N-API 注册位置（待代码确认）

| 符号 | 位置 | 说明 |
|------|------|------|
| `NAPI_MODULE` | 接口入口文件 | 模块注册宏 |
| `napi_define_properties` | 接口绑定 | 定义 JS 属性 |
| `napi_create_function` | 接口绑定 | 创建 JS 函数 |

---

## 错误码参考（待代码确认）

| 错误码 | 名称 | 含义 |
|--------|------|------|
| -1 | TODO | 待代码补充 |

---

## 最佳实践

### 1. 优先使用同步 API 处理小文件
```javascript
// 小文件读取
let data = fileio.readFileSync("/data/small.txt");
```

### 2. 大文件使用流式 API
```javascript
// 大文件复制
let input = fileio.createStreamSync(src, "r");
let output = fileio.createStreamSync(dst, "w");
let buf = new ArrayBuffer(8192);
while (input.readSync(buf) > 0) {
    output.writeSync(buf);
}
input.closeSync();
output.closeSync();
```

### 3. 异步操作必须错误处理
```javascript
fileio.unlink("/data/temp.txt", (err) => {
    if (err) {
        console.error('delete failed:', err.code);
        return;
    }
    console.log('deleted');
});
```

---

## 参考

- [项目概览](00_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [安全评审](04_Security_Review.md)

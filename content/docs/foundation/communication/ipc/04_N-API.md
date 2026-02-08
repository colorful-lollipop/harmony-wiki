# N-API 接口文档

## 4.1 模块概述

**模块名**: `@ohos/rpc`  
**注册入口**: `ipc/native/src/napi/src/napi_rpc_native_module.cpp:48`  
**宏定义**: `NAPI_MODULE(rpc, RpcExport)`

### N-API 导出清单

| 导出类/模块 | 描述 | 实现文件 |
|------------|------|----------|
| `MessageParcel` | 消息数据包（遗留接口） | `napi_common/source/napi_message_parcel_write.cpp` |
| `MessageSequence` | 消息序列（推荐新接口） | `napi_common/source/napi_message_sequence_write.cpp` |
| `Ashmem` | 匿名共享内存 | `napi_common/source/napi_ashmem.cpp` |
| `IPCSkeleton` | IPC 骨架静态方法 | `napi/src/napi_ipc_skeleton.cpp` |
| `RemoteObject` | 远程对象（服务端） | `napi_common/source/napi_remote_object.cpp` |
| `RemoteProxy` | 远程代理（客户端） | `napi/src/napi_remote_proxy.cpp` |
| `MessageOption` | 消息选项 | `napi/src/napi_message_option.cpp` |
| `ErrorCode` | 错误码枚举 | `napi_common/source/napi_rpc_error.cpp` |
| `CallingInfo` | 调用信息 | `napi/src/napi_calling_info.cpp` |

### 模块导入

```javascript
// 标准导入方式
import rpc from '@ohos.rpc';

// 或按需导入
import { MessageParcel, MessageOption, RemoteObject } from '@ohos.rpc';
```

### 版本兼容性

| 接口 | 推荐版本 | 兼容性说明 |
|------|----------|------------|
| `MessageParcel` | Legacy | 向后兼容，已稳定 |
| `MessageSequence` | v3.0+ | 新推荐接口，性能更优 |
| `MessageOption` | v3.0+ | 支持更多配置选项 |

**证据**: `napi_rpc_native_module.cpp:30-42` - 模块导出函数

## 4.2 IPCSkeleton API

IPCSkeleton 提供了一系列静态方法，用于获取调用方身份信息、管理线程池等 IPC 框架核心功能。这些方法主要用于服务端代码，用于验证调用者身份。

### 静态方法详解

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `getContextObject()` | 无 | `IPCSkeleton` | 获取当前进程的 IPC 上下文对象，可用于获取本地 Binder 对象 |
| `getCallingPid()` | 无 | `number` | **重要**：获取当前调用链中**直接调用方**的进程 ID，注意可能不是原始调用方 |
| `getCallingUid()` | 无 | `number` | 获取当前调用链中直接调用方的用户 ID |
| `getCallingTokenId()` | 无 | `number` | 获取调用方的安全令牌 ID，用于权限验证 |
| `getCallingDeviceID()` | 无 | `string` | 获取调用方所在设备的网络标识符 |
| `getLocalDeviceID()` | 无 | `string` | 获取**当前进程所在设备**的网络标识符 |
| `isLocalCalling()` | 无 | `boolean` | 判断当前调用是否来自**同一设备**，跨设备调用返回 `false` |
| `flushCmdBuffer()` | 无 | `void` | 刷新当前线程的命令缓冲区，确保所有待处理的 IPC 命令已发送 |
| `flushCommands()` | 无 | `void` | 与 `flushCmdBuffer` 功能相同，提供向后兼容性 |
| `resetCallingIdentity()` | 无 | `boolean` | 重置当前线程的调用身份为系统身份，用于执行特权操作 |
| `restoreCallingIdentity(token: string)` | `string` | `boolean` | 恢复之前保存的调用身份，`token` 为 `resetCallingIdentity` 的返回值 |
| `setCallingIdentity(token: string)` | `string` | `boolean` | 设置当前线程的调用身份，`token` 为 Base64 编码的身份字符串 |

### 身份管理详解

IPCSkeleton 的身份管理机制是 IPC 安全性的重要组成部分。以下是身份操作的标准流程：

```javascript
import rpc from '@ohos.rpc';

class MyRemoteObject extends rpc.RemoteObject {
    constructor(descriptor) {
        super(descriptor);
    }

    onRemoteMessageRequest(code, data, reply, option) {
        // 1. 验证调用者身份
        const callingPid = rpc.IPCSkeleton.getCallingPid();
        const callingUid = rpc.IPCSkeleton.getCallingUid();
        const callingTokenId = rpc.IPCSkeleton.getCallingTokenId();
        const isLocal = rpc.IPCSkeleton.isLocalCalling();

        console.log(`Calling PID: ${callingPid}, UID: ${callingUid}`);
        console.log(`Is local call: ${isLocal}`);

        // 2. 权限检查示例
        if (callingUid !== 0 && !this.checkPermission(callingTokenId)) {
            reply.writeInt32(rpc.ErrorCode.ERR_PERMISSION);
            return false;
        }

        // 3. 执行特权操作时重置身份
        if (needsPrivilegedOperation) {
            const originalToken = rpc.IPCSkeleton.resetCallingIdentity();
            try {
                // 执行需要高权限的操作
                this.performPrivilegedOperation();
            } finally {
                // 恢复原始身份
                rpc.IPCSkeleton.restoreCallingIdentity(originalToken);
            }
        }

        return true;
    }

    private checkPermission(tokenId: number): boolean {
        // 权限检查逻辑
        return true;
    }
}
```

### 设备标识操作

```javascript
import rpc from '@ohos.rpc';

class DeviceAwareService extends rpc.RemoteObject {
    onRemoteMessageRequest(code, data, reply, option) {
        // 获取本地设备 ID
        const localDeviceId = rpc.IPCSkeleton.getLocalDeviceID();
        console.log(`Local device: ${localDeviceId}`);

        // 获取调用方设备 ID
        const callingDeviceId = rpc.IPCSkeleton.getCallingDeviceID();

        // 判断是否为跨设备调用
        const isCrossDevice = !rpc.IPCSkeleton.isLocalCalling();
        if (isCrossDevice) {
            console.log(`Cross-device call from: ${callingDeviceId}`);
            // 跨设备调用需要额外的安全验证
            return this.validateCrossDeviceRequest(callingDeviceId, data);
        }

        return true;
    }
}
```

### 线程池管理

```javascript
import rpc from '@ohos.rpc';

// 注意：线程池管理通常在 Native 层配置
// 以下为接口说明，非 JS 层可用方法
// IPCSkeleton.SetMaxWorkThreadNum(int maxNum) - Native only
// IPCSkeleton.JoinWorkThread() - Native only
// IPCSkeleton.StopWorkThread() - Native only
```

### 最佳实践

1. **始终验证调用者身份**：在处理敏感请求前，调用 `getCallingPid()` 和 `getCallingUid()` 验证调用者。
2. **注意直接调用与原始调用的区别**：`getCallingPid()` 返回的是直接调用方，而非链式调用中的原始调用方。
3. **跨设备调用验证**：使用 `isLocalCalling()` 判断是否跨设备，跨设备调用需要额外验证。
4. **谨慎使用身份切换**：`resetCallingIdentity()` 和 `restoreCallingIdentity()` 应成对使用，避免身份泄露。

**证据**: `ipc/native/src/napi/src/napi_ipc_skeleton.cpp:472-486` - IPCSkeleton N-API 导出定义

## 4.3 MessageParcel API

### 类方法

| JS 方法 | 参数 | 返回值 | 描述 |
|---------|------|--------|------|
| `static create()` | 无 | `MessageParcel` | 创建实例 |
| `reclaim()` | 无 | `void` | 释放资源 |
| `writeRemoteObject()` | `IRemoteObject` | `boolean` | 写入远程对象 |
| `readRemoteObject()` | 无 | `IRemoteObject` | 读取远程对象 |
| `writeInterfaceToken()` | `string` | `boolean` | 写入接口令牌 |
| `readInterfaceToken()` | 无 | `string` | 读取接口令牌 |
| `writeInt32()` | `number` | `boolean` | 写入 32 位整数 |
| `readInt32()` | 无 | `number` | 读取 32 位整数 |
| `writeString()` | `string` | `boolean` | 写入字符串 |
| `readString()` | 无 | `string` | 读取字符串 |
| `writeAshmem()` | `Ashmem` | `boolean` | 写入共享内存 |
| `readAshmem()` | 无 | `Ashmem` | 读取共享内存 |
| `writeFileDescriptor()` | `number` | `boolean` | 写入文件描述符 |
| `readFileDescriptor()` | 无 | `number` | 读取文件描述符 |
| `getRawDataCapacity()` | 无 | `number` | 获取原始数据容量 |
| `getSize()` | 无 | `number` | 获取数据大小 |
| `getCapacity()` | 无 | `number` | 获取缓冲区容量 |

### 序列化方法详解

MessageParcel 提供了丰富的数据序列化方法，支持基本类型、字符串、数组、原始数据等多种数据格式。以下是各类方法的详细说明：

#### 基本数据类型读写

| 写方法 | 读方法 | 数据类型 | 长度 |
|--------|--------|----------|------|
| `writeBoolean(value: boolean)` | `readBoolean(): boolean` | 布尔值 | 1 字节 |
| `writeInt8(value: number)` | `readInt8(): number` | 8 位整数 | 1 字节 |
| `writeInt16(value: number)` | `readInt16(): number` | 16 位整数 | 2 字节 |
| `writeInt32(value: number)` | `readInt32(): number` | 32 位整数 | 4 字节 |
| `writeInt64(value: number)` | `readInt64(): number` | 64 位整数 | 8 字节 |
| `writeFloat32(value: number)` | `readFloat32(): number` | 单精度浮点 | 4 字节 |
| `writeFloat64(value: number)` | `readFloat64(): number` | 双精度浮点 | 8 字节 |

#### 字符串序列化

| 写方法 | 读方法 | 字符编码 | 长度前缀 |
|--------|--------|----------|----------|
| `writeString(value: string)` | `readString(): string` | UTF-8 | 4 字节 |
| `writeString16(value: string)` | `readString16(): string` | UTF-16 | 4 字节 |
| `writeShortString(value: string)` | `readShortString(): string` | UTF-8 | 1 字节 |

#### 数组序列化

| 写方法 | 读方法 | 元素类型 |
|--------|--------|----------|
| `writeBooleanArray(array: boolean[])` | `readBooleanArray(): boolean[]` | 布尔数组 |
| `writeInt8Array(array: number[])` | `readInt8Array(): number[]` | Int8 数组 |
| `writeInt16Array(array: number[])` | `readInt16Array(): number[]` | Int16 数组 |
| `writeInt32Array(array: number[])` | `readInt32Array(): number[]` | Int32 数组 |
| `writeInt64Array(array: number[])` | `readInt64Array(): number[]` | Int64 数组 |
| `writeFloat32Array(array: number[])` | `readFloat32Array(): number[]` | Float32 数组 |
| `writeFloat64Array(array: number[])` | `readFloat64Array(): number[]` | Float64 数组 |
| `writeStringArray(array: string[])` | `readStringArray(): string[]` | 字符串数组 |

#### 原始数据操作

```javascript
// 写入原始数据
writeRawData(buffer: ArrayBuffer): boolean
writeRawData(buffer: ArrayBuffer, size: number): boolean

// 读取原始数据
readRawData(size: number): ArrayBuffer
```

#### 远程对象操作

```javascript
// 序列化远程对象
writeRemoteObject(object: IRemoteObject): boolean

// 反序列化远程对象
readRemoteObject(): IRemoteObject
```

#### 文件描述符操作

```javascript
// 写入文件描述符（需要 TF_ACCEPT_FDS 标志）
writeFileDescriptor(fd: number): boolean

// 读取文件描述符
readFileDescriptor(): number
```

#### 接口令牌操作

```javascript
// 写入接口令牌（用于接口校验）
writeInterfaceToken(token: string): boolean

// 读取接口令牌
readInterfaceToken(): string
```

### 完整使用示例

以下是一个完整的 IPC 调用示例，演示如何正确使用 MessageParcel 进行数据传输：

```javascript
import rpc from '@ohos.rpc';

// 定义消息码
const TRANSACTION_GET_DATA = 1;
const TRANSACTION_SEND_DATA = 2;
const TRANSACTION_GET_FILE = 3;

// 客户端发送请求
async function sendRequest(proxy: rpc.RemoteProxy): Promise<number> {
    // 创建消息
    const data = rpc.MessageParcel.create();
    const reply = rpc.MessageParcel.create();
    const option = new rpc.MessageOption();

    try {
        // 1. 写入接口令牌（必须与 Stub 端匹配）
        data.writeInterfaceToken('com.example.IMyService');

        // 2. 写入请求参数
        data.writeInt32(100);
        data.writeString('Hello from client');

        // 3. 发送请求（Promise 模式）
        const result = await proxy.sendRequest(
            TRANSACTION_GET_DATA,
            data,
            reply,
            option
        );

        // 4. 检查错误码
        if (result.errCode !== 0) {
            console.error(`IPC error: ${result.errCode}`);
            return -1;
        }

        // 5. 读取响应数据
        const responseCode = reply.readInt32();
        const responseMessage = reply.readString();

        console.log(`Response: ${responseCode} - ${responseMessage}`);
        return responseCode;

    } finally {
        // 6. 释放资源
        data.reclaim();
        reply.reclaim();
    }
}

// 服务端处理请求
class MyRemoteObject extends rpc.RemoteObject {
    constructor(descriptor: string) {
        super(descriptor);
    }

    onRemoteMessageRequest(
        code: number,
        data: rpc.MessageParcel,
        reply: rpc.MessageParcel,
        option: rpc.MessageOption
    ): boolean {
        // 1. 验证接口令牌
        const token = data.readInterfaceToken();
        if (token !== 'com.example.IMyService') {
            reply.writeInt32(rpc.ErrorCode.ERR_INVALID_PARAMS);
            return false;
        }

        // 2. 根据消息码处理
        switch (code) {
            case TRANSACTION_GET_DATA:
                return this.handleGetData(data, reply);

            case TRANSACTION_SEND_DATA:
                return this.handleSendData(data, reply);

            case TRANSACTION_GET_FILE:
                return this.handleGetFile(data, reply, option);

            default:
                // 未知消息码，调用父类处理
                return super.onRemoteMessageRequest(code, data, reply, option);
        }
    }

    private handleGetData(
        data: rpc.MessageParcel,
        reply: rpc.MessageParcel
    ): boolean {
        try {
            // 读取请求参数
            const requestCode = data.readInt32();
            const requestMessage = data.readString();

            console.log(`Received: ${requestCode} - ${requestMessage}`);

            // 写入响应
            reply.writeInt32(200);
            reply.writeString('Data received successfully');
            return true;

        } catch (e) {
            console.error(`Handle error: ${e}`);
            reply.writeInt32(rpc.ErrorCode.ERR_UNKNOWN);
            return false;
        }
    }

    private handleGetFile(
        data: rpc.MessageParcel,
        reply: rpc.MessageParcel,
        option: rpc.MessageOption
    ): boolean {
        // 读取文件路径
        const filePath = data.readString();

        // 需要 TF_ACCEPT_FDS 标志才能传输文件描述符
        if (option.getFlags() & rpc.MessageOption.TF_ACCEPT_FDS) {
            try {
                const fd = this.openFile(filePath);
                reply.writeFileDescriptor(fd);
                return true;
            } catch (e) {
                return false;
            }
        } else {
            // 不支持 FD 传输
            return false;
        }
    }

    private openFile(path: string): number {
        // 打开文件并返回文件描述符
        return -1;
    }
}
```

### 错误处理模式

MessageParcel 的读写方法在失败时会返回 `false` 或抛出异常。以下是推荐的错误处理模式：

```javascript
import rpc from '@ohos.rpc';

function safeReadData(data: rpc.MessageParcel): { success: boolean; value?: any; error?: number } {
    try {
        // 尝试读取数据
        if (!data.isReadable()) {
            return { success: false, error: rpc.ErrorCode.ERR_INVALID_PARAMS };
        }

        const value = data.readInt32();
        return { success: true, value };

    } catch (e) {
        // 捕获解析异常
        console.error(`Parse error: ${e}`);
        return { success: false, error: rpc.ErrorCode.ERR_UNKNOWN };
    }
}

// 使用异步回调模式发送请求
function sendRequestWithCallback(
    proxy: rpc.RemoteProxy,
    callback: (result: number) => void
): void {
    const data = rpc.MessageParcel.create();
    const reply = rpc.MessageParcel.create();
    const option = new rpc.MessageOption();

    data.writeInterfaceToken('interface.token');
    data.writeInt32(42);

    proxy.sendRequest(
        1,
        data,
        reply,
        option,
        (asyncResult: rpc.SendRequestResult) => {
            try {
                if (asyncResult.errCode !== 0) {
                    callback(asyncResult.errCode);
                    return;
                }

                const response = asyncResult.reply.readInt32();
                callback(response);

            } finally {
                data.reclaim();
                reply.reclaim();
            }
        }
    );
}
```

### 数据容量与限制

MessageParcel 对数据大小有严格限制，开发时需要注意：

| 限制项 | 默认值 | 说明 |
|--------|--------|------|
| **单次传输最大数据量** | ~1 MB | 超过此限制应使用 Ashmem |
| **缓冲区初始容量** | 动态分配 | 通过 `getCapacity()` 获取 |
| **当前数据大小** | 动态 | 通过 `getSize()` 获取 |

```javascript
import rpc from '@ohos.rpc';

function checkCapacityBeforeWrite(data: rpc.MessageParcel, dataSize: number): boolean {
    const capacity = data.getCapacity();
    const currentSize = data.getSize();

    // 检查是否有足够空间
    if (currentSize + dataSize > capacity) {
        console.warn(`Insufficient capacity: ${currentSize}/${capacity}`);

        // 建议使用 Ashmem 传输大文件
        if (dataSize > 64 * 1024) { // > 64KB
            console.info('Consider using Ashmem for large data');
        }
        return false;
    }
    return true;
}
```

**证据**:
- `napi_common/source/napi_message_parcel_write.cpp` - MessageParcel N-API 导出实现
- `interfaces/innerkits/ipc_core/include/message_parcel.h:26-164` - MessageParcel 头文件接口定义

## 4.4 MessageOption API

### 构造函数

```javascript
// 同步调用（默认）
let optionSync = new rpc.MessageOption();

// 异步调用
let optionAsync = new rpc.MessageOption(rpc.MessageOption.TF_ASYNC);

// 带文件描述符
let optionFd = new rpc.MessageOption(rpc.MessageOption.TF_ACCEPT_FDS);
```

### 静态属性详解

MessageOption 定义了一组标志位，用于控制 IPC 调用的行为模式。以下是各标志位的详细说明：

| 标志常量 | 十六进制值 | 十进制值 | 描述 | 使用场景 |
|----------|-----------|----------|------|----------|
| `TF_SYNC` | `0x00` | 0 | **同步调用**（默认）：调用方阻塞等待响应，超时由 `setWaitTime()` 控制 | 简单的请求-响应模式 |
| `TF_ASYNC` | `0x01` | 1 | **异步调用**：调用方不阻塞，通过回调或 Promise 返回结果 | 不需要立即响应的场景 |
| `TF_STATUS_CODE` | `0x08` | 8 | **状态码模式**：启用详细状态码返回，提供更丰富的错误信息 | 需要精确错误诊断时 |
| `TF_ACCEPT_FDS` | `0x10` | 16 | **接受文件描述符**：允许在消息中传递 Unix 文件描述符 | 传递打开的文件、Socket 等 |
| `TF_WAIT_TIME` | `0x08` | 8 | **超时标志**：配合 `setWaitTime()` 使用，标记启用自定义超时 | 自定义超时设置 |

### 实例方法详解

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `setFlags(flags: number): void` | `flags` 为上述标志的组合值 | `void` | 设置消息选项标志，可使用位运算组合多个标志 |
| `getFlags(): number` | 无 | `number` | 获取当前设置的标志位 |
| `setWaitTime(waitTime: number): void` | `waitTime` 毫秒数 | `void` | 设置同步调用的超时时间，单位为毫秒，默认值通常为 5000ms |
| `getWaitTime(): number` | 无 | `number` | 获取当前设置的等待时间 |

### 标志组合使用

```javascript
import rpc from '@ohos.rpc';

// 组合多个标志
const SYNC_WITH_FDS = rpc.MessageOption.TF_SYNC | rpc.MessageOption.TF_ACCEPT_FDS;
const ASYNC_WITH_STATUS = rpc.MessageOption.TF_ASYNC | rpc.MessageOption.TF_STATUS_CODE;

const option1 = new rpc.MessageOption();
option1.setFlags(SYNC_WITH_FDS);
option1.setWaitTime(10000); // 10秒超时

const option2 = new rpc.MessageOption();
option2.setFlags(ASYNC_WITH_STATUS);
```

### 使用场景示例

#### 场景一：同步调用

```javascript
import rpc from '@ohos.rpc';

async function synchronousCall(proxy: rpc.RemoteProxy): Promise<number> {
    const data = rpc.MessageParcel.create();
    const reply = rpc.MessageParcel.create();

    // 默认同步模式，无需额外设置
    const option = new rpc.MessageOption();

    try {
        data.writeInt32(100);
        data.writeString('sync request');

        // 发送请求并等待响应
        const result = await proxy.sendRequest(1, data, reply, option);

        if (result.errCode !== 0) {
            console.error(`Error: ${result.errCode}`);
            return -1;
        }

        return reply.readInt32();

    } finally {
        data.reclaim();
        reply.reclaim();
    }
}
```

#### 场景二：异步调用（回调模式）

```javascript
import rpc from '@ohos.rpc';

function asynchronousCallWithCallback(
    proxy: rpc.RemoteProxy,
    onComplete: (result: number) => void
): void {
    const data = rpc.MessageParcel.create();
    const reply = rpc.MessageParcel.create();
    const option = new rpc.MessageOption();

    // 设置异步标志
    option.setFlags(rpc.MessageOption.TF_ASYNC);

    data.writeInt32(100);

    // 使用回调接收结果
    proxy.sendRequest(
        1,
        data,
        reply,
        option,
        (asyncResult: rpc.SendRequestResult) => {
            try {
                if (asyncResult.errCode !== 0) {
                    onComplete(-1);
                    return;
                }
                const result = asyncResult.reply.readInt32();
                onComplete(result);
            } finally {
                data.reclaim();
                reply.reclaim();
            }
        }
    );
}
```

#### 场景三：传输文件描述符

```javascript
import rpc from '@ohos.rpc';

async function sendFileDescriptor(
    proxy: rpc.RemoteProxy,
    filePath: string
): Promise<boolean> {
    const data = rpc.MessageParcel.create();
    const reply = rpc.MessageParcel.create();
    const option = new rpc.MessageOption();

    // 必须设置 TF_ACCEPT_FDS 才能传输文件描述符
    option.setFlags(rpc.MessageOption.TF_ACCEPT_FDS);

    try {
        // 写入文件路径
        data.writeInterfaceToken('com.example.IFileService');
        data.writeString(filePath);

        const result = await proxy.sendRequest(1, data, reply, option);

        if (result.errCode !== 0) {
            console.error(`Send FD failed: ${result.errCode}`);
            return false;
        }

        // 读取服务器返回的文件描述符
        const serverFd = reply.readFileDescriptor();
        return serverFd >= 0;

    } finally {
        data.reclaim();
        reply.reclaim();
    }
}
```

#### 场景四：自定义超时

```javascript
import rpc from '@ohos.rpc';

class IPCClient {
    private readonly DEFAULT_TIMEOUT = 3000;
    private readonly LONG_TIMEOUT = 30000;

    async quickRequest(proxy: rpc.RemoteProxy): Promise<any> {
        const option = new rpc.MessageOption();
        option.setWaitTime(this.DEFAULT_TIMEOUT);

        return this.sendWithTimeout(proxy, 1, {}, option);
    }

    async longRequest(proxy: rpc.RemoteProxy): Promise<any> {
        const option = new rpc.MessageOption();
        option.setWaitTime(this.LONG_TIMEOUT);

        return this.sendWithTimeout(proxy, 2, {}, option);
    }

    private async sendWithTimeout(
        proxy: rpc.RemoteProxy,
        code: number,
        data: any,
        option: rpc.MessageOption
    ): Promise<any> {
        const parcel = rpc.MessageParcel.create();

        try {
            // 使用 Promise.race 实现超时控制
            const result = await Promise.race([
                proxy.sendRequest(code, parcel, rpc.MessageParcel.create(), option),
                new Promise((_, reject) =>
                    setTimeout(() => reject(new Error('IPC timeout')), option.getWaitTime())
                )
            ]);
            return result;
        } catch (e) {
            console.error(`IPC failed: ${e.message}`);
            throw e;
        } finally {
            parcel.reclaim();
        }
    }
}
```

**证据**: `ipc/native/src/napi/src/napi_message_option.cpp` - MessageOption N-API 导出实现

## 4.5 RemoteObject API

### 构造函数

```javascript
import rpc from '@ohos.rpc';

// 创建 RemoteObject
class MyRemoteObject extends rpc.RemoteObject {
    constructor(descriptor) {
        super(descriptor);
    }

    onRemoteMessageRequest(code, data, reply, option) {
        // 处理请求
        return true;
    }
}

let remoteObj = new MyRemoteObject('my.service.Descriptor');
```

### 实例方法

| JS 方法 | 参数 | 返回值 | 描述 |
|---------|------|--------|------|
| `sendRequest()` | `code`, `data`, `reply`, `option` | `Promise<SendRequestResult>` | 发送请求（Promise） |
| `sendRequest()` | `code`, `data`, `reply`, `option`, `callback` | `void` | 发送请求（回调） |
| `sendMessageRequest()` | 同上 | 同上 | 发送消息请求 |
| `getCallingPid()` | 无 | `number` | 获取调用方 PID |
| `getCallingUid()` | 无 | `number` | 获取调用方 UID |
| `getInterfaceDescriptor()` | 无 | `string` | 获取接口描述符 |
| `addDeathRecipient()` | `DeathRecipient` | `boolean` | 添加死亡通知 |
| `removeDeathRecipient()` | `DeathRecipient` | `boolean` | 移除死亡通知 |
| `isObjectDead()` | 无 | `boolean` | 对象是否已死亡 |
| `reclaim()` | 无 | `void` | 释放对象 |

### 静态属性

| 属性 | 值 | 描述 |
|------|-----|------|
| `PING_TRANSACTION` | `number` | Ping 事务码 |
| `DUMP_TRANSACTION` | `number` | Dump 事务码 |
| `INTERFACE_TRANSACTION` | `number` | 接口事务码 |
| `MIN_TRANSACTION_ID` | `number` | 最小事务码 |
| `MAX_TRANSACTION_ID` | `number` | 最大事务码 |

**证据**: `napi_common/source/napi_remote_object.cpp:1712-1732`

## 4.6 RemoteProxy API

### 构造函数

```javascript
import rpc from '@ohos.rpc';

// RemoteProxy 需要通过 SAMgr 获取，不能直接构造
```

### 实例方法

| JS 方法 | 参数 | 返回值 | 描述 |
|---------|------|--------|------|
| `queryLocalInterface()` | `string` | `IRemoteObject` | 查询本地接口 |
| `getInterfaceDescriptor()` | 无 | `string` | 获取接口描述符 |
| `sendRequest()` | `code`, `data`, `reply`, `option` | `Promise<SendRequestResult>` | 发送请求 |
| `sendRequest()` | `code`, `data`, `reply`, `option`, `callback` | `void` | 发送请求（回调） |
| `isObjectDead()` | 无 | `boolean` | 对象是否已死亡 |
| `addDeathRecipient()` | `DeathRecipient` | `boolean` | 添加死亡通知 |
| `removeDeathRecipient()` | `DeathRecipient` | `boolean` | 移除死亡通知 |

**证据**: `ipc/native/src/napi/src/napi_remote_proxy.cpp:729-750`

## 4.7 Ashmem API

### 静态方法

| JS 方法 | 参数 | 返回值 | 描述 |
|---------|------|--------|------|
| `mapAshmem()` | `string`, `number` | `Ashmem` | 映射匿名共享内存 |
| `mapAshmem()` | `number`, `number` | `Ashmem` | 通过 fd 映射 |

### 实例方法

| JS 方法 | 参数 | 返回值 | 描述 |
|---------|------|--------|------|
| `writeToAshmem()` | `ArrayBuffer`, `number`, `number` | `boolean` | 写入数据 |
| `readFromAshmem()` | `ArrayBuffer`, `number`, `number` | `number` | 读取数据 |
| `getAshmemSize()` | 无 | `number` | 获取大小 |
| `unmapAshmem()` | 无 | `void` | 解除映射 |
| `closeAshmem()` | 无 | `void` | 关闭共享内存 |

**证据**: `napi_common/source/napi_ashmem.cpp` - Ashmem 导出

## 4.8 错误码

### 错误码枚举

| 错误码 | 名称 | 描述 |
|--------|------|------|
| 0 | `ERR_OK` | 成功 |
| -1 | `ERR_UNKNOWN` | 未知错误 |
| 1 | `ERR_PERMISSION` | 权限错误 |
| 2 | `ERR_INVALID_PARAMS` | 参数错误 |
| 3 | `ERR_MEMORY` | 内存错误 |
| 4 | `ERR_IPC_SEND_FAILED` | IPC 发送失败 |
| 5 | `ERR_IPC_READ_FAILED` | IPC 读取失败 |
| 6 | `ERR_REMOTE_OBJECT` | 远程对象错误 |

**证据**: `napi_common/source/napi_rpc_error.cpp` - 错误码导出

### MessageSequence API（推荐新接口）

MessageSequence 是 MessageParcel 的**推荐替代接口**，自 OpenHarmony 3.0 起引入，提供更一致的行为和更好的性能优化。

#### 构造函数

```javascript
// 创建 MessageSequence 实例
const sequence = rpc.MessageSequence.create();
```

#### 核心方法

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `create()` | 无 | `MessageSequence` | 静态工厂方法，创建新实例 |
| `reclaim()` | 无 | `void` | 释放序列对象及其关联的内存 |
| `writeRemoteObject(object: IRemoteObject)` | `IRemoteObject` | `boolean` | 写入远程对象引用 |
| `readRemoteObject()` | 无 | `IRemoteObject` | 读取远程对象引用 |
| `writeInterfaceToken(token: string)` | `string` | `boolean` | 写入接口令牌 |
| `readInterfaceToken()` | 无 | `string` | 读取接口令牌 |
| `writeInt32(value: number)` | `number` | `boolean` | 写入 32 位整数 |
| `readInt32()` | 无 | `number` | 读取 32 位整数 |
| `writeLong(value: number)` | `number` | `boolean` | 写入 64 位长整数 |
| `readLong()` | 无 | `number` | 读取 64 位长整数 |
| `writeDouble(value: number)` | `number` | `boolean` | 写入双精度浮点 |
| `readDouble()` | 无 | `number` | 读取双精度浮点 |
| `writeString(value: string)` | `string` | `boolean` | 写入字符串 |
| `readString()` | 无 | `string` | 读取字符串 |
| `writeBoolean(value: boolean)` | `boolean` | `boolean` | 写入布尔值 |
| `readBoolean()` | 无 | `boolean` | 读取布尔值 |
| `writeFileDescriptor(fd: number)` | `number` | `boolean` | 写入文件描述符 |
| `readFileDescriptor()` | 无 | `number` | 读取文件描述符 |
| `getSize()` | 无 | `number` | 获取当前数据大小 |
| `getCapacity()` | 无 | `number` | 获取缓冲区容量 |
| `getDataSize()` | 无 | `number` | 获取数据部分大小 |
| `getBuffersSize()` | 无 | `number` | 获取缓冲区总大小 |

#### MessageParcel 与 MessageSequence 对比

| 特性 | MessageParcel | MessageSequence |
|------|---------------|-----------------|
| **引入版本** | 早期版本 | OpenHarmony 3.0+ |
| **推荐程度** | 兼容使用 | **强烈推荐** |
| **API 一致性** | 部分不一致 | 完全一致 |
| **性能优化** | 标准 | 优化提升 |
| **Ashmem 集成** | 需要单独处理 | 内置支持 |
| **跨设备支持** | 需要额外配置 | 原生支持 |

#### MessageSequence 使用示例

```javascript
import rpc from '@ohos.rpc';

class MyServiceSequence extends rpc.RemoteObject {
    constructor(descriptor: string) {
        super(descriptor);
    }

    onRemoteMessageRequest(
        code: number,
        data: rpc.MessageSequence,
        reply: rpc.MessageSequence,
        option: rpc.MessageOption
    ): boolean {
        // 验证接口令牌
        const token = data.readInterfaceToken();
        if (token !== this.getInterfaceDescriptor()) {
            reply.writeInt32(rpc.ErrorCode.ERR_INVALID_PARAMS);
            return false;
        }

        switch (code) {
            case 1: // GET_DATA
                return this.handleGetData(data, reply);

            case 2: // SEND_DATA
                return this.handleSendData(data, reply);

            case 3: // GET_FILE
                return this.handleGetFile(data, reply);

            default:
                return false;
        }
    }

    private handleGetData(
        data: rpc.MessageSequence,
        reply: rpc.MessageSequence
    ): boolean {
        const id = data.readLong();
        const name = data.readString();

        console.log(`Request: id=${id}, name=${name}`);

        // 写入响应
        reply.writeLong(id);
        reply.writeString(`Hello, ${name}!`);
        reply.writeInt32(200);
        return true;
    }

    private handleSendData(
        data: rpc.MessageSequence,
        reply: rpc.MessageSequence
    ): boolean {
        // 接收数据
        const values = [];
        while (data.getDataSize() > 0) {
            values.push(data.readString());
        }

        // 处理数据
        console.log(`Received ${values.length} items`);

        reply.writeInt32(values.length);
        return true;
    }

    private handleGetFile(
        data: rpc.MessageSequence,
        reply: rpc.MessageSequence
    ): boolean {
        const path = data.readString();

        // 检查 FD 标志
        if (option.getFlags() & rpc.MessageOption.TF_ACCEPT_FDS) {
            const fd = this.openFile(path);
            reply.writeFileDescriptor(fd);
            return true;
        }
        return false;
    }

    private openFile(path: string): number {
        return -1;
    }
}
```

**证据**:
- `napi_common/source/napi_message_sequence_write.cpp` - MessageSequence N-API 导出实现
- `interfaces/innerkits/ipc_core/include/message_parcel.h` - MessageSequence 接口定义

### CallingInfo API

CallingInfo 提供了一种便捷的方式来获取当前调用链的详细信息，适用于需要记录调用追踪或审计的场景。

#### 静态方法

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `get()` | 无 | `CallingInfo` | 获取当前调用链的 CallingInfo 对象 |

#### CallingInfo 属性

| 属性名 | 类型 | 描述 |
|--------|------|------|
| `callingPid` | `number` | 调用方进程 ID |
| `callingUid` | `number` | 调用方用户 ID |
| `callingTokenId` | `number` | 调用方令牌 ID |
| `callingDeviceId` | `string` | 调用方设备 ID |
| `localDeviceId` | `string` | 本地设备 ID |
| `isLocalCalling` | `boolean` | 是否本地调用 |
| `firstPid` | `number` | 原始调用方 PID（调用链起点） |
| `firstUid` | `number` | 原始调用方 UID |

#### 使用示例

```javascript
import rpc from '@ohos.rpc';

class AuditService extends rpc.RemoteObject {
    constructor(descriptor: string) {
        super(descriptor);
    }

    onRemoteMessageRequest(
        code: number,
        data: rpc.MessageParcel,
        reply: rpc.MessageParcel,
        option: rpc.MessageOption
    ): boolean {
        // 获取调用链信息
        const callingInfo = rpc.CallingInfo.get();

        // 记录审计日志
        this.logAudit({
            timestamp: Date.now(),
            callerPid: callingInfo.callingPid,
            callerUid: callingInfo.callingUid,
            callerDevice: callingInfo.callingDeviceId,
            firstPid: callingInfo.firstPid,
            firstUid: callingInfo.firstUid,
            isLocal: callingInfo.isLocalCalling
        });

        // 如果是跨设备调用，需要额外验证
        if (!callingInfo.isLocalCalling) {
            return this.validateCrossDeviceRequest(callingInfo);
        }

        return true;
    }

    private logAudit(info: any): void {
        console.log(`[AUDIT] ${JSON.stringify(info)}`);
    }

    private validateCrossDeviceRequest(info: any): boolean {
        // 跨设备验证逻辑
        return true;
    }
}
```

**证据**: `ipc/native/src/napi/src/napi_calling_info.cpp` - CallingInfo N-API 导出实现

### DeathRecipient API

DeathRecipient 用于监听远程对象的死亡通知，当远程对象所在进程崩溃或终止时，会触发回调。

#### 创建 DeathRecipient

```javascript
import rpc from '@ohos.rpc';

const deathRecipient = {
    onRemoteDied(): void {
        console.log('Remote object died!');
        // 执行清理操作
        this.cleanup();
    },
    cleanup(): void {
        // 资源清理
    }
};

// 注册死亡通知
proxy.addDeathRecipient(deathRecipient);

// 移除死亡通知
proxy.removeDeathRecipient(deathRecipient);
```

#### 使用示例

```javascript
import rpc from '@ohos.rpc';

class RobustClient {
    private proxy: rpc.RemoteProxy | null = null;
    private deathRecipient: any = null;

    async connectToService(serviceDescriptor: string): Promise<boolean> {
        try {
            // 获取服务代理
            const samgr = rpc.IPCSkeleton.getContextObject();
            const remoteObject = samgr.getSystemAbility(serviceDescriptor);

            this.proxy = remoteObject;

            // 创建死亡通知接收器
            this.deathRecipient = {
                onRemoteDied: (): void => {
                    console.log('Service died, attempting reconnect...');
                    this.handleRemoteDeath();
                }
            };

            // 注册死亡通知
            this.proxy.addDeathRecipient(this.deathRecipient);

            return true;

        } catch (e) {
            console.error(`Connection failed: ${e}`);
            return false;
        }
    }

    private async handleRemoteDeath(): Promise<void> {
        // 移除旧的死亡通知
        if (this.proxy && this.deathRecipient) {
            this.proxy.removeDeathRecipient(this.deathRecipient);
        }

        // 等待后重连
        await this.delay(5000);

        // 尝试重新连接
        await this.connectToService('my.service');
    }

    private delay(ms: number): Promise<void> {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    async sendRequest(data: any): Promise<any> {
        if (!this.proxy || this.proxy.isObjectDead()) {
            throw new Error('Proxy is not available');
        }

        // 发送请求
        // ...
    }
}
```

### Ashmem API（匿名共享内存）

Ashmem 用于在进程间共享大块内存数据，避免通过 IPC 传递大数据的性能开销。

#### 静态方法

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `createAshmem(name: string, size: number)` | 内存名称、大小（字节） | `Ashmem` | 创建匿名共享内存区域 |
| `createAshmemFromExisting(fd: number, size: number)` | 文件描述符、大小 | `Ashmem` | 从现有 FD 创建 Ashmem |
| `mapAshmem(name: string, size: number)` | 名称、大小 | `Ashmem` | 映射已有 Ashmem |

#### 实例方法

| 方法名 | 参数类型 | 返回类型 | 描述 |
|--------|----------|----------|------|
| `writeToAshmem(data: ArrayBuffer, size: number, offset: number)` | 数据缓冲区、大小、偏移 | `boolean` | 写入数据到 Ashmem |
| `readFromAshmem(data: ArrayBuffer, size: number, offset: number)` | 数据缓冲区、大小、偏移 | `number` | 从 Ashmem 读取数据 |
| `getAshmemSize(): number` | 无 | `number` | 获取 Ashmem 大小 |
| `getAshmemReadOnlySize(): number` | 无 | `number` | 获取只读区域大小 |
| `unmapAshmem(): void` | 无 | `void` | 解除内存映射 |
| `closeAshmem(): void` | 无 | `void` | 关闭 Ashmem，释放资源 |
| `setProtection(protection: number): void` | 保护级别（0-3） | `void` | 设置内存保护级别 |

#### 内存保护级别

| 级别 | 值 | 描述 |
|------|-----|------|
| `PROT_READ` | 1 | 只读 |
| `PROT_WRITE` | 2 | 可写 |
| `PROT_READ_WRITE` | 3 | 可读写 |

#### 使用示例

```javascript
import rpc from '@ohos.rpc';

class LargeDataTransfer {
    // 发送端：使用 Ashmem 发送大文件
    async sendLargeData(
        proxy: rpc.RemoteProxy,
        filePath: string
    ): Promise<boolean> {
        try {
            // 1. 读取文件数据
            const fileData = await this.readFile(filePath);

            // 2. 创建 Ashmem
            const ashmem = rpc.Ashmem.createAshmem('large_file', fileData.byteLength);

            // 3. 写入数据到 Ashmem
            ashmem.writeToAshmem(fileData, fileData.byteLength, 0);

            // 4. 设置只读保护（发送后防止修改）
            ashmem.setProtection(1); // PROT_READ

            // 5. 创建 MessageParcel 传输 Ashmem 描述符
            const data = rpc.MessageParcel.create();
            const reply = rpc.MessageParcel.create();
            const option = new rpc.MessageOption();

            data.writeInterfaceToken('com.example.ILargeData');
            data.writeAshmem(ashmem);

            // 6. 发送请求
            const result = await proxy.sendRequest(1, data, reply, option);

            if (result.errCode !== 0) {
                console.error(`Transfer failed: ${result.errCode}`);
                return false;
            }

            console.log('Large data transfer completed');
            return true;

        } finally {
            // 7. 清理 Ashmem
            ashmem?.closeAshmem();
        }
    }

    // 接收端：接收 Ashmem
    async receiveLargeData(
        data: rpc.MessageParcel
    ): Promise<ArrayBuffer | null> {
        try {
            // 读取 Ashmem
            const ashmem = data.readAshmem();
            if (!ashmem) {
                console.error('Failed to read Ashmem');
                return null;
            }

            // 获取数据大小
            const size = ashmem.getAshmemSize();

            // 创建缓冲区接收数据
            const buffer = new ArrayBuffer(size);
            const bytesRead = ashmem.readFromAshmem(buffer, size, 0);

            if (bytesRead !== size) {
                console.error(`Partial read: ${bytesRead}/${size}`);
                return null;
            }

            console.log(`Received ${bytesRead} bytes via Ashmem`);
            return buffer;

        } catch (e) {
            console.error(`Receive error: ${e}`);
            return null;
        }
    }

    private async readFile(path: string): Promise<ArrayBuffer> {
        // 文件读取实现
        return new ArrayBuffer(0);
    }
}
```

#### 最佳实践

1. **及时释放**：使用完毕后立即调用 `closeAshmem()` 释放系统资源
2. **设置适当保护**：发送数据后设置为 `PROT_READ`，防止接收方误修改
3. **大小限制**：单个 Ashmem 不建议超过 16MB
4. **错误处理**：所有 Ashmem 操作应检查返回值

**证据**: `napi_common/source/napi_ashmem.cpp` - Ashmem N-API 导出实现

## 4.9 完整 API 速查表

### MessageParcel 速查

```javascript
// 创建和释放
const parcel = rpc.MessageParcel.create();
parcel.reclaim();

// 写入
parcel.writeInterfaceToken(token);
parcel.writeInt32(value);
parcel.writeString(str);
parcel.writeRemoteObject(obj);
parcel.writeAshmem(ashmem);
parcel.writeFileDescriptor(fd);

// 读取
const token = parcel.readInterfaceToken();
const value = parcel.readInt32();
const str = parcel.readString();
const obj = parcel.readRemoteObject();
const ashmem = parcel.readAshmem();
const fd = parcel.readFileDescriptor();

// 查询
const size = parcel.getSize();
const capacity = parcel.getCapacity();
```

### MessageSequence 速查

```javascript
// 创建和释放
const sequence = rpc.MessageSequence.create();
sequence.reclaim();

// 写入
sequence.writeInterfaceToken(token);
sequence.writeLong(value);
sequence.writeDouble(value);
sequence.writeString(str);
sequence.writeRemoteObject(obj);

// 读取
const token = sequence.readInterfaceToken();
const value = sequence.readLong();
const value = sequence.readDouble();
const str = sequence.readString();
const obj = sequence.readRemoteObject();

// 查询
const size = sequence.getSize();
const capacity = sequence.getCapacity();
```

### MessageOption 速查

```javascript
// 创建
const option = new rpc.MessageOption();

// 设置模式
option.setFlags(rpc.MessageOption.TF_SYNC);        // 同步
option.setFlags(rpc.MessageOption.TF_ASYNC);       // 异步
option.setFlags(rpc.MessageOption.TF_ACCEPT_FDS); // 接受 FD

// 设置超时
option.setWaitTime(5000); // 5秒

// 获取
const flags = option.getFlags();
const waitTime = option.getWaitTime();
```

### IPCSkeleton 速查

```javascript
// 调用者信息
const pid = rpc.IPCSkeleton.getCallingPid();
const uid = rpc.IPCSkeleton.getCallingUid();
const tokenId = rpc.IPCSkeleton.getCallingTokenId();
const deviceId = rpc.IPCSkeleton.getCallingDeviceID();
const isLocal = rpc.IPCSkeleton.isLocalCalling();

// 设备信息
const localDevice = rpc.IPCSkeleton.getLocalDeviceID();

// 上下文
const context = rpc.IPCSkeleton.getContextObject();

// 身份管理
const token = rpc.IPCSkeleton.resetCallingIdentity();
rpc.IPCSkeleton.restoreCallingIdentity(token);
```

---

*证据来源*:
- `napi_rpc_native_module.cpp:30-42` - 模块导出函数定义
- `napi_ipc_skeleton.cpp:472-486` - IPCSkeleton N-API 导出定义
- `napi_message_option.cpp` - MessageOption N-API 导出定义
- `napi_message_parcel_write.cpp` - MessageParcel N-API 导出实现
- `napi_message_sequence_write.cpp` - MessageSequence N-API 导出实现
- `napi_ashmem.cpp` - Ashmem N-API 导出实现
- `napi_calling_info.cpp` - CallingInfo N-API 导出实现
- `interfaces/innerkits/ipc_core/include/message_parcel.h` - MessageParcel/C 头文件接口

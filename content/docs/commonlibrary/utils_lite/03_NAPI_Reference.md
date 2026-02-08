# N-API 参考

> utils_lite 提供的 JavaScript API 完整参考。

## JSI 框架说明

utils_lite 使用 **JSI (JavaScript Interface)** 框架而非标准 N-API。JSI 是 OpenHarmony ACELite 框架的轻量级 JavaScript 绑定机制。

**注册模式**：
```cpp
JSI::SetModuleAPI(exports, "methodName", HandlerFunc);
```

**证据来源**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp:255-261`

---

## KV 存储 API

**命名空间**：无（全局方法）
**模块路径**：`js/builtin/kvstorekit/`

### API 清单

| JS 方法 | C++ 处理函数 | 同步/异步 | 导出位置 |
|---------|-------------|----------|----------|
| `get(key)` | `NativeapiKv::Get` | 异步 | line 263-266 |
| `set(key, value)` | `NativeapiKv::Set` | 异步 | line 268-271 |
| `delete(key)` | `NativeapiKv::Delete` | 异步 | line 273-276 |
| `clear()` | `NativeapiKv::Clear` | 异步 | line 278-291 |

### get(key)

**功能**：获取指定 key 对应的 value

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| key | string | 是 | 键名，长度 1-32 字节 |

**返回值**：Promise\<string | undefined\>

**错误码**：
| 错误码 | 说明 |
|--------|------|
| 成功 | 返回 value 字符串 |
| -1 | 操作失败 |
| -9 | 参数错误 |

**参数校验**：
- Key 长度检查：`IsValidKey()` - 1-32 字节
- 字符集检查：仅小写字母、数字、下划线、点

**代码证据**：`js/builtin/kvstorekit/src/nativeapi_kv.cpp:30-43`

### set(key, value)

**功能**：设置或更新 key-value 对

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| key | string | 是 | 键名，长度 1-32 字节 |
| value | string | 是 | 值，长度 1-128 字节 |

**返回值**：Promise\<boolean\>

**错误码**：
| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 失败 |
| -9 | 参数错误 |

**参数校验**：
- Key：`IsValidKey()` - 同上
- Value：`IsValidValue()` - 1-128 字节

**代码证据**：`js/builtin/kvstorekit/src/nativeapi_kv_impl.c:33-43`

### delete(key)

**功能**：删除指定 key

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| key | string | 是 | 键名 |

**返回值**：Promise\<boolean\>

### clear()

**功能**：清除所有缓存的 key-value 对

**前置条件**：`FEATURE_KV_CACHE` 需启用

**返回值**：Promise\<void\>

---

## 文件操作 API

**命名空间**：`@ohos.fileio` 或类似
**模块路径**：`js/builtin/filekit/`

### API 清单

| JS 方法 | C++ 处理函数 | 同步/异步 | 条件编译 | 导出位置 |
|---------|-------------|----------|----------|----------|
| `move(src, dest)` | `NativeapiFs::MoveFile` | 异步 | - | line 532-535 |
| `copy(src, dest)` | `NativeapiFs::CopyFile` | 异步 | - | line 537-540 |
| `delete(path)` | `NativeapiFs::DeleteFile` | 异步 | - | line 542-545 |
| `list(dirPath)` | `NativeapiFs::GetFileList` | 异步 | - | line 547-550 |
| `get(filePath)` | `NativeapiFs::GetFileInfo` | 异步 | - | line 552-555 |
| `writeText(path, content)` | `NativeapiFs::WriteTextFile` | 异步 | - | line 557-560 |
| `readText(path)` | `NativeapiFs::ReadTextFile` | 异步 | - | line 562-565 |
| `access(path)` | `NativeapiFs::Access` | 异步 | - | line 567-570 |
| `mkdir(path)` | `NativeapiFs::CreateDir` | 异步 | - | line 572-575 |
| `rmdir(path)` | `NativeapiFs::RemoveDir` | 异步 | - | line 577-580 |
| `readArrayBuffer(path)` | `NativeapiFs::ReadArrayFile` | 异步 | 条件编译 | line 583-586 |
| `writeArrayBuffer(path, buffer)` | `NativeapiFs::WriteArrayFile` | 异步 | 条件编译 | line 588-591 |

### move(src, dest)

**功能**：移动文件（重命名或跨路径移动）

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| src | string | 是 | 源文件路径 |
| dest | string | 是 | 目标路径 |

**路径校验**：`IsValidPath()` - 必须以 `internal://app` 开头，禁止 `..` 路径遍历

**代码证据**：`js/builtin/filekit/src/nativeapi_fs.cpp:30-50`

### copy(src, dest)

**功能**：复制文件

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| src | string | 是 | 源文件路径 |
| dest | string | 是 | 目标路径 |

### delete(path)

**功能**：删除文件

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 文件路径 |

### readText(path)

**功能**：读取文本文件

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 文件路径 |

**返回值**：Promise\<string\>

### writeText(path, content)

**功能**：写入文本文件

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 文件路径 |
| content | string | 是 | 写入内容 |

### access(path)

**功能**：检查文件/目录是否存在

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 路径 |

**返回值**：Promise\<boolean\>

### mkdir(path)

**功能**：创建目录

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 目录路径 |

### rmdir(path)

**功能**：删除空目录

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| path | string | 是 | 目录路径 |

---

## 设备信息 API

**命名空间**：`@ohos.deviceInfo`
**模块路径**：`js/builtin/deviceinfokit/`

### API 清单

| JS 方法/属性 | 类型 | 同步/异步 | 导出位置 |
|--------------|------|----------|----------|
| `getInfo()` | 方法 | 异步 | line 82-85 |
| `deviceType` | 属性 | 同步 | line 27-36 |
| `manufacture` | 属性 | 同步 | line 38-47 |
| `brand` | 属性 | 同步 | line 49-58 |
| `marketName` | 属性 | 同步 | line 60-69 |
| `productSeries` | 属性 | 同步 | line 71-80 |
| `productModel` | 属性 | 同步 | line 82-91 |
| `softwareModel` | 属性 | 同步 | line 93-102 |
| `hardwareModel` | 属性 | 同步 | line 104-113 |
| `hardwareProfile` | 属性 | 同步 | line 115-124 |
| `serial` | 属性 | 同步 | line 126-135 |
| `bootloaderVersion` | 属性 | 同步 | line 137-146 |
| `abiList` | 属性 | 同步 | line 148-157 |
| `securityPatchTag` | 属性 | 同步 | line 159-168 |
| `displayVersion` | 属性 | 同步 | line 170-179 |
| `incrementalVersion` | 属性 | 同步 | line 181-190 |
| `osReleaseType` | 属性 | 同步 | line 192-201 |
| `osFullName` | 属性 | 同步 | line 203-211 |
| `majorVersion` | 属性 | 同步 | line 213-223 |
| `seniorVersion` | 属性 | 同步 | line 225-235 |
| `featureVersion` | 属性 | 同步 | line 237-247 |
| `buildVersion` | 属性 | 同步 | line 249-259 |
| `sdkApiVersion` | 属性 | 同步 | line 261-271 |
| `firstApiVersion` | 属性 | 同步 | line 273-283 |
| `versionId` | 属性 | 同步 | line 285-294 |
| `buildType` | 属性 | 同步 | line 296-305 |
| `buildUser` | 属性 | 同步 | line 307-316 |
| `buildHost` | 属性 | 同步 | line 318-327 |
| `buildTime` | 属性 | 同步 | line 329-338 |
| `buildRootHash` | 属性 | 同步 | line 340-349 |
| `udid` | 属性 | 同步 | line 351-362 |
| `distributionOSName` | 属性 | 同步 | line 364-372 |
| `distributionOSVersion` | 属性 | 同步 | line 374-382 |
| `distributionOSApiVersion` | 属性 | 同步 | line 384-394 |
| `distributionOSReleaseType` | 属性 | 同步 | line 396-404 |

### getInfo()

**功能**：获取设备信息对象

**返回值**：Promise\<DeviceInfo\>

**DeviceInfo 对象包含**：上述所有属性

### 设备属性说明

| 属性 | 说明 | 示例值 |
|------|------|--------|
| deviceType | 设备类型 | "phone" |
| manufacture | 制造商 | "HUAWEI" |
| brand | 品牌 | "HUAWEI" |
| productSeries | 产品系列 | "MateSeries" |
| productModel | 产品型号 | "HUAWEI Mate 60" |
| hardwareModel | 硬件型号 | "Kirin 9000" |
| osFullName | 操作系统全名 | "HarmonyOS" |
| sdkApiVersion | SDK API 版本 | 10 |
| udid | 设备唯一标识 | - |

**代码证据**：`js/builtin/deviceinfokit/src/nativeapi_ohos_deviceinfo.cpp`

---

## 公共工具 API

**模块路径**：`js/builtin/common/`

### 回调处理函数

| 函数 | 功能 | 位置 |
|------|------|------|
| `FailCallBack()` | 错误回调处理 | line 22-55 |
| `SuccessCallBack()` | 成功回调处理 | line 57-75 |
| `IsValidJSIValue()` | 参数有效性校验 | line 77-83 |

### FuncParams 结构体

```cpp
struct FuncParams {
    JSIValue args = JSI::CreateUndefined();
    JSIValue thisVal = JSI::CreateUndefined();
    bool flag = false;
};
```

**代码证据**：`js/builtin/common/src/nativeapi_common.cpp`

---

## 错误码参考

| 错误码 | 说明 | 适用模块 |
|--------|------|----------|
| 0 | 成功 | 全部 |
| -1 | 操作失败 | 全部 |
| -9 | 参数错误 | KV Store, File |
| 0x100+ | 定时器状态错误 | Timer |

---

## 相关跳转

- [概述](00_Overview.md) - 项目定位
- [目录结构](01_Directory_Structure.md) - 模块布局
- [架构说明](02_Architecture.md) - 调用链
- [内部 API](04_Inner_API.md) - C/C++ 接口
- [故障排查](08_Troubleshooting.md) - 常见问题

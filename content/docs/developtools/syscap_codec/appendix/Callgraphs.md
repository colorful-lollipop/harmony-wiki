# Appendix A - 调用链详情

## 目的

本文档详细记录 `syscap_codec` 的关键调用链。

## 1. PCID 编码调用链

```
main.c:71 main()
    │
    ├──▶ main.c:99 OperateByBitMap() [bitMap = 0x111]
    │
    ├──▶ include/syscap_tool.h:40 CreatePCID()
    │
    └──▶ src/create_pcid.c:247 CreatePCID()
            │
            ├──▶ src/context_tool.c:95 CheckFileAndGetFileContext()
            │       │
            │       └──▶ src/context_tool.c:40 GetFileContext()
            │               │
            │               ├──▶ stat() - 获取文件信息
            │               ├──▶ malloc() - 分配缓冲区
            │               ├──▶ fopen() - 打开文件
            │               ├──▶ fread() - 读取内容
            │               └──▶ fclose() - 关闭文件
            │
            ├──▶ cJSON_ParseWithLength() - 解析JSON
            │
            ├──▶ cJSON_GetObjectItem() - 获取syscap对象
            │
            ├──▶ src/create_pcid.c:165 GetOsAndPriSyscapSize()
            │       │
            │       ├──▶ cJSON_GetObjectItem() - 获取os数组
            │       ├──▶ cJSON_GetObjectItem() - 获取private数组
            │       ├──▶ cJSON_GetArraySize() - 获取数组大小
            │       └──▶ 返回osCapSize和privateCapSize
            │
            ├──▶ src/create_pcid.c:201 GetPriSyscapLen()
            │       └──▶ 计算私有syscap总长度
            │
            ├──▶ malloc() - 分配PCID缓冲区
            │
            ├──▶ src/create_pcid.c:70 SetOsSyscap()
            │       │
            │       ├──▶ cJSON_GetArrayItem() - 遍历syscap数组
            │       ├──▶ cJSON_GetObjectItem() - 查询syscap编号
            │       └──▶ 设置位图: pcidBuffer->osSyscap[sector] |= 1 << pos
            │
            ├──▶ src/create_pcid.c:104 SetPriSyscap()
            │       │
            │       ├──▶ cJSON_GetArrayItem() - 遍历private数组
            │       ├──▶ strchr() - 查找'.'位置
            │       └──▶ strcat_s() - 拼接字符串
            │
            ├──▶ src/create_pcid.c:131 SetPCIDHeader()
            │       │
            │       ├──▶ cJSON_GetObjectItem() - 获取api_version
            │       ├──▶ cJSON_GetObjectItem() - 获取system_type
            │       ├──▶ cJSON_GetObjectItem() - 获取manufacturer_id
            │       ├──▶ src/endian_internal.c:46 HtonsInter() - 转换字节序
            │       └──▶ src/endian_internal.c:36 HtonlInter() - 转换字节序
            │
            └──▶ src/context_tool.c:108 ConvertedContextSaveAsFile()
                    │
                    ├──▶ realpath/strncpy_s() - 处理路径
                    ├──▶ strncat_s() - 拼接文件名
                    ├──▶ fopen() - 创建文件
                    ├──▶ fwrite() - 写入内容
                    └──▶ fclose() - 关闭文件
```

## 2. RPCID 编码调用链

```
main.c:71 main()
    │
    ├──▶ main.c:99 OperateByBitMap() [bitMap = 0x109]
    │
    ├──▶ include/syscap_tool.h:35 RPCIDEncode()
    │
    └──▶ src/syscap_tool.c:145 RPCIDEncode()
            │
            ├──▶ src/context_tool.c:95 CheckFileAndGetFileContext()
            │       └──▶ src/context_tool.c:40 GetFileContext()
            │
            ├──▶ cJSON_ParseWithLength() - 解析JSON
            │
            ├──▶ cJSON_GetObjectItem() - 获取syscap数组
            │
            ├──▶ cJSON_GetArraySize() - 获取数组大小
            │
            ├──▶ malloc() - 分配输出缓冲区
            │
            ├──▶ src/syscap_tool.c:83 FillOsCapLength()
            │       │
            │       ├──▶ cJSON_GetObjectItem() - 获取api_version
            │       ├──▶ src/endian_internal.c:46 HtonsInter() - 转换apiVersion
            │       ├──▶ 设置RPCIDHead->apiVersionType = 1
            │       ├──▶ 设置SysCapType = 2 (网络字节序)
            │       ├──▶ 设置SysCapLength (网络字节序)
            │       ├──▶ cJSON_GetArrayItem() - 遍历syscap数组
            │       ├──▶ strchr() - 查找'.'位置
            │       ├──▶ strncmp() - 验证"SystemCapability."前缀
            │       └──▶ memcpy_s() - 复制syscap名称(去掉前缀)
            │
            └──▶ src/context_tool.c:108 ConvertedContextSaveAsFile()
```

## 3. 运行时查询调用链 (N-API)

```
JS: systemCapability.querySystemCapabilities()
    │
    ▼
napi/napi_query_syscap.cpp:177 QuerySystemCapability()
    │
    ├──▶ napi/napi_query_syscap.cpp:152 PreHandleSystemCapability()
    │       │
    │       ├──▶ napi_get_cb_info() - 获取调用信息
    │       ├──▶ napi_typeof() - 检查参数类型
    │       ├──▶ (可选) napi_create_reference() - 保存callback
    │       └──▶ napi_create_promise() - 创建Promise
    │
    ├──▶ napi_create_string_utf8() - 创建工作名称
    │
    ├──▶ napi_create_async_work() - 创建异步工作
    │       │
    │       ├── 执行函数 (工作线程):
    │       │   napi/napi_query_syscap.cpp:186-194
    │       │   │
    │       │   └──▶ napi/napi_query_syscap.cpp:110 GetSystemCapability()
    │       │           │
    │       │           ├──▶ interfaces/inner_api/syscap_interface.c:81 EncodeOsSyscap()
    │       │           │       │
    │       │           │       ├──▶ src/context_tool.c:40 GetFileContext("/system/etc/pcid.sc")
    │       │           │       ├──▶ memcpy_s() - 复制前128字节
    │       │           │       └──▶ FreeContextBuffer()
    │       │           │
    │       │           ├──▶ interfaces/inner_api/syscap_interface.c:110 EncodePrivateSyscap()
    │       │           │       │
    │       │           │       ├──▶ src/context_tool.c:40 GetFileContext("/system/etc/pcid.sc")
    │       │           │       ├──▶ calloc() - 分配内存
    │       │           │       ├──▶ strncpy_s() - 复制私有syscap
    │       │           │       └──▶ FreeContextBuffer()
    │       │           │
    │       │           ├──▶ sprintf_s() - 转换uint32为字符串
    │       │           │
    │       │           ├──▶ interfaces/inner_api/syscap_interface.c:221 DecodePrivateSyscap()
    │       │           │       │
    │       │           │       ├──▶ GetPriSyscapCount() - 统计逗号数量
    │       │           │       ├──▶ malloc() - 分配数组
    │       │           │       ├──▶ strtok_r() - 分割字符串
    │       │           │       └──▶ sprintf_s() - 添加前缀
    │       │           │
    │       │           └──▶ napi/napi_query_syscap.cpp:53 CalculateAllStringLength()
    │       │                   │
    │       │                   ├──▶ strlen() - 计算各部分长度
    │       │                   ├──▶ malloc() - 分配最终缓冲区
    │       │                   ├──▶ memset_s() - 清零
    │       │                   └──▶ sprintf_s() - 拼接字符串
    │       │
    │       └── 完成回调 (主线程):
    │           napi/napi_query_syscap.cpp:196-223
    │           │
    │           ├──▶ napi_get_undefined() / napi_create_string_utf8()
    │           ├──▶ (可选) napi_create_error()
    │           ├──▶ napi_resolve_deferred() / napi_reject_deferred()
    │           ├──▶ (可选) napi_call_function() - 调用callback
    │           ├──▶ napi_delete_reference() - 删除引用
    │           └──▶ napi_delete_async_work() - 删除工作
    │
    └──▶ napi_queue_async_work() - 加入队列
```

## 4. 运行时查询调用链 (ANI/Taihe)

```
JS: querySystemCapabilitie()
    │
    ▼
taihe/syscap/src/ohos.systemCapability.impl.cpp:144 querySystemCapabilitie()
    │
    ├──▶ new SystemCapabilityAsyncContext() - 创建上下文
    │
    ├──▶ taihe/syscap/src/ohos.systemCapability.impl.cpp:100 GetSystemCapability()
    │       │
    │       ├──▶ interfaces/inner_api/syscap_interface.c:81 EncodeOsSyscap()
    │       │       └──▶ 读取 /system/etc/pcid.sc
    │       │
    │       ├──▶ interfaces/inner_api/syscap_interface.c:110 EncodePrivateSyscap()
    │       │       └──▶ 读取 /system/etc/pcid.sc
    │       │
    │       ├──▶ sprintf_s() - 转换uint32为字符串
    │       │
    │       ├──▶ interfaces/inner_api/syscap_interface.c:221 DecodePrivateSyscap()
    │       │       └──▶ 解码私有syscap
    │       │
    │       └──▶ taihe/syscap/src/ohos.systemCapability.impl.cpp:43 CalculateAllStringLength()
    │               └──▶ 拼接字符串
    │
    ├──▶ 检查状态
    │       ├── 成功: 返回字符串
    │       └── 失败: taihe::set_business_error(-1, "key does not exist")
    │
    └──▶ delete asyncContext - 释放上下文
```

## 5. 兼容性比较调用链

```
main.c:71 main()
    │
    ├──▶ main.c:99 OperateByBitMap() [bitMap = 0x60 或 0x64]
    │
    ├──▶ include/syscap_tool.h:41 ComparePcidWithRpcidString()
    │
    └──▶ src/syscap_tool.c:714 ComparePcidWithRpcidString()
            │
            ├──▶ (可选) src/context_tool.c:40 GetFileContext() - 读取文件
            │
            ├──▶ src/syscap_tool.c:595 SeparateSyscapFromString()
            │       │
            │       ├──▶ src/syscap_tool.c:532 CopyInputString()
            │       │       ├──▶ strlen() - 检查长度
            │       │       ├──▶ malloc() - 分配内存
            │       │       └──▶ strcpy_s() - 复制字符串
            │       │
            │       ├──▶ sscanf_s() - 解析32个uint32
            │       │
            │       └──▶ src/syscap_tool.c:558 GetPriSyscapData()
            │               │
            │               ├──▶ 统计逗号数量
            │               ├──▶ malloc() - 分配数组
            │               ├──▶ strtok_r() - 分割字符串
            │               └──▶ strncpy_s() - 复制字符串
            │
            ├──▶ src/syscap_tool.c:688 CompareVersion()
            │       │
            │       ├──▶ src/endian_internal.c:51 NtohsInter() - 转换字节序
            │       └──▶ 比较版本号
            │
            ├──▶ src/syscap_tool.c:638 CompareOsSyscap()
            │       │
            │       ├──▶ 位图比较: (pcid ^ rpcid) & rpcid
            │       ├──▶ 遍历每个bit
            │       ├──▶ src/syscap_tool.c:627 GetSyscapByIndex()
            │       │       └──▶ 在g_arraySyscap中查找
            │       └──▶ printf() - 输出缺失的syscap
            │
            ├──▶ src/syscap_tool.c:664 ComparePriSyscap()
            │       │
            │       ├──▶ 双重循环比较字符串
            │       └──▶ printf() - 输出缺失的syscap
            │
            └──▶ SafeFree() - 释放内存
```

## 6. 内部 API 调用关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                    内部API层 (syscap_interface.c)                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  EncodeOsSyscap() ──────────────────────────────────────────────┐  │
│    │                                                            │  │
│    ├──▶ GetFileContext("/system/etc/pcid.sc")                   │  │
│    ├──▶ memcpy_s(output, PCID_MAIN_BYTES, contextBuffer, 128)   │  │
│    └──▶ FreeContextBuffer()                                     │  │
│                                                                 │  │
│  EncodePrivateSyscap() ─────────────────────────────────────────┤  │
│    │                                                            │  │
│    ├──▶ GetFileContext("/system/etc/pcid.sc")                   │  │
│    ├──▶ calloc(priLen, sizeof(char))                            │  │
│    ├──▶ strncpy_s(outputStr, priLen, ...)                       │  │
│    └──▶ FreeContextBuffer()                                     │  │
│                                                                 │  │
│  DecodeOsSyscap() ──────────────────────────────────────────────┤  │
│    │                                                            │  │
│    ├──▶ 遍历位图每个bit                                         │  │
│    ├──▶ 在g_arraySyscap中查找对应字符串                         │  │
│    ├──▶ malloc(count * SINGLE_SYSCAP_LEN)                       │  │
│    └──▶ strcpy_s() - 复制字符串                                 │  │
│                                                                 │  │
│  DecodePrivateSyscap() ─────────────────────────────────────────┤  │
│    │                                                            │  │
│    ├──▶ GetPriSyscapCount() - 统计逗号                          │  │
│    ├──▶ malloc(syscapCnt * SINGLE_SYSCAP_LEN)                   │  │
│    ├──▶ strtok_r() - 分割字符串                                 │  │
│    └──▶ sprintf_s() - 添加"SystemCapability."前缀               │  │
│                                                                 │  │
│  ComparePcidString() ───────────────────────────────────────────┤  │
│    │                                                            │  │
│    ├──▶ SeparateSyscapFromString() - 解析PCID字符串             │  │
│    ├──▶ SeparateSyscapFromString() - 解析RPCID字符串            │  │
│    ├──▶ ComparePcidWithOsSyscap() - 比较OS syscap               │  │
│    │     └──▶ CheckPcidEachBit()                                │  │
│    │           └──▶ CopySyscopToRet()                           │  │
│    └──▶ ComparePcidWithPriSyscap() - 比较Private syscap         │  │
│                                                                 │  │
│  FreeCompareError() ────────────────────────────────────────────┘  │
│    │                                                               │
│    └──▶ free(result->syscap[i]) - 释放每个元素                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 关键函数索引

| 函数 | 文件 | 行号 | 功能 |
|------|------|------|------|
| `main` | main.c | 71 | 程序入口 |
| `OperateByBitMap` | main.c | 106 | 命令分发 |
| `RPCIDEncode` | syscap_tool.c | 145 | RPCID编码 |
| `RPCIDDecode` | syscap_tool.c | 257 | RPCID解码 |
| `CreatePCID` | create_pcid.c | 247 | PCID创建 |
| `DecodePCID` | create_pcid.c | 478 | PCID解码 |
| `GetFileContext` | context_tool.c | 40 | 读取文件 |
| `ConvertedContextSaveAsFile` | context_tool.c | 108 | 写入文件 |
| `EncodeOsSyscap` | syscap_interface.c | 81 | 编码OS syscap |
| `EncodePrivateSyscap` | syscap_interface.c | 110 | 编码Private syscap |
| `DecodeOsSyscap` | syscap_interface.c | 157 | 解码OS syscap |
| `DecodePrivateSyscap` | syscap_interface.c | 221 | 解码Private syscap |
| `ComparePcidString` | syscap_interface.c | 649 | 比较PCID字符串 |
| `QuerySystemCapability` | napi_query_syscap.cpp | 177 | N-API查询接口 |
| `GetSystemCapability` | napi_query_syscap.cpp | 110 | 获取系统能力 |
| `querySystemCapabilitie` | ohos.systemCapability.impl.cpp | 144 | ANI查询接口 |

## 相关跳转

- [架构说明](../02_Architecture.md) - 架构概览
- [N-API接口](../03_NAPI_Interface.md) - JS接口说明
- [内部API](../04_Inner_API.md) - C/C++接口说明

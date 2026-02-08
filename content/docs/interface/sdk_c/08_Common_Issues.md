# 常见问题与定位

> **目的**: 汇总构建、运行、调试的常见问题及解决方案  
> **适用范围**: 遇到问题需要排查的开发者  
> **生成时间**: 2025-02-06

---

## 1. 构建问题

### 1.1 编译失败：找不到头文件

**现象**:
```
fatal error: 'napi/native_api.h' file not found
#include "napi/native_api.h"
         ^~~~~~~~~~~~~~~~~~~
```

**原因**:
- 未正确配置 sysroot
- 头文件路径错误

**解决方案**:
```cmake
# CMakeLists.txt
set(OHOS_SYSROOT "/path/to/sdk-native/sysroot")

include_directories(
    ${OHOS_SYSROOT}/usr/include
    ${OHOS_SYSROOT}/usr/include/napi
)
```

### 1.2 编译失败：链接错误

**现象**:
```
undefined reference to `napi_create_function'
undefined reference to `OH_LOG_Print'
```

**原因**:
- 未链接对应的 NDK 库
- 库链接顺序错误

**解决方案**:
```cmake
# CMakeLists.txt
target_link_libraries(
    myapp
    ace_napi.z     # NAPI 运行时
    hilog          # 日志库
    # 其他需要的库
)
```

### 1.3 编译失败：架构不匹配

**现象**:
```
error: unable to execute command: Executable doesn't exist!
'/path/to/clang' file not found
```

**原因**:
- 工具链路径错误
- 主机架构与工具链不匹配

**解决方案**:
```bash
# 检查工具链路径
ls $OHOS_SDK/native/llvm/bin/clang

# 设置正确的环境变量
export OHOS_SDK=/path/to/ohos-sdk-native
export PATH=$OHOS_SDK/native/llvm/bin:$PATH
```

---

## 2. 运行问题

### 2.1 库加载失败

**现象**:
```
Error: dlopen failed: library "libxxx.so" not found
```

**原因**:
- 库未打包到应用
- 库路径错误

**排查步骤**:
```bash
# 1. 检查应用包内的库文件
unzip -l myapp.hap | grep \.so

# 2. 检查设备上是否存在系统库
adb shell ls /system/lib/ndk/ | grep libxxx

# 3. 检查依赖关系
readelf -d libmyapp.so | grep NEEDED
```

**解决方案**:
```json
// module.json5
{
    "module": {
        "type": "native",
        "abiFilters": ["arm64-v8a"],
        "libEntries": [
            "libmyapp.so"
        ]
    }
}
```

### 2.2 N-API 调用崩溃

**现象**:
应用调用 N-API 函数时崩溃

**原因**:
- 参数类型不匹配
- 空指针访问
- 内存越界

**排查步骤**:
```c
// 1. 添加参数校验
static napi_value MyApi(napi_env env, napi_callback_info info) {
    // 检查参数数量
    size_t argc = 1;
    napi_value args[1] = {nullptr};
    napi_get_cb_info(env, info, &argc, args, nullptr, nullptr);
    
    if (argc < 1) {
        napi_throw_error(env, nullptr, "Expected 1 argument");
        return nullptr;
    }
    
    // 检查参数类型
    napi_valuetype type;
    napi_typeof(env, args[0], &type);
    if (type != napi_number) {
        napi_throw_type_error(env, nullptr, "Expected number");
        return nullptr;
    }
    
    // ...
}
```

### 2.3 权限拒绝

**现象**:
```
Error: 201 - Permission denied
```

**原因**:
- 未申请所需权限
- 权限申请被拒绝

**解决方案**:
```json
// module.json5
{
    "module": {
        "requestPermissions": [
            {
                "name": "ohos.permission.INTERNET"
            },
            {
                "name": "ohos.permission.CAMERA"
            }
        ]
    }
}
```

---

## 3. 调试问题

### 3.1 日志不输出

**现象**:
HiLog 日志未在控制台显示

**原因**:
- 日志级别设置过高
- 日志标签过滤

**解决方案**:
```c
#include "hilog/log.h"

// 使用正确的日志级别
OH_LOG_INFO(LOG_APP, "Info message");
OH_LOG_DEBUG(LOG_APP, "Debug message");

// 检查日志标签
#undef LOG_TAG
#define LOG_TAG "MyApp"
```

```bash
# 查看日志
hilog | grep MyApp

# 设置日志级别
hilog -b D  # 显示 Debug 及以上级别
```

### 3.2 调试器连接失败

**现象**:
无法附加调试器到 Native 代码

**解决方案**:
```json
// module.json5
{
    "module": {
        "type": "native",
        "debuggable": true
    }
}
```

```bash
# 使用 lldb 调试
lldb -p $(pgrep myapp)
```

---

## 4. 性能问题

### 4.1 N-API 调用耗时高

**优化建议**:
```c
// 1. 使用异步 API
napi_create_async_work(env, resource, resource_name, 
    ExecuteWork, CompleteWork, data, &work);
napi_queue_async_work(env, work);

// 2. 减少跨语言边界调用
// 批量处理数据，而不是多次调用

// 3. 使用 TypedArray 传递大数据
// 避免大量数据拷贝
```

### 4.2 内存泄漏

**排查**:
```c
// 1. 确保释放 N-API 引用
napi_ref ref;
napi_create_reference(env, value, 1, &ref);
// 使用完毕后
napi_delete_reference(env, ref);

// 2. 释放异步工作项
napi_delete_async_work(env, work);
```

---

## 5. 问题定位路径

### 5.1 构建问题定位流程

```
编译错误
    │
    ├─> 头文件找不到？
    │       │
    │       ├─> 检查 sysroot 路径
    │       └─> 检查 include_directories
    │
    ├─> 链接错误？
    │       │
    │       ├─> 检查 target_link_libraries
    │       └─> 检查库文件是否存在
    │
    └─> 架构错误？
            │
            ├─> 检查工具链路径
            └─> 检查 abiFilters
```

### 5.2 运行问题定位流程

```
运行崩溃
    │
    ├─> 启动崩溃？
    │       │
    │       ├─> 检查库加载
    │       └─> 检查初始化代码
    │
    ├─> N-API 调用崩溃？
    │       │
    │       ├─> 检查参数校验
    │       └─> 检查空指针
    │
    └─> 权限错误？
            │
            ├─> 检查权限声明
            └─> 检查权限申请
```

---

## 6. 相关跳转

- **上一章**: [安全分析](./07_Security_Analysis.md)
- **构建指南**: `docs/howto_add.md`
- **用户指南**: `docs/user_guide.md`
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**常见问题文档 - 基于代码生成**

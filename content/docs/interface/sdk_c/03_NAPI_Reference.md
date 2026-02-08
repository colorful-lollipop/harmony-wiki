# 对外 N-API 接口文档

> **目的**: 提供完整的 N-API 接口清单、参数说明、错误码和权限要求  
> **适用范围**: N-API 开发者、JS-C 互操作开发者  
> **生成时间**: 2025-02-06

---

## 1. N-API 概述

### 1.1 什么是 N-API

N-API（Native API）是 OpenHarmony 提供的**原生模块扩展开发框架**，基于 Node.js N-API 规范开发，用于实现 ArkTS/JavaScript 与 C/C++ 代码之间的互操作。

### 1.2 N-API 核心价值

| 场景 | 说明 |
|------|------|
| **性能需求** | CPU 密集型、IO 密集型或硬件直接操作场景 |
| **代码复用** | 复用现有 C/C++ 生态库（OpenCV、TensorFlow Lite 等） |
| **系统访问** | 访问操作系统底层功能（文件、网络、硬件） |

### 1.3 N-API 架构

```
ArkTS/JS 代码
    │
    │ 调用
    ▼
┌─────────────────────────────────────┐
│  NAPI 接口层                         │
│  ├─ napi_create_function()          │
│  ├─ napi_call_function()            │
│  ├─ napi_get/set_property()         │
│  └─ ...                             │
└─────────────────────────────────────┘
    │
    │ 绑定
    ▼
┌─────────────────────────────────────┐
│  C/C++ 原生实现                      │
│  ├─ 业务逻辑                         │
│  ├─ 系统调用                         │
│  └─ 第三方库                         │
└─────────────────────────────────────┘
```

---

## 2. 基础 N-API 清单

### 2.1 模块注册

| API 名称 | 功能 | 引入版本 |
|----------|------|----------|
| `napi_module_register` | 注册 NAPI 模块 | 8 |
| `NAPI_MODULE` | 模块注册宏 | 8 |
| `NAPI_MODULE_INIT` | 简化注册宏 | 8 |

**代码示例**:
```c
// 注册模块
NAPI_MODULE(myaddon, Init)

// 初始化函数
static napi_value Init(napi_env env, napi_value exports) {
    // 导出接口
    return exports;
}
```

### 2.2 值类型操作

| API 类别 | 主要 API | 数量 |
|----------|----------|------|
| **创建值** | `napi_create_*` | 30+ |
| **获取值** | `napi_get_value_*` | 15+ |
| **类型检查** | `napi_is_*`, `napi_typeof` | 10+ |
| **类型转换** | `napi_coerce_to_*` | 5+ |

**常用 API 列表**:
```c
// 创建基本类型
napi_status napi_create_int32(napi_env env, int32_t value, napi_value* result);
napi_status napi_create_int64(napi_env env, int64_t value, napi_value* result);
napi_status napi_create_double(napi_env env, double value, napi_value* result);
napi_status napi_create_string_utf8(napi_env env, const char* str, size_t length, napi_value* result);
napi_status napi_create_string_utf16(napi_env env, const char16_t* str, size_t length, napi_value* result);
napi_status napi_create_array(napi_env env, napi_value* result);
napi_status napi_create_array_with_length(napi_env env, size_t length, napi_value* result);
napi_status napi_create_object(napi_env env, napi_value* result);
napi_status napi_create_arraybuffer(napi_env env, size_t byte_length, void** data, napi_value* result);
napi_status napi_create_typedarray(napi_env env, napi_typedarray_type type, size_t length, napi_value arraybuffer, size_t byte_offset, napi_value* result);

// 获取基本类型
napi_status napi_get_value_int32(napi_env env, napi_value value, int32_t* result);
napi_status napi_get_value_int64(napi_env env, napi_value value, int64_t* result);
napi_status napi_get_value_double(napi_env env, napi_value value, double* result);
napi_status napi_get_value_string_utf8(napi_env env, napi_value value, char* buf, size_t bufsize, size_t* result);
napi_status napi_get_array_length(napi_env env, napi_value value, uint32_t* result);

// 类型检查
napi_status napi_typeof(napi_env env, napi_value value, napi_valuetype* result);
napi_status napi_is_array(napi_env env, napi_value value, bool* result);
napi_status napi_is_arraybuffer(napi_env env, napi_value value, bool* result);
napi_status napi_is_typedarray(napi_env env, napi_value value, bool* result);
napi_status napi_is_date(napi_env env, napi_value value, bool* result);
```

### 2.3 对象操作

| API 类别 | 主要 API | 功能 |
|----------|----------|------|
| **属性操作** | `napi_set/get/has/delete_property` | 对象属性读写 |
| **元素操作** | `napi_set/get/has/delete_element` | 数组元素读写 |
| **命名属性** | `napi_set/get/has_named_property` | 命名字段访问 |
| **属性定义** | `napi_define_properties` | 批量定义属性 |

**代码示例**:
```c
// 定义属性描述符
napi_property_descriptor desc[] = {
    {"method1", nullptr, Method1, nullptr, nullptr, nullptr, napi_default, nullptr},
    {"method2", nullptr, Method2, nullptr, nullptr, nullptr, napi_default, nullptr},
};

// 批量定义属性
napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
```

### 2.4 函数操作

| API 名称 | 功能 | 说明 |
|----------|------|------|
| `napi_create_function` | 创建函数 | 绑定 C 函数到 JS |
| `napi_call_function` | 调用函数 | 调用 JS 函数 |
| `napi_new_instance` | 创建实例 | new 操作符 |
| `napi_get_cb_info` | 获取回调信息 | 获取参数和 this |
| `napi_get_new_target` | 获取 new.target | 构造函数判断 |

### 2.5 异步操作

| API 名称 | 功能 | 引入版本 |
|----------|------|----------|
| `napi_create_async_work` | 创建异步任务 | 8 |
| `napi_queue_async_work` | 投递异步任务 | 8 |
| `napi_cancel_async_work` | 取消异步任务 | 8 |
| `napi_delete_async_work` | 删除异步任务 | 8 |
| `napi_queue_async_work_with_qos` | 带 QoS 的异步任务 | 11 |

**异步模式代码示例**:
```c
// 1. 创建异步工作项
napi_create_async_work(env, resource, resource_name, 
    ExecuteWork,   // 在工作线程执行
    CompleteWork,  // 在主线程回调
    data, &work);

// 2. 投递到线程池
napi_queue_async_work(env, work);

// 3. 执行回调（工作线程）
void ExecuteWork(napi_env env, void* data) {
    // 执行耗时操作
}

// 4. 完成回调（主线程）
void CompleteWork(napi_env env, napi_status status, void* data) {
    // 返回结果到 JS
}
```

### 2.6 Promise 支持

| API 名称 | 功能 | 引入版本 |
|----------|------|----------|
| `napi_create_promise` | 创建 Promise | 10 |
| `napi_resolve_deferred` | 解决 Promise | 10 |
| `napi_reject_deferred` | 拒绝 Promise | 10 |
| `napi_is_promise` | 检查 Promise | 10 |

### 2.7 线程安全函数

| API 名称 | 功能 | 引入版本 |
|----------|------|----------|
| `napi_create_threadsafe_function` | 创建线程安全函数 | 10 |
| `napi_call_threadsafe_function` | 调用线程安全函数 | 10 |
| `napi_call_threadsafe_function_with_priority` | 带优先级的调用 | 12 |
| `napi_acquire_threadsafe_function` | 获取引用 | 10 |
| `napi_release_threadsafe_function` | 释放引用 | 10 |

### 2.8 OpenHarmony 特有扩展

| API 名称 | 功能 | 引入版本 |
|----------|------|----------|
| `napi_load_module` | 加载模块 | 11 |
| `napi_load_module_with_info` | 带信息加载模块 | 12 |
| `napi_create_ark_runtime` | 创建 Ark 运行时 | 12 |
| `napi_destroy_ark_runtime` | 销毁 Ark 运行时 | 12 |
| `napi_run_event_loop` | 运行事件循环 | 12 |
| `napi_stop_event_loop` | 停止事件循环 | 12 |
| `napi_serialize` | 序列化对象 | 12 |
| `napi_deserialize` | 反序列化对象 | 12 |
| `napi_create_sendable_array` | 创建 Sendable 数组 | 12 |
| `napi_create_sendable_object_with_properties` | 创建 Sendable 对象 | 12 |
| `napi_define_sendable_class` | 定义 Sendable 类 | 12 |
| `napi_fatal_exception` | 抛出致命异常 | 12 |

---

## 3. 领域 N-API 分类

### 3.1 ArkUI Native API

**头文件**: `arkui/ace_engine/native/native_interface.h`

| API 类别 | 主要功能 | 头文件 |
|----------|----------|--------|
| **节点操作** | 创建、修改、删除 UI 节点 | `native_node.h` |
| **动画** | 属性动画、过渡动画 | `native_animate.h` |
| **手势** | 点击、滑动、捏合等 | `native_gesture.h` |
| **事件** | 触摸、按键、输入 | `ui_input_event.h` |
| **渲染** | XComponent、自定义绘制 | `native_interface_xcomponent.h` |

### 3.2 多媒体 N-API

| 模块 | 头文件 | 主要功能 |
|------|--------|----------|
| **音频** | `multimedia/audio_framework/*.h` | 采集、渲染、管理 |
| **视频编解码** | `multimedia/av_codec/*.h` | H.264/H.265/AAC 编解码 |
| **相机** | `multimedia/camera_framework/camera.h` | 预览、拍照、录像 |
| **图像** | `multimedia/image_framework/include/image/*.h` | PixelMap、ImageSource |
| **播放** | `multimedia/player_framework/avplayer/*.h` | 媒体播放 |

### 3.3 图形 N-API

| 模块 | 头文件 | 主要功能 |
|------|--------|----------|
| **原生绘制** | `graphic/graphic_2d/native_drawing/*.h` | 2D 图形绘制 |
| **原生窗口** | `graphic/graphic_2d/native_window/*.h` | 窗口管理 |
| **OpenGL** | `graphic/graphic_2d/GLES2/GLES3/*.h` | OpenGL ES 接口 |
| **Vulkan** | `graphic/graphic_2d/vulkan/*.h` | Vulkan 图形 API |

---

## 4. 错误码体系

### 4.1 N-API 状态码

| 状态码 | 值 | 说明 |
|--------|-----|------|
| `napi_ok` | 0 | 成功 |
| `napi_invalid_arg` | 1 | 非法参数 |
| `napi_object_expected` | 2 | 期望对象 |
| `napi_string_expected` | 3 | 期望字符串 |
| `napi_name_expected` | 4 | 期望名称 |
| `napi_function_expected` | 5 | 期望函数 |
| `napi_number_expected` | 6 | 期望数字 |
| `napi_boolean_expected` | 7 | 期望布尔值 |
| `napi_array_expected` | 8 | 期望数组 |
| `napi_generic_failure` | 9 | 通用失败 |
| `napi_pending_exception` | 10 | 有待处理异常 |
| `napi_cancelled` | 11 | 已取消 |
| `napi_escape_called_twice` | 12 | escape 调用两次 |
| `napi_handle_scope_mismatch` | 13 | handle scope 不匹配 |
| `napi_callback_scope_mismatch` | 14 | callback scope 不匹配 |
| `napi_queue_full` | 15 | 队列已满 |
| `napi_closing` | 16 | 正在关闭 |
| `napi_bigint_expected` | 17 | 期望 BigInt |
| `napi_date_expected` | 18 | 期望 Date |

### 4.2 模块特定错误码

| 模块 | 错误码范围 | 说明 |
|------|-----------|------|
| **AbilityKit** | 160000xx | Ability 运行时错误 |
| **多媒体** | 540000xx | 媒体播放/录制错误 |
| **音频** | 680000xx | 音频框架错误 |
| **图像** | 290000xx | 图像处理错误 |
| **安全** | 120000xx | HUKS 错误 |

---

## 5. 代码证据

| 结论 | 证据文件 | 关键内容 |
|------|----------|----------|
| N-API 定义 | `arkui/napi/libnapi.ndk.json` | 306 个符号定义 |
| N-API 头文件 | `arkui/napi/native_api.h` | OpenHarmony 扩展 |
| Node 兼容层 | `third_party/node/src/node_api.h` | 标准 N-API |
| 错误码定义 | `third_party/node/src/js_native_api_types.h` | napi_status 枚举 |

---

## 6. 相关跳转

- **上一章**: [架构说明](./02_Architecture.md)
- **下一章**: [内部 API](./04_Internal_API.md)
- **线程模型**: [架构 - 线程模型](./02_Architecture.md#线程模型)
- **返回导航**: [SUMMARY.md](./SUMMARY.md)

---

**N-API 接口文档 - 基于代码生成**

# VK-GL-CTS Patch 分析

## 分析结论

**VK-GL-CTS 在 OpenHarmony 中没有传统意义上的 `.patch` 文件。**

这与大多数第三方库的适配方式不同。OpenHarmony 采用了**平台层完全重写**的适配策略，而非通过 Patch 修改上游代码。

## 为什么不需要传统 Patch？

### 1. 架构设计优势

VK-GL-CTS 采用高度模块化的架构设计：

```
VK-GL-CTS 架构层次：
┌─────────────────────────────────────────────┐
│  测试用例层 (Test Cases)                     │
│  - OpenGL ES 测试                            │
│  - Vulkan 测试                               │
│  - EGL 测试                                  │
├─────────────────────────────────────────────┤
│  框架层 (Framework)                          │
│  - 测试框架 tcutil                           │
│  - 工具库 delibs                             │
│  - OpenGL/Vulkan 封装                        │
├─────────────────────────────────────────────┤
│  平台抽象层 (Platform Abstraction)            │
│  - Android                                   │
│  - Linux                                     │
│  - Windows                                   │
│  - OHOS ← 完全独立实现                       │
└─────────────────────────────────────────────┘
```

平台抽象层的设计允许：
- 每个平台独立实现
- 不修改上层通用代码
- 通过继承和多态实现平台特定功能

### 2. 零侵入式适配

传统 Patch 方式：
```
上游源码 → 应用 Patch → 修改后的源码 → 构建
              ↓
         修改原有文件
         升级时需重新适配
```

OpenHarmony 方式：
```
上游源码 (不变) ────────────────────────┐
                                        ├──→ 构建
OHOS 平台层 (新增) ────────────────────┘
              ↓
         新增独立文件
         升级时通常只需验证接口
```

## 替代 Patch 的适配工作

虽然没有 `.patch` 文件，但 OpenHarmony 进行了大量适配工作：

### 1. 平台层完全重写

**目录**: `framework/platform/ohos/`

| 文件/模块 | 代码行数 | 功能 |
|-----------|----------|------|
| `tcuOhosPlatform.cpp` | ~150 | 平台主实现 |
| `display/` 目录 | ~800 | EGL Display/Window/Pixmap 适配 |
| `context/` 目录 | ~400 | OpenGL/Vulkan 上下文适配 |
| `rosen_context/` 目录 | ~600 | Rosen 框架集成 |
| **总计** | **~2000+** | **全新平台实现** |

这相当于一个**超大 Patch**，只是以独立文件形式存在。

### 2. Vulkan 扩展新增

**目录**: `build/external/vulkancts/framework/vulkan/`

OpenHarmony 添加了以下 Vulkan 扩展定义：

#### VK_OpenHarmony_OHOS_surface

```c
#define VK_OpenHarmony_OHOS_SURFACE_EXTENSION_NAME "VK_OpenHarmony_OHOS_surface"

// 新增结构体
typedef struct VkOHOSSurfaceCreateInfoOpenHarmony {
    VkStructureType                        sType;
    const void*                            pNext;
    VkOHOSSurfaceCreateFlagsOpenHarmony    flags;
    struct OH_NativeWindow*                window;
} VkOHOSSurfaceCreateInfoOpenHarmony;

// 新增函数
VkResult vkCreateOHOSSurfaceOpenHarmony(
    VkInstance instance,
    const VkOHOSSurfaceCreateInfoOpenHarmony* pCreateInfo,
    const VkAllocationCallbacks* pAllocator,
    VkSurfaceKHR* pSurface);
```

#### VK_OpenHarmony_external_memory_OHOS_native_buffer

```c
#define VK_OPENHARMONY_EXTERNAL_MEMORY_OHOS_NATIVE_BUFFER_EXTENSION_NAME \
    "VK_OpenHarmony_external_memory_OHOS_native_buffer"

// 新增外部内存句柄类型
VK_EXTERNAL_MEMORY_HANDLE_TYPE_OHOS_NATIVE_BUFFER_BIT_OPENHARMONY = 0x00004000

// 新增函数
VkResult vkGetOHOSNativeBufferPropertiesOpenHarmony(...);
VkResult vkGetMemoryOHOSNativeBufferOpenHarmony(...);
```

这些扩展的代码量是**生成代码**，由 Vulkan 代码生成工具根据 XML 定义生成。

### 3. BUILD.gn 构建适配

**文件**: 40+ 个 BUILD.gn 文件

将所有 CMake 构建逻辑转换为 GN 构建系统：

| 适配类型 | 数量 | 说明 |
|----------|------|------|
| BUILD.gn 文件 | 40+ | 替代 CMakeLists.txt |
| vk_gl_cts.gni | 1 | 公共配置定义 |
| 编译选项 | 20+ | OHOS 特有 flags |
| 预定义宏 | 15+ | 如 `DE_OS=DE_OS_UNIX` |

### 4. 与 Rosen 框架集成

**目录**: `framework/platform/ohos/rosen_context/`

这是 OpenHarmony 特有的适配工作：

```cpp
// ohos_context_i.h - OHOS 上下文接口
namespace OHOS {
class OhosContextI {
public:
    // EGL 相关
    virtual EGLBoolean OH_makeCurrent(...);
    virtual EGLBoolean OH_swapBuffers(...);
    virtual EGLContext OH_createContext(...);
    
    // Vulkan 相关
    virtual void* GetNativeWindow(...);
    virtual uint64_t CreateWindow(...);
    
    // Rosen 集成
    virtual bool InitRosenContext(...);
};
}
```

## Patch 分类分析

如果将这些适配工作按照传统 Patch 分类：

| 类型 | 工作量 | 说明 |
|------|--------|------|
| **Bugfix** | 无 | 未发现上游代码 Bug 修复 |
| **Feature** | 极大 | OHOS 平台支持、Vulkan 扩展 |
| **OH 适配** | 极大 | Rosen 集成、构建系统迁移 |
| **性能优化** | 无 | 未修改性能相关代码 |

## 回归风险评估

### 升级上游版本时的注意事项

虽然没有 Patch 文件，但升级时仍需关注：

| 风险点 | 等级 | 说明 |
|--------|------|------|
| **平台接口变更** | 中 | 上层框架可能修改平台抽象接口 |
| **Vulkan 版本升级** | 低 | OH 扩展命名空间独立，冲突概率低 |
| **测试用例新增** | 低 | 可能新增对平台功能的依赖 |
| **构建系统变更** | 低 | GN 构建逻辑独立于上游 CMake |

### 升级验证清单

升级上游版本时建议验证：

- [ ] `framework/platform/tcuPlatform.hpp` 接口是否变化
- [ ] `framework/egl/` 接口是否变化
- [ ] `external/vulkancts/framework/vulkan/` 接口是否变化
- [ ] 新增测试用例是否依赖未实现的 OHOS 功能

## 维护建议

### 1. 上游代码更新

由于没有 Patch，可以直接替换上游代码：

```bash
# 1. 备份 OHOS 平台层
cp -r framework/platform/ohos /tmp/ohos_platform_backup

# 2. 替换上游代码
# (解压新版本的 VK-GL-CTS)

# 3. 恢复 OHOS 平台层
cp -r /tmp/ohos_platform_backup framework/platform/ohos

# 4. 验证编译
./build.sh ...
```

### 2. Rosen 接口变更

由于与 Rosen 框架深度耦合，需关注：

- `foundation/graphic/graphic_2d/rosen/` 的接口变更
- `OH_NativeWindow` 相关接口变更
- `OH_NativeBuffer` 相关接口变更

### 3. 文档建议

建议在每次升级时记录：
- 上游版本变化
- 接口兼容性验证结果
- 新增测试用例覆盖情况

## 总结

VK-GL-CTS 的适配方式展示了**优秀架构设计**的价值：

1. **平台抽象层**设计使得零侵入式适配成为可能
2. **2000+ 行**平台适配代码相当于大型 Patch，但以独立文件维护
3. **Vulkan 扩展**以标准方式添加，不破坏上游代码
4. **升级友好** - 无 Patch 冲突，升级流程简单

这种适配模式值得其他需要多平台支持的第三方库参考。

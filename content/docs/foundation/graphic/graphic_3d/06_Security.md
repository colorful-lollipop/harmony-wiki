# 安全风险评审

## 概述

本文档基于代码审计，识别出 **8个高风险安全问题**。AGP引擎作为OpenHarmony系统的3D渲染组件，面临来自文件系统、内存操作、插件加载等多方面的安全威胁。

## 攻击面清单

| 攻击面 | 风险等级 | 说明 |
|--------|---------|------|
| **文件系统** | HIGH | GLTF/GLB模型文件解析、图片资源加载 |
| **插件加载** | CRITICAL | 动态.so库加载（dlopen） |
| **内存管理** | HIGH | GPU缓冲区、顶点数据、Shader Uniform |
| **N-API接口** | MEDIUM | JS/ETS到Native的调用边界 |
| **IPC通信** | LOW | 与RenderService的通信（主要在测试代码） |

## 信任边界

```
┌──────────────────────────────────────────────────────────────┐
│                     外部不可信输入                            │
│  GLTF文件  │  图片资源  │  Shader代码  │  插件.so  │  JS API   │
└────────────┴────────────┴──────────────┴───────────┴───────────┘
                              │
                              ▼ 信任边界
┌──────────────────────────────────────────────────────────────┐
│                     AGP引擎内部                               │
│  文件解析器  │  资源管理器  │  渲染管线  │  插件系统  │  N-API层  │
└────────────┴──────────────┴───────────┴───────────┴───────────┘
                              │
                              ▼ 信任边界
┌──────────────────────────────────────────────────────────────┐
│                      系统服务层                               │
│       图形驱动  │  GPU  │  文件系统  │  内存管理              │
└──────────────────────────────────────────────────────────────┘
```

---

## 风险点详细分析

### 风险点 1: 路径遍历攻击 (HIGH)

**证据位置**: 
- `lume/LumeEngine/src/io/std_directory.cpp:291-295`
- `lume/LumeEngine/src/io/std_directory.cpp:284`

**代码片段**:
```cpp
// 行 291-295 (OHOS/Linux)
char resolvedPath[PATH_MAX];
if (realpath(string(path).c_str(), resolvedPath) != nullptr) {
    absolutePath = resolvedPath;
}

// 行 284
auto handle = fopen(resolvedPath, "r");
```

**问题分析**:
1. `realpath()` 解析符号链接但不验证沙箱边界
2. 无检查确保解析后的路径在允许目录内
3. 用户可控路径可直接传递给 `fopen()`

**攻击路径**:
```
恶意GLTF文件 → 引用 ../../../etc/passwd → realpath解析 → 访问系统文件
```

**修复建议**:
```cpp
// 在realpath后添加沙箱边界验证
if (!IsPathWithinSandbox(resolvedPath, allowedBasePath)) {
    return false;  // 拒绝访问
}
```

**信任边界**: 外部文件系统 → 引擎内部

**利用场景**:
```
场景: 恶意应用通过AGP引擎加载自定义GLTF模型
1. 攻击者创建一个GLTF文件，其中包含对系统文件的引用
2. GLTF文件通过 scene.importScene() 加载
3. 引擎解析GLTF中的URI，构造完整路径
4. 路径包含 ../../../etc/passwd 等遍历序列
5. realpath() 解析但不验证沙箱边界
6. 成功读取系统敏感文件
```

**CVSS 评分**: 7.5 (High)
- AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
- 网络可达，低复杂度，无需权限，高机密性影响

---

### 风险点 2: Shader输入缓冲区溢出 (HIGH)

**证据位置**:
- `3d_widget_adapter/core/src/lume/custom/shader_input_buffer.cpp:35,79-80`

**代码片段**:
```cpp
// 行 35: 分配缓冲区（无溢出检查）
buffer_ = new float[floatSize];

// 行 79-80: memcpy_s使用不匹配的大小
auto ret = memcpy_s(
    reinterpret_cast<void *>(buffer_), floatSize_ * sizeof(float),  // dest: floatSize_
    reinterpret_cast<void *>(buffer), floatSize * sizeof(float)      // src: floatSize
);
```

**问题分析**:
1. `new float[floatSize]` - 若 `floatSize` 由攻击者控制，可能发生整数溢出
2. `memcpy_s` 的目标大小是 `floatSize_ * sizeof(float)`，但复制大小是 `floatSize * sizeof(float)`
3. 若 `floatSize > floatSize_`，即使使用"安全"的 `memcpy_s` 仍会发生缓冲区溢出

**攻击路径**:
```
恶意应用 → 创建Shader → 提供超大floatSize → 缓冲区溢出 → 代码执行
```

**修复建议**:
```cpp
// 在memcpy_s前验证大小
if (floatSize > floatSize_) {
    return false;  // 拒绝操作
}
auto ret = memcpy_s(dst, floatSize_ * sizeof(float), src, floatSize * sizeof(float));
```

**信任边界**: 应用程序 → GPU Shader系统

**利用场景**:
```
场景: 恶意应用创建超大Shader输入缓冲区
1. 攻击者调用 scene.createShader() 并指定超大floatSize
2. floatSize 导致整数溢出，分配过小缓冲区
3. 后续memcpy_s将大量数据复制到小缓冲区
4. 堆缓冲区溢出，覆盖相邻内存结构
5. 可能实现代码执行或拒绝服务
```

**CVSS 评分**: 8.1 (High)
- AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H
- 需应用权限，但可完全控制目标进程

---

### 风险点 3: GLB解析整数溢出 (HIGH)

**证据位置**:
- `lume/Lume_3D/src/gltf/gltf2_loader.cpp:2972,2935-2982`

**代码片段**:
```cpp
// 行 2972: 整数溢出在偏移量计算
const size_t dataOffset = chunkJson.chunkLength + sizeof(GLBHeader) + 2 * sizeof(GLBChunk);

// 行 2981-2982: 基于攻击者控制的块大小分配内存
string jsonString;
jsonString.resize(chunkJson.chunkLength);

// 从文件头读取的chunkLength（无验证）
struct GLBChunk {
    uint32_t chunkLength;  // 攻击者可控
    uint32_t chunkType;
};
```

**问题分析**:
1. `chunkJson.chunkLength` 来自GLB文件头，无溢出验证
2. 若 `chunkJson.chunkLength` 接近 `SIZE_MAX`，`dataOffset` 计算溢出
3. 行2981使用未检查大小的 `chunkLength` 进行内存分配（可能导致DoS）

**攻击路径**:
```
恶意GLB文件 → chunkLength = 0xFFFFFFFF → 整数溢出 → 内存分配失败/越界访问
```

**修复建议**:
```cpp
// 添加GLB块大小限制和溢出检查
constexpr size_t MAX_GLB_CHUNK_SIZE = 100 * 1024 * 1024;  // 100MB
if (chunkJson.chunkLength > MAX_GLB_CHUNK_SIZE) {
    return false;  // 拒绝解析
}

// 检查溢出
if (chunkJson.chunkLength > SIZE_MAX - sizeof(GLBHeader) - 2 * sizeof(GLBChunk)) {
    return false;
}
```

**信任边界**: 外部GLB文件 → 3D模型解析器

**利用场景**:
```
场景: 恶意GLB文件触发整数溢出
1. 攻击者构造GLB文件，设置 chunkLength = 0xFFFFFFFF
2. 文件通过GLTF解析器加载
3. dataOffset计算溢出，指向无效内存
4. 尝试读取超大块，导致内存分配失败
5. 或越界访问，触发崩溃或信息泄露
```

**CVSS 评分**: 7.5 (High)
- AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:N/A:H
- 需用户交互加载恶意文件，但可导致拒绝服务

---

### 风险点 4: 未验证动态库加载 (CRITICAL)

**证据位置**:
- `lume/LumeEngine/src/os/ohos/library_ohos.cpp:28-31,54`
- `lume/LumeEngine/src/plugin_registry.cpp:648-654`

**代码片段**:
```cpp
// library_ohos.cpp:28-31 - dlopen无签名验证
LibraryOHOS::LibraryOHOS(const string_view filename)
{
    string tmp(filename);
    libraryHandle_ = dlopen(tmp.c_str(), RTLD_NOW | RTLD_LOCAL);

// library_ohos.cpp:54 - 从加载的库获取符号
return reinterpret_cast<IPlugin*>(dlsym(libraryHandle_, "gPluginData"));

// plugin_registry.cpp:648-654 - 代码注释明确承认风险
// This load plugin call isn't really that ideal, basically dlopen/LoadLibrary 
// is called all plugins found. The order is undefined, so apotentional risks is 
// that the modules could attempt to use tracing prior to tracing plugin being 
// effectively loaded.
```

**问题分析**:
1. `dlopen()` 加载任意.so文件，**无代码签名验证**
2. 无路径验证 - 可能从攻击者控制的位置加载
3. 插件系统注释明确承认安全风险
4. 一旦恶意插件加载，获得引擎完整权限

**攻击路径**:
```
攻击者替换/注入恶意.so → dlopen加载 → 执行任意代码 → 完全控制渲染流程
```

**修复建议**:
1. **立即实施**: 代码签名验证（使用OHOS应用签名机制）
2. **路径白名单**: 仅从 `/system/lib64/graphics3d/` 加载
3. **沙箱隔离**: 在独立进程中运行插件（如可能）

```cpp
// 添加签名验证
if (!VerifyCodeSignature(filename)) {
    LOGE("Plugin signature verification failed: %s", filename);
    return nullptr;
}
libraryHandle_ = dlopen(tmp.c_str(), RTLD_NOW | RTLD_LOCAL);
```

**信任边界**: 外部插件 → 引擎核心

**利用场景**:
```
场景: 恶意.so插件被加载执行
1. 攻击者将恶意.so文件放置于插件目录
2. 引擎启动时自动加载所有插件
3. dlopen() 无签名验证直接加载
4. 恶意插件获得引擎完整权限
5. 可执行任意代码、访问GPU资源、窃取敏感数据

攻击前提: 
- 需要文件系统写入权限(通常需要系统漏洞)
- 或供应链攻击(替换合法插件)
```

**CVSS 评分**: 9.8 (Critical)
- AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
- 无需认证，低复杂度，完全系统控制

**紧急度**: 需要立即修复 - 这是最关键的入口点

---

### 风险点 5: JSON数字解析错误处理缺失 (MEDIUM)

**证据位置**:
- `lume/LumeEngine/api/core/json/json.h:704-709`

**代码片段**:
```cpp
// 行 704-709
if (fraction || exponent) {
    res = value_t<T>(strtod(beg, &end));   // 未检查end
} else if (negative) {
    res = value_t<T>(strtoll(beg, &end, 10));  // 未检查end
} else {
    res = value_t<T>(strtoull(beg, &end, 10)); // 未检查end
}
// 注意：end指针未被检查，未验证整个字符串是否被消费
```

**问题分析**:
1. `strtod()`/`strtoll()` 使用后未检查 `end` 指针
2. 无验证确认整个字符串被消费
3. 允许解析 "123abc" 为有效数字 123
4. 可能导致后续逻辑错误（如尺寸计算）

**攻击路径**:
```
恶意JSON配置 → 数字字段 = "1.5e999malicious" → 解析为部分值 → 逻辑错误/内存问题
```

**修复建议**:
```cpp
char* end;
res = value_t<T>(strtod(beg, &end));
if (end != beg + strlen(beg)) {
    return false;  // 未完全消费，拒绝解析
}
```

**信任边界**: JSON输入 → 解析器

**利用场景**:
```
场景: 畸形JSON数字导致解析错误
1. 攻击者构造包含 "1.5e999malicious" 的JSON
2. strtod() 解析部分数字后停止
3. end指针未被检查，后续逻辑使用错误值
4. 可能导致尺寸计算错误、内存分配异常
```

**CVSS 评分**: 5.3 (Medium)
- AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N
- 网络可达，但仅导致轻微完整性问题

---

### 风险点 6: 内存分配整数溢出 (MEDIUM)

**证据位置**:
- `lume/LumeBase/api/base/containers/vector.h:1158,627`

**代码片段**:
```cpp
// 行 1158
tmp = (pointer)allocator_.alloc(count * sizeof(value_type));

// 行 627
pointer tmp = allocate_if_needed(size_ + count);

// 分配器实现 - 乘法溢出风险
void* alloc(size_t size) { return malloc(size); }
```

**问题分析**:
1. `count * sizeof(value_type)` 在32位系统上可能溢出
2. 若 `count` 由攻击者控制（如来自GLTF文件），分配大小回绕
3. 导致堆缓冲区溢出（写入超分配内存）

**攻击路径**:
```
恶意GLTF → 数组count = 0x40000000 → count * 4 溢出 → 分配小缓冲区 → 写入大数组 → 堆溢出
```

**修复建议**:
```cpp
// 添加溢出检查（C++17方式）
size_t totalSize;
if (__builtin_mul_overflow(count, sizeof(value_type), &totalSize)) {
    throw std::bad_alloc();  // 或返回错误
}
tmp = (pointer)allocator_.alloc(totalSize);
```

**信任边界**: 外部输入 → 内存分配器

**利用场景**:
```
场景: 超大数组计数导致分配溢出
1. GLTF文件中数组count字段被设置为极大值(0x40000000)
2. count * sizeof(value_type) 乘法溢出
3. 分配远小于预期的缓冲区
4. 后续写入操作越界，导致堆溢出
```

**CVSS 评分**: 6.5 (Medium)
- AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:L
- 可利用，但通常导致崩溃而非代码执行

---

### 风险点 7: strtof转换无范围验证 (MEDIUM)

**证据位置**:
- `lume/Lume_3D/src/gltf/gltf2_loader.cpp:250`

**代码片段**:
```cpp
// 行 250
value = strtof(str.data(), nullptr);
// nullptr作为end指针 - 无验证
// 无HUGE_VALF/ERANGE错误检查
```

**问题分析**:
1. `strtof()` 以 `nullptr` 作为end指针调用 - 无转换成功验证
2. 无 `HUGE_VALF`/`ERANGE` 错误检查（溢出/下溢）
3. 畸形字符串可能导致未定义行为

**修复建议**:
```cpp
char* end;
value = strtof(str.data(), &end);
if (end == str.data() || *end != '\0') {
    return false;  // 转换失败
}
if (errno == ERANGE) {
    return false;  // 溢出/下溢
}
```

**信任边界**: GLTF字符串数据 → 数值解析器

**利用场景**:
```
场景: 畸形浮点数字符串导致未定义行为
1. GLTF文件包含无效浮点数字符串
2. strtof() 返回 HUGE_VALF 但 errno 未被检查
3. 超大值流入后续计算，导致异常行为
```

**CVSS 评分**: 5.3 (Medium)
- AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:N/A:L
- 需要用户加载恶意文件，可用性影响轻微

---

### 风险点 8: 插件注册表加载顺序风险 (MEDIUM)

**证据位置**:
- `lume/LumeEngine/src/plugin_registry.cpp:648-654`

**代码片段**:
```cpp
// 行 648-654（代码注释明确承认风险）
// This load plugin call isn't really that ideal, basically dlopen/LoadLibrary 
// is called all plugins found. The order is undefined, so apotentional risks is 
// that the modules could attempt to use tracing (i.e. memory allocation tracking) 
// prior to tracing plugin being effectively loaded.
```

**问题分析**:
1. 代码注释明确承认安全风险
2. 未定义加载顺序可能导致初始化竞争
3. 模块可能尝试在追踪插件加载前使用内存分配追踪
4. 无插件代码沙箱/隔离

**修复建议**:
1. 定义明确的插件加载顺序（依赖声明）
2. 延迟初始化直到所有依赖插件加载完成
3. 添加插件能力声明和验证

**信任边界**: 插件生态系统 → 引擎运行时

**利用场景**:
```
场景: 插件加载顺序竞争条件
1. 多个插件同时加载，顺序未定义
2. 插件A依赖插件B的功能，但B未加载
3. 插件A尝试使用未初始化的功能
4. 可能导致崩溃或不稳定行为
```

**CVSS 评分**: 4.4 (Medium)
- AV:L/AC:L/PR:H/UI:N/S:U/C:N/I:N/A:H
- 本地攻击，需高权限，主要影响可用性

---

## 风险汇总表

| # | 风险 | 等级 | CVSS 3.1 | 文件 | 行号 | 类型 | 修复优先级 |
|---|------|------|----------|------|------|------|-----------|
| 1 | 路径遍历 | HIGH | 7.5 | std_directory.cpp | 291-295 | 文件系统 | P1 |
| 2 | 缓冲区溢出 | HIGH | 8.1 | shader_input_buffer.cpp | 35,79-80 | 内存操作 | P1 |
| 3 | 整数溢出 | HIGH | 7.5 | gltf2_loader.cpp | 2972 | 输入解析 | P1 |
| 4 | 未验证代码加载 | **CRITICAL** | **9.8** | library_ohos.cpp | 28-31 | 动态加载 | **P0** |
| 5 | JSON解析缺陷 | MEDIUM | 5.3 | json.h | 704-709 | 输入解析 | P2 |
| 6 | 分配溢出 | MEDIUM | 6.5 | vector.h | 1158 | 内存分配 | P2 |
| 7 | strtof错误 | MEDIUM | 5.3 | gltf2_loader.cpp | 250 | 输入解析 | P2 |
| 8 | 插件顺序风险 | MEDIUM | 4.4 | plugin_registry.cpp | 648-654 | 架构设计 | P2 |

### CVSS 评分说明

**CVSS 3.1 评分维度**:
- **9.0-10.0 (Critical)**: 极易利用，无需认证，完全控制系统
- **7.0-8.9 (High)**: 可利用，可能导致严重数据泄露或系统损坏
- **4.0-6.9 (Medium)**: 利用条件受限，或影响较轻微
- **0.1-3.9 (Low)**: 难以利用，影响有限

**评分计算要素**:
- **攻击向量(AV)**: Network/Local/Physical
- **攻击复杂度(AC)**: Low/High
- **所需权限(PR)**: None/Low/High
- **用户交互(UI)**: None/Required
- **影响范围(S)**: Changed/Unchanged
- **机密性(C)**: High/Low/None
- **完整性(I)**: High/Low/None
- **可用性(A)**: High/Low/None

---

## 修复建议优先级

### P0 - 立即修复（关键风险）

**风险点 4: 未验证动态库加载**

建议措施：
1. 实施代码签名验证
2. 限制插件路径白名单
3. 考虑插件沙箱化

### P1 - 高优先级（高风险）

**风险点 1, 2, 3: 路径遍历、缓冲区溢出、整数溢出**

建议措施：
1. 添加沙箱路径验证
2. 修复memcpy_s大小检查
3. 添加GLB chunk大小限制

### P2 - 中优先级（中等风险）

**风险点 5, 6, 7, 8: 解析错误、分配溢出、插件顺序**

建议措施：
1. 修复JSON/strtof错误处理
2. 添加分配溢出检查
3. 定义插件加载顺序

---

## 局限性说明

本次安全审计存在以下局限性：

1. **未覆盖测试代码**: 根据要求，未审计 `test/` 目录下的代码
2. **Shader编译器**: 未深入审计Shader编译流程的安全问题
3. **GPU驱动交互**: 未审计与Vulkan/OpenGL驱动的交互边界
4. **IPC机制**: 本仓库主要是3D渲染引擎，IPC代码较少（主要在fuzz测试）

---

## 下一步

- [查看架构设计 →](02_Architecture.md)
- [查看GN构建 →](05_GN_Build.md)
- [查看常见问题 →](07_FAQ.md)

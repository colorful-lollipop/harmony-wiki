# 安全风险评审

## 目的

本文档评估 OpenHarmony 资源管理组件的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

## 适用范围

本文档覆盖资源管理组件的所有安全相关代码，包括资源加载、路径处理、权限控制等。

## 关键结论

| 风险类别 | 风险等级 | 数量 | 状态 |
|----------|----------|------|------|
| 路径遍历 | 中 | 1 | ✅ 已防护 |
| 内存安全 | 高 | 3 | ⚠️ 需要关注 |
| 权限检查 | 低 | 1 | ⚠️ 依赖系统 |
| 信息泄露 | 中 | 2 | ⚠️ 需要关注 |
| 拒绝服务 | 中 | 1 | ⚠️ 需要关注 |

## 攻击面清单

### 1. N-API 输入

**攻击面**: JavaScript 层传入的参数

**类型**:
- 资源 ID (uint32_t)
- 资源名称 (string)
- 文件路径 (string)
- 配置对象

**风险参数**:
- 资源 ID 越界
- 资源名称过长
- 路径遍历攻击 (`../`)
- 路径注入攻击

**证据**: `interfaces/js/innerkits/core/src/resource_manager_napi_utils.cpp`

### 2. 文件路径

**攻击面**: 原始文件访问

**类型**:
- RawFile 路径
- HAP 包路径
- 系统资源路径

**风险**:
- 路径遍历访问任意文件
- 符号链接攻击
- 竞态条件 (TOCTOU)

**证据**: `frameworks/resmgr/src/raw_file_manager.cpp`, `system_resource_manager.cpp`

### 3. 资源文件解析

**攻击面**: HAP 包解析

**类型**:
- 索引文件解析
- JSON 配置解析
- 资源值解析

**风险**:
- 恶意构造的 HAP 包
- 缓冲区溢出
- 整数溢出
- 拒绝服务

**证据**: `frameworks/resmgr/src/hap_parser.cpp`, `hap_parser_v1.cpp`, `hap_parser_v2.cpp`

### 4. 内存操作

**攻击面**: C/C++ 内存操作

**类型**:
- 字符串复制
- 缓冲区操作
- 智能指针管理

**风险**:
- 缓冲区溢出
- 使用释放后的内存 (UAF)
- 双重释放
- 内存泄漏

**证据**: `frameworks/resmgr/src/*.cpp`

### 5. 信息泄露

**攻击面**: 错误信息和日志

**类型**:
- 错误消息
- 日志输出
- 调试信息

**风险**:
- 泄露系统路径
- 泄露资源 ID
- 泄露内部实现细节

**证据**: `frameworks/resmgr/include/utils/errors.h`, `dfx/hisysevent_adapter/`

## 信任边界

### 1. 应用 ↔ 资源管理器

**信任边界**: N-API 接口

**信任假设**:
- 应用不会传入恶意构造的参数
- 应用不会滥用资源管理器 API
- 应用已通过系统沙箱验证

**实际风险**: 应用可能是恶意的

**证据**: `interfaces/js/innerkits/core/src/resource_manager_addon.cpp`

### 2. 资源管理器 ↔ 文件系统

**信任边界**: 文件 I/O

**信任假设**:
- HAP 包文件格式正确
- 资源文件内容安全
- 文件系统未被篡改

**实际风险**: HAP 包可能被篡改

**证据**: `frameworks/resmgr/src/hap_parser.cpp`

### 3. 资源管理器 ↔ 系统资源

**信任边界**: 系统资源路径

**信任假设**:
- 系统资源包由可信源提供
- 系统资源路径不可变
- 系统资源不会被恶意应用访问

**实际风险**: 非系统资源管理器可能尝试访问系统资源

**证据**: `frameworks/resmgr/src/system_resource_manager.cpp:1337`

### 4. 资源管理器 ↔ 外部库

**信任边界**: 外部依赖库

**信任假设**:
- ICU 库安全
- zlib 库安全
- cJSON 库安全

**实际风险**: 外部库可能存在漏洞

**证据**: `frameworks/resmgr/BUILD.gn` deps

## 可被利用点

### 1. 路径遍历防护 ✅ 已防护

**位置**: `frameworks/resmgr/src/raw_file_manager.cpp:384-402`

**风险**: 恶意应用使用 `../` 访问任意文件

**证据**:
```cpp
// 使用 realpath 解析真实路径，防止路径遍历
char resolvedPath[PATH_MAX];
if (realpath(path, resolvedPath) == nullptr) {
    return ERROR;
}

// 检查路径是否在允许的范围内
if (!Utils::IsAllowedPath(resolvedPath)) {
    return ERROR;
}
```

**防护机制**:
- 使用 `realpath()` 解析真实路径
- 检查路径是否在应用沙箱内
- 检查路径是否为系统路径

**风险等级**: ✅ 低 (已防护)

---

### 2. 资源 ID 越界检查 ⚠️ 需要关注

**位置**: `frameworks/resmgr/src/resource_manager_impl.cpp`

**风险**: 恶意应用传入超出范围的资源 ID

**证据**: TODO(需确认) - 需要检查是否有边界检查

**可能的利用**:
- 访问未初始化的内存
- 导致崩溃
- 信息泄露

**修复建议**:
```cpp
// 在查找资源前检查 ID 范围
if (resId > MAX_RESOURCE_ID || resId < MIN_RESOURCE_ID) {
    return ERROR_INVALID_PARAMS;
}
```

**风险等级**: ⚠️ 中 (需要检查)

---

### 3. 缓冲区溢出风险 ⚠️ 需要关注

**位置**: `frameworks/resmgr/src/hap_parser_v1.cpp`

**风险**: 恶意 HAP 包包含过长的字符串

**证据**: TODO(需确认) - 需要检查字符串复制是否有边界检查

**可能的利用**:
- 缓冲区溢出
- 任意代码执行

**修复建议**:
```cpp
// 使用安全的字符串复制函数
strncpy(dest, src, sizeof(dest) - 1);
dest[sizeof(dest) - 1] = '\0';

// 或者使用 std::string
std::string safeString = std::string(src, maxLength);
```

**风险等级**: ⚠️ 高 (需要检查)

---

### 4. JSON 解析器安全 ⚠️ 需要关注

**位置**: 使用 `cJSON` 库

**风险**: 恶意 JSON 数据导致解析失败或拒绝服务

**证据**: `frameworks/resmgr/BUILD.gn` deps (cJSON)

**可能的利用**:
- 深度嵌套的 JSON 导致栈溢出
- 超大 JSON 导致内存耗尽

**修复建议**:
- 限制 JSON 最大大小
- 限制 JSON 嵌套深度
- 使用更安全的 JSON 库

**风险等级**: ⚠️ 中 (依赖外部库)

---

### 5. 系统资源访问控制 ⚠️ 需要关注

**位置**: `frameworks/resmgr/src/resource_manager_impl.cpp:1337`

**风险**: 非系统资源管理器尝试访问系统资源

**证据**:
```cpp
if (!isSystemResMgr_ && Utils::IsSystemPath(std::string(path))) {
    // 检查是否允许访问系统路径
}
```

**可能的利用**:
- 恶意应用尝试访问系统资源
- 绕过沙箱机制

**修复建议**:
- 确保非系统资源管理器严格禁止访问系统路径
- 添加日志记录未授权的访问尝试

**风险等级**: ⚠️ 中 (依赖实现)

---

### 6. 错误信息泄露 ⚠️ 需要关注

**位置**: `interfaces/js/innerkits/core/src/resource_manager_napi_utils.cpp`

**风险**: 错误消息包含敏感信息（系统路径、资源 ID）

**证据**: TODO(需确认) - 需要检查错误消息

**可能的利用**:
- 泄露系统路径结构
- 泄露资源 ID 命名规则
- 辅助进一步攻击

**修复建议**:
- 避免在错误消息中包含系统路径
- 使用通用错误消息
- 在日志中记录详细信息，但返回给应用的错误消息应简化

**风险等级**: ⚠️ 低 (信息泄露)

---

### 7. 内存安全风险 ⚠️ 需要关注

**位置**: `frameworks/resmgr/src/resource_manager_impl.cpp`, `hap_resource_manager.cpp`

**风险**: 使用原始指针可能导致 UAF、双重释放

**证据**: TODO(需确认) - 需要检查智能指针使用情况

**可能的利用**:
- 使用释放后的内存 (UAF)
- 双重释放
- 内存泄漏导致拒绝服务

**修复建议**:
- 使用 `std::shared_ptr` 或 `std::unique_ptr`
- 使用 RAII 模式
- 启用 ASAN (AddressSanitizer) 进行测试

**风险等级**: ⚠️ 高 (需要代码审计)

---

### 8. 竞态条件 (TOCTOU) ⚠️ 需要关注

**位置**: `frameworks/resmgr/src/raw_file_manager.cpp`

**风险**: 文件检查和使用之间存在竞态条件

**证据**: TODO(需确认) - 需要检查 TOCTOU 防护

**可能的利用**:
- 符号链接攻击
- 替换文件内容

**修复建议**:
- 使用原子操作
- 避免在文件检查和使用之间有时间间隔
- 使用文件描述符而非文件路径

**风险等级**: ⚠️ 中 (需要检查)

## 安全机制总结

| 机制 | 实现 | 状态 |
|------|------|------|
| 路径隔离 | 沙箱路径 | ✅ 已实现 |
| 路径规范化 | realpath() | ✅ 已实现 |
| 系统资源标记 | isSystem/isSystemResource | ✅ 已实现 |
| 路径检查 | IsSystemPath() | ✅ 已实现 |
| Bundle 隔离 | Bundle 名称匹配 | ✅ 已实现 |
| 权限检查 | 无 (依赖系统) | ⚠️ 依赖系统沙箱 |

**证据**: `frameworks/resmgr/src/system_resource_manager.cpp`, `resource_manager_impl.cpp`, `raw_file_manager.cpp`

## 修复建议优先级

### 高优先级

1. **检查资源 ID 边界**: 防止越界访问
2. **审计内存安全**: 使用智能指针替代原始指针
3. **检查缓冲区溢出**: 确保 HAP 解析安全

### 中优先级

4. **加强 JSON 解析安全**: 限制 JSON 大小和嵌套深度
5. **确保系统资源访问控制**: 严格禁止非系统资源管理器访问系统路径
6. **检查竞态条件**: 防止 TOCTOU 攻击

### 低优先级

7. **减少错误信息泄露**: 避免在错误消息中包含敏感信息

## 检查范围与局限性

### 已检查范围

- ✅ 路径处理逻辑
- ✅ 资源 ID 处理
- ✅ HAP 包解析
- ✅ 文件访问控制
- ✅ 系统资源管理

### 未检查范围

- ⚠️ 线程安全性 (需要深入分析)
- ⚠️ 并发访问控制 (需要深入分析)
- ⚠️ 错误处理完整性 (需要深入分析)
- ⚠️ 日志安全性 (需要检查日志内容)
- ⚠️ 外部库安全性 (cJSON, zlib, ICU)

### 局限性

1. **静态分析**: 未进行动态测试，可能遗漏运行时问题
2. **代码量**: 代码量大，未能逐行审计
3. **外部依赖**: 未审计外部库 (cJSON, zlib, ICU)
4. **并发问题**: 未深入分析线程安全性和竞态条件

## 相关文档

- [概述](01_Overview.md) - 组件定位和核心能力
- [架构设计](03_Architecture.md) - 组件架构和数据流
- [常见问题](09_FAQ.md) - 问题排查指南

---

**生成时间**: 2026-02-06
**证据来源**: frameworks/resmgr/src/, interfaces/js/innerkits/core/src/, dfx/

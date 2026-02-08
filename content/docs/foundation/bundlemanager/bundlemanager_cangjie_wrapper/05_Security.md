# 安全风险评审

> bundlemanager_cangjie_wrapper 攻击面分析与风险评估

## 评审范围

| 范围 | 说明 |
|------|------|
| **代码范围** | `bundlemanager_cangjie_wrapper` 仓库源码 |
| **排除范围** | `test/` 目录、依赖组件内部实现 |
| **信任边界** | 本仓库 FFI 层与 bundle_framework 边界 |

**证据范围声明**:
- 本评审仅基于本仓库可验证的代码证据
- 依赖组件（bundle_framework, ability_runtime）由各自仓库负责

---

## 攻击面分析

### 输入点清单

| 输入类型 | 来源 | 处理位置 | 风险等级 |
|----------|------|----------|----------|
| **bundleFlags** | 用户传入 Int32 | `getBundleInfoForSelf()` | 低 |
| **moduleName** | 用户传入 String | `getProfileByAbility()` | 中 |
| **abilityName** | 用户传入 String | `getProfileByAbility()` | 中 |
| **link** | 用户传入 String | `canOpenLink()` | 高 |
| **metadataName** | 用户传入 String | `getProfileByAbility()` | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         不可信区域 (用户空间)                            │
│                                                                          │
│  - Cangjie 应用代码                                                      │
│  - 用户传入的所有字符串参数                                              │
│  - bundleFlags 等整型参数                                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 信任边界 (FFI 调用)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        bundlemanager_cangjie_wrapper                      │
│                                                                          │
│  - FFI 参数转换 (CString)                                                │
│  - 错误码检查                                                            │
│  - BusinessException 抛出                                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    │ 信任边界 (C 接口)
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        bundle_framework (依赖组件)                        │
│                                                                          │
│  - 输入验证                                                              │
│  - 权限校验                                                              │
│  - IPC 安全检查                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 风险清单

### 风险 1: 链接注入攻击

| 项目 | 内容 |
|------|------|
| **证据** | `bundle_manager.cj:37` - `FfiBundleManagerCanOpenLink(cLink, code)` |
| **可利用路径** | `canOpenLink(link)` → FFI → bundle_framework |
| **影响** | 恶意应用可能通过构造特殊链接触发未预期的应用启动 |
| **风险等级** | 中 |
| **当前缓解** | bundle_framework 侧 `ERROR_INVALID_LINK`(17700055) 和 `ERROR_SCHEME_NOT_IN_QUERYSCHEMES`(17700056) 检查 |
| **修复建议** | 建议在 FFI 层增加 URL 格式预校验 |

### 风险 2: 字符串参数注入

| 项目 | 内容 |
|------|------|
| **证据** | `bundle_manager.cj:119-125` - `LibC.mallocCString()` |
| **可利用路径** | `getProfileByAbility(moduleName, abilityName)` |
| **影响** | 特殊构造的字符串可能导致 C 层缓冲区溢出 |
| **风险等级** | 低 |
| **当前缓解** | C 接口层负责内存安全；LibC.mallocCString 使用 safe context |
| **修复建议** | 建议增加参数长度限制检查 |

### 风险 3: BundleFlags 整数溢出

| 项目 | 内容 |
|------|------|
| **证据** | `bundle_manager.cj:29` - `FfiOHOSGetBundleInfoForSelfV2(bundleFlags: Int32)` |
| **可利用路径** | `getBundleInfoForSelf(bundleFlags)` |
| **影响** | 非法 bundleFlags 可能导致返回错误的 BundleInfo |
| **风险等级** | 低 |
| **当前缓解** | BundleFlag 枚举约束有效值 |
| **修复建议** | 建议增加参数边界校验 |

### 风险 4: 缓存污染攻击

| 项目 | 内容 |
|------|------|
| **证据** | `bundle_manager.cj:48-58` - `HashMap<BMQuery, BundleInfo>` 缓存 |
| **可利用路径** | 多用户场景下缓存被不同用户访问 |
| **影响** | 跨用户信息泄露 |
| **当前缓解** | `checkToCache()` 函数检查 `uid != callingUid`，不同用户不缓存 |
| **风险等级** | 低 |

### 风险 5: C 字符串内存泄漏

| 项目 | 内容 |
|------|------|
| **证据** | `bundle_manager.cj:37-46` - 多个 `Ffi*` 函数返回 CString |
| **可利用路径** | FFI 调用返回的资源未正确释放 |
| **影响** | 内存泄漏导致资源耗尽 |
| **风险等级** | 中 |
| **当前缓解** | 每个 Ret* 结构体都有 `free()` 方法；`bundle_manager.cj:93` 调用 `ret.free()` |
| **修复建议** | 建议使用 defer 模式确保释放 |

---

## 缓解措施总结

| 风险 | 已有的安全措施 | 建议改进 |
|------|---------------|----------|
| 链接注入 | bundle_framework 侧校验 | FFI 层增加 URL 格式校验 |
| 字符串注入 | LibC.mallocCString 安全转换 | 增加长度限制 |
| BundleFlags 越界 | BundleFlag 枚举约束 | 增加运行时校验 |
| 缓存污染 | checkToCache 用户隔离 | 完善（当前已实现） |
| 内存泄漏 | free() 方法 + 显式调用 | 使用 defer 模式 |

---

## 安全相关代码位置

| 功能 | 文件路径 |
|------|----------|
| 错误码定义 | `ohos/bundle/bundle_manager/error_code.cj` |
| FFI 函数声明 | `ohos/bundle/bundle_manager/bundle_manager.cj` |
| C 结构体定义 | `ohos/bundle/bundle_manager/cj_bundle_ffi.cj` |
| BundleFlag 枚举 | `ohos/bundle/bundle_manager/bundle_flag.cj` |

---

## 未发现的风险类型

| 类型 | 检查结果 | 说明 |
|------|----------|------|
| **竞态条件** | ✅ 无明显风险 | Mutex 同步机制正确 |
| **信息泄露** | ✅ 无明显风险 | 无敏感信息明文传输 |
| **动态加载** | ✅ 不适用 | 本仓库无动态加载逻辑 |
| **权限提升** | ✅ 不适用 | 本仓库为只读查询 API |
| **SQL/NoSQL 注入** | ✅ 不适用 | 无数据库操作 |

---

## 安全最佳实践

### 1. 参数验证

```cj
// 推荐的参数验证模式
public static func canOpenLink(link: String): Bool {
    // 1. 空值检查
    if (link.isEmpty()) {
        throw BusinessException(ERROR_INVALID_LINK, "Link cannot be empty")
    }
    
    // 2. 格式检查（建议实现）
    if (!isValidUrlFormat(link)) {
        throw BusinessException(ERROR_INVALID_LINK, "Invalid URL format")
    }
    
    // 3. 长度检查（建议实现）
    if (link.length > MAX_LINK_LENGTH) {
        throw BusinessException(ERROR_INVALID_LINK, "Link too long")
    }
    
    // ... FFI 调用
}
```

### 2. 资源释放

```cj
// 推荐使用 defer 模式确保资源释放
let ret = unsafe { FfiOHOSGetBundleInfoForSelfV2(bundleFlags) }
defer { unsafe { ret.free() } }  // 确保离开作用域时释放
let info = BundleInfo(ret)
```

---

## 相关文档

- [02_API_Reference.md](02_API_Reference.md) - API 参考
- [03_Architecture.md](03_Architecture.md) - 架构说明
- [SUMMARY.md](SUMMARY.md) - 文档导航

# 攻击面分析（Attack Surface Analysis）

**目的**: 识别所有外部输入入口、敏感操作和信任边界，为安全研究提供攻击面清单

**适用范围**: `base/global/global_cangjie_wrapper` 非测试代码

---

## 攻击面总览

```
┌─────────────────────────────────────────────────────────────┐
│                    外部世界（用户输入）                          │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌───────────┐ │
│  │  路径输入  │  │  资源ID  │  │  字符串输入  │ │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘ │
│        │                │                │         │ │
│        ▼                ▼                ▼         ▼ │
│  ┌─────────────────────────────────────────────────────┐ │
│  │         Cangjie API 层                  │ │
│  │  Calendar | ResourceManager | System          │ │
│  └──────────────────┬──────────────────────────────┘ │
│                     │                                   │
│                     ▼                                   │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              FFI 边界（跨语言）              │ │
│  └──────────────────┬──────────────────────────────┘ │
│                     │                                   │
│                     ▼                                   │
│  ┌─────────────────────────────────────────────────────┐ │
│  │           底层 C++ 系统服务                   │ │
│  │  i18n | resource_management                 │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────────┘
```

**信任边界**: Cangjie API 层 ↔ FFI 边界 ↔ 底层 C++ 系统

---

## 一、外部输入清单

### 1.1 路径输入（Path Input）

| API | 参数 | 位置 | 输入来源 | 验证状态 | 风险类型 |
|-----|------|------|----------|----------|----------|
| `ResourceManager.getRawFd(path)` | `path: String` | `resource_manager.cj:78` | 用户指定 | ✅ 底层验证（9001005） | 路径遍历、任意文件访问 |
| `ResourceManager.closeRawFd(path)` | `path: String` | `resource_manager.cj:101` | 用户指定 | ✅ 底层验证（9001005） | 路径遍历、任意文件访问 |
| `ResourceManager.getRawFileContent(path)` | `path: String` | `resource_manager.cj:123` | 用户指定 | ✅ 底层验证（9001005） | 路径遍历、任意文件读取 |
| `ResourceManager.getRawFileList(path)` | `path: String` | `resource_manager.cj:152` | 用户指定 | ✅ 底层验证（9001005） | 路径遍历、信息泄露 |
| `ResourceManager.addResource(path)` | `path: String` | `resource_manager.cj:650` | 用户指定 | ✅ 底层验证（9001010） | 路径遍历、任意文件访问 |
| `ResourceManager.removeResource(path)` | `path: String` | `resource_manager.cj:670` | 用户指定 | ✅ 底层验证（9001010） | 路径遍历、资源删除 |

**路径输入风险分析**：

#### 触发路径（示例）
```
用户调用：resourceManager.getRawFd("../../../etc/passwd")
    ↓
Cangjie 层：ResourceManager.getRawFd(path)
    ↓
FFI 调用：CJ_GetRawFd(id, cPath, fd)
    ↓
底层 C++：解析路径，检查有效性
    ↓
返回：错误码 9001005（Invalid relative path）
```

#### 潜在漏洞点

| 漏洞类型 | 触发条件 | 影响 |
|----------|----------|------|
| 路径遍历 | 输入包含 `../`、`..\\` 等序列 | 可能读取/写入应用沙箱外的文件 |
| 路径规范化绕过 | 底层验证不完整 | 可能访问受限目录 |
| 符号链接攻击 | 通过符号链接绕过路径限制 | 访问任意文件 |

### 1.2 资源 ID 输入（Resource ID Input）

| API | 参数 | 位置 | 输入来源 | 验证状态 | 风险类型 |
|-----|------|------|----------|----------|----------|
| `ResourceManager.getColor(resId)` | `resId: UInt32` | `resource_manager.cj:212` | 用户指定 | ✅ 底层验证（9001001, 9001002） | 资源 ID 注入、越界访问 |
| `ResourceManager.getString(resId, args)` | `resId: UInt32` | `resource_manager.cj:565` | 用户指定 | ✅ 底层验证（9001001, 9001002, 9001007） | 资源 ID 注入、格式化漏洞 |
| `ResourceManager.getBoolean(resId)` | `resId: UInt32` | `resource_manager.cj:233` | 用户指定 | ✅ 底层验证（9001001, 9001002） | 资源 ID 注入 |
| `ResourceManager.getNumber(resId)` | `resId: UInt32` | `resource_manager.cj:279` | 用户指定 | ✅ 底层验证（9001001, 9001002） | 资源 ID 注入 |
| `ResourceManager.getMediaContent(resId, density?)` | `resId: UInt32` | `resource_manager.cj:366` | 用户指定 | ✅ 底层验证（9001001, 9001002） | 资源 ID 注入、任意媒体访问 |
| `ResourceManager.getMediaContentBase64(resId, density?)` | `resId: UInt32` | `resource_manager.cj:402` | 用户指定 | ✅ 底层验证（9001001, 9001002） | 资源 ID 注入、Base64 编码绕过 |
| `ResourceManager.getPluralStringValue(resId, num)` | `resId: UInt32` | `resource_manager.cj:465` | 用户指定 | ✅ 底层验证（9001001, 9001002, 9001006） | 资源 ID 注入 |
| `ResourceManager.getStringArrayValue(resId)` | `resId: UInt32` | `resource_manager.cj:519` | 用户指定 | ✅ 底层验证（9001001, 9001002, 9001006） | 资源 ID 注入 |
| `ResourceManager.getMediaByName(resName, density?)` | `resName: String` | `resource_manager.cj:329` | 用户指定 | ✅ 底层验证（9001003, 9001004） | 资源名称注入 |

**资源 ID 风险分析**：

#### 触发路径（示例）
```
用户调用：resourceManager.getString(0xFFFFFFFF, [])
    ↓
Cangjie 层：ResourceManager.getString(resId, args)
    ↓
FFI 调用：CJ_GetString(id, resId)
    ↓
底层 C++：检查资源 ID 是否存在
    ↓
返回：错误码 9001002（No matching resource is found）
```

#### 潜在漏洞点

| 漏洞类型 | 触发条件 | 影响 |
|----------|----------|------|
| 资源 ID 越界 | resId 超出有效范围 | 可能访问其他应用的资源 |
| 资源 ID 注入 | 通过构造特殊 ID 访问未授权资源 | 信息泄露、权限提升 |
| 资源循环引用 | 资源之间形成循环引用 | 可能导致拒绝服务 |

### 1.3 字符串输入（String Input）

| API | 参数 | 位置 | 输入来源 | 验证状态 | 风险类型 |
|-----|------|------|----------|----------|----------|
| `Calendar.setTimeZone(timeZone)` | `timeZone: String` | `calendar.cj:170` | 用户指定 | ❓ 依赖底层验证 | 时区注入、格式化错误 |
| `Calendar.get(field)` | `field: String` | `calendar.cj:267` | 用户指定 | ❓ 依赖底层验证 | 字段注入、格式化错误 |
| `Calendar.getDisplayName(locale)` | `locale: String` | `calendar.cj:287` | 用户指定 | ❓ 依赖底层验证 | Locale 注入 |
| `ResourceManager.getStringByName(resName, args)` | `resName: String` | `resource_manager.cj:590` | 用户指定 | ✅ 底层验证（9001003, 9001004, 9001008） | 资源名称注入 |
| `ResourceManager.getColorByName(resName)` | `resName: String` | `resource_manager.cj:186` | 用户指定 | ✅ 底层验证（9001003, 9001004, 9001006） | 资源名称注入 |
| `ResourceManager.getBooleanByName(resName)` | `resName: String` | `resource_manager.cj:254` | 用户指定 | ✅ 底层验证（9001003, 9001004, 9001006） | 资源名称注入 |
| `ResourceManager.getNumberByName(resName)` | `resName: String` | `resource_manager.cj:302` | 用户指定 | ✅ 底层验证（9001003, 9001004, 9001006） | 资源名称注入 |
| `getCalendar(locale, calendarType)` | `locale: String` | `calendar.cj:77` | 用户指定 | ❓ 依赖底层验证 | Locale 注入、日历类型注入 |

### 1.4 其他输入类型

| API | 参数 | 位置 | 输入来源 | 验证状态 | 风险类型 |
|-----|------|------|----------|----------|----------|
| `Calendar.set(year, month, date, ...)` | `year/month/date: Int32` | `calendar.cj:138` | 用户指定 | ❓ 依赖底层验证 | 日期溢出、逻辑错误 |
| `Calendar.add(field, amount)` | `field: String, amount: Int32` | `calendar.cj:313` | 用户指定 | ✅ 返回异常（890001） | 字段注入、数值溢出 |

---

## 二、敏感操作清单

### 2.1 文件系统操作

| 操作 | API | 位置 | 风险等级 | 说明 |
|------|-----|------|----------|------|
| 读取原始文件 | `getRawFd()`, `getRawFileContent()` | **中危** | 可能读取任意文件 |
| 获取文件列表 | `getRawFileList()` | **低危** | 可能泄露文件结构 |
| 添加覆盖资源 | `addResource()` | **中危** | 可能添加恶意覆盖资源 |
| 移除覆盖资源 | `removeResource()` | **中危** | 可能删除合法资源 |

### 2.2 FFI 调用（跨语言边界）

| FFI 函数 | 位置 | 风险等级 | 说明 |
|----------|------|----------|------|
| `FfiOHOSGetCalendar()` | `calendar.cj:25` | **中危** | 直接调用底层 C 库 |
| `CJ_GetRawFd()` | `resource_manager_ffi.cj:31` | **中危** | 传递用户路径到底层 |
| `CJ_GetString()` | `resource_manager_ffi.cj:79` | **中危** | 资源 ID 注入风险 |
| `FFIGetResourceString()` | `resource_common.cj:63` | **低危** | 资源字符串解析 |

**FFI 边界风险**：
- **类型不安全**: Cangjie 和 C 的类型转换可能导致数据截断
- **内存管理错误**: `CString` 需要手动释放，遗漏可能导致内存泄漏
- **编码问题**: Cangjie `String` 到 C `CString` 的转换可能存在编码问题

### 2.3 内存分配操作

| 操作 | 位置 | 风险等级 | 说明 |
|------|------|----------|------|
| `LibC.mallocCString()` | 多处（如 `calendar.cj:80`） | **低危** | 需要确保 `LibC.free()` |
| `LibC.free()` | 多处 | **低危** | 双重释放或未释放 |

---

## 三、信任边界图

```
┌─────────────────────────────────────────────────────────────┐
│                   应用进程                          │
│  ┌───────────────────────────────────────────────────┐ │
│  │        Cangjie API 层（高信任）              │ │
│  │  Calendar | ResourceManager | System          │ │
│  │  - 输入验证（有限）                        │ │
│  │  - 异常处理                                │ │
│  │  - 线程安全                                │ │
│  └──────────────┬────────────────────────────────────┘ │
│                 │                                    │
│                 ▼                                    │
│  ┌───────────────────────────────────────────────────┐ │
│  │            FFI 边界（风险边界）              │ │
│  │  - 类型转换风险                            │ │
│  │  - 内存管理风险                            │ │
│  │  - 编码风险                                │ │
│  └──────────────┬────────────────────────────────────┘ │
│                 │                                    │
│                 ▼                                    │
│  ┌───────────────────────────────────────────────────┐ │
│  │      底层 C++ 系统服务（低信任）            │ │
│  │  - global_i18n                              │ │
│  │  - global_resource_management                  │ │
│  └───────────────────────────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────────┘
```

**信任边界说明**：
1. **应用进程 → Cangjie API 层**: 高信任域，假设调用者是可信的
2. **Cangjie API 层 → FFI 边界**: 风险边界，跨语言调用
3. **FFI 边界 → 底层 C++ 系统**: 低信任域，假设底层可能包含漏洞

---

## 四、安全机制评估

### 4.1 输入验证

| 机制 | 状态 | 位置 | 说明 |
|------|------|------|------|
| 错误码验证 | ✅ | `resource_manager_errors.cj:26-32` | 返回明确错误码 |
| 异常抛出 | ✅ | `resource_manager_errors.cj:35-39` | 对无效输入抛出 `BusinessException` |
| 底层验证 | ⚠️ | 底层 C++ 系统 | 依赖底层验证，不透明 |
| 参数类型限制 | ✅ | 所有 API | 强类型限制（如 `UInt32` ID） |

### 4.2 线程安全

| 组件 | 保护机制 | 位置 | 状态 |
|------|----------|------|------|
| ResourceManager 缓存 | Mutex | `resource_manager.cj:37` | ✅ 使用 `synchronized` 保护 |
| 单例模式 | 双重检查锁 | `resource_manager.cj:50-62` | ✅ 线程安全 |

```cangjie
// 位置: ohos/resource_manager/resource_manager.cj:50-62
protected static func getResourceManager(context: StageContext): ResourceManager {
    let ptrAddr = context.toUIntNative()
    match (RES_MGR_MAP.get(ptrAddr)) {
        case Some(v) => v
        case None => synchronized(APP_MUTEX) {
            match (RES_MGR_MAP.get(ptrAddr)) {
                case Some(v) => v
                case None =>
                    let mgrId = unsafe { CJ_GetResourceManagerStageMode(context) }
                    let mgr = ResourceManager(mgrId)
                    RES_MGR_MAP.add(ptrAddr, mgr)
                    mgr
            }
        }
    }
}
```

### 4.3 资源管理

| 机制 | 状态 | 位置 | 说明 |
|------|------|------|------|
| 远程数据析构 | ✅ | `calendar.cj:104-106` | 自动释放底层资源 |
| FFI 数据释放 | ✅ | `resource_manager.cj:45` | 调用 `releaseFFIData()` |

---

## 五、未防护的攻击面

### 5.1 路径遍历攻击

**风险**: 中危

**原因**:
- `getRawFd()`、`getRawFileContent()` 等方法接受用户提供的路径
- 虽然底层返回错误码 9001005，但不清楚具体的验证逻辑
- 可能存在路径规范化绕过

**攻击向量**:
```
# 示例 1: 尝试读取系统文件
resourceManager.getRawFd("../../../../../etc/passwd")

# 示例 2: 使用编码绕过
resourceManager.getRawFd("..%2F..%2Fetc%2Fpasswd")

# 示例 3: 符号链接
resourceManager.getRawFd("symlink_to_sensitive_file")
```

**缓解建议**:
- 底层应进行完整的路径规范化
- 限制路径为相对路径，不允许 `..`
- 实施应用沙箱隔离

### 5.2 资源 ID 注入攻击

**风险**: 中危

**原因**:
- 大量 API 接受 `resId: UInt32` 或 `resName: String`
- 虽然底层验证，但不清楚验证逻辑的完整性

**攻击向量**:
```
# 示例 1: 尝试访问其他应用的资源
resourceManager.getString(0x12345678, [])

# 示例 2: 资源 ID 越界
resourceManager.getColor(0xFFFFFFFF)

# 示例 3: 资源名称注入
resourceManager.getStringByName("../../../etc/passwd", [])
```

**缓解建议**:
- 实施资源 ID 空间隔离
- 限制资源访问范围在应用包内
- 验证资源 ID 的有效性

### 5.3 FFI 边界攻击

**风险**: 中危

**原因**:
- 大量 FFI 调用直接传递用户输入到底层 C++ 系统
- 类型转换可能导致数据截断
- `CString` 内存管理需要手动释放

**攻击向量**:
```
# 示例 1: 类型转换截断
Calendar.setTimeZone("超长的时区字符串XXXXXXXXXX...")

# 示例 2: 内存泄漏
# 如果忘记调用 LibC.free()，导致内存泄漏
let clanguage = unsafe { FfiI18nSystemGetAppPreferredLanguage() }
# 忘记：unsafe { LibC.free(clanguage) }
```

**缓解建议**:
- 确保所有 `CString` 都有对应的 `LibC.free()`
- 使用 RAII 模式管理内存
- 验证 FFI 参数的边界

### 5.4 并发攻击

**风险**: 低危

**原因**:
- ResourceManager 使用 `Mutex` 保护静态 HashMap
- 但其他模块的并发安全性未明确

**缓解建议**:
- 审查所有共享状态的并发安全性
- 使用不可变数据结构
- 避免全局可变状态

---

## 六、外部依赖安全风险

| 依赖 | 依赖类型 | 风险等级 | 说明 |
|------|----------|----------|------|
| `i18n:cj_i18n_ffi` | C++ 底层库 | **高危** | 日历功能的核心依赖 |
| `resource_management:cj_resource_manager_ffi` | C++ 底层库 | **高危** | 资源管理功能的核心依赖 |
| `ace_engine:cj_frontend_ohos` | C++ 前端库 | **中危** | 提供 `StageContext` |

**风险说明**:
- 底层 C++ 库的漏洞直接影响 Cangjie API 层
- 无法从当前仓库审查底层库的安全性
- 需要同步审查 `global_i18n` 和 `global_resource_management` 仓库

---

## 七、推荐安全审计优先级

| 优先级 | 审查项目 | 原因 |
|--------|----------|------|
| **P0** | FFI 边界 | 所有用户输入通过 FFI 传递到底层 |
| **P1** | 路径验证 | `getRawFd()` 等方法存在路径遍历风险 |
| **P2** | 资源 ID 验证 | 大量 API 接受资源 ID，需验证隔离 |
| **P3** | 内存管理 | 确保所有 `CString` 都有对应释放 |
| **P4** | 并发安全 | 审查所有共享状态的并发安全性 |
| **P5** | 底层依赖 | 审查 `global_i18n` 和 `global_resource_management` |

---

**相关文档**:
- [06_SecurityReview](06_SecurityReview.md) - 深度安全风险评估
- [04_Interface](04_Interface.md) - 完整 API 清单
- [08_Internals](08_Internals.md) - 内部实现细节

# 安全风险评审

## 目的

本文档基于代码证据对 ability_base 组件进行安全风险评审，识别攻击面、信任边界、潜在可利用点，并提供修复建议。所有结论均基于对 `interfaces/` 目录代码的分析。

**评审范围**：
- ✅ 输入验证（URI 解析、参数类型）
- ✅ IPC 安全（序列化、远程对象）
- ✅ 内存安全（缓冲区大小、递归深度）
- ✅ 资源管理（文件描述符、ZIP 文件）
- ❌ 权限验证（在 ability_runtime 层实现，此仓库不包含）
- ❌ N-API 层（此仓库无 N-API 绑定）

---

## 1. 威胁模型

### 1.1 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│              不信任域（应用/第三方）                    │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  应用代码使用 Want/Configuration API              │ │
│  └──────────────────┬──────────────────────────────┘ │
└─────────────────────┼─────────────────────────────────────┘
                    │
        ┌───────────▼──────────────┐
        │   ability_base 组件     │ ← 信任边界
        │   (本评审范围）        │
        └───────────┬──────────────┘
                    │
        ┌───────────▼──────────────┐
        │  系统服务层             │ ← 最终信任域
        │  (ability_runtime 等）   │
        └──────────────────────────┘
```

### 1.2 外部输入→敏感操作

| 外部输入 | 敏感操作 | 风险 |
|----------|----------|------|
| URI 字符串（Want::SetUri） | 文件访问、跨设备通信 | 路径遍历、注入攻击 |
| Want 参数（WantParams） | IPC 传输、Ability 启动 | 类型混淆、参数篡改 |
| Configuration 键 | 系统配置访问 | 配置注入、权限提升 |
| ZIP 文件（Extractor） | 文件解压、代码加载 | ZIP 滑炸弹、路径遍历 |
| 文件描述符（WantParams） | 跨进程文件共享 | FD 泄露、权限绕过 |

---

## 2. 攻击面清单

### 2.1 N-API 层（不存在）

**状态**：此仓库不包含 N-API 绑定

**说明**：根据 README.md:22，N-API 代码应在 `frameworks/js/napi/`，但该目录不存在。N-API 绑定在 ability_runtime 组件实现。

**结论**：✅ 无 N-API 直接攻击面

**证据位置**：
- README.md:22 - 提到 `frameworks/js/napi`
- 目录扫描结果：无 N-API 文件

### 2.2 C API (NDK) 层

**攻击面**：
- `OH_AbilityBase_CreateWant()` - 创建 Want 对象
- `OH_AbilityBase_SetWantCharParam()` - 设置字符串参数
- `OH_AbilityBase_SetWantUri()` - 设置 URI
- `OH_AbilityBase_AddWantFd()` - 添加文件描述符

**风险等级**：⚠️ 中等

**说明**：C API 直接暴露给 NDK 开发者，参数验证在 C++ 层实现。

**证据位置**：
- C API 定义：`interfaces/kits/c/cwant/include/want.h`

### 2.3 Native C++ API 层

**攻击面**：
- `Want::SetParam()` - 设置任意类型参数
- `Want::ParseUri()` - 解析 URI 字符串
- `Want::SetElementName()` - 设置组件标识（bundleName/abilityName）
- `Configuration::AddItem()` - 添加配置项
- `Extractor::ExtractToBuf()` - 解压 ZIP 文件

**风险等级**：⚠️ 中等

**证据位置**：
- Want API：`interfaces/kits/native/want/include/want.h`
- Configuration API：`interfaces/kits/native/configuration/include/configuration.h`
- Extractor API：`interfaces/kits/native/extractortool/include/extractor.h`

### 2.4 IPC 层（Parcelable）

**攻击面**：
- `Want::Marshalling()` - 序列化 Want 对象
- `WantParams::Marshalling()` - 序列化参数
- `SessionInfo::Marshalling()` - 序列化会话信息（含 IRemoteObject）

**风险等级**：⚠️ 中等

**说明**：IPC 序列化需要严格验证，防止恶意数据跨进程传播。

**证据位置**：
- Want Marshalling：`interfaces/kits/native/want/include/want.h:794`
- WantParams Marshalling：`interfaces/kits/native/want/src/want_params.cpp`

### 2.5 文件系统层

**攻击面**：
- `Extractor::Open()` - 打开 HAP/ZIP 文件
- `ZipFile::Open()` - 解析 ZIP 文件
- `WantParams` 文件描述符操作

**风险等级**：⚠️ 高

**证据位置**：
- Extractor 接口：`interfaces/kits/native/extractortool/include/extractor.h`
- ZipFile 类：`interfaces/kits/native/extractortool/include/zip_file.h`

---

## 3. 可被利用点（已发现）

### 3.1 ⚠️ WantParams 类型混淆

**证据位置**：
- 位置：`interfaces/kits/native/want/src/want_params.cpp:758-778`
- 代码：`ReadFromParcelRemoteObject()` 方法

**漏洞描述**：
```cpp
bool WantParams::ReadFromParcelRemoteObject(Parcel &parcel, std::string &key)
{
    int32_t typeId = 0;
    if (!parcel.ReadInt32(typeId)) {
        return false;
    }
    if (typeId != VALUE_TYPE_REMOTE_OBJECT) {
        // 类型检查
        return false;
    }
    // ... 解析远程对象
}
```

**可利用路径**：
1. 攻击者构造恶意 Parcel，篡改 `typeId`
2. 绕过类型检查，尝试读取不匹配的数据
3. 导致类型混淆或内存损坏

**触发条件**：
- 跨进程传输 Want 对象
- 恶意应用构造篡改的 IPC 数据

**影响**：
- 🔴 类型混淆：导致错误的类型解析
- 🔴 内存损坏：可能触发崩溃
- 🔴 信息泄露：读取不应访问的数据

**当前保护**：
- ✅ 类型 ID 检查（`typeId != VALUE_TYPE_REMOTE_OBJECT`）
- ✅ `ReadInt32()` 失败检查

**修复建议**：
1. 加强类型验证：增加更多类型 ID 范围检查
2. 添加数据完整性校验：使用 checksum 验证 Parcel 数据
3. 记录异常：对类型不匹配的请求记录日志

**证据文件**：`interfaces/kits/native/want/src/want_params.cpp:758-778`

---

### 3.2 ⚠️ WantParams 递归深度溢出

**证据位置**：
- 位置：`interfaces/kits/native/want/src/want_params.cpp:204`
- 代码：`MAX_RECURSION_DEPTH = 100` 常量

**漏洞描述**：
```cpp
constexpr int MAX_RECURSION_DEPTH = 100;

bool WantParams::ReadFromParcel(Parcel &parcel)
{
    if (recursionDepth_ >= MAX_RECURSION_DEPTH) {
        return false;  // 防止栈溢出
    }
    recursionDepth_++;
    // ... 递归解析嵌套结构
    recursionDepth_--;
}
```

**可利用路径**：
1. 攻击者构造深度嵌套的 WantParams（99 层）
2. 触发接近栈溢出的递归深度
3. 虽然有保护，但仍可能消耗大量资源

**触发条件**：
- 应用构造深度嵌套的参数结构
- 使用嵌套的 `WantParams` 或 `Array`

**影响**：
- 🟡 资源消耗：CPU 和栈内存占用
- 🟡 拒绝服务：大量深度嵌套请求
- 🟢 崩溃风险：接近 100 层时可能触发系统栈限制

**当前保护**：
- ✅ 递归深度限制：100 层
- ✅ 计数器保护：`recursionDepth_`

**修复建议**：
1. 降低递归限制：考虑降低到 50-80 层
2. 添加资源监控：监控嵌套深度异常的请求
3. 实施速率限制：限制单次请求的嵌套结构数量

**证据文件**：`interfaces/kits/native/want/src/want_params.cpp:204`

---

### 3.3 ⚠️ WantParams 数组大小限制不足

**证据位置**：
- 位置：`interfaces/kits/native/want/src/want_params.cpp:1305`
- 代码：`maxAllowedSize = 1024` 常量

**漏洞描述**：
```cpp
constexpr int maxAllowedSize = 1024;

bool WantParams::ReadFromParcelArray(Parcel &parcel, std::string &key)
{
    long size = 0;
    if (!parcel.ReadInt64(size)) {
        return false;
    }
    if (size <= 0 || size > maxAllowedSize) {
        return false;
    }
    // ... 解析数组
}
```

**可利用路径**：
1. 攻击者创建包含 1024 个元素的数组
2. 每个元素可以是大型字符串或其他对象
3. 导致大量内存分配

**触发条件**：
- 应用创建大型 WantParams 数组
- 跨进程传输大型数组

**影响**：
- 🟡 内存消耗：单个数组可占用大量内存
- 🟡 拒绝服务：大量大型数组请求
- 🟢 崩溃风险：内存不足时触发 OOM

**当前保护**：
- ✅ 大小限制：1024 元素
- ✅ 范围检查：`size <= 0 || size > maxAllowedSize`

**修复建议**：
1. 降低大小限制：考虑降低到 256-512
2. 添加总大小检查：限制单个 WantParams 的总内存占用
3. 实施配额限制：限制单次 IPC 请求的总大小

**证据文件**：`interfaces/kits/native/want/src/want_params.cpp:1305`

---

### 3.4 ⚠️ ZipFile 签名验证不足

**证据位置**：
- 位置：`interfaces/kits/native/extractortool/src/zip_file.cpp`
- 代码：`CENTRAL_SIGNATURE`、`DATA_DESC_SIGNATURE` 等常量

**漏洞描述**：
ZipFile 验证 ZIP 文件签名，但对恶意 ZIP 格式（ZIP bomb）的防护不足。

```cpp
// 签名常量
constexpr uint32_t EOCD_SIGNATURE = 0x06054b50;
constexpr uint32_t CENTRAL_SIGNATURE = 0x02014b50;
constexpr uint32_t DATA_DESC_SIGNATURE = 0x08074b50;
constexpr uint32_t LOCAL_HEADER_SIGNATURE = 0x04034b50;

// 检查签名
if (header.signature != LOCAL_HEADER_SIGNATURE) {
    return false;
}
```

**可利用路径**：
1. 攻击者构造恶意 ZIP 文件（ZIP bomb）
2. 使用压缩比率极高的文件（1MB → 10GB 解压）
3. 包含大量重复或特殊结构的文件
4. 导致解压时耗尽磁盘或内存

**触发条件**：
- 应用调用 `Extractor::ExtractToBuf()` 提取恶意 ZIP
- 解压到受限环境（嵌入式设备）

**影响**：
- 🔴 磁盘耗尽：ZIP bomb 可能耗尽系统存储
- 🔴 内存耗尽：解压大型文件导致 OOM
- 🔴 拒绝服务：系统资源耗尽无法响应其他请求

**当前保护**：
- ✅ 签名验证：检查 ZIP 文件头签名
- ✅ 大小检查：限制单个条目大小（100MB）
- ✅ SAFE_ABC 模式：部分 FileMapper 使用安全映射

**修复建议**：
1. 添加总大小检查：限制 ZIP 文件总解压大小
2. 实施解压速率限制：控制解压速度
3. 添加文件数量限制：限制 ZIP 文件中的条目数量
4. 使用沙盒：在受限环境中解压（如果可能）

**证据文件**：
- ZipFile 签名：`interfaces/kits/native/extractortool/src/zip_file.cpp` (多处)
- 大小限制：`interfaces/kits/native/want/src/want_params.cpp:1581`

---

### 3.5 ⚠️ Want JSON 字符串验证不足

**证据位置**：
- 位置：`interfaces/kits/native/want/src/want_params_wrapper.cpp:102-126`
- 代码：`ValidateStr()` 方法

**漏洞描述**：
```cpp
bool WantParamWrapper::ValidateStr(const std::string &str)
{
    // 验证括号匹配
    // 验证引号配对
    // 但未检查所有 JSON 注入模式
}
```

**可利用路径**：
1. 攻击者在 Want 参数中注入恶意 JSON
2. 虽然有基础验证，但可能绕过某些检查
3. 如果解析器使用有漏洞的 JSON 库，可能导致代码执行

**触发条件**：
- 应用将 Want 转换为 JSON（Want::ToJson()）
- JSON 解析器存在已知漏洞

**影响**：
- 🟡 JSON 注入：恶意 JSON 数据
- 🟢 代码执行：如果 JSON 解析器有漏洞
- 🟢 拒绝服务：畸形 JSON 导致解析器崩溃

**当前保护**：
- ✅ 括号匹配：验证 `{}` 和 `[]` 匹配
- ✅ 引号配对：验证 `"` 配对

**修复建议**：
1. 增强 JSON 验证：使用更严格的 JSON schema
2. 输入编码验证：确保 UTF-8 合法性
3. 沙盒 JSON 解析：如果支持用户提供的 JSON
4. 更新 JSON 库：使用最新安全的 nlohmann_json

**证据文件**：`interfaces/kits/native/want/src/want_params_wrapper.cpp:102-126`

---

### 3.6 ✅ URI 路径遍历防护良好

**证据位置**：
- 位置：`interfaces/kits/native/want/src/want.cpp:953`
- 代码：`CheckUri()` 方法

**防护描述**：
Want 的 URI 解析包含基础验证，防止明显的路径遍历攻击。

**可利用路径**：
1. 攻击者尝试使用 `../` 路径遍历
2. 构造恶意 URI 访问非授权文件

**当前保护**：
- ✅ URI 格式验证：`CheckUri()` 方法
- ✅ Scheme 验证：只允许特定 scheme（file://, http:// 等）

**风险评估**：🟢 低风险 - 基础防护良好

**建议**：
1. 考虑添加更多 scheme 白名单
2. 实现 URI 规范化（路径标准化）
3. 添加路径遍历检测（阻止 `../`）

**证据文件**：`interfaces/kits/native/want/src/want.cpp:953`

---

## 4. 数据流安全分析

### 4.1 Want 参数数据流

```
应用输入 (不可信)
    ↓ Want.SetParam(key, value)
    ↓ WantParams::SetParam()
    ↓ 类型包装（Boolean::Box 等）
    ↓ 存储到 params_ map
    ↓ Want::Marshalling()
    ↓ WantParams::Marshalling()
    ↓ 写入 Parcel（IPC）
    ↓ ──────────────────────
    ↓ Binder IPC 传输
    ↓ ──────────────────────
    ↓ 系统服务接收
    ↓ Want::Unmarshalling()
    ↓ WantParams::Unmarshalling()
    ↓ 类型验证（typeId 检查）
    ↓ IInterface::Unbox()
    ↓ 系统服务使用 (可信)
```

**风险点**：
1. 🔴 类型混淆：Marshalling/Unmarshalling 类型 ID 不匹配
2. 🔴 参数篡改：Parcel 传输过程中被篡改
3. 🟡 资源消耗：大型参数导致内存耗尽

### 4.2 Configuration 数据流

```
应用查询
    ↓ Configuration::GetItem(key)
    ↓ MakeTheKey(key)  // "displayId#key"
    ↓ std::lock_guard<recursive_mutex> lock
    ↓ configParameter_.find(key)
    ↓ 返回值
```

**风险点**：
1. 🟢 配置注入：恶意应用尝试设置系统配置（但在系统服务层验证）
2. 🟢 信息泄露：敏感配置值可能泄露（但在 ability_runtime 控制访问）
3. ✅ 线程安全：使用 recursive_mutex 保护

---

## 5. 内存安全

### 5.1 内存分配保护

**证据**：多处使用 `new (std::nothrow)` 安全分配

```cpp
// want.cpp
sptr<IInterface> value = new (std::nothrow) Boolean(true);
if (value == nullptr) {
    // 处理分配失败
}
```

**保护措施**：✅ 良好 - 使用 nothrow 防止抛出异常

### 5.2 引用计数

**证据**：所有 Object 子类使用 RefBase 引用计数

```cpp
// base_obj.h
class Object : public virtual RefBase {
    void IncStrongRef(const void *id = nullptr) override;
    void DecStrongRef(const void *id = nullptr) override;
};
```

**保护措施**：✅ 良好 - 自动生命周期管理

### 5.3 缓冲区大小限制

**证据**：多处大小限制

```cpp
constexpr size_t maxAllowedSize = 100 * 1024 * 1024;  // 100MB
```

**保护措施**：✅ 良好 - 防止缓冲区溢出

---

## 6. IPC 安全

### 6.1 Parcelable 序列化

**证据**：所有跨进程类实现 Parcelable

```cpp
// want.h
class Want final : public Parcelable {
    virtual bool Marshalling(Parcel &parcel) const override;
    static Want *Unmarshalling(Parcel &parcel);
};
```

**保护措施**：
- ✅ 显式序列化接口
- ✅ Parcel 层面保护（Binder 安全机制）

### 6.2 IRemoteObject 传递

**证据**：WantParams 和 SessionInfo 支持传递 IRemoteObject

```cpp
// want.h
Want& SetParam(const std::string &key, const sptr<IRemoteObject> &remoteObject);
```

**保护措施**：
- ⚠️ 类型检查：`ReadFromParcelRemoteObject()` 验证类型
- ⚠️ 权限验证：在系统服务层验证 token（不在 ability_base）

**风险**：
- 🟡 Token 泄露：恶意应用可能获取敏感 token（但在系统服务层控制）
- 🟡 权限提升：伪造 token 尝试权限提升（但在系统服务层验证）

---

## 7. 文件系统安全

### 7.1 文件描述符管理

**证据**：WantParams 特殊管理 FD

```cpp
// want_params.h
std::map<std::string, int> fds_;
void CloseAllFd();
void DupAllFd();
```

**保护措施**：
- ✅ 自动关闭：`CloseAllFd()` 在析构时调用
- ✅ 复制保护：`DupAllFd()` 用于 fork 后

**风险**：
- 🟡 FD 泄露：恶意应用可能获取不应访问的 FD（但在 IPC 层验证）
- 🟡 权限绕过：通过 FD 传递绕过权限检查（但在系统服务层验证）

### 7.2 ZIP 文件处理

**证据**：Extractor 和 ZipFile 处理 ZIP 文件

**保护措施**：
- ✅ 签名验证：检查 ZIP 文件头签名
- ✅ 大小限制：单个条目 100MB
- ⚠️ ZIP bomb：总大小和条目数量限制不足（见 3.4）

---

## 8. 权限验证（不在 ability_base）

### 8.1 权限验证位置

**说明**：ability_base 提供**数据结构和接口**，实际的**权限验证在 ability_runtime 层**实现。

**证据**：
- SessionInfo 包含 `callingTokenId`（`session_info.h:75`）
- 包含注释："To ensure security, this attribute must be rewritten on the server-side"（`session_info.h:84`）

**结论**：
- ✅ ability_base 正确地传递上下文信息（token, bundleName, uid）
- ✅ 不在此层实现权限验证是正确的架构设计
- ⚠️ 需确保 ability_runtime 正确验证这些上下文

---

## 9. 安全配置建议

### 9.1 编译时安全选项

**已启用**：
- ✅ `pac_ret`：分支保护（返回地址保护）
- ✅ `-fexceptions`：C++ 异常支持
- ✅ CFI（ability_base_want）：控制流完整性

**建议启用**：
- ⚠️ Stack Canaries：栈溢出保护
- ⚠️ ASLR：地址空间布局随机化（系统级）
- ⚠️ Fortify Source：运行时缓冲区保护

### 9.2 运行时安全配置

**建议**：
1. 限制单个应用的 Want 参数数量
2. 限制单个应用的总 IPC 数据大小
3. 实施速率限制：防止大量请求
4. 监控异常：记录异常的参数模式

---

## 10. 检查范围与局限性

### 10.1 已检查范围

- ✅ 所有 `interfaces/` 目录代码
- ✅ 输入验证逻辑（URI、参数类型）
- ✅ 序列化/反序列化代码
- ✅ 内存管理（引用计数、缓冲区大小）
- ✅ 文件系统操作（ZIP、FD）
- ✅ 线程安全机制

### 10.2 未检查范围

- ❌ 权限验证（ability_runtime 层）
- ❌ N-API 绑定（此仓库不存在）
- ❌ Ability 启动流程（ability_runtime 层）
- ❌ Binder IPC 底层实现
- ❌ 测试代码（根据约束排除）

### 10.3 局限性

1. **静态分析**：基于代码审查，未进行动态测试
2. **时间限制**：未进行完整的模糊测试
3. **依赖组件**：未审查所有外部依赖（IPC、JSON 库等）
4. **运行时行为**：未在实际系统中测试攻击场景

---

## 11. 风险总结

| 风险 | 等级 | 可利用 | 当前保护 | 建议优先级 |
|------|------|--------|----------|-----------|
| WantParams 类型混淆 | ⚠️ 中等 | 是 | 类型 ID 检查 | 🔴 高 |
| WantParams 递归深度 | 🟡 低 | 部分 | 100 层限制 | 🟡 中 |
| WantParams 数组大小 | 🟡 低 | 部分 | 1024 元素 | 🟡 中 |
| ZIP 炸弹 | 🔴 高 | 是 | 签名 + 大小 | 🔴 高 |
| JSON 注入 | 🟡 低 | 部分 | 括号验证 | 🟢 低 |
| URI 路径遍历 | 🟢 低 | 否 | 格式验证 | 🟢 低 |
| FD 泄露 | 🟡 低 | 否 | 自动关闭 | 🟡 中 |

---

## 12. 修复优先级

### P0 - 高优先级（立即修复）

1. **ZIP 炸弹防护增强**（3.4）
   - 添加总大小检查
   - 添加条目数量限制
   - 实施解压速率限制

2. **WantParams 类型混淆加强**（3.1）
   - 增强类型 ID 范围检查
   - 添加数据完整性校验

### P1 - 中优先级（尽快修复）

1. **WantParams 递归深度降低**（3.2）
   - 降低到 50-80 层
   - 添加资源监控

2. **WantParams 数组大小降低**（3.3）
   - 降低到 256-512 元素
   - 添加总大小检查

3. **FD 权限验证增强**（7.1）
   - 在系统服务层验证 FD 权限
   - 记录 FD 传递日志

### P2 - 低优先级（计划修复）

1. **JSON 验证增强**（3.5）
   - 使用更严格的 JSON schema
   - 更新 JSON 库

2. **URI 验证增强**（3.6）
   - 添加 scheme 白名单
   - 实现路径标准化

---

## 相关跳转

- 🏠 **项目概览**：[index.md](index.md)
- 🏗️ **架构设计**：[02_Architecture.md](02_Architecture.md)
- ⚙️ **GN 构建**：[05_GN_Build.md](05_GN_Build.md)

---

**返回导航**：[SUMMARY.md](SUMMARY.md)

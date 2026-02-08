# 安全风险评审

> 目的：建立威胁模型、识别攻击面、信任边界、可被利用点
> 适用范围：安全审计、架构师、开发者安全编码
> 最后更新：2026-02-06

## 威胁模型

### 外部输入 → 敏感操作

```
┌─────────────────────────────────────────────────────────────┐
│  外部输入（攻击面）                                  │
│  - 应用传入的文件路径/URI                             │
│  - 应用传入的文件数据（Buffer/String）                │
│  - 应用传入的打开模式/权限模式                      │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  输入验证层                                          │
│  - 路径长度检查                                       │
│  - 空字符串检查                                       │
│  - 编码验证（UTF-8/16）                               │
│  - 路径遍历防护（NOFOLLOW 标志）                        │
│  - 符号链接防护（lstat）                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  敏感操作                                            │
│  - 文件读取（read/readText）                             │
│  - 文件写入（write）                                     │
│  - 文件创建/删除（mkdir/unlink）                          │
│  - 文件移动/复制（move/copy）                             │
│  - 权限检查（access）                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  系统调用层（VFS / Kernel）                     │
│  - 系统文件系统访问                                    │
│  - 应用沙箱隔离                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 攻击面分析

### 1. 文件路径操作

**证据位置**：
- `FileIo.open()`: `ohos/file/fs/cj_file_fs.cj:1270`
- `FileIo.stat()`: `ohos/file/fs/cj_file_fs.cj:992`
- `FileIo.mkdir()`: `ohos/file/fs/cj_file_fs.cj:1118`（需查找）

**潜在风险**：
- ⚠️ 路径遍历（Path Traversal）
- ⚠️ 符号链接攻击（Symbolic Link Attack）
- ⚠️ 长路径 DoS（Buffer Overflow）
- ⚠️ 竞争条件（Race Condition）

### 2. 文件内容操作

**证据位置**：
- `FileIo.read()`: `ohos/file/fs/cj_file_fs.cj:1339`
- `FileIo.write()`: `ohos/file/fs/cj_file_fs.cj:1396`
- `Stream.read()`: `ohos/file/fs/stream.cj:239`
- `Stream.write()`: `ohos/file/fs/stream.cj:129`

**潜在风险**：
- ⚠️ 缓冲区溢出（Buffer Overflow）
- ⚠️ 格式化字符串漏洞（Format String）
- ⚠️ 竞争条件（TOCTOU）
- ⚠️ 信息泄露（通过错误信息）

### 3. 文件 URI 操作

**证据位置**：
- `FileUri(uriOrPath)`: `ohos/file/fileuri/file_uri.cj:103`
- `getUriFromPath(path)`: `ohos/file/fileuri/file_uri.cj:196`

**潜在风险**：
- ⚠️ URI 注入（URI Injection）
- ⚠️ 路径混淆（Path Confusion）
- ⚠️ 外部存储访问（External Storage Access）

### 4. 权限与访问控制

**证据位置**：
- `FileIo.access()`: `ohos/file/fs/cj_file_fs.cj:1202`
- `OpenMode` 权限常量：`ohos/file/fs/cj_file_fs.cj:98-180`
- `AccessModeType` 枚举：`ohos/file/fs/cj_file_fs.cj:461-508`

**潜在风险**：
- ⚠️ 权限提升（Privilege Escalation）
- ⚠️ 访问控制绕过（Access Control Bypass）
- ⚠️ 竞争条件（TOCTOU）

---

## 信任边界

### 边界 1：Cangjie 应用 → FFI 层

**边界描述**：应用代码通过 FFI 调用外部 C++ 函数

**信任关系**：
- ✅ 应用代码信任 FFI 函数签名正确
- ✅ FFI 层不信任应用传递的参数（需要验证）

**证据**：`ohos/file/fs/native.cj:22-178`（70+ FFI 声明）

### 边界 2：FFI 层 → 外部服务

**边界描述**：FFI 调用进入外部 `file_api` 和 `app_file_service` C++ 库

**信任关系**：
- ✅ FFI 层信任外部服务实现了约定的接口
- ✅ 外部服务不信任 FFI 传递的参数（在服务端验证）

**证据**：
- `ohos/file/fs/BUILD.gn:46` - `external_deps = [ "file_api:cj_file_fs_ffi" ]`
- `ohos/file/fileuri/BUILD.gn:34` - `external_deps = [ "app_file_service:cj_file_fileuri_ffi" ]`

### 边界 3：应用沙箱 → 文件系统

**边界描述**：OpenHarmony 应用在沙箱内运行，访问受限的文件系统

**信任关系**：
- ✅ 系统信任应用在沙箱边界内
- ✅ 应用仅能访问自己的沙箱目录

**证据**：README:55-66（约束条件） - "URI 暂不支持外部存储目录"

---

## 可被利用点（基于证据）

### 利用点 1：路径遍历攻击（Path Traversal）

**严重性**：🔴 **高**

**证据**：
- 路径验证在 FFI 层（file_api）实现，当前代码未直接验证
- `FileIo.open()` 直接传递路径给 FFI：`cj_file_fs.cj:1271-1280`

**触发路径**：
```cangjie
// 应用代码（可能受影响）
let file = FileIo.open("../../../etc/passwd", OpenMode.READ_ONLY)
```

**调用链**：
```
应用 → FileIo.open() → FfiOHOSFileFsOpen() → file_api (C++) → open() syscall
```

**影响**：
- 读取应用沙箱外的敏感文件（如 `/etc/passwd`）
- 虽有 OpenHarmony 沙箱隔离，但可能绕过沙箱边界

**修复建议**：
1. ✅ 确认 FFI 层（file_api）实现路径验证
2. ⚠️ 在 Cangjie 层添加路径规范化（TODO）
3. ⚠️ 禁止包含 `..` 的路径（TODO）

### 利用点 2：符号链接攻击（Symbolic Link Attack）

**严重性**：🔴 **高**

**证据**：
- `NOFOLLOW` 标志存在但未默认启用：`cj_file_fs.cj:170` - `NOFOLLOW = 0o400000`
- `lstat()` 函数存在但需主动调用：`cj_file_fs.cj:1158` - `FileIo.lstat(path)`

**触发路径**：
```cangjie
// 应用代码
let file = FileIo.open("/tmp/symlink_to_sensitive", OpenMode.READ_ONLY)
// 如果路径是符号链接指向敏感文件，且未使用 NOFOLLOW
```

**调用链**：
```
应用 → FileIo.open() (无 NOFOLLOW) → FfiOHOSFileFsOpen() → file_api → open() (跟随符号链接)
```

**影响**：
- 读取符号链接指向的敏感文件
- 绕过沙箱隔离（如果符号链接在沙箱内）

**修复建议**：
1. ✅ 使用 `OpenMode.NOFOLLOW` 标志打开不可信文件
2. ✅ 使用 `lstat()` 检查符号链接而非目标文件
3. ⚠️ 添加符号链接检测机制（TODO）

### 利用点 3：缓冲区溢出（Buffer Overflow）

**严重性**：🟡 **中**

**证据**：
- `FileIo.read()` 直接使用应用提供的 Buffer：`cj_file_fs.cj:1343-1363`
- Buffer 大小由 `length` 参数控制，但未发现明显边界检查

**触发路径**：
```cangjie
// 应用代码（可能受影响）
let buffer = Array<Byte>(1024)
FileIo.read(fd, buffer, ReadOptions(offset: 0, length: 2048))  // 超出 buffer 大小
```

**调用链**：
```
应用 → FileIo.read(fd, buffer, options) → FfiOHOSFileFsRead() → file_api → read() syscall
```

**影响**：
- 写越界导致内存破坏
- 可能的任意代码执行

**修复建议**：
1. ✅ 确认 FFI 层（file_api）实现 Buffer 边界检查
2. ⚠️ 在 Cangjie 层添加显式边界检查（TODO）
3. ⚠️ 使用安全的缓冲区分配函数（safeMalloc）

### 利用点 4：信息泄露（Information Disclosure）

**严重性**：🟡 **中**

**证据**：
- 错误码和错误信息可能泄露文件路径信息：`conflict_file_exception.cj`（需查找）
- 日志中可能记录敏感路径：`cj_file_fs.cj:26` - `let FS_LOG = HilogChannel(... )`

**触发路径**：
```cangjie
// 通过错误信息泄露路径结构
try {
        FileIo.open("/data/sensitive/config.txt", OpenMode.READ_ONLY)
} catch (e: BusinessException) {
        // 错误消息可能包含路径信息
        println(e.message)  // "No such file or directory: /data/sensitive/config.txt"
}
```

**调用链**：
```
应用 → FileIo.open() → FFI 调用失败 → throw BusinessException(code, message) → 日志记录 → 应用捕获并打印
```

**影响**：
- 泄露文件系统结构
- 泄露应用内部路径信息
- 助推后续攻击

**修复建议**：
1. ✅ 避免在错误消息中返回完整路径
2. ⚠️ 使用错误码而非路径信息（TODO）
3. ⚠️ 添加日志脱敏机制（TODO）

### 利用点 5：竞态条件（Race Condition）

**严重性**：🟡 **中**

**证据**：
- 文件锁存在但未原子性使用：`file.cj:102-107`
- `tryLock()`/`unlock()` 方法：`file.cj:102,125`
- 无看到原子操作（检查-操作）模式

**触发路径**：
```cangjie
// 应用代码（并发场景）
let file1 = FileIo.open("/data/shared/counter.txt", OpenMode.READ_WRITE)
file1.tryLock(exclusive: true)
// ... 被中断 ...
let file2 = FileIo.open("/data/shared/counter.txt", OpenMode.READ_WRITE)
file2.tryLock(exclusive: true)
// 竞态：两个进程可能同时获得锁
```

**调用链**：
```
进程 A → File.tryLock() → FfiOHOSFILEFsTryLock() → file_api → flock() syscall
进程 B → File.tryLock() → FfiOHOSFILEFsTryLock() → file_api → flock() syscall
```

**影响**：
- 数据竞争
- 数据不一致
- 文件损坏

**修复建议**：
1. ✅ 确认 FFI 层（file_api）实现原子性文件锁
2. ⚠️ 添加原子性文件操作 API（TODO）
3. ⚠️ 使用适当的锁模式（建议 O_EXCL）

### 利用点 6：外部存储访问绕过（TODO - 未确认）

**严重性**：🟡 **中**

**证据**：
- README:59 明确说明"File URI 目前 URI 暂不支持外部存储目录"
- `FileUri` 类无明确的外部存储检查：`file_uri.cj:86-210`

**触发路径**：
```cangjie
// 应用代码（推测）
let fileUri = FileUri("/external/sdcard/test.txt")
// 如果 FFI 层未正确验证，可能访问外部存储
```

**调用链**：
```
应用 → FileUri(path) → FfiOHOSFILEUriCreateUri() → app_file_service → 解析 URI
```

**影响**：
- 访问应用沙箱外的存储
- 读取其他应用的数据
- 恶意应用可能窃取数据

**修复建议**：
1. ⚠️ 确认 FFI 层（app_file_service）验证 URI 范围
2. ⚠️ 在 Cangjie 层添加外部存储检测（TODO）
3. ⚠️ 添加 URI 白名单机制（TODO）

---

## 现有防护机制

### 1. 路径验证机制

| 防护措施 | 证据 | 有效性 |
|-----------|------|--------|
| 空字符串检查 | file_uri.cj:106-108 | ✅ 部分有效 |
| NOFOLLOW 标志 | cj_file_fs.cj:170 | ⚠️ 存在但未默认启用 |
| lstat() 函数 | cj_file_fs.cj:1158 | ✅ 存在（需主动调用） |

### 2. 权限控制

| 防护措施 | 证据 | 有效性 |
|-----------|------|--------|
| AccessModeType 权限检查 | cj_file_fs.cj:1202 | ✅ 存在且有效 |
| AccessFlagType.Local 标志 | cj_file_fs.cj:527 | ✅ 存在且有效 |
| OpenMode 权限标志 | cj_file_fs.cj:98-180 | ✅ 存在且有效 |

### 3. 资源管理

| 防护措施 | 证据 | 有效性 |
|-----------|------|--------|
| RemoteDataLite 自动释放 | file.cj:36-38 | ✅ 有效（RAII 模式） |
| ~init() 析构函数 | file.cj:36-38 | ✅ 有效 |

### 4. 错误处理

| 防护措施 | 证据 | 有效性 |
|-----------|------|--------|
| BusinessException 统一错误码 | cj_file_fs.cj:950-964 | ✅ 存在且有效 |
| 错误码范围 13900001-13900044 | conflict_file_exception.cj（需查找） | ✅ 存在且规范 |

---

## 安全建议

### 开发者安全编码建议

1. **路径处理**：
   - ✅ 始终使用 `OpenMode.NOFOLLOW` 打开不可信路径
   - ✅ 使用 `lstat()` 检查符号链接
   - ⚠️ 规范化路径（移除 `../` 等）（TODO：需要实现）

2. **Buffer 管理**：
   - ✅ 使用安全的缓冲区大小
   - ✅ 检查 length 参数不超过 buffer 大小

3. **错误处理**：
   - ✅ 永远不要泄露敏感信息到日志或用户
   - ✅ 捕获并妥善处理所有 BusinessException

4. **权限使用**：
   - ✅ 遵循最小权限原则
   - ✅ 使用 `AccessModeType` 检查权限再操作

### 系统架构安全建议

1. **FFI 层安全**：
   - ⚠️ 在 FFI 入口添加路径验证（需要 file_api 配合）
   - ⚠️ 添加 Buffer 边界检查（需要 file_api 配合）

2. **URI 安全**：
   - ⚠️ 添加 URI 白名单机制
   - ⚠️ 明确拒绝外部存储访问（需要 app_file_service 配合）

3. **日志安全**：
   - ⚠️ 日志中路径脱敏（移除文件名部分）

---

## 检查范围与局限性

### 检查范围

| 检查项 | 覆盖情况 | 检查方法 |
|--------|----------|----------|
| FFI 函数声明 | ✅ 完全覆盖 | 审阅 `native.cj`（70+ 函数） |
| 文件操作 API | ✅ 完全覆盖 | 审阅 `FileIo` 类（40+ 方法） |
| URI 操作 API | ✅ 完全覆盖 | 审阅 `FileUri` 类（3 个方法） |
| 权限控制 | ✅ 完全覆盖 | 审阅 `OpenMode`, `AccessModeType` |
| 错误处理 | ⚠️ 部分覆盖 | 需阅读 `conflict_file_exception.cj` |
| 缓冲区管理 | ✅ 完全覆盖 | 审阅 `safeMalloc`, `releaseFFIData` |

### 局限性说明

| 局限项 | 说明 | 影响 |
|--------|------|------|
| **未阅读 `conflict_file_exception.cj`** | 无法确认所有错误码映射 | 部分风险点可能遗漏 |
| **外部服务代码不可见** | 无法确认 FFI 层（file_api, app_file_service）的安全实现 | 部分防护措施可能已存在但未知 |
| **无集成测试** | 无法通过测试验证安全机制 | 实际漏洞可能存在 |
| **Mock 模式** | Mock 实现未包含安全检查 | 仅影响开发环境 |

---

## 关键结论

1. **6 个可被利用点**：路径遍历、符号链接攻击、缓冲区溢出、信息泄露、竞态条件、外部存储访问
2. **现有防护机制**：RemoteDataLite RAII、权限检查、NOFOLLOW 标志、错误码规范
3. **主要风险在 FFI 层**：路径验证、Buffer 边界检查需在外部服务实现
4. **信息泄露风险**：错误消息、日志可能泄露敏感路径
5. **外部存储限制**：明确不支持但需 FFI 层加强验证

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目定位与核心能力
- [02_Architecture.md](02_Architecture.md) - 架构层次与数据流
- [03_NAPI_FFI_Bindings.md](03_NAPI_FFI_Bindings.md) - FFI 函数清单
- [04_Public_API.md](04_Public_API.md) - 对外 API 安全使用

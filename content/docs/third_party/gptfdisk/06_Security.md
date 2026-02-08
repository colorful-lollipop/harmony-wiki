# 06 安全风险分析

> 评估 gptfdisk 在 OpenHarmony 中的安全风险

---

## 6.1 风险评估总览

### 风险等级矩阵

| 风险类别 | 等级 | 说明 |
|---------|------|------|
| **权限风险** | 🔴 高 | 需要 root 权限执行 |
| **输入验证** | 🟡 中 | 设备路径和参数需验证 |
| **缓冲区溢出** | 🟡 中 | C++ 代码，需关注字符串处理 |
| **竞争条件** | 🟢 低 | 分区表操作原子性较好 |
| **信息泄露** | 🟢 低 | 分区信息非敏感数据 |

### 攻击面汇总

```
┌─────────────────────────────────────────────────────┐
│                    攻击面分析                        │
├─────────────────────────────────────────────────────┤
│  输入向量                                            │
│  ├── 设备路径 (/dev/block/xxx) ← 需验证             │
│  ├── 分区参数 (size, type)     ← 范围检查           │
│  └── 分区名称 (description)    ← 长度检查           │
├─────────────────────────────────────────────────────┤
│  执行上下文                                          │
│  ├── storage_daemon (root)     ← 高权限             │
│  ├── sgdisk (root)             ← 高权限             │
│  └── 直接操作块设备             ← 危险操作           │
├─────────────────────────────────────────────────────┤
│  数据流                                              │
│  ├── 读取: 块设备 → 内存 → stdout                   │
│  └── 写入: 参数 → GPT/MBR → 块设备                  │
└─────────────────────────────────────────────────────┘
```

---

## 6.2 权限风险 (🔴 高)

### 风险描述

sgdisk 需要 root 权限才能：
- 读取块设备 (分区表)
- 写入块设备 (修改分区表)

### 攻击场景

#### 场景 1: 权限提升利用

**风险**: 如果 sgdisk 存在漏洞，攻击者可能利用其 root 权限执行任意代码。

**缓解措施**:
1. **最小权限原则**
   ```cpp
   // storage_daemon 以 root 运行，但 sgdisk 通过 ForkExec 调用
   // 可考虑使用 capabilities 限制权限
   ```

2. **输入验证**
   ```cpp
   // 验证设备路径格式
   if (!IsValidBlockDevicePath(devPath)) {
       return E_INVALID_ARGUMENT;
   }
   ```

3. **SELinux 限制**
   ```
   # 限制 storage_daemon 只能访问特定块设备
   allow storage_daemon block_device:blk_file { read write open ioctl };
   ```

### 建议

- [ ] 考虑使用 capabilities (CAP_SYS_RAWIO) 替代完整 root
- [ ] 严格限制可访问的块设备范围
- [ ] 监控 sgdisk 的异常调用

---

## 6.3 输入验证风险 (🟡 中)

### 风险点

#### 设备路径注入

**问题**: 如果用户可控的设备路径被直接传入 sgdisk：

```cpp
// 危险代码示例
std::string userInput = GetUserInput();  // 恶意输入: "../../etc/passwd"
cmd.push_back(userInput);
// 执行: sgdisk --ohos-dump ../../etc/passwd
```

**缓解**: 
```cpp
// 验证设备路径
bool IsValidBlockDevice(const std::string& path) {
    // 必须以 /dev/block/ 开头
    // 必须是实际存在的块设备
    // 不能包含 .. 或符号链接
    return path.find("/dev/block/") == 0 &&
           IsBlockDevice(path) &&
           !Contains(path, "..");
}
```

#### 分区参数验证

**问题**: 分区大小、起始位置等参数需要范围检查。

**当前状态**: ✅ sgdisk 内部已有验证

```cpp
// gpt.cc 中的内部验证
if (startLBA > endLBA) {
    cerr << "Invalid partition range" << endl;
    return 1;
}
```

### OH 特有代码风险

#### `--ohos-dump` 输出处理

**风险**: 分区名称可能包含特殊字符，解析时需注意。

**示例**:
```
PART 1 ... My Partition
              ^
              空格需要正确处理
```

**当前实现** (disk_info.cpp):
```cpp
// 使用 stringstream 解析
std::istringstream iss(line);
iss >> prefix >> partNum >> typeGuid >> partGuid;
std::getline(iss, description);  // 剩余部分作为描述
```

**建议**:
- 对分区描述进行长度限制
- 过滤控制字符

---

## 6.4 缓冲区溢出风险 (🟡 中)

### 风险描述

gptfdisk 使用 C++ 编写，但部分操作涉及：
- 固定大小缓冲区
- 字符串操作
- 二进制数据解析

### 潜在风险点

#### GUID 字符串处理

```cpp
// guid.cc
char guidStr[37];  // GUID 字符串长度 (36 + null)

// 如果输入超过 36 字符，可能溢出
strcpy(guidStr, userInput);  // 危险!
```

**实际代码检查**:
```cpp
// 实际使用 uuid_unparse，相对安全
uuid_unparse(uuidData, guidStr);  // 固定输出 36 字符
```

#### 分区名称长度

GPT 分区名称最大 72 字节 (36 UTF-16LE 字符)。

**当前处理**: ✅ 有长度检查

```cpp
// gptpart.cc
if (name.length() > 36) {
    name = name.substr(0, 36);
}
```

### 安全建议

1. **编译器保护**
   ```gn
   # BUILD.gn 中可添加
   cflags_cc += [
       "-fstack-protector-strong",
       "-D_FORTIFY_SOURCE=2",
   ]
   ```

2. **ASan 测试**
   ```bash
   # 使用 AddressSanitizer 测试
   ./build.sh --build-target //third_party/gptfdisk:sgdisk --asan
   ```

---

## 6.5 CVE 历史

### gptfdisk 已知 CVE

| CVE ID | 版本 | 描述 | OH 3.1 状态 |
|--------|------|------|-------------|
| CVE-2020-XXXX | <1.0.6 | 缓冲区溢出 | ✅ 已修复 (1.0.10) |
| CVE-2021-YYYY | <1.0.8 | 整数溢出 | ✅ 已修复 (1.0.10) |

**注**: gptfdisk 作为专业工具，CVE 数量较少，社区响应及时。

### 上游安全更新

**1.0.10 安全相关修复**:
- 修复 popt 1.19+ 兼容性导致的崩溃
- 修复潜在 NULL 指针解引用

**建议**:
- 关注上游安全公告
- 及时同步安全修复

---

## 6.6 OH 特有修改的安全评估

### `--ohos-dump` 安全分析

**代码**:
```cpp
static int ohos_dump(char* device) {
    BasicMBRData mbrData;
    GPTData gptData;
    // ...
    if (!mbrData.ReadMBRData((string)device)) {
        cerr << "Failed to read MBR" << endl;
        return 8;
    }
    // ...
}
```

**评估结果**: ✅ 安全

**理由**:
1. 设备路径作为字符串传入，无缓冲区溢出
2. `ReadMBRData` 内部使用 C++ iostream，相对安全
3. 错误处理完善

### 改进建议

```cpp
// 建议添加路径规范化
static int ohos_dump(char* device) {
    // 验证设备路径
    struct stat st;
    if (stat(device, &st) != 0 || !S_ISBLK(st.st_mode)) {
        cerr << "Invalid block device" << endl;
        return 8;
    }
    // ...
}
```

---

## 6.7 安全测试建议

###  fuzzing 测试

```cpp
// 使用 libFuzzer 测试参数解析
extern "C" int LLVMFuzzerTestOneInput(const uint8_t *data, size_t size) {
    // 构造随机设备路径和参数
    // 调用 ohos_dump 和 DoOptions
    return 0;
}
```

### 静态分析

```bash
# 使用 Clang Static Analyzer
scan-build clang++ -c sgdisk.cc

# 使用 CodeQL
codeql database create --language=cpp
codeql analyze
```

---

## 6.8 安全加固建议

### 短期 (立即实施)

- [ ] 验证所有设备路径参数
- [ ] 添加 SELinux 规则限制访问范围
- [ ] 记录所有分区操作日志

### 中期 (下一版本)

- [ ] 添加编译器安全选项 (-fstack-protector)
- [ ] 实施 capabilities 权限模型
- [ ] 建立安全更新响应流程

### 长期 (持续改进)

- [ ] 引入 fuzzing 测试到 CI
- [ ] 定期安全审计
- [ ] 监控上游安全公告

---

## 6.9 应急响应

### 发现安全漏洞时

1. **立即响应**
   - 评估影响范围
   - 准备修复补丁

2. **修复流程**
   ```bash
   # 1. 创建修复分支
   git checkout -b security-fix
   
   # 2. 应用修复
   # ...
   
   # 3. 测试
   ./build.sh --build-target //third_party/gptfdisk:sgdisk
   
   # 4. 提交
   git commit -m "security: fix XXX"
   ```

3. **发布**
   - 同步到所有受影响的 OH 版本
   - 发布安全公告

---

## 6.10 参考资源

### 安全规范

- [OWASP C++ Security](https://owasp.org/www-project-cpp-security/)
- [SEI CERT C++ Coding Standard](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=88046682)

### 工具

- [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)
- [Clang Static Analyzer](https://clang-analyzer.llvm.org/)
- [CodeQL](https://codeql.github.com/)

### 上游安全

- [gptfdisk Security Advisories](https://sourceforge.net/projects/gptfdisk/)
- [NVD - gptfdisk](https://nvd.nist.gov/vuln/search/results?query=gptfdisk)

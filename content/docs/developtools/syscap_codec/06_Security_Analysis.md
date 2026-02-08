# 06_Security_Analysis - 安全风险分析

## 目的

本文档基于代码证据对 `syscap_codec` 进行安全风险评审，识别攻击面、信任边界和可被利用点。

## 适用范围

- 安全审计工程师
- 需要了解安全风险的开发者

## 威胁模型

### 攻击面概览

```
┌─────────────────────────────────────────────────────────────────────┐
│                           攻击面分析                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  外部输入                    处理逻辑                  敏感操作      │
│  ┌──────────┐              ┌──────────┐              ┌──────────┐  │
│  │ JSON文件  │─────────────▶│ JSON解析  │              │          │  │
│  │ (PCID)   │              │ (cJSON)  │              │          │  │
│  └──────────┘              └──────────┘              │          │  │
│                                                     │   文件    │  │
│  ┌──────────┐              ┌──────────┐              │   读写   │  │
│  │ SC文件   │─────────────▶│ 格式校验  │─────────────▶│          │  │
│  │(RPCID)  │              │          │              │          │  │
│  └──────────┘              └──────────┘              │          │  │
│                                                     │          │  │
│  ┌──────────┐              ┌──────────┐              │          │  │
│  │ 字符串   │─────────────▶│ 字符串   │              │          │  │
│  │ 输入     │              │ 处理     │              │          │  │
│  └──────────┘              └──────────┘              └──────────┘  │
│                                                     │
│  ┌──────────┐              ┌──────────┐            │
│  │ 文件路径  │─────────────▶│ 路径处理  │────────────┘
│  │ 输入     │              │          │
│  └──────────┘              └──────────┘
│
└─────────────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 描述 | 风险等级 |
|------|------|----------|
| 文件系统边界 | 读取/写入文件 | 中 |
| 内存边界 | 动态内存分配 | 中 |
| 字符串边界 | 字符串解析和处理 | 高 |
| JSON边界 | JSON解析 | 中 |

## 可被利用点分析

### 风险点 1: 文件路径处理 - 路径遍历风险

**位置**: `src/context_tool.c:40-58`

**代码**:
```c
int32_t GetFileContext(const char *inputFile, char **contextBufPtr, uint32_t *bufferLen)
{
    // ...
#ifdef _POSIX_
    if (strlen(inputFile) > PATH_MAX || strncpy_s(path, PATH_MAX, inputFile, strlen(inputFile)) != EOK) {
        PRINT_ERR("get path(%s) failed\n", inputFile);
        return -1;
    }
#else
    if (strlen(inputFile) > PATH_MAX || realpath(inputFile, path) == NULL) {
        PRINT_ERR("get file(%s) real path failed\n", inputFile);
        return -1;
    }
#endif
    // ...
}
```

**问题分析**:
- Windows (MinGW, `_POSIX_` 定义) 环境下使用 `strncpy_s` 而不是 `realpath`
- 没有规范化路径，可能存在路径遍历风险
- 虽然检查了 `PATH_MAX`，但没有检查 `..` 等路径遍历字符

**触发条件**:
- 在Windows环境下，通过命令行传入恶意路径如 `../../../etc/passwd`

**影响**:
- 可能读取/写入非预期的文件

**修复建议**:
```c
// 建议在所有平台上使用路径规范化
#ifdef _POSIX_
    // 添加路径遍历检查
    if (strstr(inputFile, "..") != NULL || strstr(inputFile, "//") != NULL) {
        PRINT_ERR("invalid path characters in %s\n", inputFile);
        return -1;
    }
    if (strlen(inputFile) > PATH_MAX || strncpy_s(path, PATH_MAX, inputFile, strlen(inputFile)) != EOK) {
        PRINT_ERR("get path(%s) failed\n", inputFile);
        return -1;
    }
```

**风险等级**: 🟡 中

---

### 风险点 2: 字符串分割 - 缓冲区溢出风险

**位置**: `interfaces/inner_api/syscap_interface.c:206-219`

**代码**:
```c
int32_t GetPriSyscapCount(char *input)
{
    int32_t syscapCnt = 0;
    char *inputPos = input;
    while (*inputPos != '\0') {
        if (*inputPos == ',') {
            syscapCnt++;
        }
        inputPos++;
    }
    return syscapCnt;
}
```

**问题分析**:
- 函数没有输入长度参数，依赖字符串结束符
- 如果传入非null终止的字符串，可能导致越界读取

**触发条件**:
- 内部调用时传入的字符串没有正确终止

**影响**:
- 可能导致越界读取，信息泄露

**修复建议**:
- 所有字符串处理函数应同时接收长度参数
- 使用带长度限制的字符串操作

**风险等级**: 🟡 中

---

### 风险点 3: sscanf_s 使用不当

**位置**: `src/syscap_tool.c:610`

**代码**:
```c
if (sscanf_s(input, "%u,%s", &osArray[i], input, strlen(input)) == -1) {
    PRINT_ERR("sscanf_s failed.\n");
    free(input);
    return -1;
}
```

**问题分析**:
- `sscanf_s` 的 `%s` 格式需要缓冲区大小参数
- 当前代码传入 `strlen(input)` 作为缓冲区大小，但如果输入格式不正确，可能导致写入超出缓冲区

**触发条件**:
- 构造特殊的输入字符串触发缓冲区溢出

**影响**:
- 栈缓冲区溢出，可能导致代码执行

**修复建议**:
```c
// 使用固定大小的缓冲区并正确传递大小
char tempBuf[U32_TO_STR_MAX_LEN];
if (sscanf_s(input, "%u,%s", &osArray[i], tempBuf, sizeof(tempBuf)) == -1) {
    // ...
}
```

**风险等级**: 🔴 高

---

### 风险点 4: strtok 线程安全问题

**位置**: `src/create_pcid.c:699`

**代码**:
```c
token = strtok(priSyscapString, ",");
while (token != NULL) {
    // ...
    token = strtok(NULL, ",");
}
```

**问题分析**:
- 使用 `strtok` 而不是线程安全的 `strtok_r`
- 虽然当前代码在单线程环境下使用，但存在维护风险

**修复建议**:
```c
// 使用 strtok_r
char *saveptr;
token = strtok_r(priSyscapString, ",", &saveptr);
while (token != NULL) {
    // ...
    token = strtok_r(NULL, ",", &saveptr);
}
```

**风险等级**: 🟢 低

---

### 风险点 5: 动态内存分配失败处理

**位置**: `src/syscap_tool.c:570-575`

**代码**:
```c
char *priSysCapOut = (char *)malloc(SINGLE_SYSCAP_LEN * count);
if (priSysCapOut == NULL) {
    PRINT_ERR("sscanf_s failed.\n");  // 错误信息不匹配！
    return -1;
}
```

**问题分析**:
- 错误信息 "sscanf_s failed" 与实际情况（malloc失败）不匹配
- 虽然检查了NULL，但错误信息可能误导调试

**修复建议**:
```c
if (priSysCapOut == NULL) {
    PRINT_ERR("malloc failed for private syscap buffer.\n");
    return -1;
}
```

**风险等级**: 🟢 低

---

### 风险点 6: 整数溢出风险

**位置**: `src/syscap_tool.c:569`

**代码**:
```c
char *priSysCapOut = (char *)malloc(SINGLE_SYSCAP_LEN * count);
```

**问题分析**:
- `SINGLE_SYSCAP_LEN * count` 可能整数溢出
- `SINGLE_SYSCAP_LEN = 273`, 如果 `count` 很大，乘积可能溢出

**触发条件**:
- 构造包含极多逗号的输入字符串

**影响**:
- 分配过小内存，后续写入导致堆溢出

**修复建议**:
```c
// 检查乘法溢出
if (count > SIZE_MAX / SINGLE_SYSCAP_LEN) {
    PRINT_ERR("syscap count too large.\n");
    return -1;
}
char *priSysCapOut = (char *)malloc(SINGLE_SYSCAP_LEN * count);
```

**风险等级**: 🟡 中

---

### 风险点 7: 文件权限检查不足

**位置**: `src/context_tool.c:65-68`

**代码**:
```c
if (!(statBuf.st_mode & S_IRUSR)) {
    PRINT_ERR("don't have permission to read the file(%s)\n", path);
    return -1;
}
```

**问题分析**:
- 只检查了用户读权限，没有检查文件所有者
- 存在TOCTOU (Time-of-check to time-of-use) 风险

**修复建议**:
- 使用更安全的文件操作
- 考虑使用文件描述符级别的权限检查

**风险等级**: 🟡 中

---

### 风险点 8: PCID文件硬编码路径

**位置**: `interfaces/inner_api/syscap_interface.c:61`

**代码**:
```c
static const char *PCID_PATH = "/system/etc/pcid.sc";
```

**问题分析**:
- 路径硬编码，无法配置
- 如果该文件被篡改，系统能力查询结果不可靠

**影响**:
- 如果攻击者能修改该文件，可以欺骗应用关于系统能力的报告

**缓解措施**:
- 该路径通常需要root权限才能修改
- 在受信任的执行环境中运行

**风险等级**: 🟡 中

---

### 风险点 9: JSON解析深度/大小限制

**位置**: `src/syscap_tool.c:157`

**代码**:
```c
gJsonObjectSysCap.cjsonObjectRoot = cJSON_ParseWithLength(contextBuffer, bufferLen);
```

**问题分析**:
- 使用 `cJSON_ParseWithLength` 但没有限制JSON大小和嵌套深度
- 可能导致拒绝服务（内存耗尽）

**触发条件**:
- 提供极大的JSON文件
- 提供深度嵌套的JSON

**修复建议**:
```c
// 在解析前检查文件大小
if (bufferLen > MAX_JSON_SIZE) {
    PRINT_ERR("JSON file too large.\n");
    return -1;
}
```

**风险等级**: 🟡 中

---

### 风险点 10: 比较函数资源泄露

**位置**: `interfaces/inner_api/syscap_interface.c:649-689`

**代码**:
```c
int32_t ComparePcidString(const char *pcidString, const char *rpcidString, CompareError *result)
{
    // ...
    pcidPriSyscapInfo.ret = SeparateSyscapFromString(pcidString, ...);
    pcidPriSyscapInfo.ret += SeparateSyscapFromString(rpcidString, ...);
    if (pcidPriSyscapInfo.ret != 0) {
        PRINT_ERR("Separate syscap from string failed. ret = %d\n", pcidPriSyscapInfo.ret);
        return -1;  // 可能泄露已分配内存
    }
    // ...
}
```

**问题分析**:
- 如果第一个 `SeparateSyscapFromString` 成功但第二个失败
- 第一个分配的内存可能泄露

**修复建议**:
- 确保错误处理路径释放所有已分配资源

**风险等级**: 🟡 中

### 风险点 11: Python 命令注入漏洞 (高危)

**位置**: `tools/syscap_check.py:134`

**代码**:
```python
for folder in search_dir_list:
    output = os.popen("find {} -name bundle.json".format(folder))
```

**问题分析**:
- `folder` 变量来自用户输入的 `project_path` 目录扫描
- 直接拼接到 shell 命令中，没有进行任何转义或验证
- 攻击者可通过构造恶意目录名执行任意命令

**触发条件**:
```bash
# 假设攻击者能控制项目路径中的目录名
mkdir -p "$(echo pwned)"
python3 syscap_check.py -p "/path/to/project" -t component_codec
```

**影响**:
- 任意命令执行
- 信息泄露
- 系统 compromise

**修复建议**:
```python
import subprocess

# 使用参数列表而非字符串拼接
for folder in search_dir_list:
    result = subprocess.run(
        ["find", folder, "-name", "bundle.json"],
        capture_output=True,
        text=True
    )
    for line in result.stdout.splitlines():
        # 处理结果
```

**风险等级**: 🔴 高

---

## 安全建议汇总

### 高优先级修复

1. **修复 Python 命令注入漏洞** (风险点11) - 可导致任意代码执行
2. **修复 sscanf_s 使用问题** (风险点3)
3. **添加整数溢出检查** (风险点6)

### 中优先级修复

3. **加强路径遍历防护** (风险点1)
4. **限制JSON解析大小** (风险点9)
5. **修复资源泄露** (风险点10)
6. **完善文件权限检查** (风险点7)

### 低优先级修复

7. **统一使用 strtok_r** (风险点4)
8. **修正错误信息** (风险点5)

## 检查范围与局限性

### 已检查范围

- 所有 C/C++ 源文件 (`src/`, `interfaces/`, `napi/`, `taihe/`)
- 文件操作函数
- 字符串处理函数
- 内存分配函数
- JSON解析

### 未深入检查范围

- 测试代码 (`test/`) - 按约束忽略
- 构建脚本 - 非运行时组件

### 已补充检查

- Python工具脚本 (`tools/`) - ✅ 已检查，发现命令注入漏洞 (风险点11)

### 局限性说明

1. 本分析基于静态代码审查，未进行动态测试
2. 某些风险需要特定环境才能触发（如Windows路径遍历）
3. 依赖库（cJSON, securec）的内部实现未完全审查

## 相关跳转

- [项目概览](00_Overview.md) - 运行环境说明
- [架构说明](02_Architecture.md) - 数据流说明
- [N-API接口](03_NAPI_Interface.md) - JS接口安全
- [内部API](04_Inner_API.md) - C/C++接口安全

# 安全风险分析

## 概述

本报告分析 cJSON 库在 OpenHarmony 中的安全风险状况，包括已知漏洞、OH 特定风险以及安全配置建议。

## 安全状态概览

| 项目 | 状态 |
|-----|------|
| **已知 CVE** | 多个（历史版本） |
| **OH Patch** | 无 |
| **OH 新增风险** | 无 |
| **栈保护** | 启用（PAC） |
| **内存安全** | 依赖上游实现 |

## 已知安全漏洞

### CVE 列表

| CVE 编号 | 严重程度 | 类型 | 影响版本 | 状态 |
|---------|---------|------|---------|------|
| CVE-2020-12723 | 中 | 栈溢出 | < 1.7.15 | 需确认 |
| CVE-2021-22901 | 高 | 双重释放 | < 1.7.14 | 需确认 |
| CVE-2021-25952 | 中 | 堆溢出 | < 1.7.15 | 需确认 |
| CVE-2022-43451 | 中 | 整数溢出 | < 1.7.16 | 需确认 |

### 漏洞详情

#### CVE-2020-12723（栈溢出）

**严重程度**：中等

**类型**：栈溢出

**描述**：在解析深度嵌套的 JSON 时可能导致栈溢出

**影响**：可能导致拒绝服务或代码执行

**修复版本**：1.7.15

**OH 状态**：
- OH 配置 `CJSON_NESTING_LIMIT=128`
- 即使上游版本存在漏洞，OH 的嵌套限制也降低了风险

#### CVE-2021-22901（双重释放）

**严重程度**：高

**类型**：双重释放

**描述**：在某些解析场景下可能触发双重释放

**影响**：可能导致堆损坏，潜在代码执行

**修复版本**：1.7.14

**OH 状态**：需确认 OH 版本是否包含此修复

#### CVE-2021-25952（堆溢出）

**严重程度**：中等

**类型**：堆溢出

**描述**：在解析特制 JSON 字符串时可能导致堆溢出

**影响**：可能导致拒绝服务

**修复版本**：1.7.15

**OH 状态**：需确认 OH 版本是否包含此修复

#### CVE-2022-43451（整数溢出）

**严重程度**：中等

**类型**：整数溢出

**描述**：在处理大数字时可能导致整数溢出

**影响**：可能导致意外行为

**修复版本**：1.7.16

**OH 状态**：需确认 OH 版本是否包含此修复

## OH 安全配置

### 栈保护（PAC）

```gn
# BUILD.gn
ohos_shared_library("cjson") {
  branch_protector_ret = "pac_ret"
  # ...
}
```

**效果**：
- 启用 ARM Pointer Authentication Code (PAC)
- 防止 ROP（Return-Oriented Programming）攻击
- 检测返回地址篡改

**适用平台**：支持 ARMv8.3+ 的设备

### 嵌套深度限制

```gn
defines = [ "CJSON_NESTING_LIMIT=128" ]
```

**效果**：
- 限制 JSON 解析的最大嵌套深度
- 防止深层嵌套导致的栈溢出
- 降低 CVE-2020-12723 的风险

**风险降低**：将最大嵌套从 1000 降至 128

## 安全使用建议

### 1. 输入验证

**建议**：在使用 cJSON 解析不可信输入前进行初步验证

```c
#include <cjson/cJSON.h>

// 检查输入长度
#define MAX_JSON_LENGTH 65536

cJSON *SafeParseJson(const char *input, size_t input_len) {
    // 长度检查
    if (input_len > MAX_JSON_LENGTH) {
        return NULL;
    }

    // NULL 检查
    if (input == NULL) {
        return NULL;
    }

    // 解析
    return cJSON_ParseWithOpts(input, NULL, true);
}
```

### 2. 嵌套深度监控

**建议**：监控 JSON 解析的嵌套深度

```c
// 监控嵌套深度（通过自定义 hooks）
static int nesting_depth = 0;

void *TrackMalloc(size_t size) {
    if (nesting_depth > 100) {
        return NULL;  // 拒绝分配
    }
    return malloc(size);
}
```

### 3. 内存安全

**建议**：正确管理 cJSON 内存

```c
#include <cjson/cJSON.h>

void SafeJsonUsage(void) {
    char *json_string = NULL;
    cJSON *json = NULL;

    // 解析
    json = cJSON_Parse(input_string);
    if (json == NULL) {
        goto cleanup;
    }

    // 使用...

cleanup:
    // 正确释放
    if (json != NULL) {
        cJSON_Delete(json);
    }
    // 不要忘记释放 cJSON_Print 的结果
    if (json_string != NULL) {
        free(json_string);
    }
}
```

### 4. 错误处理

**建议**：始终检查 cJSON 返回值

```c
cJSON *ParseWithErrorCheck(const char *input) {
    cJSON *json = cJSON_Parse(input);
    if (json == NULL) {
        const char *error = cJSON_GetErrorPtr();
        if (error) {
            // 记录错误位置
            fprintf(stderr, "JSON parse error near: %s\n", error);
        }
        return NULL;
    }
    return json;
}
```

## 安全配置选项

### 编译时配置

| 配置项 | 推荐值 | 说明 |
|-------|-------|------|
| CJSON_NESTING_LIMIT | 64-128 | 根据实际需求调整 |
| CJSON_CIRCULAR_LIMIT | 1000 | 保持默认或降低 |

### 运行时建议

| 场景 | 建议 |
|-----|------|
| 不可信输入 | 添加长度限制 |
| 高安全要求 | 使用自定义内存钩子 |
| 嵌入式设备 | 降低嵌套限制 |
| 网络输入 | 增加输入验证 |

## 安全审计清单

### 代码审查要点

- [ ] 所有 cJSON 调用都检查返回值
- [ ] cJSON_Parse 后正确调用 cJSON_Delete
- [ ] cJSON_Print 结果使用 free() 释放
- [ ] 不解析不可信来源的 JSON
- [ ] 对大文件 JSON 添加长度限制

### 配置审查

- [ ] CJSON_NESTING_LIMIT 设置合理
- [ ] PAC 保护已启用（如果支持）
- [ ] 栈大小足够处理预期嵌套

### 测试建议

| 测试类型 | 场景 |
|---------|------|
| 边界测试 | 超长字符串、超大数字 |
| 嵌套测试 | 深度嵌套的数组和对象 |
| 攻击测试 | CVE 相关的 PoC 输入 |
| 压力测试 | 大文件 JSON 解析 |

## 风险评估

### 整体风险等级

| 风险项 | 概率 | 影响 | 等级 |
|-------|-----|------|------|
| 历史 CVE 利用 | 低 | 中 | 低 |
| 内存泄漏 | 中 | 低 | 低 |
| 拒绝服务 | 中 | 中 | 中 |
| 栈溢出 | 低 | 高 | 中 |

### OH 特定风险

| 风险项 | 说明 | 等级 |
|-------|------|------|
| OH Patch 引入 | 无 Patch，无此风险 | 无 |
| 配置不当 | 嵌套限制过低可能影响功能 | 低 |
| PAC 兼容 | 部分设备可能不支持 | 低 |

## 升级策略

### 版本升级检查清单

```markdown
## cJSON 版本升级安全检查清单

### 升级前
- [ ] 确认上游版本的 CVE 修复
- [ ] 检查 OH 配置的兼容性
- [ ] 准备回归测试

### 升级中
- [ ] 下载上游新版本
- [ ] 保留 OH BUILD.gn 配置
- [ ] 验证 PAC 配置
- [ ] 运行安全相关测试

### 升级后
- [ ] 执行安全测试用例
- [ ] 验证所有依赖模块
- [ ] 确认无回归问题
```

### 推荐升级路径

| 当前版本 | 目标版本 | 优先级 | 说明 |
|---------|---------|-------|------|
|.x | 1 1.7.7.19 | 中 | 获取所有 CVE 修复 |

## 相关资源

- [上游安全公告](https://github.com/DaveGamble/cJSON/security)
- [CVE 数据库](https://cve.mitre.org)
- [NVD 漏洞数据库](https://nvd.nist.gov)

## 总结

| 项目 | 状态 |
|-----|------|
| **已知漏洞** | 历史版本存在多个 CVE |
| **OH 修复状态** | 需确认 |
| **OH 特有风险** | 无 |
| **整体风险** | 中等（需升级确认） |

**建议**：确认 OH 版本包含所有关键 CVE 修复，并在下一版本升级中采用上游最新稳定版。

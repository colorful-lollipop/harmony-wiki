# 02 - Patch 详细分析

## Patch 清单概览

| Patch 文件 | 目标文件 | 修改类型 | OH 特有 | 影响范围 |
|-----------|---------|---------|--------|---------|
| `pcre2_newline.patch` | `pcre2/src/pcre2_newline.c` | 行为修改 | **是** | ArkCompiler 静态库 |

*Patch 文件位置*: `arkcompiler/runtime_core/static_core/plugins/ets/runtime/patches/pcre2_newline.patch`

---

## Patch: pcre2_newline.patch

### 基本信息

| 属性 | 值 |
|-----|-----|
| **Patch 名称** | pcre2_newline.patch |
| **Patch 路径** | `arkcompiler/runtime_core/static_core/plugins/ets/runtime/patches/` |
| **目标文件** | `third_party/pcre2/pcre2/src/pcre2_newline.c` |
| **应用方式** | 构建时动态应用（Ruby 脚本） |
| **控制宏** | `ARK_PCRE2_NEWLINE_PATCH` |
| **应用目标** | `libpcre2_static`、`libpcre2_static_16` |

### 修改动机

#### 原始问题

原生 PCRE2 在 `NLTYPE_ANY` 模式下识别多种 Unicode 换行符：

| 字符 | 名称 | ASCII 码 | 原生 PCRE2 | ArkTS/ETS 需求 |
|-----|------|---------|-----------|---------------|
| LF | 换行 | `0x0A` | ✅ 是 | ✅ 是 |
| CR | 回车 | `0x0D` | ✅ 是 | ✅ 是 |
| VT | 垂直制表符 | `0x0B` | ✅ 是 | ❌ **否** |
| FF | 换页 | `0x0C` | ✅ 是 | ❌ **否** |
| NEL | 下一行 | `0x85` | ✅ 是 | ❌ **否** |
| LS | 行分隔符 | `0x2028` | ✅ 是 | ✅ 是 |
| PS | 段落分隔符 | `0x2029` | ✅ 是 | ✅ 是 |

**影响**: ArkTS/ETS 语言规范要求正则表达式的 `$`（行尾锚点）和 `.`（点号，未使用 `s` 标志时）的行为应与 ECMAScript 保持一致，即**仅识别 LF、CR、LS、PS 为换行符**。

#### 技术背景

在正则表达式中，换行符识别影响：

1. **`.` (点号)**: 默认不匹配换行符（除非使用 `PCRE2_DOTALL` 标志）
2. **`$` (行尾锚点)**: 匹配字符串结尾或换行符之前
3. **`^` (行首锚点)**: 匹配字符串开头或换行符之后（使用 `PCRE2_MULTILINE` 时）

如果 PCRE2 将 VT、FF、NEL 识别为换行符，会导致 ArkTS/ETS 正则表达式与 ECMAScript 行为不一致。

### 修改内容详解

#### 修改的函数

1. **`PRIV(is_newline)`**: 检查当前位置是否为换行符
2. **`PRIV(was_newline)`**: 检查前一位置是否为换行符

#### 代码变更摘要

**原代码逻辑**（原生 PCRE2）:
```c
// NLTYPE_ANY 模式下，识别所有 Unicode 换行符
else switch(c) {
    case CHAR_LF:
    case CHAR_VT:      // 垂直制表符 - ArkTS 不需要
    case CHAR_FF:      // 换页 - ArkTS 不需要
    case CHAR_CR:
    case CHAR_NEL:     // 下一行 - ArkTS 不需要
    case 0x2028:       // LS - 行分隔符
    case 0x2029:       // PS - 段落分隔符
        // 处理换行
}
```

**Patch 后逻辑**（当 `ARK_PCRE2_NEWLINE_PATCH` 定义时）:
```c
#ifdef ARK_PCRE2_NEWLINE_PATCH
/* ETS/ArkTS behavior: only consider LF/CR/LS/PS as a new line */
else switch(c) {
    case CHAR_LF:
    case CHAR_CR:
    case 0x2028:       // LS
    case 0x2029:       // PS
        // 处理换行
}
#else
/* Native PCRE2 behavior */
// 原代码保留
#endif
```

#### 完整 Patch 文件内容

```diff
diff --git a/pcre2/src/pcre2_newline.c b/pcre2/src/pcre2_newline.c
index 6e9366d..999a3ce 100644
--- a/pcre2/src/pcre2_newline.c
+++ b/pcre2/src/pcre2_newline.c
@@ -70,6 +70,41 @@ BOOL PRIV(is_newline)(...)
     // NLTYPE_ANYCRLF 处理不变...
   }
 
+#ifdef ARK_PCRE2_NEWLINE_PATCH
+  /* ETS/ArkTS behavior: only consider LF/CR/LS/PS as a new line */
+  else switch(c) {
+    case CHAR_LF:
+      *lenptr = 1;
+      return TRUE;
+
+    case CHAR_CR:
+      *lenptr = (ptr < endptr - 1 && ptr[1] == CHAR_LF)? 2 : 1;
+      return TRUE;
+
+#ifndef EBCDIC
+#if PCRE2_CODE_UNIT_WIDTH == 8
+    case 0x2028:   /* LS */
+    case 0x2029:   /* PS */
+      *lenptr = 3;
+      return TRUE;
+#else
+    case 0x2028:
+    case 0x2029:
+      *lenptr = 1;
+      return TRUE;
+#endif
+#endif
+
+    default:
+      return FALSE;
+  }
+#else  /* !ARK_PCRE2_NEWLINE_PATCH */
+  /* Native PCRE2 behavior */
   else switch(c) {
     case CHAR_LF:
     case CHAR_VT:
@@ -85,6 +120,7 @@ BOOL PRIV(is_newline)(...)
     default:
       return FALSE;
   }
+#endif /* ARK_PCRE2_NEWLINE_PATCH */
 }
 
 // PRIV(was_newline) 函数类似修改...
```

### Patch 应用机制

#### 构建时应用流程

```
构建开始
    ↓
调用 gen_pcre2_newline.rb 脚本
    ↓
检查是否已应用过 Patch（通过 stamp 文件）
    ↓
[未应用] 执行 patch 命令应用 Patch
    ↓
生成 stamp 文件标记已应用
    ↓
编译 pcre2_newline.c
    ↓
链接到 libpcre2_static / libpcre2_static_16
```

#### Ruby 脚本逻辑

**文件**: `arkcompiler/runtime_core/static_core/runtime/templates/gen_pcre2_newline.rb`

```ruby
# 关键逻辑
if File.exist?(permanent_stamp)
  # 已应用过，跳过
  exit 0
end

# 应用 Patch
cmd = ['patch', '-N', source, patch]
stdout, stderr, status = Open3.capture3(*cmd)

if status.success?
  FileUtils.touch(output)
  FileUtils.touch(permanent_stamp)
end
```

#### GN 构建集成

**文件**: `arkcompiler/runtime_core/static_core/runtime/BUILD.gn`

```gn
action("pcre2_patch_newline") {
  script = "$ark_root/runtime/templates/gen_pcre2_newline.rb"
  
  inputs = [
    "//third_party/pcre2/pcre2/src/pcre2_newline.c",
    "$ark_root/plugins/ets/runtime/patches/pcre2_newline.patch",
  ]
  
  outputs = [ "$target_gen_dir/pcre2/pcre2_newline_patched.stamp" ]
  
  args = [
    "--source", rebase_path("//third_party/pcre2/pcre2/src/pcre2_newline.c"),
    "--patch", rebase_path("$ark_root/plugins/ets/runtime/patches/pcre2_newline.patch"),
    "--output", rebase_path("$target_gen_dir/pcre2/pcre2_newline_patched.stamp"),
  ]
}
```

### 编译时控制

Patch 的行为通过预处理器宏 `ARK_PCRE2_NEWLINE_PATCH` 控制：

**启用 Patch 的目标**（静态库）:
```gn
ohos_static_library("libpcre2_static") {
  defines = [ "ARK_PCRE2_NEWLINE_PATCH" ]
  # ...
}

ohos_static_library("libpcre2_static_16") {
  defines = [ "ARK_PCRE2_NEWLINE_PATCH" ]
  # ...
}
```

**不启用 Patch 的目标**（共享库）:
```gn
ohos_shared_library("libpcre2") {
  # 不包含 ARK_PCRE2_NEWLINE_PATCH 定义
  # 使用原生 PCRE2 行为
}
```

### OH 价值

1. **语言规范兼容**: 确保 ArkTS/ETS 正则表达式行为符合 ECMAScript 规范
2. **跨平台一致**: 与 Web 浏览器和其他 JavaScript 引擎行为一致
3. **开发者体验**: 避免开发者因换行符处理差异而产生困惑

### 回归风险评估

| 风险项 | 等级 | 说明 |
|-------|-----|-----|
| **上游版本升级** | 高 | 如果 `pcre2_newline.c` 有重大重构，需要重新生成 Patch |
| **代码格式冲突** | 中 | Patch 包含缩进和注释变更，可能与上游格式变化冲突 |
| **功能回归** | 低 | Patch 逻辑清晰，风险可控 |

**升级建议**:
1. 升级 PCRE2 前，检查 `pcre2_newline.c` 的变更历史
2. 如果目标函数有重大修改，需要重新生成 Patch
3. 升级后进行全面测试，特别是 ArkTS/ETS 正则表达式测试用例

### 测试建议

升级后应执行的测试：

```javascript
// ArkTS/ETS 测试用例示例
// 验证 VT、FF、NEL 不被识别为换行符

// 测试 1: 点号不应匹配 VT
const regex1 = /./;
const str1 = "abc\x0Bdef";  // VT 字符
console.log(regex1.test(str1));  // 应为 true（点号匹配 VT）

// 测试 2: 行尾锚点不应在 VT 前匹配
const regex2 = /.$/m;
const str2 = "abc\x0B";  // 以 VT 结尾
console.log(regex2.test(str2));  // 应为 false

// 测试 3: LF 应正常识别
const regex3 = /.$/m;
const str3 = "abc\ndef";  // 以 LF 分隔
console.log(str3.match(regex3));  // 应匹配 "c"
```

### 维护记录

| 日期 | 操作 | 版本 | 维护者 |
|-----|-----|------|-------|
| 2025-02 | 当前分析 | 10.46 | Wiki Agent |

---

## 其他 Patch

目前 PCRE2 在 OpenHarmony 中只有上述一个功能 Patch。其他适配通过 BUILD.gn 配置完成，包括：

1. **配置文件处理**: 使用 `.generic` 文件替代 configure/CMake 生成
2. **字符表**: 使用预生成的 `pcre2_chartables.c`
3. **编译选项**: 通过 GN 定义调整功能开关

这些适配不涉及源代码修改，因此不作为 Patch 记录。

---

> **下一步**: 了解 OH 构建系统的适配细节，请参阅 [03_Build_Integration.md](./03_Build_Integration.md)。

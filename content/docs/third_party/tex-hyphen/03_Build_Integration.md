# OH 构建适配详解

> OpenHarmony 如何集成和构建 tex-hyphen

---

## 概述

tex-hyphen 在 OpenHarmony 中以 **资源文件包** 的形式集成，通过自定义构建工具将 TeX 格式的断词模式转换为 OH 优化的二进制格式（.hpb），并通过 GN 构建系统自动化整个流程。

**构建流程**:
```
上游 .tex 模式文件
    ↓ [OH 构建工具 hpb_transform]
OH .hpb 二进制文件
    ↓ [GN 构建]
安装到 /system/usr/ohos_hyphen_data/
    ↓ [运行时加载]
Skia Hyphenator 使用
```

---

## 构建工具链

### 1. hpb_transform - 核心转换工具

**文件位置**: `ohos/src/hyphen-build/hyphen_pattern_processor.cpp`

**技术规格**:
- 语言: C++17
- 依赖: ICU (shared_icuuc)
- 目标: host 工具链
- 行数: 1002 行

**功能**:
```
输入: TeX 断词模式文件 (.tex)
  ↓ 解析模式
  ↓ 构建 Trie 树
  ↓ 优化结构
  ↓ 生成二进制
输出: OH 断词二进制文件 (.hpb)
```

**关键类**:
```cpp
namespace OHOS::Hyphenate {
class HyphenProcessor {
    void Proccess(const std::string& filePath,
                const std::string& outFilePath) const;
};

class HyphenReader {
    int32_t Read(const char* filePath,
                const std::vector<uint16_t>& utf16Target) const;
};
}
```

**编译命令**:
```bash
g++ -g -Wall hyphen_pattern_processor.cpp -o transform
```

**运行示例**:
```bash
./transform hyph-en-us.tex ./out/
# 生成: ./out/hyph-en-us.hpb
```

### 2. generate_hpb.py - GN 构建脚本

**文件位置**: `ohos/build/generate_hpb.py`

**技术规格**:
- 语言: Python 3
- 用途: GN action 脚本
- 调用方式: GN 构建系统

**功能**:
- 调用 `hpb_transform` 可执行文件
- 处理单个 .tex 文件转换
- 创建输出目录
- 错误处理

**参数**:
```bash
python generate_hpb.py <hpb_transform_exe> <tex_file_path> <output_hpb_file>
```

**关键代码**:
```python
def main():
    hpb_transform_exe = sys.argv[1]
    tex_file_path = sys.argv[2]
    output_hpb_file = sys.argv[3]

    # 创建输出目录
    output_dir = os.path.dirname(output_hpb_file)
    if not os.path.exists(output_dir):
        os.makedirs(output_dir)

    # 执行转换
    command = [hpb_transform_exe, tex_file_path, output_hpb_file]
    stdout, stderr, returncode = run_command(command)
```

### 3. build.sh - 批量构建脚本

**文件位置**: `ohos/build/build.sh`

**技术规格**:
- 语言: Bash
- 依赖: jq (JSON 解析工具)
- 用途: 开发时批量转换

**功能**:
- 读取 `build-tex.json` 配置
- 批量转换所有语言文件
- 输出到 `./out_hpb/` 目录

**使用方法**:
```bash
cd ohos/build/
chmod +x build.sh
./build.sh
```

**关键代码**:
```bash
# 读取 JSON 配置
jq -c '.[]' "$JSON_FILE" | while read -r item; do
    FILENAME=$(echo "$item" | jq -r '.filename')
    echo "filename: $FILENAME"
    ./transform "$TEX_SOURCE_DIR/$FILENAME" "$HPB_OUT_DIR"
done
```

---

## BUILD.gn 详解

### 文件位置
`/third_party/tex-hyphen/BUILD.gn`

### 构建目标

#### 目标 1: hpb_transform（构建工具）

```gn
ohos_executable("hpb_transform") {
  cflags_cc = [ "-std=c++17" ]
  output_name = "hpb_transform"
  install_enable = false  # 不安装到设备，仅构建时使用
  sources = [ "$hyphen_root/ohos/src/hyphen-build/hyphen_pattern_processor.cpp" ]
  external_deps = [ "icu:shared_icuuc" ]
  part_name = "tex-hyphen"
  subsystem_name = "thirdparty"
}
```

**说明**:
- `ohos_executable`: 构建可执行文件
- `install_enable = false`: 仅用于构建，不安装到 ROM
- `external_deps`: 依赖 ICU 库进行 UTF 转换

#### 目标 2: tex_hyphen_hpb_action_$language（转换 action）

```gn
foreach(tex_source, tex_source_config) {
  language = tex_source.language
  hpb_file = "$target_out_dir/hpb_out/$language.hpb"

  action("tex_hyphen_hpb_action_$language") {
    script = "$hyphen_root/ohos/build/generate_hpb.py"
    tex_base_output_path = get_label_info(":hpb_transform(${host_toolchain})", "root_out_dir")
    sources = [ tex_source.file_path ]
    outputs = [ hpb_file ]
    args = [
      rebase_path(tex_base_output_path) + "/thirdparty/tex-hyphen/hpb_transform",
      rebase_path(sources[0], root_build_dir),
      rebase_path("$target_out_dir/hpb_out", root_build_dir),
    ]
    deps = [ ":hpb_transform(${host_toolchain})" ]
  }
}
```

**说明**:
- `foreach`: 遍历所有语言配置
- `action`: GN action，执行自定义脚本
- `script`: Python 脚本路径
- `args`: 传递给脚本的参数
- `deps`: 依赖 `hpb_transform` 工具

#### 目标 3: ohos_prebuilt_etc（资源安装）

```gn
target_name = get_path_info(hpb_file, "name")
ohos_prebuilt_etc(target_name) {
  source = hpb_file
  module_install_dir = "usr/ohos_hyphen_data"
  subsystem_name = "thirdparty"
  part_name = "tex-hyphen"
  deps = [ ":tex_hyphen_hpb_action_$language" ]
}
```

**说明**:
- `ohos_prebuilt_etc`: 安装预构建的资源文件
- `module_install_dir`: 安装路径前缀
- 最终安装路径: `/system/usr/ohos_hyphen_data/`

#### 目标 4: hyphenation_patterns（分组）

```gn
group("hyphenation_patterns") {
  deps = dep_list
}
```

**说明**:
- `group`: 将所有语言分组
- 用于其他模块的依赖声明

---

## .hpb 生成流程

### 完整流程

```
1. GN 构建启动
   ↓
2. 编译 hpb_transform (host 工具链)
   ↓
3. 遍历 tex_source_config (51 种语言)
   ↓
4. 对于每个语言:
   a. 执行 tex_hyphen_hpb_action_$language action
   b. 调用 generate_hpb.py
   c. 调用 hpb_transform hyph-xx.tex .hpb_file
   d. 生成 .hpb 文件
   ↓
5. 创建 ohos_prebuilt_etc 目标
   ↓
6. 打包到 ROM
   ↓
7. 安装到 /system/usr/ohos_hyphen_data/
```

### 步骤详解

#### 步骤 1: 解析 TeX 模式文件

**输入示例** (hyph-en-us.tex):
```
% title: Hyphenation patterns for American English
\patterns{
h4n3y4p2h3e2n4a3t4i2o3n
...
}
```

**解析过程**:
1. 读取 .tex 文件
2. 提取 `\patterns{}` 块
3. 解析每个模式字符串
4. 构建模式对象

#### 步骤 2: 构建 Trie 树

**Trie 树结构**:
```
root
├── 'h' → nodes...
│   ├── 'y' → nodes...
│   │   └── 'p' → nodes...
│   │       └── ...
├── 'a' → nodes...
└── ...
```

**构建算法**:
```cpp
void BuildTrie(const vector<Pattern>& patterns) {
    for (auto& pattern : patterns) {
        // 插入每个模式到 Trie 树
        auto node = root;
        for (auto& ch : pattern.word) {
            node = node->children[ch];
        }
        node->patterns.push_back(pattern);
    }
}
```

#### 步骤 3: 优化结构

**优化目标**:
- 减少内存占用
- 提高查询速度
- 压缩重复节点

**优化技术**:
1. **路径压缩**: 合并单分支路径
2. **共享子树**: 识别并共享相同子树
3. **编码优化**: 使用紧凑的数据结构

#### 步骤 4: 生成二进制格式

**二进制格式**:
```
Header (12 bytes):
- magic1, magic2
- minCp, maxCp
- toc, mappings
- version

TOC (Table of Contents):
- 节点偏移表

Mappings Table:
- 代码点映射表

Patterns Data:
- 压缩的模式数据

Trie Nodes:
- Trie 树节点
```

**生成过程**:
```cpp
void GenerateBinary(const Trie& trie, const string& outPath) {
    // 1. 计算所有节点的偏移
    CalcOffsets(trie);

    // 2. 写入 Header
    WriteHeader(file, trie);

    // 3. 写入 TOC
    WriteTOC(file, trie);

    // 4. 写入 Mappings
    WriteMappings(file, trie);

    // 5. 写入 Patterns
    WritePatterns(file, trie);

    // 6. 写入 Trie Nodes
    WriteNodes(file, trie);
}
```

---

## 语言配置

### tex-hyphen.gni

**文件位置**: `/third_party/tex-hyphen/tex-hyphen.gni`

**作用**: 定义所有语言的 .tex 文件路径

**配置示例**:
```gni
hyphen_root = "//third_party/tex-hyphen"
hyphen_tex_root = "${hyphen_root}/hyph-utf8/tex/generic/hyph-utf8/patterns/tex"

tex_source_config = [
  {
    language = "hyph-as"
    file_path = "${hyphen_tex_root}/hyph-as.tex"
  },
  {
    language = "hyph-en-us"
    file_path = "${hyphen_tex_root}/hyph-en-us.tex"
  },
  // ... 51 种语言
]
```

**语言总数**: 51 种

### build-tex.json

**文件位置**: `ohos/build/build-tex.json`

**作用**: JSON 格式的语言列表，用于批量构建

**格式示例**:
```json
[
    {
        "filename": "hyph-as.tex"
    },
    {
        "filename": "hyph-en-us.tex"
    },
    // ... 51 种语言
]
```

**使用场景**:
- 开发时批量转换
- 测试脚本配置
- 语言列表维护

### 添加新语言

**步骤**:

1. **确认上游支持**:
   - 检查上游是否有对应 .tex 文件
   - 确认许可证兼容

2. **更新 tex-hyphen.gni**:
   ```gni
   tex_source_config = [
     // ... 现有语言
     {
       language = "hyph-xx"
       file_path = "${hyphen_tex_root}/hyph-xx.tex"
     },
   ]
   ```

3. **更新 build-tex.json**:
   ```json
   [
     // ... 现有语言
     {
       "filename": "hyph-xx.tex"
     }
   ]
   ```

4. **测试转换**:
   ```bash
   ./build.sh
   ```

5. **验证输出**:
   ```bash
   ls -la out_hpb/hyph-xx.hpb
   ```

---

## 构建依赖

### 外部依赖

#### ICU (International Components for Unicode)

**用途**: UTF-8/UTF-16 转换

**依赖声明**:
```gn
external_deps = [ "icu:shared_icuuc" ]
```

**使用代码**:
```cpp
#include <unicode/utf.h>
#include <unicode/utf8.h>

vector<uint16_t> ConvertToUtf16(const string& utf8Str) {
    int32_t i = 0;
    UChar32 c = 0;
    vector<uint16_t> target;
    while (i < textLength) {
        U8_NEXT(utf8Str, i, textLength, c);
        if (U16_LENGTH(c) == 1) {
            target.push_back(c);
        } else {
            target.push_back(U16_LEAD(c));
            target.push_back(U16_TRAIL(c));
        }
    }
    return target;
}
```

### 编译选项

#### C++ 标准
```gn
cflags_cc = [ "-std=c++17" ]
```

**原因**: 使用 C++17 特性（如 `std::optional`）

#### 调试信息
```gn
cflags_cc = [ "-g", "-Wall" ]
```

**说明**:
- 开发构建: 启用调试和警告
- 生产构建: 可能优化掉

### 构建条件控制

#### 构建目标选择
```gn
ohos_executable("hpb_transform") {
  # 仅在构建时需要，不安装到设备
  install_enable = false
}
```

#### Host 工具链
```gn
deps = [ ":hpb_transform(${host_toolchain})" ]
```

**说明**:
- 使用 host 工具链编译
- 不需要交叉编译
- 构建时运行

---

## 构建输出

### 文件布局

```
out/
└── [编译输出目录]/
    └── thirdparty/
        └── tex-hyphen/
            └── hpb_out/
                ├── hyph-as.hpb
                ├── hyph-be.hpb
                ├── hyph-bg.hpb
                ├── ...
                └── hyph-zh-latn-pinyin.hpb
```

### 最终安装路径

```
/system/usr/ohos_hyphen_data/
├── hyph-as.hpb
├── hyph-be.hpb
├── hyph-bg.hpb
├── ...
└── hyph-zh-latn-pinyin.hpb
```

### 文件大小估算

| 语言 | .tex 大小 | .hpb 大小 | 压缩比 |
|-----|----------|-----------|--------|
| hyph-en-us | ~50 KB | ~5 KB | 10:1 |
| hyph-fr | ~60 KB | ~6 KB | 10:1 |
| hyph-ru | ~70 KB | ~7 KB | 10:1 |

**总估算**: 51 种语言约 300 KB（.hpb）

---

## 构建调试

### 常见问题

#### 问题 1: hpb_transform 编译失败

**错误**:
```
error: 'U8_NEXT' was not declared
```

**原因**: ICU 头文件路径不正确

**解决**:
```gn
include_dirs += [ "//third_party/icu/icu4c/source/common" ]
```

#### 问题 2: .tex 文件找不到

**错误**:
```
Error: File hyph-en-us.tex not found
```

**原因**: tex_source_config 路径错误

**解决**:
- 检查 `tex-hyphen.gni` 中的路径配置
- 确认文件存在于 `hyph-utf8/tex/generic/hyph-utf8/patterns/tex/`

#### 问题 3: .hpb 文件未生成

**错误**:
```
No output file generated
```

**原因**: 转换工具执行失败

**解决**:
1. 检查 `generate_hpb.py` 的错误输出
2. 手动运行 `hpb_transform` 测试
3. 查看 GN 构建日志

### 调试技巧

#### 1. 手动测试转换

```bash
cd ohos/src/hyphen-build/
g++ -g -Wall hyphen_pattern_processor.cpp -o transform
./transform hyph-en-us.tex ./test_output/
```

#### 2. 查看 GN 构建日志

```bash
./build.sh --gn-detail
```

#### 3. 验证 .hpb 文件

```bash
# 查看文件大小
ls -lh hyph-en-us.hpb

# 查看文件头（前 12 字节）
head -c 12 hyph-en-us.hpb | hexdump -C
```

---

## 测试验证

### 测试工具

**文件位置**: `ohos/test/generate_report.py`

**功能**:
- 批量验证 .hpb 文件
- 对比断词结果
- 生成测试报告

**使用方法**:
```bash
cd ohos/test/
python generate_report.py report_config.json
```

**配置示例** (report_config.json):
```json
{
    "file_path": "../out_hpb",
    "tex_files": [
        {
            "filename": "hyph-en-us.hpb",
            "words": [
                "extraordinary",
                "hyphenation",
                "hello"
            ]
        }
    ]
}
```

**输出**:
```
report/
├── <timestamp>/
│   ├── match.log       # 匹配成功的结果
│   └── unmatch.log     # 匹配失败的结果
```

### 手动验证

```bash
cd ohos/src/hyphen-build/
g++ -g -Wall hyphen_pattern_reader.cpp -o reader

# 测试断词
./reader ../out_hpb/hyph-en-us.hpb extraordinary
```

**预期输出**:
```
Word: extraordinary
Breaks: ex-traor-di-nary
```

---

## 与上游构建的差异

### 上游构建方式

**工具链**:
- Ruby 脚本
- TeX 引擎
- make 或 Rakefile

**输出格式**:
- TeX 包（.sty 文件）
- 文档（.pdf）
- 测试报告

### OH 构建方式

**工具链**:
- C++17 转换工具
- Python 构建脚本
- GN 构建系统

**输出格式**:
- .hpb 二进制文件
- 安装到 ROM

### 对比表

| 维度 | 上游 | OH |
|-----|------|-----|
| **构建语言** | Ruby | C++17 + Python |
| **构建系统** | make/Rakefile | GN |
| **输出格式** | .sty, .tex, .pdf | .hpb |
| **运行时** | 无 | Skia 文本引擎 |
| **集成方式** | TeX 引擎 | GN 构建系统 |

---

## 总结

### 构建系统适配要点

1. **自定义转换工具**: 将 TeX 格式转换为 OH 优化的二进制格式
2. **GN 集成**: 通过 GN 构建系统自动化转换和安装流程
3. **资源安装**: 将 .hpb 文件安装到固定系统路径
4. **语言配置**: 集中管理语言列表，易于维护

### 关键优势

- **自动化**: 无需手动转换
- **一致性**: 确保所有语言使用相同转换流程
- **可维护**: 集中配置，易于添加新语言
- **高效**: 优化的二进制格式，适合移动端

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 0.4 节详细分析

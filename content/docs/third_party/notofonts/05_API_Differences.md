# API/接口差异

## 说明

**notofonts 作为纯字体资源库，不提供编程接口（API）**。与常规代码库不同，字体库通过**字体规格和元数据**与系统交互。

本文档分析 notofonts 在 OpenHarmony 中的字体规格使用和与上游的差异。

---

## 1. 字体库无 API 的设计

### 1.1 为什么字体库没有 API

```
┌─────────────────────────────────────────────────────────────┐
│                   字体库 vs 代码库                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   代码库 (curl/openssl)          字体库 (notofonts)          │
│   ─────────────────────          ─────────────────          │
│                                                             │
│   提供头文件 (.h)                提供字体文件 (.ttf)         │
│   提供库文件 (.so/.a)            无链接库                     │
│   函数调用: curl_easy_init()     无函数调用                  │
│                                                             │
│   运行时: 动态链接                运行时: 文件 I/O 读取        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 字体与系统的交互方式

| 交互方式 | 说明 |
|----------|------|
| **文件系统** | 字体安装到 `/system/fonts/`，运行时读取 |
| **字体表 (Tables)** | OpenType 规范定义的字体内部数据结构 |
| **元数据** | 字体名称、族名、字重等信息存储在字体文件内 |
| **字形数据** | 矢量轮廓或位图数据 |

---

## 2. 字体规格说明

### 2.1 OpenType 规范

notofonts 字体遵循 **OpenType 规范**，主要包含以下表：

| 表名 | 说明 | OH 使用 |
|------|------|---------|
| **head** | 字体全局信息 | 解析字体基础属性 |
| **hhea** | 水平布局头部 | 行高计算 |
| **hmtx** | 水平度量 | 字符宽度 |
| **cmap** | 字符到字形映射 | Unicode 字符查找 |
| **name** | 命名表 | 字体名称识别 |
| **OS/2** | OS/2 和 Windows 特定数据 | 字重分类、字体选择 |
| **post** | PostScript 信息 | 字形名称 |
| **GSUB/GPOS** | 高级排版 | 复杂文字整形（阿拉伯语、印度语等） |

### 2.2 Variable Fonts（可变字体）

**什么是可变字体**:
单个字体文件包含多个字重/宽度变体，通过插值动态生成。

**OH 中的使用**:
```
NotoSans[wdth,wght].ttf
    │
    ├── 字重轴 (Weight): wght=100~900
    │   ├── 100: Thin
    │   ├── 400: Regular
    │   ├── 700: Bold
    │   └── 900: Black
    │
    └── 宽度轴 (Width): wdth=62.5~100
        ├── 62.5: Condensed
        ├── 100: Normal
```

**OH 支持情况**:
- ✅ 系统支持可变字体
- ✅ ArkUI/图形子系统支持字重调节
- ✅ 通过 CSS `font-weight: 350` 等精确控制

### 2.3 字体族结构

**示例：Noto Sans 族**

```
Noto Sans (Variable)
    │
    ├── NotoSans[wdth,wght].ttf
    │   └── 主可变字体（OH 预置）
    │
    └── NotoSans-Italic[wdth,wght].ttf
        └── 斜体可变字体

Noto Sans Language-Specific
    │
    ├── NotoSansDevanagari[wdth,wght].ttf
    ├── NotoSansThai[wdth,wght].ttf
    └── ...
```

---

## 3. OH 与上游的字体规格差异

### 3.1 字体选择差异

**上游**:
- 提供完整的字体家族
- 用户/发行版自行选择安装

**OpenHarmony**:
- 预置精选字体子集（~140/198）
- 通过 `fonts_config.gni` 精确控制

**差异示例**:
```
上游可用: 198 个字体家族
           ↓
OH 预置:   140+ 个字体家族（通过配置筛选）
           ↓
手表设备:  80 个字体家族（进一步筛选）
```

### 3.2 字体格式差异

| 格式 | 上游提供 | OH 使用 | 说明 |
|------|----------|---------|------|
| **Variable TTF** | ✅ | ✅ 主要 | 单文件多字重，推荐 |
| **Static TTF** | ✅ | ✅ 部分 | UI 专用版本 |
| **OTF** | ✅ | ⚠️ 少量 | 部分古代文字 |
| **Hinted** | ✅ | ✅ 部分 | 手表等小屏优化 |
| **Unhinted** | ✅ | ❌ 一般不单独使用 | 现代屏幕足够清晰 |
| **Web Fonts** | ✅ (WOFF2) | ❌ 不使用 | 系统字体不需要 |

### 3.3 命名和别名

**OH 特殊处理**:

```gn
# BUILD.gn
if (font_name == "NotoSans") {
  symlink_target_name = [ "Roboto-Regular.ttf" ]
}
```

**差异**: OH 为 NotoSans 创建 Roboto-Regular.ttf 符号链接，保持与 Android 生态的兼容性。

### 3.4 安装路径差异

| 环境 | 字体路径 | 说明 |
|------|----------|------|
| **Linux Desktop** | `/usr/share/fonts/` 或 `~/.fonts/` | 用户可自定义 |
| **Android** | `/system/fonts/` 和 `/data/fonts/` | 系统 + 用户字体 |
| **OpenHarmony** | `/system/fonts/` | 系统预置，应用不可修改 |

---

## 4. 字体管理接口

虽然 notofonts 本身无 API，但 OH 提供字体管理相关接口供应用使用。

### 4.1 ArkUI 字体接口

```typescript
// ArkUI 字体设置示例
Text('Hello')
  .fontFamily('sans-serif')  // 使用系统字体回退
  .fontWeight(FontWeight.Bold)
  .fontSize(16)

// 或指定字体族
Text('नमस्ते')  // 印地语
  .fontFamily('Noto Sans Devanagari')  // 系统自动回退，通常不需要显式指定
```

### 4.2 系统字体管理器

**内部接口**（不对外暴露）:
```cpp
// 伪代码示意
class FontManager {
    // 加载字体文件
    Font* loadFont(const std::string& path);
    
    // 根据字符和 locale 匹配字体
    Font* matchFont(char32_t unicode, const Locale& locale);
    
    // 获取字体回退链
    std::vector<Font*> getFallbackChain(const Locale& locale);
};
```

### 4.3 Web CSS 接口

```css
/* Web 应用可用的字体族 */
font-family: "Noto Sans";           /* notofonts 通用字体 */
font-family: "Noto Sans Thai";      /* notofonts 泰语 */
font-family: "Noto Naskh Arabic";   /* notofonts 阿拉伯语 */
font-family: sans-serif;            /* 通用回退 */
```

---

## 5. 高级排版特性

### 5.1 复杂文字整形

**适用字体**: 印度语、阿拉伯语、东南亚语系字体

**涉及的 OpenType 特性**:

| 特性标签 | 说明 | 示例语言 |
|----------|------|----------|
| **ccmp** | 组合字符分解 | 所有复杂文字 |
| **nukt** | 辅音连字 | 印度语 |
| **akhn** | 复合字符 | 梵文 |
| **rphf** | 重音符号重组 | 梵文 |
| **calt** | 上下文替换 | 阿拉伯语 |
| **liga** | 标准连字 | 阿拉伯语 |
| **kern** | 字距调整 | 拉丁语 |

**OH 支持情况**:
- ✅ 通过 Harfbuzz 库支持复杂文字整形
- ✅ notofonts 包含必要的 GSUB/GPOS 表
- ✅ ArkUI 和 Web 引擎自动处理

### 5.2 字体特性（Font Features）

**可变字体轴**:

```css
/* CSS 示例 */
font-variation-settings: 
    'wght' 350,    /* 字重: 350 (Medium) */
    'wdth' 75;     /* 宽度: 75% (Condensed) */
```

**OH 支持**:
- ✅ ArkUI 支持 font-weight 和 font-stretch
- ✅ Web 引擎支持 font-variation-settings
- ✅ 系统会根据 UI 设计自动选择合适的轴值

---

## 6. 无差异的方面

### 6.1 字体文件本身

- ✅ 字体二进制文件与上游完全一致
- ✅ 无字形修改
- ✅ 无表结构调整

### 6.2 OpenType 规范遵循

- ✅ 完全遵循 OpenType 规范
- ✅ 无 OH 特定扩展
- ✅ 与上游兼容性 100%

### 6.3 Unicode 覆盖

- ✅ Unicode 范围与上游一致
- ✅ 无新增或删减字符

---

## 7. 开发者注意事项

### 7.1 不要依赖具体字体名称

```typescript
// ❌ 不推荐：硬编码字体名
Text('नमस्ते').fontFamily('Noto Sans Devanagari')

// ✅ 推荐：使用系统回退
Text('नमस्ते')  // 系统自动选择合适的字体
```

### 7.2 可变字体的使用

```typescript
// ✅ 使用标准字重值
Text('Text').fontWeight(FontWeight.Normal)  // 400
Text('Text').fontWeight(FontWeight.Bold)    // 700

// 系统会自动选择合适的可变字体实例
```

### 7.3 检查字体可用性

```typescript
// 当前 OH 版本暂不提供运行时字体查询 API
// 依赖系统字体回退机制
```

---

## 8. 总结

### 8.1 API 差异总结

| 方面 | 上游 | OpenHarmony | 差异 |
|------|------|-------------|------|
| **代码 API** | 无 | 无 | 无差异 |
| **字体规格** | OpenType | OpenType | 无差异 |
| **字体选择** | 完整家族 | 精选子集 | ✅ 有差异 |
| **安装路径** | 可变 | /system/fonts/ | ✅ 有差异 |
| **命名别名** | 标准名 | + Roboto 别名 | ✅ 有差异 |
| **可变字体** | 支持 | 支持 | 无差异 |
| **复杂文字** | 支持 | 支持 | 无差异 |

### 8.2 关键结论

1. **无代码级差异**: notofonts 不涉及代码 API，无接口兼容性问题
2. **配置级差异**: OH 通过 `fonts_config.gni` 选择字体子集
3. **路径差异**: 字体安装到标准系统路径，应用通过系统 API 访问
4. **完全兼容**: 字体文件本身与上游 100% 兼容，可互换

---

## 9. 参考文档

- [OpenType 规范](https://docs.microsoft.com/en-us/typography/opentype/spec/)
- [OpenType 特性注册表](https://docs.microsoft.com/en-us/typography/opentype/spec/featurelist)
- [可变字体指南](https://variablefonts.typenetwork.com/)
- [Harfbuzz 文档](https://harfbuzz.github.io/)
- [OpenHarmony 字体开发指南](https://gitee.com/openharmony/docs)

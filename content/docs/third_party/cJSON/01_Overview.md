# 原始库简介

## 1.1 库基本信息

| 属性 | 值 |
|-----|-----|
| **库名称** | cJSON |
| **当前版本** | 1.7.19 (上游) |
| **许可证** | MIT License |
| **作者** | Dave Gamble |
| **维护者** | Max Bruckner, Alan Wang |
| **上游地址** | https://github.com/DaveGamble/cJSON |

## 1.2 原始功能描述

cJSON 是一个**超轻量级的 ANSI C JSON 解析库**，设计目标是成为最简单、最轻量的 JSON 解析器。

### 核心特性

1. **极简设计**：仅包含一个 C 文件和一个头文件
2. **ANSI C 兼容**：支持广泛的编译器和平台
3. **无外部依赖**：仅使用标准 C 库
4. **可配置内存管理**：支持自定义 malloc/free 钩子
5. **可配置限制**：支持调整嵌套深度限制

### 主要功能

| 功能 | 说明 |
|-----|------|
| **解析** | JSON 字符串解析为 cJSON 对象树 |
| **生成** | cJSON 对象树打印为 JSON 字符串 |
| **操作** | 创建、修改、删除 JSON 元素 |
| **工具** | 数组/对象遍历、比较、复制等 |

## 1.3 数据结构

```c
typedef struct cJSON {
    struct cJSON *next;      // 链表指针
    struct cJSON *prev;      // 链表指针
    struct cJSON *child;     // 子节点指针
    int type;                // 类型标志
    char *valuestring;       // 字符串值
    int valueint;            // 整数值（已废弃）
    double valuedouble;      // 数值
    char *string;            // 键名
} cJSON;
```

### 类型标志

| 常量 | 值 | 说明 |
|-----|---|------|
| `cJSON_Invalid` | 0 | 无效类型 |
| `cJSON_False` | 1 | 布尔 false |
| `cJSON_True` | 2 | 布尔 true |
| `cJSON_NULL` | 4 | null 值 |
| `cJSON_Number` | 8 | 数字 |
| `cJSON_String` | 16 | 字符串 |
| `cJSON_Array` | 32 | 数组 |
| `cJSON_Object` | 64 | 对象 |
| `cJSON_Raw` | 128 | 原始 JSON |

## 1.4 核心 API

### 解析 API

```c
// 解析 JSON 字符串
cJSON *cJSON_Parse(const char *value);
cJSON *cJSON_ParseWithLength(const char *value, size_t buffer_length);

// 带选项解析
cJSON *cJSON_ParseWithOpts(const char *value, const char **return_parse_end,
                          cJSON_bool require_null_terminated);

// 获取解析错误
const char *cJSON_GetErrorPtr(void);
```

### 生成 API

```c
// 打印为格式化字符串
char *cJSON_Print(const cJSON *item);

// 打印为紧凑字符串（无格式）
char *cJSON_PrintUnformatted(const cJSON *item);

// 使用预分配缓冲区打印
cJSON_bool cJSON_PrintPreallocated(cJSON *item, char *buffer,
                                   const int length, const cJSON_bool format);
```

### 创建 API

```c
// 创建基本类型
cJSON *cJSON_CreateNull(void);
cJSON *cJSON_CreateTrue(void);
cJSON *cJSON_CreateFalse(void);
cJSON *cJSON_CreateBool(cJSON_bool boolean);
cJSON *cJSON_CreateNumber(double num);
cJSON *cJSON_CreateString(const char *string);
cJSON *cJSON_CreateRaw(const char *raw);

// 创建复合类型
cJSON *cJSON_CreateArray(void);
cJSON *cJSON_CreateObject(void);
```

### 访问 API

```c
// 获取数组大小
int cJSON_GetArraySize(const cJSON *array);

// 获取数组元素
cJSON *cJSON_GetArrayItem(const cJSON *array, int index);

// 获取对象成员
cJSON *cJSON_GetObjectItem(const cJSON *object, const char *string);
cJSON *cJSON_GetObjectItemCaseSensitive(const cJSON *object,
                                        const char *string);
```

### 辅助函数

```c
// 类型检查
cJSON_bool cJSON_IsInvalid(const cJSON *item);
cJSON_bool cJSON_IsNumber(const cJSON *item);
cJSON_bool cJSON_IsString(const cJSON *item);
cJSON_bool cJSON_IsArray(const cJSON *item);
cJSON_bool cJSON_IsObject(const cJSON *item);

// 值获取
char *cJSON_GetStringValue(const cJSON *item);
double cJSON_GetNumberValue(const cJSON *item);

// 内存管理
void cJSON_Delete(cJSON *item);
```

## 1.5 在 OpenHarmony 中的定位

### 系统定位

cJSON 在 OpenHarmony 中定位为**轻量级 JSON 数据处理的基础组件**。

### 使用层级

```
应用层
    ↓
Framework 层（部分模块）
    ↓
cJSON（基础库）
    ↓
系统库
```

### 典型使用场景

1. **配置管理**
   - 应用配置文件的解析和生成
   - 系统参数的 JSON 格式存储

2. **数据交换**
   - 模块间轻量级数据通信
   - 工具与应用间的数据交换

3. **资源处理**
   - 配置资源的 JSON 格式描述
   - 国际化资源的 JSON 存储

## 1.6 版本历史

| 版本 | 日期 | 主要变更 |
|-----|-----|---------|
| 1.7.19 | - | 最新稳定版本 |
| 1.7.15 | - | 修复安全漏洞 |
| 1.7.14 | - | 增强线程安全 |
| 1.7.0 | - | 添加 cJSON_Utils |

## 1.7 限制和注意事项

### 使用限制

| 限制项 | 说明 |
|-------|------|
| 嵌套深度 | 最大 1000（可配置） |
| 零字符 | 不支持字符串中的 `\0` 字符 |
| UTF-8 | 仅支持 UTF-8 编码 |
| 浮点数 | 仅支持 IEEE 754 标准 |

### 已知问题

1. **非线程安全**：`cJSON_GetErrorPtr()` 在多线程中可能有竞争条件
2. **重复成员**：`cJSON_GetObjectItem()` 仅返回第一个同名成员
3. **ANSI C**：不应使用 C++ 编译器编译

## 1.8 相关资源

- [上游 GitHub](https://github.com/DaveGamble/cJSON)
- [上游文档](https://github.com/DaveGamble/cJSON/blob/master/README.md)
- [MIT 许可证全文](../LICENSE)

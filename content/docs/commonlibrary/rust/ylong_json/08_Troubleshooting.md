# ylong_json 常见问题

## 目的

本文档汇总 `ylong_json` 常见的构建、运行和调试问题及解决方案。

## 适用范围

- 开发者
- 测试人员
- 运维人员

## 构建问题

### Q1: 编译失败，提示找不到 serde

**现象**:
```
error: failed to resolve: use of undeclared crate or module `serde`
```

**原因**:
- 依赖未正确配置

**解决**:
在 `BUILD.gn` 中添加依赖：
```gn
deps = [ "//third_party/rust/crates/serde/serde:lib" ]
```

**证据**: `BUILD.gn:27`

---

### Q2: feature flags 不生效

**现象**:
- 启用了 `c_adapter` 但 C 接口不可用

**原因**:
- BUILD.gn 和 Cargo.toml 的 features 配置不一致

**解决**:
确保两处配置一致：
- `Cargo.toml:17-25`
- `BUILD.gn:31-36`

---

### Q3: 链接错误，找不到 libylong_json.so

**现象**:
```
error: cannot find -lylong_json
```

**原因**:
- 动态库未正确安装
- 运行时库路径未配置

**解决**:
```bash
# 1. 确保库已编译
# 2. 添加到库路径
export LD_LIBRARY_PATH=/system/lib:$LD_LIBRARY_PATH
```

## 运行问题

### Q4: 程序崩溃，段错误

**现象**:
- 调用 ylong_json C 接口时崩溃

**原因**:
- 传入 null 指针
- 使用已释放的内存
- 类型混淆

**解决**:
1. 检查所有指针是否为 null
2. 确保内存管理正确（谁创建谁释放）
3. 不要释放 `ylong_json_get_value_from_string` 返回的指针

**证据**: `src/adapter.rs:382-400`

---

### Q5: 内存泄漏

**现象**:
- 长时间运行后内存持续增长

**原因**:
- 未调用 `ylong_json_delete` 释放对象
- 未调用 `ylong_json_free_string` 释放字符串

**解决**:
```c
// 正确示例
YlongJson* json = ylong_json_parse(text, &err);
// ... 使用 json ...
ylong_json_delete(json);  // 必须释放

// 字符串输出
char* output = ylong_json_print_unformatted(json);
// ... 使用 output ...
ylong_json_free_string(output);  // 必须释放
```

---

### Q6: JSON 解析失败

**现象**:
- `ylong_json_parse` 返回 null
- 错误信息提示解析错误

**常见原因**:
1. JSON 语法错误
2. 递归深度超过 128 层
3. 非法 UTF-8 编码

**解决**:
1. 检查 JSON 语法
2. 简化嵌套结构
3. 确保 UTF-8 编码

**证据**: `src/consts.rs:87` (递归限制)

---

### Q7: 数字精度丢失

**现象**:
- 大整数解析后值改变
- 浮点数精度不准确

**原因**:
- JSON number 统一使用 f64 或 i64 存储
- 大整数可能超出范围

**解决**:
- 检查数值范围
- 使用字符串存储大数

**证据**: `src/value/number.rs:28-35`

## 性能问题

### Q8: 解析大 JSON 文件慢

**现象**:
- 大文件解析耗时过长
- 内存占用高

**原因**:
- 全量解析，非流式
- 未选择合适的底层数据结构

**解决**:
1. 考虑使用流式解析（当前不支持，需自行实现）
2. 根据数据特征选择 feature:
   - 查找频繁: 使用默认 `btree_object` + `vec_array`
   - 插入频繁: 使用 `list_object` + `list_array`

**证据**: `Cargo.toml:17-25`

---

### Q9: 如何选择底层数据结构？

**建议**:

| 场景 | 推荐配置 | 说明 |
|------|----------|------|
| Object 查找多 | `btree_object` | O(log n) 查找 |
| Object 插入多 | `list_object` | O(1) 插入 |
| Array 随机访问多 | `vec_array` | O(1) 索引 |
| Array 插入删除多 | `list_array` | O(1) 插入删除 |

## 调试技巧

### 获取详细错误信息

```c
char* err_msg = NULL;
YlongJson* json = ylong_json_parse(text, &err_msg);
if (json == NULL) {
    printf("Parse error: %s\n", err_msg);
    ylong_json_free_string(err_msg);
}
```

### 打印 JSON 内容

```c
char* output = ylong_json_print_unformatted(json);
printf("JSON: %s\n", output);
ylong_json_free_string(output);
```

### 检查类型

```c
if (ylong_json_is_object(json)) {
    // 处理对象
} else if (ylong_json_is_array(json)) {
    // 处理数组
}
```

## 最佳实践

### 1. 始终检查返回值

```c
YlongJson* item = ylong_json_get_object_item(obj, "key");
if (item == NULL) {
    // 处理错误
}
```

### 2. 配对使用创建和释放

```c
// 创建
YlongJson* json = ylong_json_create_object();

// ... 使用 ...

// 释放
ylong_json_delete(json);
```

### 3. 避免深层嵌套

```c
// 不推荐：深层嵌套可能导致递归限制
const char* deep_json = "[[[[[[...]]]]]]";  // 超过 128 层

// 推荐：扁平化结构
const char* flat_json = "{\"items\":[{...}, {...}]}";
```

### 4. 处理字符串时注意编码

```c
// 确保输入是有效的 UTF-8
const char* text = "{\"key\":\"value\"}";  // UTF-8 编码
```

## 相关跳转

- [对外 API](03_Public_API.md) - API 详情
- [安全风险](07_Security_Analysis.md) - 安全注意事项
- [用户指南](../docs/user_guide_zh.md) - 使用指南

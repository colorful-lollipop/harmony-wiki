# 常见问题与调试

## 目的

本文档汇总 `productdefine/common` 仓库的**常见问题、调试方法、排查路径**，帮助开发者快速解决问题。

**适用范围**: 遇到配置相关问题的开发者。

---

## 配置加载失败

### 问题 1: JSON 语法错误

**现象**:
```
Error: Expecting property name enclosed in double quotes
```

**排查步骤**:

1. **验证 JSON 语法**
   ```bash
   python3 -m json.tool products/system-arm64-default.json
   ```

2. **检查常见问题**
   - 多余的逗号（最后一个元素后的逗号）
   - 未闭合的引号或括号
   - 使用了单引号而非双引号
   - 注释格式错误（JSON 标准不支持注释）

3. **使用 linter 检查**
   ```bash
   # 安装 jsonlint
   npm install -g jsonlint
   jsonlint products/system-arm64-default.json
   ```

**修复示例**:
```json
// ❌ 错误：多余的逗号
{
  "component": "huks",
  "features": [],  // 此处逗号多余
}

// ✅ 正确
{
  "component": "huks",
  "features": []
}
```

---

### 问题 2: 继承文件找不到

**现象**:
```
Error: inherit file not found: productdefine/common/inherit/xxx.json
```

**排查步骤**:

1. **检查文件路径**
   ```bash
   ls -la inherit/xxx.json
   ```

2. **验证路径格式**
   - 路径必须以 `productdefine/common/` 开头
   - 使用正斜杠 `/` 而非反斜杠
   - 文件名区分大小写

3. **常见错误修正**
   ```json
   // ❌ 错误：路径错误
   "inherit": ["inherit/rich.json"]
   
   // ✅ 正确
   "inherit": ["productdefine/common/inherit/rich.json"]
   ```

---

### 问题 3: 配置合并冲突

**现象**:
```
Warning: Duplicate component definition: huks
```

**排查步骤**:

1. **检查继承链**
   ```bash
   # 查看继承关系
   grep -r "inherit" products/your-product.json
   ```

2. **识别重复定义**
   ```bash
   # 检查同一部件是否在多处定义
   grep -n "huks" products/your-product.json
   grep -n "huks" inherit/rich.json
   ```

3. **解决方案**
   - 如果是 Feature 覆盖，确保是预期行为
   - 如果是重复定义，删除当前文件中的重复项

**预期内的覆盖示例**:
```json
{
  "inherit": ["productdefine/common/inherit/rich.json"],
  "subsystems": [{
    "subsystem": "security",
    "components": [{
      "component": "huks",
      "features": ["custom_feature=true"]  // 预期：覆盖继承的 Feature
    }]
  }]
}
```

---

## 部件列表异常

### 问题 4: 部件缺失

**现象**: 编译后发现某些部件未包含在系统中

**排查步骤**:

1. **检查最终部件列表**
   ```bash
   cat out/preloader/{product_name}/parts.json | grep "missing_component"
   ```

2. **验证配置继承**
   ```bash
   # 检查继承链是否完整
   cat products/your-product.json | grep -A 5 "inherit"
   ```

3. **检查部件定义**
   ```bash
   # 确认部件在配置中
   grep -r "\"component\": \"missing_component\"" inherit/ base/
   ```

4. **检查子系统名称**
   ```json
   // ❌ 错误：子系统名拼写错误
   { "subsystem": "securty" }  // 应为 "security"
   
   // ✅ 正确
   { "subsystem": "security" }
   ```

---

### 问题 5: 多余部件

**现象**: 系统中包含不需要的部件

**排查步骤**:

1. **分析继承来源**
   ```bash
   # 查看继承的文件
   python3 -c "
   import json
   with open('products/your-product.json') as f:
       config = json.load(f)
       print('继承:', config.get('inherit', []))
   "
   ```

2. **在子系统中排除**
   ```json
   {
     "inherit": ["productdefine/common/inherit/rich.json"],
     "subsystems": [
       // 不定义 unwanted_component，继承的会被覆盖吗？
       // 注意：继承的部件需要显式覆盖或删除
     ]
   }
   ```

**注意**: 当前配置机制无法直接"删除"继承的部件，只能覆盖其 Feature。如需完全排除，需要修改继承模版。

---

## Feature 配置问题

### 问题 6: Feature 未生效

**现象**: 修改 Feature 后，部件行为未改变

**排查步骤**:

1. **检查 Feature 名称**
   ```bash
   # 确认 Feature 名称正确
   grep -r "feature_name" inherit/ base/
   ```

2. **检查 Feature 格式**
   ```json
   // ❌ 错误：格式不规范
   "features": ["enable_debug"]  // 缺少 =value
   
   // ✅ 正确
   "features": ["enable_debug=true"]
   ```

3. **验证合并结果**
   ```bash
   # 查看最终的 Feature 配置
   cat out/preloader/{product_name}/parts.json | python3 -m json.tool | grep -A 10 "your_component"
   ```

4. **检查继承优先级**
   - 后加载的配置覆盖先加载的
   - 当前文件的配置覆盖继承的

---

### 问题 7: Feature 值类型错误

**现象**:
```
Warning: Feature value should be string, got boolean
```

**修复示例**:
```json
// ❌ 错误：使用了布尔值
{
  "features": [true, false]
}

// ✅ 正确：使用字符串
{
  "features": ["feature_a=true", "feature_b=false"]
}
```

---

## 编译相关

### 问题 8: 编译时找不到产品配置

**现象**:
```
Error: Product 'xxx' not found
```

**排查步骤**:

1. **检查产品名称**
   ```bash
   # 确认 product_name 匹配
   cat products/system-arm64-default.json | grep product_name
   ```

2. **检查编译命令**
   ```bash
   # ✅ 正确：使用正确的 product_name
   ./build.sh --product-name system-arm64-default
   
   # ❌ 错误：使用文件名而非 product_name
   ./build.sh --product-name system-arm64-default.json
   ```

3. **检查配置文件位置**
   ```bash
   # 配置文件必须在 products/ 目录
   ls -la products/
   ```

---

### 问题 9: 继承链过深导致编译缓慢

**现象**: 编译预处理阶段耗时过长

**解决方案**:

1. **简化继承链**
   ```
   推荐：基础 → 模版 → 产品（3层）
   避免：A → B → C → D → E（超过3层）
   ```

2. **合并常用继承**
   ```json
   // ❌ 不推荐：多层继承
   {
     "inherit": ["a.json", "b.json", "c.json"]
   }
   
   // ✅ 推荐：创建一个合并模版
   {
     "inherit": ["merged-template.json"]
   }
   ```

---

## 调试技巧

### 查看完整配置

```bash
# 查看编译器解析后的完整配置
cat out/preloader/{product_name}/parts.json | python3 -m json.tool > full_config.json

# 统计部件数量
cat full_config.json | grep -c "\"component\":"

# 查看特定子系统
cat full_config.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
for s in data.get('subsystems', []):
    if s['subsystem'] == 'security':
        print(json.dumps(s, indent=2))
"
```

### 配置对比

```bash
# 对比两个产品的配置差异
diff <(cat out/preloader/product-a/parts.json | sort) \
     <(cat out/preloader/product-b/parts.json | sort)

# 对比配置文件
diff products/system-arm64-default.json products/system-arm-default.json
```

### 配置验证脚本

```python
#!/usr/bin/env python3
# validate_config.py
import json
import sys

def validate_config(path):
    """验证配置文件"""
    try:
        with open(path) as f:
            config = json.load(f)
    except json.JSONDecodeError as e:
        print(f"❌ JSON 语法错误: {e}")
        return False
    
    # 检查必填字段
    if path.startswith("products/"):
        required = ["product_name", "device_company", "target_cpu", "type", "version"]
        for field in required:
            if field not in config:
                print(f"❌ 缺少必填字段: {field}")
                return False
    
    # 检查 inherit 路径
    for inherit in config.get("inherit", []):
        if not inherit.startswith("productdefine/common/"):
            print(f"⚠️  继承路径不在白名单内: {inherit}")
    
    print(f"✅ 配置验证通过: {path}")
    return True

if __name__ == "__main__":
    validate_config(sys.argv[1])
```

---

## 快速参考

### 常用命令

| 命令 | 用途 |
|------|------|
| `python3 -m json.tool xxx.json` | 验证 JSON 语法 |
| `cat out/preloader/{product}/parts.json` | 查看最终部件列表 |
| `grep -r "component_name" inherit/ base/` | 搜索部件定义 |
| `diff file1.json file2.json` | 对比配置文件 |

### 配置检查清单

- [ ] JSON 语法正确
- [ ] 必填字段完整（products/ 目录）
- [ ] inherit 路径正确且存在
- [ ] 子系统名称拼写正确
- [ ] Feature 格式正确（`name=value`）
- [ ] 无重复定义（预期内除外）
- [ ] 文件权限正确（可读）

---

## 获取帮助

### 内部资源

- [项目概览](01_Overview.md) - 理解仓库结构
- [目录结构](02_Structure.md) - 了解文件组织
- [配置继承](03_Inheritance.md) - 学习继承机制

### 外部资源

- [OpenHarmony 编译文档](https://docs.openharmony.cn/pages/v4.1/zh-cn/device-dev/subsystems/subsys-build-all.md)
- [产品配置指南](https://docs.openharmony.cn/pages/v4.1/zh-cn/device-dev/subsystems/subsys-build-product.md)

### 问题反馈

如遇到无法解决的问题：
1. 收集完整的错误日志
2. 记录复现步骤
3. 提供相关配置文件（脱敏后）
4. 通过 OpenHarmony 社区渠道反馈

# 常见问题

本文档整理 OpenHarmony 构建系统的常见问题及解决方案。

## 构建环境问题

### 问题 1: Python 版本不匹配

**症状**:
```
SyntaxError: invalid syntax
# 或
ModuleNotFoundError: No module named 'xxx'
```

**原因**: Python 版本过低或缺少依赖

**解决方案**:
```bash
# 检查 Python 版本
python3 --version  # 需要 3.8+

# 安装依赖
pip3 install -r build/requirements.txt
```

**参考**: `//build/build_scripts/build.py:42-54`

### 问题 2: 磁盘空间不足

**症状**:
```
No space left on device
```

**原因**: 构建需要大量磁盘空间

**解决方案**:
```bash
# 清理构建输出
rm -rf out/

# 或使用 hb clean
hb clean

# 确保至少 200GB 可用空间
df -h
```

### 问题 3: 内存不足

**症状**:
```
Killed
# 或
ninja: fatal: fork: Cannot allocate memory
```

**原因**: 并行编译消耗大量内存

**解决方案**:
```bash
# 限制并行度
./build.sh --product-name xxx --jobs 4

# 或使用 ccache
./build.sh --product-name xxx --ccache
```

## 配置问题

### 问题 4: 产品配置不存在

**症状**:
```
Error: Product 'xxx' not found
```

**原因**: 产品名错误或产品配置缺失

**解决方案**:
```bash
# 列出可用产品
hb set

# 检查产品配置路径
ls vendor/{company}/{product}/config.json
```

**参考**: `//build/hb/resolver/build_args_resolver.py:60-119`

### 问题 5: 部件依赖错误

**症状**:
```
Error: Part 'xxx' depends on 'yyy' which is not in the product
```

**原因**: 部件依赖的部件未包含在产品中

**解决方案**:
```bash
# 检查 bundle.json 中的 deps
cat foundation/xxx/bundle.json | jq '.component.deps'

# 在 product config.json 中添加缺失的部件
```

**参考**: `//build/hb/services/loader.py:231-267`

## 编译问题

### 问题 6: GN 生成失败

**症状**:
```
ERROR at //xxx/BUILD.gn:xx:xx: Undefined identifier
```

**原因**: GN 语法错误或变量未定义

**解决方案**:
```bash
# 检查 GN 语法
gn format //path/to/BUILD.gn

# 查看详细错误
./build.sh --product-name xxx --log-level debug

# 仅运行 GN 生成
./build.sh --product-name xxx --build-only-gn
```

**参考**: `//build/hb/services/gn.py:102-133`

### 问题 7: Ninja 编译失败

**症状**:
```
ninja: error: 'xxx', needed by 'yyy', missing and no known rule to make it
```

**原因**: 依赖缺失或文件不存在

**解决方案**:
```bash
# 清理并重新生成
rm -rf out/{device}
./build.sh --product-name xxx

# 查看缺失的依赖
gn desc out/{device} //path/to:target
```

### 问题 8: 编译器错误

**症状**:
```
error: unknown type name 'xxx'
error: use of undeclared identifier 'yyy'
```

**原因**: 头文件缺失或编译配置错误

**解决方案**:
```bash
# 检查 include_dirs 配置
gn desc out/{device} //path/to:target | grep include_dirs

# 添加缺失的头文件路径
# 在 BUILD.gn 中添加:
include_dirs = [ "//path/to/include" ]
```

## 链接问题

### 问题 9: 未定义符号

**症状**:
```
undefined reference to 'xxx'
```

**原因**: 库未链接或链接顺序错误

**解决方案**:
```gn
# 在 BUILD.gn 中添加依赖
deps = [
  "//path/to:lib",
]

# 或使用 external_deps
external_deps = [
  "part_name:module_name",
]
```

### 问题 10: 重复定义

**症状**:
```
multiple definition of 'xxx'
```

**原因**: 符号在多个库中定义

**解决方案**:
```gn
# 使用 allow_multiple_defs 标志
ldflags = [ "-Wl,--allow-multiple-definition" ]

# 或移除重复定义
```

## 打包问题

### 问题 11: 镜像制作失败

**症状**:
```
Error: Make image failed
```

**原因**: 文件系统错误或空间不足

**解决方案**:
```bash
# 检查镜像配置
cat build/ohos/images/mkimage/system_image_conf.txt

# 手动制作镜像
python3 build/ohos/images/mkimage/mkextimage.py ...
```

**参考**: `//build/ohos/images/BUILD.gn:431`

### 问题 12: HAP 签名失败

**症状**:
```
Error: Sign hap failed
```

**原因**: 密钥或证书问题

**解决方案**:
```bash
# 检查密钥配置
ls developtools/hapsigner/dist/OpenHarmony.p12

# 生成调试密钥
# 参考 developtools/hapsigner 文档
```

**参考**: `//build/ohos_var.gni:392-404`

## 性能问题

### 问题 13: 构建速度慢

**原因**: 未启用优化选项

**解决方案**:
```bash
# 启用 ccache
./build.sh --product-name xxx --ccache

# 启用快速重建
./build.sh --product-name xxx --fast-rebuild

# 指定并行度
./build.sh --product-name xxx --jobs 16
```

### 问题 14: 增量构建失效

**症状**: 每次构建都重新编译所有文件

**原因**: 时间戳或依赖关系问题

**解决方案**:
```bash
# 检查文件时间戳
find . -type f -newer reference_file

# 使用 ninja 的 -d explain 查看原因
ninja -C out/{device} -d explain
```

## 调试技巧

### 查看详细日志

```bash
# 设置日志级别
./build.sh --product-name xxx --log-level debug

# 保留 Ninja 日志
ninja -C out/{device} -v
```

### 分析构建图

```bash
# 生成构建图
ninja -C out/{device} -t graph > build.dot
dot -Tpng build.dot -o build.png

# 查看依赖路径
gn path out/{device} //from:target //to:target
```

### 检查目标详情

```bash
# 查看目标配置
gn desc out/{device} //path/to:target

# 查看目标依赖
gn desc out/{device} //path/to:target deps --tree
```

## 常见错误代码

| 错误代码 | 含义 | 解决方案 |
|---------|------|---------|
| 0001 | 通用错误 | 查看详细日志 |
| 0002 | 参数错误 | 检查命令行参数 |
| 0003 | 配置错误 | 检查配置文件 |
| 0004 | 编译错误 | 检查源码和编译选项 |
| 0005 | 链接错误 | 检查库依赖 |
| 3001 | GN 错误 | 检查 GN 配置 |
| 3004 | Ninja 错误 | 检查构建依赖 |

**参考**: `//build/hb/containers/arg.py`

## 获取帮助

### 查看帮助信息

```bash
# build.sh 帮助
./build.sh --help

# hb 帮助
hb --help
hb build --help
```

### 相关文档

- [构建系统详解](04_Build_System.md)
- [GN Targets 梳理](05_GN_Targets.md)
- [官方文档](https://gitee.com/openharmony/docs)

### 社区支持

- [OpenHarmony Issues](https://gitee.com/openharmony/build/issues)
- [OpenHarmony 论坛](https://forums.openharmony.cn)

---

*文档生成时间: 2025-02-06*

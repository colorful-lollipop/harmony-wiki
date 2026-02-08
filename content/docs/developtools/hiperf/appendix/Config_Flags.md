# 附录：配置 Flags

## 目的

本文档汇总 hiperf 的所有编译时配置 Flags。

## 定义位置

**文件**: `hiperf.gni`

## Flags 列表

### 构建目标

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hiperf_target_host` | bool | false | 编译 Host 端工具 |
| `hiperf_target_static` | bool | false | 静态链接 |
| `hiperf_independent_compilation` | bool | true | 独立编译模式 |

### 调试与测试

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hiperf_debug` | bool | true | 启用调试日志 |
| `hiperf_check_time` | bool | false | 启用时间检查 |
| `hiperf_test_coverage` | bool | false | 启用测试覆盖率 |
| `hiperf_test_fuzz` | bool | true | 启用 Fuzz 测试 |
| `hiperf_sanitize` | bool | false | 启用 Sanitizer |
| `hiperf_code_analyze` | bool | false | 启用代码分析 |

### 功能特性

| Flag | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `hiperf_use_libunwind` | bool | false | 使用 libunwind |
| `hiperf_use_libunwinder` | bool | true | 使用 libunwinder |
| `hiperf_use_syspara` | bool | true | 使用系统参数 |
| `bundle_framework_enable` | bool | false | 启用 Bundle 框架支持 |
| `ability_base_enable` | bool | false | 启用 Ability Base 支持 |
| `hiperf_sandbox_log_path_mapping` | bool | false | 启用沙箱路径映射 |
| `hiperf_feature_support_usr_symlink` | bool | false | 启用 /usr 符号链接 |

## 使用示例

```bash
# 编译 Host 端工具
--gn-args "hiperf_target_host=true"

# 静态链接
--gn-args "hiperf_target_static=true"

# 关闭调试
--gn-args "hiperf_debug=false"

# 启用代码分析
--gn-args "hiperf_code_analyze=true"

# 组合使用
--gn-args "hiperf_target_host=true hiperf_debug=false"
```

## 自动检测

以下 Flags 会根据系统配置自动设置：

```gni
# 如果系统包含 bundle_framework 部件
bundle_framework_enable = true

# 如果系统包含 ability_base 部件
ability_base_enable = true
```

## 相关跳转

- [GN Targets](../06_GN_Targets.md) - 构建目标
- [编译产物](../07_Build_Artifacts.md) - 产物说明

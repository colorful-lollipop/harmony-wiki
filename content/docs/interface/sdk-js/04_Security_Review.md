# 安全评审

## 概述

本章节对 `interface/sdk-js` 仓库进行安全评审。

**评审范围**:
- API 声明文件安全
- 构建工具安全
- GN 构建配置安全
- 输入验证与文件操作

**排除范围**:
- 测试代码 (`test/`, `*_test.*`)
- 第三方依赖库（仅评审源码）

## 项目安全特性

### 1. 声明式仓库特性

**发现**: 本仓库为**纯声明式仓库**，不包含运行时逻辑

**影响**:
- 无直接代码执行风险
- 安全风险主要存在于工具链
- API 声明需正确标注权限 (`@syscap`)

**证据**: `api/` 目录全部为 `.d.ts` TypeScript 声明文件

## 攻击面分析

### 1. 工具输入源

| 输入类型 | 来源 | 风险等级 |
|---------|------|---------|
| `.d.ts` 文件 | `api/`, `kits/`, `arkts/` | 低 |
| Python 脚本 | `*.py` | 中 |
| JavaScript 脚本 | `*.js` | 中 |
| 配置文件 | `remove_list.json`, `config.json` | 低 |

### 2. 文件操作

| 操作 | 工具 | 潜在风险 |
|------|------|---------|
| 文件复制 | `process_internal.py` | 路径遍历 |
| 文件删除 | `delete_systemapi_plugin/` | 路径遍历 |
| 文件解析 | `dts_parser/` | 代码注入 |
| 命令执行 | `BUILD.gn` actions | 命令注入 |

## 风险清单

### 高风险项

#### R1: Python 脚本路径遍历风险

**发现**: 部分 Python 脚本使用用户可控路径参数

**证据**: `process_internal.py` 接收 `--input`, `--output`, `--remove` 参数

**触发场景**:
```bash
python process_internal.py \
  --input "../../../sensitive/path" \
  --output "/tmp/output"
```

**影响**: 可读取/写入非授权目录

**修复建议**:
```python
# 1. 路径白名单验证
ALLOWED_BASE = "/path/to/sdk-js"
if not path.startswith(ALLOWED_BASE):
    raise SecurityError("Path traversal detected")

# 2. 使用 realpath 规范化路径
resolved_path = os.path.realpath(user_path)
```

**证据**: `BUILD.gn:103-118` (参数传递)

#### R2: JavaScript 脚本代码注入风险

**发现**: `dts_parser` 使用 `eval` 或动态代码执行

**证据**: `dts_parser/src/coreImpl/parser/parser.ts` (需进一步确认)

**触发场景**: 解析恶意构造的 `.d.ts` 文件

**影响**: 可执行注入的恶意代码

**修复建议**:
- 使用 AST 解析替代 `eval`
- 沙箱化解析环境
- 输入过滤敏感字符

### 中风险项

#### R3: GN 构建脚本参数注入

**发现**: `BUILD.gn` 中的 `action` 和 `action_with_pydeps` 接收外部参数

**证据**: `BUILD.gn:42-61` (ohos_base_split 参数)

```gn
args = [
  "--root-build-dir", rebase_path("//", root_build_dir),
  "--node-js", rebase_path(nodejs, root_build_dir),
  "--output-interface-sdk", rebase_path(interface_sdk_path, root_build_dir),
]
```

**影响**: 恶意构建配置可注入参数

**修复建议**:
- 参数白名单验证
- 使用 `rebase_path` 规范化路径
- 避免直接传递用户输入

#### R4: API 权限标注不准确

**发现**: 部分 API 可能缺少 `@syscap` 权限标注

**证据**: 需代码审计 `api/@ohos.*.d.ts` 文件

**影响**:
- 应用可能请求超出需求的权限
- 权限校验失效

**修复建议**:
- 使用 `api_check_plugin` 定期扫描
- 完善 JSDoc 规范检查

### 低风险项

#### R5: OAT.xml 配置过宽

**发现**: OAT 配置文件可能允许过多 License 类型

**证据**: `OAT.xml:32-47` (policyList 定义)

**修复建议**: 审查 License 策略，限制为 Apache 2.0

#### R6: Node.js 依赖版本过旧

**发现**: 工具使用 Node.js 14.x

**影响**: 可能存在已知安全漏洞

**修复建议**: 评估升级到 Node.js 18+

**证据**: `build-tools/api_check_plugin/README_zh.md:213`

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                     外部输入边界                                  │
├─────────────────────────────────────────────────────────────────┤
│  .d.ts 文件 (api/, kits/, arkts/)                                │
│  Python/JS 脚本 (build-tools/)                                   │
│  配置文件 (remove_list.json, config.json)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     构建处理边界                                  │
├─────────────────────────────────────────────────────────────────┤
│  GN 构建 (BUILD.gn)                                              │
│  Python 脚本处理 (process_*.py)                                   │
│  JavaScript 工具 (dts_parser, api_check_plugin)                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     输出边界                                      │
├─────────────────────────────────────────────────────────────────┤
│  SDK 产物 (.d.ts 文件)                                           │
│  中间产物 (out/sdk/obj/interface/sdk-js/)                        │
└─────────────────────────────────────────────────────────────────┘
```

## 最佳实践

### 1. 输入验证

```python
# 工具应实现输入验证
def validate_input_path(path: str) -> str:
    # 路径白名单
    ALLOWED_PREFIXES = [
        "/workspace/sdk-js/api",
        "/workspace/sdk-js/kits",
    ]

    real_path = os.path.realpath(path)
    for prefix in ALLOWED_PREFIXES:
        if real_path.startswith(prefix):
            return real_path

    raise ValueError(f"Invalid input path: {path}")
```

### 2. 路径安全

```python
import os

def safe_path_join(base: str, *parts: str) -> str:
    """安全路径拼接，防止遍历攻击"""
    full_path = os.path.join(base, *parts)
    real_path = os.path.realpath(full_path)

    # 确保结果在允许范围内
    if not real_path.startswith(os.path.realpath(base)):
        raise SecurityError("Path traversal detected")

    return full_path
```

### 3. API 权限标注

```typescript
/**
 * 敏感 API 必须标注完整权限
 * @syscap SystemCapability.Security.Core
 * @since 9
 */
export function sensitiveOperation(): void;

/**
 * API 变更需更新权限标注
 * @permission ohos.permission.XXX
 * @syscap SystemCapability.XXX
 * @since 10
 */
export function newApi(): void;
```

## 安全测试建议

| 测试项 | 方法 | 频率 |
|--------|------|------|
| 路径遍历 | 构造恶意路径参数 | 每次工具更新 |
| 代码注入 | 构造恶意 d.ts 文件 | 每次工具更新 |
| 权限校验 | 扫描所有 API 标注 | 每月 |
| 依赖扫描 | npm audit, pip audit | 每周 |

## 相关文档

- [构建工具](02_Build_Tools.md)
- [GN 配置](03_Build_Configuration.md)
- [API 声明文件](01_API_Declarations.md)

## 评审结论

| 类别 | 风险等级 | 说明 |
|------|---------|------|
| 工具安全 | 中 | Python/JS 脚本需加强输入验证 |
| 声明安全 | 低 | 需完善权限标注检查 |
| 构建安全 | 中 | GN 参数传递需验证 |
| 依赖安全 | 低 | 建议升级 Node.js 版本 |

**总体评估**: 本仓库为声明式仓库，安全风险可控。主要风险点在于工具链的输入验证，建议实施本章节中的安全最佳实践。

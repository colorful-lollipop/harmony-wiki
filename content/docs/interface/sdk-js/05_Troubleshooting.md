# 常见问题

## 概述

本章节记录构建、运行、调试过程中的常见问题及解决方案。

## 构建问题

### Q1: GN 构建报错 "toolchain not found"

**问题描述**:
```
ERROR: Can't find toolchain for label //build/toolchain:ohos_clang
```

**解决方案**:
1. 检查 `build/ohos.gni` 是否正确导入
2. 确认 toolchain 定义存在
3. 检查 `OHOS_SDK_ROOT` 环境变量

**证据**: `BUILD.gn:15` (导入 ohos.gni)

### Q2: Python 脚本路径错误

**问题描述**:
```
FileNotFoundError: [Errno 2] No such file or directory: 'process_internal.py'
```

**解决方案**:
1. 确保在正确目录运行
2. 使用绝对路径
3. 检查文件权限

**证据**: `BUILD.gn:103` (process_internal.py 路径)

### Q3: npm 依赖安装失败

**问题描述**:
```
npm ERR! code ELIFECYCLE
npm ERR! errno 1
```

**解决方案**:
```bash
# 1. 清除 npm 缓存
npm cache clean --force

# 2. 删除 node_modules
rm -rf node_modules package-lock.json

# 3. 重新安装
npm install

# 4. 检查 Node.js 版本 (需 14.x)
node --version
```

**证据**: `build-tools/api_check_plugin/README_zh.md:213`

### Q4: 权限检查失败

**问题描述**:
```
Permission check failed: missing ohos.permission.XXX
```

**解决方案**:
1. 检查 `api_check_plugin/config/config.json` 配置
2. 更新权限配置文件
3. 运行 `npm run test` 重新检查

**证据**: `build-tools/api_check_plugin/README_zh.md:18-26`

## 工具问题

### Q5: d.ts 解析结果为空

**问题描述**:
```bash
node ./dts_parser/src/main.ts -N collect -C ./api
# 输出: {}
```

**解决方案**:
1. 检查输入路径是否正确
2. 确认目录包含 `.d.ts` 文件
3. 检查文件权限

```bash
# 调试步骤
ls -la api/@ohos.*.d.ts | head -5
cat api/@ohos.ability.ability.d.ts | head -20
```

### Q6: API 差异比较报错

**问题描述**:
```
Error: Invalid SDK path
```

**解决方案**:
1. 确认新旧 SDK 路径存在
2. 检查目录结构是否正确
3. 使用绝对路径

```bash
node ./diff_api/src/main.ts -N diff \
  --old "/path/to/sdk_v1/api" \
  --new "/path/to/sdk_v2/api" \
  --output ./output
```

### Q7: JSDoc 格式检查不通过

**问题描述**:
```
JSDoc check failed: missing @syscap
```

**解决方案**:
1. 检查 `@syscap` 标签是否正确
2. 确认标签顺序符合规范
3. 参考 `api_check_plugin/config/code_style_rule.json`

```typescript
/**
 * 正确示例
 * @syscap SystemCapability.Ability.AbilityRuntime.AbilityCore
 * @since 9
 */
export function api(): void;
```

## GN 构建调试

### Q8: 查看完整构建日志

**解决方案**:
```bash
# 启用详细日志
gn gen out/default --verbose

# 查看特定 target
ninja -C out/default -v ohos_ets_api
```

### Q9: 调试特定 target

**解决方案**:
```bash
# 1. 查看 target 依赖
gn deps out/default //interface/sdk-js:ohos_ets_api

# 2. 单独构建 target
ninja -C out/default //interface/sdk-js:ohos_ets_api

# 3. 查看构建产物
ls -la out/default/obj/interface/sdk-js/
```

### Q10: 自定义构建参数

**解决方案**:
```bash
# 查看可用参数
gn args out/default --list | grep sdk

# 编辑参数
gn args out/default
```

**常用参数**:
```python
sdk_build_public = true  # 公开构建
sdk_build_check_level = "strict"  # 检查级别
```

## 文件处理问题

### Q11: remove_list.json 规则不生效

**问题描述**: 配置的过滤规则未生效

**解决方案**:
1. 检查 JSON 格式正确性
2. 确认 target 名称匹配
3. 检查 `ispublic` 参数

```json
{
  "ets_component": {
    "sdk_build_public_remove": [
      "ability_component.d.ts"
    ]
  }
}
```

**证据**: `remove_list.json:8-25`

### Q12: 条件编译未正确处理

**问题描述**: 动态/静态 SDK 包含相同代码

**解决方案**:
1. 确认条件编译标签正确
2. 检查 `arkui_transformer.py` 执行
3. 验证输出目录结构

```typescript
/*** if arkts dynamic */
// 动态 SDK 代码
/*** endif */

/*** if arkts static */
// 静态 SDK 代码
/*** endif */
```

**证据**: `api/@ohos.ability.ability.d.ts:17-25`

## 环境问题

### Q13: Node.js 版本不兼容

**问题描述**:
```
Error: The engine "node" is incompatible
```

**解决方案**:
```bash
# 使用 nvm 切换版本
nvm install 14
nvm use 14

# 或使用 prebuilt 工具链
export PATH="/prebuilts/build-tools/common/nodejs/current/bin:$PATH"
```

**证据**: `BUILD.gn:446-455`

### Q14: 路径过长问题 (Windows)

**问题描述**:
```
Error: ENAMETOOLONG: name too long
```

**解决方案**:
1. 使用短路径
2. 启用长路径支持
3. 使用 WSL

```bash
# 启用 Windows 长路径
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f
```

## 性能问题

### Q15: 构建速度慢

**优化建议**:
```bash
# 1. 使用 ccache
export CCACHE_DIR=/path/to/ccache
ccache -M 100G

# 2. 并行构建
ninja -C out/default -j8

# 3. 增量构建
ninja -C out/default
```

### Q16: 工具内存占用高

**解决方案**:
1. 分批处理文件
2. 增加 Node.js 内存限制

```bash
# 增加内存限制
node --max-old-space-size=4096 ./dts_parser/src/main.ts
```

## 相关文档

- [构建工具](02_Build_Tools.md)
- [GN 配置](03_Build_Configuration.md)
- [安全评审](04_Security_Review.md)

## 反馈渠道

如遇到本文档未收录的问题：

1. 搜索 [Gitee Issues](https://gitee.com/openharmony/interface_sdk-js/issues)
2. 提交新 Issue
3. 或联系 SDK 团队

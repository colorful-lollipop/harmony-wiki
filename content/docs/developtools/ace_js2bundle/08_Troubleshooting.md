# 08 - 常见问题与故障排查

## 构建问题

### Q1: npm install 失败

**现象**:
```
npm ERR! code E404
npm ERR! 404 Not Found
```

**原因**: 网络问题或 registry 配置错误

**解决**:
```bash
# 设置国内镜像
npm config set registry http://registry.npm.taobao.org
npm config set strict-ssl false
npm cache clean -f
npm install
```

**代码证据**: `README_zh.md:35-39`

### Q2: 编译时提示 "missing app.js"

**现象**:
```
ERROR: missing app.js
```

**原因**: 项目目录结构不正确，缺少 `app.js`

**解决**:
确保项目结构正确：
```
project/
├── pages/
│   └── index/
│       ├── index.hml
│       ├── index.css
│       └── index.js
├── app.js          # 必须存在
└── manifest.json
```

**代码证据**: `main.product.js:88-91`

### Q3: 编译时提示 "cannot both have hml && visual"

**现象**:
```
ERROR: pages/index cannot both have hml && visual
```

**原因**: 同一页面同时存在 `.hml` 和 `.visual` 文件

**解决**: 删除其中一个文件，只保留一种格式

**代码证据**: `main.product.js:125-127`

### Q4: 编译结果为空或文件未生成

**现象**: 编译成功但输出目录为空

**原因**:
1. `aceModuleBuild` 环境变量未设置
2. 输出路径权限不足

**解决**:
```bash
# Linux/Mac
export aceModuleRoot=/path/to/project
export aceModuleBuild=/path/to/output

# Windows
set aceModuleRoot=C:\path\to\project
set aceModuleBuild=C:\path\to\output
```

**代码证据**: `webpack.rich.config.js:210-213`

### Q5: 内存不足 (OOM)

**现象**:
```
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed
```

**解决**:
```bash
# 增加 Node.js 内存限制
node --max-old-space-size=4096 ./node_modules/webpack/bin/webpack.js --config webpack.rich.config.js
```

## 运行时问题

### Q6: 页面显示空白

**排查步骤**:
1. 检查 `manifest.json` 中的 `pages` 配置是否正确
2. 检查页面文件路径是否正确
3. 检查编译输出目录是否有对应文件

**代码证据**: `resource-plugin.js:207-241`

### Q7: 样式不生效

**排查步骤**:
1. 检查 CSS 文件路径是否正确
2. 检查选择器是否正确
3. 检查是否有语法错误

**常见错误**:
```css
/* 错误：使用了不支持的属性 */
.unsupported {
  display: flex;  /* 可能不支持 */
}
```

### Q8: 事件不触发

**排查步骤**:
1. 检查事件绑定语法
2. 检查事件处理函数是否存在
3. 检查方法名是否正确

**正确示例**:
```hml
<div onclick="handleClick"></div>
```

```js
export default {
  handleClick() {
    console.log('clicked');
  }
}
```

## 调试技巧

### 启用详细日志

```bash
# 设置日志级别
export logLevel=3  # 0-3，数字越大越详细
```

**代码证据**: `webpack.rich.config.js:208`

### 查看编译错误日志

编译错误会写入 `build/compile_error.log`:

```bash
cat build/compile_error.log
```

**代码证据**: `compile-plugin.js:217-223`

### 禁用代码压缩

```bash
# 设置构建模式为 debug
npm run rich -- --env buildMode=debug
```

**代码证据**: `webpack.rich.config.js:384-417`

### 保留 source map

```javascript
// webpack.rich.config.js
devtool: 'source-map'  // 使用完整 source map
```

## 性能优化

### 编译速度慢

**优化建议**:
1. 启用缓存
2. 排除不需要的目录
3. 使用增量编译

**配置**:
```javascript
// webpack.rich.config.js
config.cache = {
  type: 'filesystem',
  cacheDirectory: path.resolve(__dirname, '.webpack_cache')
};

config.watchOptions = {
  ignored: ['**/node_modules', '**/oh_modules']
};
```

**代码证据**: `webpack.rich.config.js:167-175`

### 产物体积大

**优化建议**:
1. 启用代码分割
2. 启用 Tree Shaking
3. 压缩代码

**配置**:
```javascript
// 已在默认配置中启用
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendors: { ... },
      commons: { ... }
    }
  }
}
```

**代码证据**: `webpack.rich.config.js:358-378`

## 平台相关问题

### Windows 路径问题

**现象**: 路径分隔符错误

**解决**: 使用 `path.join` 或 `path.resolve` 处理路径

```javascript
const path = require('path');
const filePath = path.join('src', 'pages', 'index.js');
```

### Mac/Linux 权限问题

**现象**: `EACCES: permission denied`

**解决**:
```bash
# 修改目录权限
chmod -R 755 /path/to/project

# 或使用 sudo（不推荐）
sudo npm run rich
```

## 常见错误码

| 错误信息 | 原因 | 解决 |
|----------|------|------|
| `ERROR: missing pages` | manifest.json 缺少 pages 字段 | 添加 pages 配置 |
| `ERROR: Invalid route` | 页面路径错误 | 检查 pages 配置和文件路径 |
| `ERROR: the manifest.json file format is invalid` | JSON 格式错误 | 检查 JSON 语法 |
| `Failed to convert file to bin` | qjsc 编译失败 | 检查 JS 语法 |
| `ERROR find build fail` | 缺少 Ark 编译器 | 安装 Ark 编译器 |

**代码证据**: `main.product.js:66-68`, `resource-plugin.js:214-216`

## 日志解读

### 编译成功日志

```
COMPILE RESULT:SUCCESS {}
```

### 编译失败日志

```
COMPILE RESULT:FAIL {"ERROR":1}
ERROR: missing app.js
```

### 带警告的编译

```
COMPILE RESULT:SUCCESS {"WARN":2}
```

**代码证据**: `compile-plugin.js:231-268`

## 获取帮助

### 查看详细错误

```bash
# 启用错误显示
export error=true
export warning=true
export note=true
```

**代码证据**: `webpack.rich.config.js:204-206`

### 检查环境变量

```bash
# 查看所有 ace 相关环境变量
env | grep ace
```

### 验证项目结构

```bash
# 检查必要文件是否存在
ls -la app.js manifest.json
ls -la pages/
```

## 相关文档

- [项目概览](./01_Overview.md)
- [目录结构](./02_Directory_Structure.md)
- [架构说明](./03_Architecture.md)

# 07 - 安全风险评估

## 评估说明

本项目为纯 Node.js 构建工具，安全风险主要集中在：
1. 文件系统操作（路径遍历、任意文件读写）
2. 代码执行（子进程调用、动态代码）
3. 输入验证（用户输入处理）
4. 依赖安全（第三方包漏洞）

## 攻击面分析

```
┌─────────────────────────────────────────────────────────────────┐
│                        攻击面分析                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  外部输入 ───────────────────────────────────────────────────┐  │
│       │                                                      │  │
│       ├──► 文件路径输入                                       │  │
│       │    ├── manifest.json                                  │  │
│       │    ├── module.json                                    │  │
│       │    ├── 项目源码文件                                    │  │
│       │    └── 资源文件                                       │  │
│       │                                                      │  │
│       ├──► 环境变量                                           │  │
│       │    ├── aceModuleRoot                                  │  │
│       │    ├── aceModuleBuild                                 │  │
│       │    └── 其他配置                                       │  │
│       │                                                      │  │
│       └──► 命令行参数                                         │  │
│            └── webpack env                                    │  │
│                                                              │  │
│  敏感操作 ───────────────────────────────────────────────────┤  │
│       │                                                      │  │
│       ├──► 文件系统操作                                       │  │
│       │    ├── 读取任意文件                                    │  │
│       │    ├── 写入任意文件                                    │  │
│       │    ├── 删除目录                                        │  │
│       │    └── 遍历目录                                        │  │
│       │                                                      │  │
│       ├──► 子进程执行                                         │  │
│       │    ├── 调用 ts2abc/es2abc                             │  │
│       │    ├── 调用 qjsc                                      │  │
│       │    └── 调用 node                                      │  │
│       │                                                      │  │
│       └──► 网络操作                                           │  │
│            └── npm install (开发时)                            │  │
│                                                              │  │
└──────────────────────────────────────────────────────────────┘  │
```

## 风险点详细分析

### 风险 1: 路径遍历攻击

**风险等级**: 中

**证据**:
```javascript
// main.product.js:86-91
const appJSPath = path.resolve(projectPath, 'app.js');
// resource-plugin.js:119-128
input = input_;
output = output_;
manifestFilePath = manifestFilePath_;
```

**问题**: 用户通过环境变量控制 `projectPath` 和 `buildPath`，如果未经验证，可能导致：
- 读取系统敏感文件
- 写入到非预期目录

**触发路径**:
```
用户设置 aceModuleRoot=/etc ──► main.product.js:209 ──► 读取 /etc/app.js
```

**影响**: 信息泄露、文件覆盖

**修复建议**:
```javascript
// 添加路径验证
function validateProjectPath(projectPath) {
  const resolved = path.resolve(projectPath);
  const cwd = process.cwd();
  if (!resolved.startsWith(cwd)) {
    throw new Error('Invalid project path: must be under current directory');
  }
  return resolved;
}
```

### 风险 2: 任意文件删除

**风险等级**: 高

**证据**:
```javascript
// main.product.js:37-51
function deleteFolderRecursive(url) {
  let files = [];
  if (fs.existsSync(url)) {
    files = fs.readdirSync(url);
    files.forEach(function(file) {
      const curPath = path.join(url, file);
      if (fs.statSync(curPath).isDirectory()) {
        deleteFolderRecursive(curPath);
      } else {
        fs.unlinkSync(curPath);
      }
    });
    fs.rmdir(url, function(err) {});
  }
}

// webpack.rich.config.js:298
deleteFolderRecursive(process.env.buildPath);
```

**问题**: `buildPath` 由环境变量控制，如果设置为系统目录，可能导致数据丢失。

**触发路径**:
```
用户设置 aceModuleBuild=/home/user/documents ──► 删除整个 documents 目录
```

**影响**: 数据丢失

**修复建议**:
```javascript
// 添加安全目录检查
function safeDeleteFolder(url) {
  const resolved = path.resolve(url);
  const homeDir = require('os').homedir();
  
  // 禁止删除家目录、根目录等敏感路径
  if (resolved === homeDir || resolved === '/') {
    throw new Error('Cannot delete protected directory: ' + url);
  }
  
  // 添加确认提示
  console.warn('About to delete: ' + resolved);
  // ... 删除逻辑
}
```

### 风险 3: 子进程命令注入

**风险等级**: 中

**证据**:
```javascript
// genAbc-plugin.js:586-592
let genAbcCmd = `${initAbcEnv().join(' ')} "@${filesInfoPath}" --file-threads "${fileThreads}"`;
childProcess.exec(genAbcCmd);

// genBin-plugin.js:94
const cmd = `"${qjsc}" -o "${outputPath}" -N buf -c "${inputPath}"`
process.execSync(cmd)
```

**问题**: 如果 `filesInfoPath` 或 `inputPath` 包含特殊字符，可能导致命令注入。

**触发路径**:
```
恶意文件名: "file; rm -rf /;.js" ──► 注入 shell 命令
```

**影响**: 任意代码执行

**修复建议**:
```javascript
// 使用 execFile 替代 exec，避免 shell 解释
const { execFile } = require('child_process');
execFile(es2abc, ['@' + filesInfoPath, '--file-threads', fileThreads]);
```

### 风险 4: JSON 解析拒绝服务

**风险等级**: 低

**证据**:
```javascript
// main.product.js:58-70
function readManifest(manifestFilePath) {
  let manifest = {};
  try {
    if (fs.existsSync(manifestFilePath)) {
      const jsonString = fs.readFileSync(manifestFilePath).toString();
      manifest = JSON.parse(jsonString);  // 可能解析大文件导致内存耗尽
    }
  } catch (e) {
    throw Error('ERROR: the manifest.json file format is invalid.').message;
  }
  return manifest;
}
```

**问题**: 如果 `manifest.json` 是精心构造的大文件（如 1GB 的 JSON），可能导致内存耗尽。

**影响**: 拒绝服务

**修复建议**:
```javascript
// 添加文件大小限制
const MAX_MANIFEST_SIZE = 10 * 1024 * 1024; // 10MB

function readManifest(manifestFilePath) {
  const stats = fs.statSync(manifestFilePath);
  if (stats.size > MAX_MANIFEST_SIZE) {
    throw new Error('Manifest file too large');
  }
  // ... 原有逻辑
}
```

### 风险 5: 原型链污染

**风险等级**: 低

**证据**:
```javascript
// resource-plugin.js:207-241
function addPageEntryObj() {
  // ...
  pages.forEach((element) => {
    // ...
    entryObj['./' + sourcePath] = path.resolve(projectPath, './' + sourcePath + '.hml?entry');
  });
  // ...
}
```

**问题**: 如果 `sourcePath` 包含 `__proto__` 或 `constructor`，可能导致原型链污染。

**触发路径**:
```json
{
  "pages": ["__proto__/polluted"]
}
```

**影响**: 对象属性污染，可能导致逻辑绕过

**修复建议**:
```javascript
// 使用 Map 替代普通对象，或使用 Object.create(null)
const entryObj = Object.create(null);
// 或
const entryObj = new Map();
```

### 风险 6: 敏感信息泄露

**风险等级**: 低

**证据**:
```javascript
// webpack.rich.config.js:180-190
output: {
  devtoolModuleFilenameTemplate: (info) => {
    const newInfo = info.absoluteResourcePath.replace(process.env.projectRootPath + path.sep, '')
      .replace(process.env.projectRootPath + path.sep, '')
      .replace(path.join(__dirname, path.sep), '');
    return newInfo;
  }
}
```

**问题**: Source map 可能包含绝对路径，泄露系统目录结构。

**影响**: 信息泄露

**修复建议**:
```javascript
// 在 release 模式下禁用 source map 或清理路径
if (env.buildMode === 'release') {
  config.devtool = false;
}
```

### 风险 7: 依赖包漏洞

**风险等级**: 中

**证据**:
```json
// ace-loader/package.json:34-58
"dependencies": {
  "@babel/cli": "7.20.7",
  "@babel/core": "7.20.12",
  "webpack": "5.72.1",
  "shelljs": "0.8.5",
  // ...
}
```

**问题**: 
- `shelljs` 存在命令注入风险
- 旧版本 webpack 可能存在已知漏洞
- 其他依赖可能存在未修复的漏洞

**影响**: 取决于具体漏洞，可能导致任意代码执行

**修复建议**:
1. 定期更新依赖包
2. 使用 `npm audit` 检查漏洞
3. 使用 `package-lock.json` 锁定版本
4. 考虑使用 `yarn` 的 `resolutions` 强制升级有漏洞的间接依赖

### 风险 8: 正则表达式拒绝服务 (ReDoS)

**风险等级**: 低

**证据**:
```javascript
// lite-transform-template.js:498-501
function cacheI18nTranslation(func) {
  if (!REGXP_LANGUAGE.test(func)) {
    return func;
  }
  const i18nExpressions = func.match(REGXP_LANGUAGE_KEY);
  // ...
}
```

**问题**: 如果正则表达式设计不当，可能导致 ReDoS。

**影响**: CPU 耗尽，拒绝服务

**修复建议**:
```javascript
// 添加正则执行超时
const { performance } = require('perf_hooks');

function safeRegexTest(regex, input, timeoutMs = 1000) {
  const start = performance.now();
  const result = regex.test(input);
  if (performance.now() - start > timeoutMs) {
    throw new Error('Regex timeout');
  }
  return result;
}
```

## 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                        信任边界                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  不可信输入                    可信处理                    输出  │
│  ───────────                  ─────────                  ────   │
│                                                                  │
│  用户源码 ──► ace_js2bundle ──► 编译产物                        │
│  (不可信)      (可信工具)       (可信输出)                       │
│                                                                  │
│  信任边界 1: 源码解析                                            │
│  信任边界 2: 编译输出                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 安全检查清单

| 检查项 | 状态 | 说明 |
|--------|------|------|
| 路径遍历防护 | ❌ | 需要添加路径验证 |
| 文件删除保护 | ❌ | 需要添加安全目录检查 |
| 命令注入防护 | ⚠️ | 部分使用 execFile，部分仍需改进 |
| JSON 大小限制 | ❌ | 未限制 |
| 原型链污染防护 | ❌ | 使用普通对象 |
| 敏感信息清理 | ⚠️ | source map 可能泄露路径 |
| 依赖漏洞管理 | ⚠️ | 需要定期更新 |
| ReDoS 防护 | ❌ | 无超时机制 |

## 修复优先级

| 优先级 | 风险 | 建议 |
|--------|------|------|
| P0 | 任意文件删除 | 添加安全目录检查 |
| P1 | 命令注入 | 统一使用 execFile |
| P1 | 路径遍历 | 添加路径验证 |
| P2 | 依赖漏洞 | 定期更新依赖 |
| P2 | JSON DoS | 添加大小限制 |
| P3 | 原型链污染 | 使用 Object.create(null) |
| P3 | ReDoS | 添加正则超时 |

## 相关文档

- [架构说明](./03_Architecture.md)
- [内部 API](./05_Internal_API.md)

# 附录 A - 关键调用链

## 1. 页面编译调用链

### 1.1 富设备页面编译

```
webpack.rich.config.js:290
    └── setConfigs(env)
        ├── process.env.projectPath = env.aceModuleRoot
        ├── process.env.buildPath = env.aceModuleBuild
        └── process.env.DEVICE_LEVEL = 'rich'
    
    └── loadEntryObj()
        ├── main.product.js:80-105
        ├── readManifest()
        │   └── main.product.js:58-70
        └── addPageEntryObj()
            └── main.product.js:113-136
    
    └── ResourcePlugin.apply()
        ├── resource-plugin.js:131-143
        │   └── circularFile() - 复制资源
        ├── resource-plugin.js:147-174
        │   └── addPageEntryObj() - 添加页面入口
        └── resource-plugin.js:175-178
            └── copyManifest() - 复制 manifest
    
    └── loader-gen.js:172
        └── codegenHmlAndCss()
            ├── generateOutput('template')
            │   └── getLoaderString('template')
            ├── generateOutput('style')
            │   └── getLoaderString('style')
            └── generateOutput('script')
                └── getLoaderString('script')
    
    └── ResultStates.apply()
        ├── compile-plugin.js:68-92
        │   └── buildModule.tap() - 收集模块
        ├── compile-plugin.js:94-102
        │   └── copyFindModule() - 复制公共模块
        └── compile-plugin.js:104-132
            └── printResult() - 打印结果
    
    └── GenAbcPlugin.apply()
        ├── genAbc-plugin.js:98-119
        │   └── emit.tap() - 生成临时文件
        └── genAbc-plugin.js:120-129
            └── afterEmit.tap() - 调用 ABC 编译
                └── invokeWorkerToGenAbc()
                    └── processWorkersOfBuildMode()
```

### 1.2 瘦设备页面编译

```
webpack.lite.config.js
    └── 类似富设备流程，但使用 lite-transform-template.js
        └── lite-transform-template.js:138
            └── transformTemplate(value)
                ├── transformNode(ast)
                ├── transformFor(node) - for 指令
                └── transformIf(node) - if 指令
```

### 1.3 卡片编译

```
webpack.rich.config.js:351-356
    └── config.module = cardModule
        └── card-loader.js:33
            └── loader(source)
                ├── findStyleFile() - 查找样式
                ├── parseFragment() - 解析 HML
                └── addJson() - 添加 JSON 配置
```

## 2. ABC 生成调用链

```
GenAbcPlugin.apply()
    └── compiler.hooks.emit.tap()
        └── genAbc-plugin.js:98-119
            └── 遍历 assets
                ├── 包装 JS: forward + content + last
                └── writeFileSync() - 写入临时文件
    
    └── compiler.hooks.afterEmit.tap()
        └── genAbc-plugin.js:120-129
            └── processMultiThreadEntry()
                ├── isTs2Abc() ? invokeWorkerToGenAbc()
                └── isEs2Abc() ? generateAbcByEs2AbcOfBundleMode()
    
    └── invokeWorkerToGenAbc()
        ├── filterIntermediateJsBundleByHashJson() - 哈希过滤
        ├── splitJsBundlesBySize() - 分组
        └── processWorkersOfBuildMode()
            ├── cluster.setupPrimary() - 设置主进程
            ├── cluster.fork() - 创建 Worker
            └── cluster.on('exit') - 处理完成
```

## 3. 资源处理调用链

```
ResourcePlugin.constructor()
    └── 初始化路径配置
        ├── input = projectPath
        ├── output = buildPath
        └── manifestFilePath

ResourcePlugin.apply()
    └── compiler.hooks.beforeCompile.tap()
        └── circularFile(input, output, '')
            ├── 遍历目录
            ├── copyFile() - 复制文件
            └── themeFileBuild() - 构建主题

    └── compiler.hooks.normalModuleFactory.tap()
        └── addPageEntryObj()
            ├── readManifest() - 读取配置
            ├── 遍历 pages
            └── 检查 hml/visual 文件存在性

    └── compiler.hooks.done.tap()
        └── copyManifest()
            └── copyFile() - 复制 manifest.json
```

## 4. 编译结果处理调用链

```
ResultStates.constructor()
    └── 初始化 GLOBAL_COMMON_MODULE_CACHE

ResultStates.apply()
    └── compiler.hooks.compilation.tap()
        └── buildModule.tap()
            └── 收集 common/i18n 路径

    └── compiler.hooks.afterCompile.tap()
        └── copyFindModule()
            ├── circularFile(commonPath, ...)
            ├── circularFile(i18nPath, ...)
            └── addCacheFiles()

    └── compiler.hooks.compilation.tap('CommonAsset')
        └── processAssets.tap()
            └── 注入 GLOBAL_COMMON_MODULE_CACHE

    └── compiler.hooks.compilation.tap('Require')
        └── renderRequire.tap()
            └── 修改 require 逻辑

    └── compiler.hooks.done.tap()
        └── printResult()
            ├── printWarning()
            ├── printError()
            └── console.log('COMPILE RESULT:...')
```

## 5. 主题处理调用链

```
ResourcePlugin.circularFile()
    └── 遇到 theme JSON 文件
        └── themeFileBuild(input, output)
            ├── 读取 theme JSON
            ├── 映射自定义主题到系统主题
            │   ├── CUSTOM_THEME_PROP_GROUPS
            │   └── OHOS_THEME_PROP_GROUPS
            └── 写入转换后的 JSON
```

## 6. Worker 处理调用链

```
main.product.js:215-233
    └── readWorkerFile()
        ├── 读取 aceBuildJson
        ├── 遍历 workers 配置
        └── 返回 workerEntryObj

webpack.rich.config.js:293
    └── readWorkerFile() 返回值传递给 plugins

ResourcePlugin.apply()
    └── normalModuleFactory.tap()
        └── loadWorker()
            ├── 遍历 workers 目录
            └── 添加 worker 入口

GenAbcPlugin.apply()
    └── emit.tap()
        └── checkWorksFile(key, workerFile)
            └── 判断是否包装 JS
```

## 7. 缓存处理调用链

```
main.product.js:249-283
    └── compareCache(cachePath)
        ├── 读取 entry.json
        ├── 检查文件是否存在
        └── deleteFolderRecursive() - 清理缓存

webpack.rich.config.js:304-305
    └── 设置 cache.cacheDirectory

GenAbcPlugin.apply()
    └── afterEmit.tap()
        └── filterIntermediateJsBundleByHashJson()
            ├── 计算文件哈希
            ├── 对比 gen_hash.json
            └── 过滤未变更文件
```

## 8. 完整构建流程调用链

```
npm run rich
    └── webpack --config webpack.rich.config.js
        └── webpack.rich.config.js:290
            └── module.exports = (env) => {
                ├── setConfigs(env)
                ├── compareCache()
                ├── readWorkerFile()
                ├── deleteFolderRecursive(buildPath)
                ├── loadEntryObj()
                ├── 配置 plugins
                │   ├── ResourcePlugin
                │   ├── ResultStates
                │   ├── DefinePlugin
                │   └── GenAbcPlugin
                └── 返回 config
            }
        
        └── webpack(config)
            ├── 初始化 compiler
            ├── 执行编译生命周期
            │   ├── beforeCompile
            │   ├── compile
            │   ├── compilation
            │   ├── make
            │   ├── afterCompile
            │   ├── emit
            │   ├── afterEmit
            │   └── done
            └── 输出结果
```

## 9. 错误处理调用链

```
编译错误
    └── compile-plugin.js:339-370
        └── printError()
            ├── 格式化错误信息
            ├── console.error() - 输出到控制台
            └── writeError() - 写入 compile_error.log

异常捕获
    └── genAbc-plugin.js:279-285
        └── try-catch
            ├── console.debug(red, `ERROR...`)
            ├── process.env.abcCompileSuccess = 'false'
            └── process.exit(FAIL)
```

## 10. 文件操作调用链

```
读取文件
    ├── fs.existsSync(path) - 检查存在
    ├── fs.readFileSync(path) - 读取内容
    └── JSON.parse() - 解析 JSON

写入文件
    ├── mkDir(parent) - 创建目录
    ├── fs.writeFileSync(path, content) - 写入
    └── fs.copyFileSync(src, dest) - 复制

删除文件
    └── deleteFolderRecursive(url)
        ├── fs.readdirSync() - 读取目录
        ├── fs.unlinkSync() - 删除文件
        └── fs.rmdir() - 删除目录
```

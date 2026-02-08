# 附录：关键调用链

## 打包调用链

### HAP 打包完整调用链

```
CompressEntrance.main()
  └─ CommandParser.commandParser(utility, args)
       └─ 解析命令行参数到 Utility 对象
  └─ CompressVerify.commandVerify(utility)
       └─ 验证参数合法性
       └─ isPathValid() 验证路径
  └─ Compressor.compressProcess(utility)
       └─ compressExcute(utility)
            └─ compressHap(utility)
                 ├─ setGenerateBuildHash(utility)
                 ├─ isModuleJSON() 检查 module.json
                 ├─ checkStageHap(utility) 验证 HAP
                 │    └─ HapVerify 相关检查
                 ├─ compressHapModeForModule(utility)
                 │    ├─ parseAtomicServiceSizeLimit() 解析大小限制
                 │    ├─ compressPackinfoIntoHap() 压缩 pack.info
                 │    ├─ compressHapModeMultiple() 压缩多文件
                 │    │    └─ compressFileList()
                 │    │         └─ PackageUtil.parallelCompress()
                 │    │              └─ ParallelScatterZipCreator
                 │    └─ buildHash(utility) 生成构建哈希
                 └─ 返回结果
```

### APP 打包调用链

```
CompressEntrance.main()
  └─ Compressor.compressProcess(utility)
       └─ compressAppMode(utility)
            ├─ parseVerifyInfoForApp() 解析验证信息
            │    └─ 读取所有 HAP 的 HapVerifyInfo
            ├─ HapVerify.checkHapIsValid(hapVerifyInfoList)
            │    ├─ checkAppFieldsAreSame() 检查 bundleName
            │    ├─ checkAppFieldsAreSame() 检查 versionCode
            │    ├─ checkModuleNameIsValid() 检查 moduleName 唯一性
            │    └─ checkApiVersion() 检查 API 版本
            ├─ compressAppModeMultiple() 压缩 APP
            │    ├─ compressPackinfoIntoApp() 压缩 pack.info
            │    └─ compressHapIntoApp() 压缩 HAP 文件
            └─ checkAppAtomicServiceCompressedSizeValid() 检查大小
```

## 拆包调用链

### HAP 拆包调用链

```
UncompressEntrance.main()
  └─ CommandParser.commandParser(utility, args)
  └─ UncompressVerify.commandVerify(utility)
       └─ isPathValid() 验证输入路径
       └─ isPathValid() 验证输出路径
  └─ Uncompress.unpackageProcess(utility)
       └─ unpackageHapMode(utility)
            ├─ unpackageLibsMode() 如果是 libs 模式
            │    └─ unzipFromFile() 解压 libs
            ├─ getRpcidFromHap() 如果是 rpcid 模式
            │    └─ 提取 rpcid.sc
            ├─ unzip() 如果是 unpackApk 模式
            │    └─ unzipFromFile()
            │         └─ dataTransfer()
            │              └─ FileUtils 文件操作
            └─ dataTransferAllFiles() 完整拆包
                 └─ FileUtils.unzip()
```

### APP 解析调用链

```
UncompressEntrance.parseApp(appPath, mode, hapName)
  └─ UncompressVerify.isPathValid() 验证路径
  └─ UncompressVerify.isParseAppModeValid() 验证模式
  └─ Uncompress.uncompressAppByPath(utility)
       ├─ PARSE_MODE_HAPLIST:
       │    └─ uncompress(deviceType, srcPath, PACK_INFO)
       │         └─ unZipHapFileFromHapFile()
       │              ├─ FileUtils.getFileStringFromZip(HARMONY_PROFILE)
       │              ├─ getResourceDataFromHap()
       │              └─ FileUtils.getFileStringFromZip(PACK_INFO)
       │         └─ uncompressPackInfo()
       │              └─ JsonUtil.parseHapList()
       ├─ PARSE_MODE_HAPINFO:
       │    └─ uncompressHapAndHspFromAppPath()
       │         ├─ ZipFile 打开 APP
       │         ├─ 遍历 entries 找指定 HAP
       │         └─ uncompressHapByStream()
       │              └─ uncompressModuleHapByInput()
       │                   └─ JsonUtil 解析
       └─ PARSE_MODE_ALL:
            └─ uncompressAllAppByPath()
                 ├─ 遍历所有 entries
                 ├─ 解析 pack.info
                 └─ 解析每个 HAP/HSP
```

## 验证调用链

### HAP 验证调用链

```
Compressor.compressHap() / compressAppMode()
  └─ HapVerify.checkHapIsValid(hapVerifyInfoList)
       ├─ checkAppFieldsAreSame(hapVerifyInfos, "bundleName")
       │    └─ 比较所有 HAP 的 bundleName
       ├─ checkAppFieldsAreSame(hapVerifyInfos, "versionCode")
       ├─ checkAppFieldsAreSame(hapVerifyInfos, "bundleType")
       ├─ checkAppFieldsAreSame(hapVerifyInfos, "debug")
       ├─ checkModuleNameIsValid(hapVerifyInfos)
       │    └─ 检查 moduleName 是否重复
       ├─ checkApiVersion(hapVerifyInfos)
       │    ├─ 检查 minCompatibleVersionCode
       │    ├─ 检查 targetAPIVersion
       │    └─ 检查 minAPIVersion
       └─ checkInstallationFreeIsValid(hapVerifyInfos)
```

### 参数验证调用链

```
CompressEntrance.main()
  └─ CompressVerify.commandVerify(utility)
       ├─ verifyMode(utility) 验证模式
       ├─ verifyJsonPath(utility) 验证 JSON 路径
       ├─ verifyOutPath(utility) 验证输出路径
       ├─ verifyHapPath(utility) 验证 HAP 路径
       ├─ verifyAppPath(utility) 验证 APP 路径
       ├─ verifyForceRewrite(utility) 验证 force 参数
       ├─ verifyCompressLevel(utility) 验证压缩级别
       ├─ verifyEntryCardPath(utility) 验证卡片路径
       ├─ verifySignaturePath(utility) 验证签名路径
       ├─ verifyCertificatePath(utility) 验证证书路径
       └─ verifyHapVerifyInfo(utility) 验证 HAP 信息
            └─ HapVerifyInfo 相关验证
```

## JSON 解析调用链

### module.json 解析

```
Compressor.compressHap()
  └─ FileUtils.getFileContent(utility.getJsonPath())
       └─ 读取文件内容
  └─ ModuleJsonUtil.parseModuleType(jsonString)
       └─ JSON.parseObject(jsonString)
       └─ jsonObject.getJSONObject("module")
       └─ module.getString("type")
  └─ ModuleJsonUtil.parseStageBundleType(jsonString)
       └─ 解析 bundleType
  └─ checkStageHap(utility)
       └─ ModuleJsonUtil 各种检查方法
```

### pack.info 解析

```
Uncompress.uncompressPackInfo()
  └─ JsonUtil.parseHapList(deviceType, packInfoJsonStr)
       └─ JSON.parseObject(packInfoJsonStr)
       └─ jsonObject.getJSONArray("packages")
       └─ 遍历 packages 数组
       └─ 创建 PackInfo 对象
            ├─ setName()
            ├─ setModuleName()
            ├─ setModuleType()
            ├─ setDeviceType()
            └─ setDeliveryWithInstall()
```

## 文件操作调用链

### Zip 压缩调用链

```
Compressor.compressFileList()
  └─ PackageUtil.parallelCompress(fileList, outputPath)
       └─ new ParallelScatterZipCreator()
       └─ for each file:
            ├─ create ZipArchiveEntry
            ├─ create InputStreamSupplier
            └─ scatterZipCreator.addArchiveEntry(entry, supplier)
       └─ scatterZipCreator.writeTo(zipOut)
            └─ 多线程并行压缩
            └─ 写入 Zip 中央目录
```

### Zip 解压调用链

```
Uncompress.dataTransferAllFiles()
  └─ FileUtils.unzip(srcPath, destDirPath)
       └─ new ZipFile(srcFile)
       └─ 遍历 entries:
            ├─ entry = entries.nextElement()
            ├─ destFile = new File(destDirPath, entry.getName())
            ├─ FileUtils.matchPattern(destFile.getCanonicalPath())
            └─ dataTransfer(zipFile, entry, destFile)
                 ├─ zipFile.getInputStream(entry)
                 ├─ new FileOutputStream(destFile)
                 └─ 流拷贝
```

## 扫描调用链

### 重复文件检测

```
ScanEntrance.main()
  └─ Scan.scanProcess(utility)
       └─ scanStatDuplicate(utility)
            └─ ScanStatDuplicate.scan()
                 ├─ 遍历 HAP 内所有文件
                 ├─ 计算每个文件的 hash
                 ├─ Map<hash, List<file>> 统计
                 └─ 输出重复文件列表
```

### 文件大小统计

```
Scan.scanProcess(utility)
  └─ scanStatFileSize(utility)
       └─ ScanStatFileSize.scan()
            ├─ 遍历 HAP 内所有文件
            ├─ 按后缀名分组
            ├─ 统计每个后缀的总大小
            └─ 输出统计结果
```

## 关键类交互图

```
┌─────────────────────────────────────────────────────────────────┐
│                         入口层                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │CompressEntrance │  │UncompressEntrance│  │  ScanEntrance   │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
└───────────┼────────────────────┼────────────────────┼──────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                       配置与解析层                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │     Utility     │  │  CommandParser  │  │    JsonUtil     │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         核心处理层                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │    Compressor   │  │    Uncompress   │  │      Scan       │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
└───────────┼────────────────────┼────────────────────┼──────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         验证层                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │ CompressVerify  │  │ UncompressVerify│  │    HapVerify    │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         工具层                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │    FileUtils    │  │   PackageUtil   │  │   ModuleJsonUtil│  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 异常处理链

```
CompressEntrance.main()
  ├─ try {
  │     CommandParser.commandParser()
  │     CompressVerify.commandVerify()
  │     Compressor.compressProcess()
  │  }
  ├─ catch (BundleException e) {
  │     LOG.error("Bundle exception: " + e.getMessage())
  │     System.exit(1)
  │  }
  ├─ catch (FileNotFoundException e) {
  │     LOG.error("File not found: " + e.getMessage())
  │     System.exit(1)
  │  }
  └─ catch (IOException e) {
        LOG.error("IO exception: " + e.getMessage())
        System.exit(1)
     }
```

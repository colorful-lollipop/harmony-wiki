# 关键调用链

## 目的与适用范围

本文档记录 `global_resource_tool` 的关键调用链，帮助开发者理解代码执行流程。

**适用对象**: 核心开发者、调试人员

---

## 调用链 1: 程序启动

### 流程图

```
main() [src/restool.cpp:24]
    │
    ├──▶ InitFaq(argv[0]) [src/restool_errors.cpp]
    │       └── 加载 restool_faq.json
    │
    └──▶ CmdParser::GetInstance() [singleton.h]
            │
            └──▶ CmdParser::Parse(argc, argv, 1) [src/cmd/cmd_parser.cpp:31]
                    │
                    ├──▶ 检查子命令 (dump)
                    │
                    └──▶ PackageParser::Parse(argc, argv) [src/cmd/package_parser.cpp:62]
                            │
                            ├──▶ InitCommand() [src/cmd/package_parser.cpp:531]
                            │       └── 注册命令处理函数
                            │
                            ├──▶ ParseCommand() [src/cmd/package_parser.cpp:615]
                            │       └── getopt_long() 解析参数
                            │
                            ├──▶ CheckParam() [src/cmd/package_parser.cpp:321]
                            │       └── 参数校验
                            │
                            └──▶ ExecCommand() [src/cmd/package_parser.cpp:662]
                                    └── ResourcePack::Package() [src/resource_pack.cpp:37]
```

### 关键代码

**入口** (`src/restool.cpp:24`):
```cpp
int main(int argc, char *argv[])
{
    // ...
    auto &parser = CmdParser::GetInstance();
    return parser.Parse(argc, argv, 1);
}
```

---

## 调用链 2: 资源打包主流程

### 流程图

```
ResourcePack::Package() [src/resource_pack.cpp:37]
    │
    ├──▶ 判断打包类型
    │       ├──▶ PackAppend() - 追加模式
    │       ├──▶ PackCombine() - 合并模式
    │       └──▶ Pack() - 普通/叠加模式
    │
    └──▶ Pack() [src/resource_pack.cpp:74]
            │
            ├──▶ InitResourcePack() [src/resource_pack.cpp:152]
            │       │
            │       ├──▶ InitHeaderCreater() [src/resource_pack.cpp:143]
            │       ├──▶ InitCompression() [src/resource_pack.cpp:61]
            │       ├──▶ InitOutput() [src/resource_pack.cpp:152]
            │       ├──▶ InitConfigJson() [src/resource_pack.cpp:190]
            │       ├──▶ InitModule() [src/resource_pack.cpp:98]
            │       │       └──▶ IdWorker::Init() [src/id_worker.cpp]
            │       └──▶ ThreadPool::Start() [src/thread_pool.cpp:39]
            │
            ├──▶ ScanResources() [resource_directory.cpp]
            │       └──▶ 遍历资源目录
            │
            ├──▶ CompileResources() [resource_compiler_factory.cpp]
            │       │
            │       ├──▶ ResourceCompilerFactory::CreateCompiler()
            │       │       ├──▶ JsonCompiler [src/json_compiler.cpp]
            │       │       └──▶ GenericCompiler [src/generic_compiler.cpp]
            │       │
            │       └──▶ IResourceCompiler::Compile()
            │               └──▶ FileManager::AddResource()
            │
            ├──▶ ResourceTable::CreateResourceTable() [src/resource_table.cpp:46]
            │       │
            │       ├──▶ FileManager::GetResources()
            │       ├──▶ SaveToResouorceIndex() [src/resource_table.cpp:247]
            │       └──▶ CreateIdDefined() [src/resource_table.cpp:73]
            │
            └──▶ GenerateHeader() [src/resource_pack.cpp:171]
                    ├──▶ GenerateTextHeader()
                    ├──▶ GenerateJsHeader()
                    ├──▶ GenerateCplusHeader()
                    └──▶ GenerateTsHeader()
```

---

## 调用链 3: JSON 资源编译

### 流程图

```
JsonCompiler::Compile() [src/json_compiler.cpp]
    │
    ├──▶ OpenJsonFile() [src/resource_util.cpp]
    │       └──▶ cJSON_Parse()
    │
    ├──▶ ParseRootJson() [src/json_compiler.cpp]
    │       │
    │       ├──▶ ParseString() / ParseInteger() / ParseColor() / ...
    │       │       │
    │       │       ├──▶ KeyParser::Parse() [src/key_parser.cpp]
    │       │       │       └── 解析限定词
    │       │       │
    │       │       ├──▶ ReferenceParser::Parse() [src/reference_parser.cpp]
    │       │       │       └── 解析资源引用 ($string:name)
    │       │       │
    │       │       └──▶ IdWorker::GenerateId() [src/id_worker.cpp]
    │       │               └── 分配资源 ID
    │       │
    │       └──▶ ResourceItem 构建
    │
    └──▶ SaveResourceItem() [src/i_resource_compiler.cpp]
            └──▶ FileManager::AddResource() [src/file_manager.cpp]
```

---

## 调用链 4: 资源索引生成

### 流程图

```
ResourceTable::CreateResourceTable() [src/resource_table.cpp:46]
    │
    ├──▶ FileManager::GetResources() [src/file_manager.cpp]
    │       └── 获取所有资源项
    │
    ├──▶ 构建 TableData 映射
    │       └── map<limitKey, vector<TableData>>
    │
    ├──▶ 判断索引格式
    │       ├──▶ SaveToResouorceIndex() - 旧格式
    │       └──▶ SaveToNewResouorceIndex() - 新格式
    │
    └──▶ SaveToResouorceIndex() [src/resource_table.cpp:247]
            │
            ├──▶ 写入 IndexHeader
            │       ├──▶ tag: "RES"
            │       ├──▶ version
            │       ├──▶ count
            │       └──▶ dataOffset
            │
            ├──▶ 写入 ResConfig (限定词配置)
            │
            ├──▶ 写入 ResData (资源数据)
            │       ├──▶ id
            │       ├──▶ resType
            │       ├──▶ dataSize
            │       └──▶ data
            │
            └──▶ 写入 DataPool
```

---

## 调用链 5: 纹理压缩

### 流程图

```
CompressionParser::Init() [src/compression_parser.cpp:89]
    │
    ├──▶ ResourceUtil::OpenJsonFile() [src/resource_util.cpp]
    │       └── 加载 opt-compression.json
    │
    ├──▶ ParseContext() [src/compression_parser.cpp:99]
    │       └── 获取 extensionPath (动态库路径)
    │
    ├──▶ LoadImageTranscoder() [src/compression_parser.cpp]
    │       │
    │       ├──▶ dlopen() / LoadLibrary() [src/compression_parser.cpp]
    │       │       └── 加载 libimage_transcoder_shared.so/.dll/.dylib
    │       │
    │       └──▶ dlsym() / GetProcAddress()
    │               └── 获取压缩函数指针
    │
    ├──▶ ParseCompression() [src/compression_parser.cpp:125]
    │       └── 解析压缩配置
    │
    └──▶ ParseFilters() [src/compression_parser.cpp]
            └── 解析过滤规则
```

---

## 调用链 6: Dump 子命令

### 流程图

```
main() [src/restool.cpp:24]
    │
    └──▶ CmdParser::Parse() [src/cmd/cmd_parser.cpp:31]
            │
            └──▶ 识别 "dump" 子命令
                    │
                    └──▶ DumpParser::Parse() [src/cmd/dump_parser.cpp]
                            │
                            ├──▶ 解析参数 (-h, config, filePath)
                            │
                            └──▶ ResourceDumper::Dump() [src/resource_dumper.cpp]
                                    │
                                    ├──▶ 打开 HAP 文件
                                    │
                                    ├──▶ 解析 resources.index
                                    │       └── ResourceTable::LoadResTable()
                                    │
                                    └──▶ 输出 JSON 格式资源信息
```

---

## 调用链 7: 错误处理

### 流程图

```
错误发生
    │
    └──▶ PrintError() [src/restool_errors.cpp]
            │
            ├──▶ GetError(errCode) [src/restool_errors.cpp]
            │       └── 从 ERRORS_MAP 获取错误信息
            │
            ├──▶ FormatCause() [include/restool_errors.h:165]
            │       └── 格式化错误原因
            │
            ├──▶ SetPosition() [include/restool_errors.h:181]
            │       └── 设置错误位置
            │
            └──▶ 输出错误信息到 stderr
                    │
                    ├──▶ 错误码
                    ├──▶ 错误类型
                    ├──▶ 错误描述
                    ├──▶ 错误原因
                    ├──▶ 解决方案 (solutions)
                    └──▶ 更多信息链接 (moreInfo)
```

---

## 调用链 8: 文件操作

### 流程图

```
FileEntry::CreateDirs() [src/file_entry.cpp:155]
    │
    └──▶ CreateDirsInner() [src/file_entry.cpp:340]
            │
            ├──▶ 逐级解析路径
            │       └── path/sub1/sub2/...
            │
            ├──▶ FileEntry::Exist() [src/file_entry.cpp:117]
            │       └── stat() / PathFileExists()
            │
            └──▶ mkdir() / CreateDirectoryW() [src/file_entry.cpp:347]
                    └── 创建目录 (权限: 755)

FileEntry::RemoveAllDir() [src/file_entry.cpp:132]
    │
    └──▶ RemoveAllDirInner() [src/file_entry.cpp:304]
            │
            ├──▶ 递归遍历子目录
            │       └── GetChilds() [src/file_entry.cpp:61]
            │
            ├──▶ 删除文件
            │       └── remove() / DeleteFileW() [src/file_entry.cpp:314]
            │
            └──▶ 删除目录
                    └── rmdir() / RemoveDirectoryW() [src/file_entry.cpp:331]
```

---

## 相关文档

- [架构说明](../02_Architecture.md) - 系统架构
- [内部接口](../04_Internal_API.md) - 模块接口

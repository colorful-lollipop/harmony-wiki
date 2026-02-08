# 常见问题与调试

## 打包问题

### 1. 打包失败："Compress failed"

**症状**:
```
LOG.error: Compress failed
Exit code: 1
```

**排查步骤**:

1. **检查输入文件是否存在**
```bash
ls -la module.json
ls -la resources/
```

2. **检查输出目录权限**
```bash
touch /path/to/output/test && rm /path/to/output/test
```

3. **查看详细错误日志**
```bash
java -jar app_packing_tool.jar ... 2>&1 | tee build.log
```

**常见原因**:
- 输入路径错误 (`adapter/ohos/CompressEntrance.java:42-54`)
- 输出目录无写入权限
- JSON 文件格式错误
- 文件被其他进程占用

**修复方法**:
```bash
# 确保所有输入文件存在
# 确保输出目录可写
mkdir -p /path/to/output
chmod 755 /path/to/output

# 验证 JSON 格式
python -m json.tool module.json > /dev/null && echo "Valid JSON"
```

### 2. HAP 验证失败

**症状**:
```
LOG.error: Verify the module.json file of the Stage HAP package failed
```

**排查步骤**:

1. **检查 module.json 格式** (`adapter/ohos/Compressor.java:655-659`):
```bash
# 验证 JSON 结构
cat module.json | python -m json.tool
```

2. **检查必填字段**:
```json
{
  "module": {
    "name": "entry",           // 必填
    "type": "entry",           // 必填
    "deviceTypes": ["phone"]   // 必填
  },
  "app": {
    "bundleName": "com.example", // 必填
    "versionCode": 1            // 必填
  }
}
```

3. **检查 module 类型**:
```bash
# HAP 的 module type 不能是 shared
grep -o '"type":\s*"[^"]*"' module.json
```

### 3. APP 打包时 HAP 验证失败

**症状**:
```
LOG.error: HapVerify::checkHapIsValid failed
```

**排查步骤** (`adapter/ohos/HapVerify.java`):

1. **检查 bundleName 一致性**:
```bash
# 所有 HAP 的 bundleName 必须相同
for hap in *.hap; do
  unzip -p "$hap" module.json | grep '"bundleName"'
done
```

2. **检查 versionCode 一致性**:
```bash
for hap in *.hap; do
  unzip -p "$hap" module.json | grep '"versionCode"'
done
```

3. **检查 moduleName 唯一性**:
```bash
for hap in *.hap; do
  unzip -p "$hap" module.json | grep '"name"' | head -1
done
```

**修复方法**:
- 确保所有 HAP 使用相同的 bundleName
- 确保所有 HAP 使用相同的 versionCode
- 确保每个 HAP 的 moduleName 唯一

## 拆包问题

### 1. 拆包失败："unpackageProcess failed"

**症状**:
```
LOG.error: UncompressEntrance::main exit, uncompress failed
```

**排查步骤** (`adapter/ohos/Uncompress.java:94-145`):

1. **检查输入文件格式**:
```bash
file input.hap
# 应该是: Zip archive data
```

2. **检查输出目录**:
```bash
# 目录必须可写
ls -ld output_dir/
```

3. **检查磁盘空间**:
```bash
df -h output_dir/
```

### 2. 解析失败："ParseApp failed"

**症状**:
```
compressResult.setMessage("ParseApp verify failed")
```

**排查步骤** (`adapter/ohos/UncompressEntrance.java:228-253`):

1. **检查 APP 文件完整性**:
```bash
unzip -t input.app
```

2. **检查 pack.info 是否存在**:
```bash
unzip -l input.app | grep pack.info
```

3. **验证解析模式**:
```java
// 有效的解析模式
PARSE_MODE_HAPLIST  // 获取 HAP 列表
PARSE_MODE_HAPINFO  // 获取指定 HAP 信息
PARSE_MODE_ALL      // 获取所有信息
```

## 验证问题

### 1. 路径验证失败

**症状**:
```
LOG.error: Input invalid file: /path/to/file
```

**排查步骤** (`adapter/ohos/FileUtils.java`):

1. **检查路径格式**:
```java
// FileUtils.matchPattern 使用正则: [0-9A-Za-z/].{0,4095}
// 只允许字母、数字、斜杠
```

2. **检查路径长度**:
```bash
# 路径长度不能超过 4096
path="/very/long/path/..."
echo ${#path}
```

**修复方法**:
- 使用相对路径
- 避免特殊字符
- 缩短路径长度

### 2. JSON 解析失败

**症状**:
```
LOG.error: JSON parse error
```

**排查步骤** (`adapter/ohos/JsonUtil.java`):

1. **验证 JSON 格式**:
```bash
python -m json.tool module.json > /dev/null
```

2. **检查文件编码**:
```bash
file -i module.json
# 应该是: text/plain; charset=utf-8
```

3. **检查 BOM**:
```bash
hexdump -C module.json | head -1
# 不应该以 EF BB BF 开头
```

**修复方法**:
```bash
# 移除 BOM
sed -i '1s/^\xEF\xBB\xBF//' module.json

# 转换编码
iconv -f GBK -t UTF-8 module.json > module.json.new
mv module.json.new module.json
```

## 构建问题

### 1. GN 构建失败

**症状**:
```
ERROR at //developtools/packing_tool/BUILD.gn:xx:xx
```

**排查步骤**:

1. **检查 GN 语法**:
```bash
gn format BUILD.gn
```

2. **检查依赖**:
```bash
gn desc out/ //developtools/packing_tool:packing_tool deps
```

3. **检查工具链**:
```bash
gn args out/ --list
```

### 2. Python 构建失败

**症状**:
```
compile module: packing_tool failed!
```

**排查步骤** (`build.py`):

1. **检查 Python 版本**:
```bash
python3 --version  # 需要 3.6+
```

2. **检查 Java 版本**:
```bash
java -version  # 需要 1.8+
javac -version
```

3. **检查依赖库**:
```bash
ls -la ../../prebuilts/packing_tool/fastjson2/
ls -la ../../prebuilts/packing_tool/compress/
```

4. **查看详细日志**:
```bash
python3 build.py ... 2>&1 | tee build.log
```

### 3. Shell 脚本失败

**症状**:
```
bash: packingTool.sh: command not found
```

**排查步骤**:

1. **检查脚本权限**:
```bash
chmod +x packingTool.sh
```

2. **检查脚本内容**:
```bash
head -20 packingTool.sh
```

3. **手动执行调试**:
```bash
bash -x packingTool.sh arg1 arg2 2>&1 | tee script.log
```

## 性能问题

### 1. 打包速度慢

**症状**: 打包大项目耗时过长

**优化建议** (`adapter/ohos/Compressor.java`):

1. **使用并行压缩**（已默认启用）:
```java
// ParallelScatterZipCreator 自动使用多线程
```

2. **调整压缩级别**:
```bash
# 压缩级别 1-9，数值越大压缩率越高速度越慢
java -jar app_packing_tool.jar ... --compress-level 1
```

3. **排除不必要文件**:
```bash
# 使用 .packingignore 排除临时文件
```

### 2. 内存不足

**症状**:
```
java.lang.OutOfMemoryError: Java heap space
```

**解决方法**:

```bash
# 增加 JVM 内存
java -Xmx4g -jar app_packing_tool.jar ...

# 或设置环境变量
export JAVA_OPTS="-Xmx4g -Xms1g"
```

## 调试技巧

### 1. 启用详细日志

```bash
# 设置日志级别
export PACKING_TOOL_LOG_LEVEL=DEBUG

# 或修改代码
// Log.java
private static final int LOG_LEVEL = LOG_LEVEL_DEBUG;
```

### 2. 使用 IDE 调试

```bash
# 1. 编译带调试信息的版本
javac -g -source 1.8 -target 1.8 ...

# 2. 在 IDE 中配置远程调试
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005 \
     -jar app_packing_tool.jar ...
```

### 3. 分析堆栈跟踪

```bash
# 获取线程 dump
jstack -l $(pgrep -f "app_packing_tool") > thread_dump.txt

# 获取堆 dump
jmap -dump:format=b,file=heap.hprof $(pgrep -f "app_packing_tool")
```

### 4. 性能分析

```bash
# 使用 async-profiler
./profiler.sh -d 30 -f profile.html $(pgrep -f "app_packing_tool")

# 使用 JFR
java -XX:+FlightRecorder -XX:StartFlightRecording=duration=60s,filename=recording.jfr \
     -jar app_packing_tool.jar ...
```

## 常见错误码

| 错误码 | 含义 | 排查方向 |
|-------|------|---------|
| 0 | 成功 | - |
| 1 | 通用错误 | 查看日志 |
| 2 | 参数错误 | 检查命令行参数 |
| 3 | 文件不存在 | 检查输入路径 |
| 4 | 验证失败 | 检查配置文件 |
| 5 | IO 错误 | 检查磁盘/权限 |

## 日志分析

### 日志格式

```
[时间] [级别] [类名]: 消息
```

示例:
```
2025-02-06 10:30:45 ERROR CompressEntrance: Compress failed
2025-02-06 10:30:45 DEBUG FileUtils: Reading file: module.json
```

### 关键日志位置

| 类 | 日志位置 | 用途 |
|---|---------|------|
| CompressEntrance | `main()` | 打包流程 |
| UncompressEntrance | `main()` | 拆包流程 |
| Compressor | `compressProcess()` | 压缩详情 |
| HapVerify | `checkHapIsValid()` | 验证详情 |

### 日志过滤

```bash
# 只看错误
grep ERROR build.log

# 只看特定类
grep CompressEntrance build.log

# 统计错误数量
grep -c ERROR build.log
```

## 联系支持

如果以上方法无法解决问题：

1. 收集以下信息:
   - 完整的命令行
   - 完整的错误日志
   - 相关配置文件（脱敏后）
   - 系统环境信息

2. 提交 Issue 到:
   - OpenHarmony 官方仓库
   - 或联系开发团队

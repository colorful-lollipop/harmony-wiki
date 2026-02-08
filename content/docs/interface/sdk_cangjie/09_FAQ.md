# 常见问题 (FAQ)

## 构建问题

### Q1: 构建时提示 "cjc not found"

**问题**: 运行构建命令时提示找不到 cjc 编译器

**解决步骤**:

1. 检查 SDK 工具链是否正确安装

```bash
ls $OHOS_SDK/cangjie/build-tools/bin/cjc
```

2. 设置环境变量

```bash
export CANGJIE_SDK=$OHOS_SDK/cangjie
export PATH=$CANGJIE_SDK/build-tools/bin:$PATH
```

3. 重新构建

```bash
./build.sh --product-name ohos-sdk --build-target out/sdk/gen/build/ohos/sdk:cangjie
```

**相关文档**: [构建指南](../docs/cangjie_sdk_build_guide.md)

---

### Q2: GN 构建失败 "cannot find import"

**问题**: GN 构建时提示找不到模块导入

**解决步骤**:

1. 检查 `sdk_cangjie.gni` 是否存在

```bash
ls interface/sdk_cangjie/sdk_cangjie.gni
```

2. 检查 GN 路径配置

```bash
# 确保在正确的工作目录下
cd $OHOS_ROOT
ls interface/sdk_cangjie/
```

3. 清理构建缓存

```bash
rm -rf out/sdk/gen/build/ohos/sdk/cangjie/*
```

---

### Q3: 交叉编译平台不支持

**问题**: 尝试构建 ohos-arm 平台失败

**解决步骤**:

1. 检查平台支持标志

```bash
# 在 BUILD.gn 或命令行中添加
--args sdk_build_cangjie_ohos_arm=true
```

2. 检查工具链配置

```bash
ls build/toolchain/
```

3. 确认系统能力

```bash
# 确保编译环境支持 ARM 工具链
which aarch64-linux-gnu-gcc
```

---

### Q4: cjo 文件生成失败

**问题**: flatc 工具无法生成 cjo 文件

**解决步骤**:

1. 检查 flatc 工具

```bash
ls third_party/flatbuffers/build/flatc
```

2. 检查 FlatBuffers Schema

```bash
ls *.fbs
```

3. 手动运行 flatc

```bash
./third_party/flatbuffers/build/flatc -c schema.fbs
```

**相关文档**: [cjo 序列化指南](../docs/cangjie_cjo_serialization_and_deserialization_guide.md)

---

## 运行问题

### Q5: 运行时提示 "permission denied"

**问题**: 应用运行时提示权限拒绝

**解决步骤**:

1. 在模块配置中声明权限

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.CAMERA"
      }
    ]
  }
}
```

2. 运行时请求权限

```cangjie
import kit.AbilityKit

let atManager = AtManager.create()
let result = await atManager.requestPermissionsFromUser(
    context,
    ["ohos.permission.CAMERA"]
)
```

3. 检查权限是否已授予

```bash
# 通过 hdc 查看权限状态
hdc shell "bm dump"
```

---

### Q6: 网络请求失败 "net error"

**问题**: HTTP 请求返回网络错误

**解决步骤**:

1. 检查网络权限

```bash
# 确认已声明权限
ohos.permission.INTERNET
```

2. 检查 URL 格式

```cangjie
// 使用完整的 URL
let response = await http.request("https://example.com/api")
```

3. 检查超时设置

```cangjie
let options = HttpRequestOptions()
options.connectTimeout = 30000  // 30 秒
```

4. 检查防火墙

```bash
# 开放网络端口
iptables -L
```

---

### Q7: 文件读写失败 "no such file"

**问题**: 文件操作返回文件不存在

**解决步骤**:

1. 检查文件路径

```cangjie
// 使用正确的沙箱路径
let file = File.open(context.filesDir + "/test.txt", OpenFlags.RDWR)
```

2. 检查目录是否存在

```bash
hdc shell "ls /data/app/el2/100/base/"
```

3. 检查权限

```bash
# 检查应用目录权限
hdc shell "ls -la /data/app/el2/100/base/"
```

---

### Q8: IPC 通信失败

**问题**: IPC 调用返回错误

**解决步骤**:

1. 检查 IPC 能力声明

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public class IPCDemo { ... }
```

2. 检查接口令牌

```cangjie
let parcel = MessageParcel.create()
parcel.writeRemoteInterfaceToken("demo.token")
```

3. 检查服务是否注册

```bash
hdc shell "sa list"
```

---

## API 使用问题

### Q9: 导入 Kit 失败

**问题**: 导入 kit 模块时报错

**解决步骤**:

1. 检查 kit 名称

```cangjie
// 正确
import kit.NetworkKit

// 错误
import kit.networkkit
```

2. 检查 SDK 是否完整安装

```bash
ls $OHOS_SDK/cangjie/api/modules/
```

3. 检查 .cj.d 文件是否存在

```bash
ls $OHOS_SDK/cangjie/api/modules/ohos/kit.NetworkKit.cj.d
```

---

### Q10: 异步 API 调用问题

**问题**: 异步调用返回 Promise pending

**解决步骤**:

1. 使用 await

```cangjie
let response = await http.request("https://example.com")
// 必须使用 await 等待结果
```

2. 检查异步函数返回类型

```cangjie
// Promise<T> 需要 await
public func request(): Promise<HttpResponse>

// AsyncCallback<T> 需要回调
public func request(callback: (Error?, HttpResponse?) => Unit)
```

3. 处理错误

```cangjie
try {
    let response = await http.request("https://example.com")
} catch (e: BusinessException) {
    // 处理异常
}
```

---

### Q11: 状态管理不生效

**问题**: ArkUI 状态变量更新后 UI 未刷新

**解决步骤**:

1. 使用正确的状态装饰器

```cangjie
// @State 用于组件内部状态
@State
var count: Int = 0

// @Prop 用于父子组件传值
@Prop
var title: String

// @Link 用于双向绑定
@Link
var count: Int
```

2. 确保状态变更后触发刷新

```cangjie
// 正确
this.count += 1

// 错误 - 直接赋值可能不触发刷新
count = count + 1
```

3. 检查组件结构

```cangjie
// 确保在 build() 中使用状态变量
Column {
    Text("Count: \(${this.count})")
    Button("Increment") {
        this.count += 1
    }
}
```

---

### Q12: 类型转换错误

**问题**: 类型不匹配错误

**解决步骤**:

1. 使用显式类型转换

```cangjie
let num: Int = 42
let str: String = num.toString()

let floatValue: Float = 3.14
let intValue: Int = int(floatValue)
```

2. 检查泛型类型

```cangjie
// Array<String> 和 Array<Int> 不兼容
let strings: Array<String> = ["a", "b"]
let result: Array<String> = strings.map { s => s }
```

3. 使用类型推断

```cangjie
// 编译器自动推断类型
let array = [1, 2, 3]  // Array<Int>
let strings = ["a", "b"]  // Array<String>
```

---

## 调试问题

### Q13: 日志输出

**问题**: 如何输出调试日志

**解决步骤**:

1. 使用 HiLog

```cangjie
import kit.PerformanceAnalysisKit

let logger = HiLogger.create("DemoTag")
logger.info("Debug message")
logger.error("Error message")
```

2. 设置日志级别

```cangjie
logger.setLevel(LogLevel.DEBUG)
```

3. 查看日志

```bash
hdc shell "hilog | grep DemoTag"
```

---

### Q14: 性能分析

**问题**: 如何进行性能分析

**解决步骤**:

1. 使用 HiTrace

```cangjie
import kit.PerformanceAnalysisKit

let trace = HiTrace.create("TraceName")
trace.start()

// 执行需要分析的操作

trace.finish()
```

2. 使用系统跟踪

```bash
# 启动跟踪
hdc shell "profiler start"

# 执行操作

# 停止并导出
hdc shell "profiler stop"
```

3. 查看火焰图

```bash
# 使用 perf 工具
perf record -g -a
perf report
```

---

### Q15: 断点调试

**问题**: 如何设置断点调试

**解决步骤**:

1. 使用 cjdb 调试器

```bash
cjdb --file app.cjdb --breakpoint main
```

2. 设置条件断点

```bash
cjdb> break main:10 if count > 100
```

3. 查看变量

```bash
cjdb> print count
cjdb> backtrace
```

---

## 兼容性

### Q16: API 版本兼容

**问题**: 不同 API Level 的兼容性

**解决步骤**:

1. 检查 API 版本

```cangjie
@!APILevel[
    since: "22"
]
public func newFeature(): Unit
```

2. 使用版本检查

```cangjie
if (apiLevel >= 22) {
    // 使用新 API
} else {
    // 使用兼容方案
}
```

3. 查看 API 变更日志

```bash
# 查看 API 变更记录
git log --oneline --grep="API" interface/sdk_cangjie/
```

---

### Q17: 与 ArkTS 互操作

**问题**: Cangjie 与 ArkTS 代码互操作

**解决步骤**:

1. 使用 N-API

```cangjie
import ohos.ark_interop.*

let env = getNapiEnv()
let value = createNapiValue(env, ptr)
```

2. 使用 FFI

```cangjie
import ohos.ffi.*

@CFunction("external_c_function")
public func externalFunction(param: Int32): Int32
```

3. 查看互操作文档

```bash
# 参考 arkcompiler_cangjie_ark_interop 仓库
ls -la arkcompiler_cangjie_ark_interop/doc/
```

---

## 工具链

### Q18: cjpm 使用问题

**问题**: 包管理器命令错误

**解决步骤**:

1. 检查 cjpm 版本

```bash
cjpm --version
```

2. 查看帮助

```bash
cjpm --help
cjpm init --help
cjpm install --help
```

3. 常见命令

```bash
# 初始化项目
cjpm init -t library

# 安装依赖
cjpm install

# 构建项目
cjpm build

# 运行测试
cjpm test
```

---

### Q19: 代码格式化

**问题**: 代码格式不符合规范

**解决步骤**:

1. 使用 cjfmt 格式化

```bash
cjfmt --fix src/
```

2. 配置格式化规则

```bash
# .cjfmt.toml
indentWidth = 4
lineWidth = 120
```

3. 集成到 IDE

```bash
# VS Code 插件
# 安装 Cangjie Language Support
```

---

### Q20: 编译优化

**问题**: 编译产物过大或运行慢

**解决步骤**:

1. 启用优化选项

```bash
cjc -O3 --opt-level 3 source.cj
```

2. 使用 LTO

```bash
cjc -flto source.cj
```

3. 去除调试信息

```bash
cjc -g0 source.cj
```

---

## 相关资源

### 官方文档

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Cangjie 语言文档](https://gitcode.com/Cangjie/cangjie_docs)
- [SDK 构建指南](../docs/cangjie_sdk_build_guide.md)

### 社区支持

- [OpenHarmony GitHub](https://github.com/openharmony)
- [Cangjie 社区](https://gitcode.com/Cangjie)

### 调试工具

| 工具 | 用途 |
|------|------|
| `hilog` | 日志查看 |
| `hdc` | 设备调试 |
| `cjdb` | 断点调试 |
| `cjpm` | 包管理 |
| `cjfmt` | 代码格式化 |
| `perf` | 性能分析 |

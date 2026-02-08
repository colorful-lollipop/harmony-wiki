# Callgraphs.md

## 目的

本文档提供 XTS Tools 的关键调用链（入口→核心逻辑），帮助开发者理解代码执行路径。

## 适用范围

- 仅涉及 tools/ 仓库的核心流程
- 不涉及测试用例具体实现

---

## C 测试用例执行调用链（Mini 系统）

```mermaid
sequenceDiagram
    participant OS as 系统上电
    participant Main as 主入口
    participant Suite as 测试套件
    participant Case as 测试用例
    participant Serial as 串口日志

    OS->>Main: 初始化
    Main->>Suite: RUN_TEST_SUITE(IntTestSuite)
    Suite->>Suite: IntTestSuiteSetup()
    Suite->>Case: LITE_TEST_CASE(TestCase001)
    Case->>Serial: 输出日志
    Suite->>Suite: IntTestSuiteTearDown()
    Serial->>Serial: "xx Tests xx Failures xx Ignored"
```

证据：README.md:310-351

---

## C++ 测试用例执行调用链（Small/Standard 系统）

```mermaid
sequenceDiagram
    participant PC as PC（NFS 服务器）
    participant Board as 开发板
    participant Bin as 测试套件（.bin）
    participant Suite as 测试套件
    participant Case as 测试用例
    participant Serial as 串口日志

    PC->>Board: mount NFS
    Board->>Bin: 执行 .bin
    Bin->>Suite: 加载测试套件
    Suite->>Suite: SetUpTestCase()
    Suite->>Case: HWTEST_F(TestCase_0001)
    Case->>Serial: 输出日志
    Suite->>Suite: TearDownTestCase()
    Serial->>Serial: 测试结果
```

证据：README.md:485-510

---

## JavaScript 测试用例执行调用链（Standard 系统）

```mermaid
sequenceDiagram
    participant HAP as HAP 包
    participant Core as HJSUnit Core
    participant Config as Config Service
    participant Test as 测试用例
    participant Log as 日志

    HAP->>Core: 加载 Core 实例
    Core->>Core: addService('expect', expectExtend)
    Core->>Core: init()
    Core->>Config: setConfig(this)
    Config->>Test: require('../../../test/List.test')
    Test->>Test: describe('appInfoTest')
    Test->>Test: it('app_info_test_001')
    Test->>Log: expect(...).assertEqual()
    Core->>Core: execute()
    Log->>Log: 测试结果
```

证据：README.md:511-649

---

## GN 构建调用链

```mermaid
sequenceDiagram
    participant Dev as 开发者
    participant GN as GN 工具
    participant Ninja as Ninja 工具
    participant Obj as 编译产物
    participant Art as 最终产物

    Dev->>GN: gn gen out/xxx
    GN->>GN: 解析 BUILD.gn/.gni
    GN->>Ninja: 生成 build.ninja
    Dev->>Ninja: ninja -C out/xxx
    Ninja->>Obj: 编译源文件
    Obj->>Art: 链接生成 .a/.bin/.hap
    Art->>Dev: 输出产物
```

证据：README.md:318-327、440-458

---

## suite.py 调用链（测试套件管理）

```mermaid
sequenceDiagram
    participant User as 用户
    participant Suite as suite.py
    participant GN as GN 工具
    participant Ninja as Ninja 工具
    participant NFS as NFS 服务器
    participant Board as 开发板

    User->>Suite: 执行测试套件
    Suite->>GN: 调用 gn gen
    GN->>Ninja: 调用 ninja
    Ninja->>Suite: 编译完成
    Suite->>NFS: 部署测试套件
    NFS->>Board: 挂载 NFS
    Board->>Board: 执行测试套件
    Board->>Suite: 返回测试结果
    Suite->>User: 输出测试报告
```

证据：glob 发现 build/suite.py；README.md:485-510

---

## check_hvigor.py 调用链（HVIGR 检查）

```mermaid
sequenceDiagram
    participant User as 用户
    participant Script as check_hvigor.py
    participant HVIGR as HVIGR 工具
    participant FS as 文件系统

    User->>Script: 执行 check_hvigor.py
    Script->>FS: 读取配置文件
    Script->>HVIGR: 调用 HVIGR 检查
    HVIGR->>FS: 扫描文件
    HVIGR->>Script: 返回检查结果
    Script->>User: 输出检查报告
```

证据：glob 发现 standard_check/check_hvigor.py

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [02_Architecture.md](02_Architecture.md)
- [05_GN_Targets.md](05_GN_Targets.md)

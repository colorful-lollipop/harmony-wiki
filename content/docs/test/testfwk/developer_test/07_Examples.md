# 示例说明

## 7.1 示例概述

Developer Test Framework 提供了多个测试用例示例，涵盖不同测试类型和编程语言。

### 7.1.1 示例列表

| 目录 | 说明 | 测试类型 | 语言 |
|------|------|---------|------|
| `calculator/` | 计算器示例 | UT, FUZZ, BENCHMARK | C++ |
| `app_info/` | 应用信息示例 | UT | JavaScript |
| `detector/` | 探测器示例 | UT | C++ |
| `sleep/` | 睡眠测试示例 | PERF | C++ |
| `distributedb/` | 分布式数据库示例 | DST | C++ |
| `lite/` | Lite 设备示例 | - | - |
| `stagetest/` | Stage 模型示例 | ACTS | ArkTS |

## 7.2 Calculator 示例

### 7.2.1 目录结构

```
examples/calculator/
├── BUILD.gn                   # 构建配置
├── include/
│   └── calculator.h           # 头文件
├── src/
│   └── calculator.cpp         # 源文件
└── test/
    ├── BUILD.gn              # 测试构建
    ├── unittest/              # 单元测试
    │   ├── common/           # 公共用例
    │   │   ├── BUILD.gn
    │   │   ├── calculator_add_test.cpp
    │   │   └── calculator_sub_test.cpp
    │   └── phone/            # phone 形态用例
    │       ├── BUILD.gn
    │       ├── calculator_div_test.cpp
    │       └── calculator_mul_test.cpp
    ├── fuzztest/             # Fuzz 测试
    │   └── common/
    │       └── Calculator_fuzzer/
    │           ├── BUILD.gn
    │           ├── calculator_fuzzer.cpp
    │           ├── calculator_fuzzer.h
    │           ├── corpus/
    │           └── project.xml
    └── benchmarktest/        # 性能测试
        └── common/
            ├── BUILD.gn
            └── benchmark_demo_test.cpp
```

### 7.2.2 业务代码

**calculator.h** (`examples/calculator/include/calculator.h`)

```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H

class Calculator {
public:
    int Add(int a, int b);
    int Sub(int a, int b);
    int Mul(int a, int b);
    int Div(int a, int b);
};

#endif // CALCULATOR_H
```

**calculator.cpp** (`examples/calculator/src/calculator.cpp`)

```cpp
#include "calculator.h"

int Calculator::Add(int a, int b) {
    return a + b;
}

int Calculator::Sub(int a, int b) {
    return a - b;
}

int Calculator::Mul(int a, int b) {
    return a * b;
}

int Calculator::Div(int a, int b) {
    return a / b;
}
```

### 7.2.3 单元测试示例

**calculator_add_test.cpp** (`examples/calculator/test/unittest/common/`)

```cpp
#include "calculator.h"
#include <gtest/gtest.h>

using namespace testing::ext;

class CalculatorAddTest : public testing::Test {
public:
    static void SetUpTestCase(void);
    static void TearDownTestCase(void);
    void SetUp();
    void TearDown();
};

void CalculatorAddTest::SetUpTestCase(void)
{
    // Setup invoked before all testcases
}

void CalculatorAddTest::TearDownTestCase(void)
{
    // Teardown invoked after all testcases
}

void CalculatorAddTest::SetUp(void)
{
    // Setup invoked before each testcases
}

void CalculatorAddTest::TearDown(void)
{
    // Teardown invoked after each testcases
}

/**
 * @tc.name: integer_add_001
 * @tc.desc: Verify the add function.
 * @tc.type: FUNC
 * @tc.require: issueNumber
 */
HWTEST_F(CalculatorAddTest, integer_add_001, TestSize.Level1)
{
    // Step 1: Call function to get result
    int actual = Add(4, 0);

    // Step 2: Use assertion to compare expected and actual result
    EXPECT_EQ(4, actual);
}
```

### 7.2.4 Fuzz 测试示例

**calculator_fuzzer.cpp** (`examples/calculator/test/fuzztest/common/Calculator_fuzzer/`)

```cpp
#include "calculator.h"
#include <fuzzer/FuzzTest.h>

// Fuzz test entry
FUZZ_TEST(const char* a, const char* b) {
    Calculator calc;
    int val1 = std::stoi(a);
    int val2 = std::stoi(b);
    
    // Call functions that need to be fuzzed
    calc.Add(val1, val2);
    calc.Sub(val1, val2);
    calc.Mul(val1, val2);
    
    // Division needs zero check
    if (val2 != 0) {
        calc.Div(val1, val2);
    }
}
```

### 7.2.5 性能测试示例

**benchmark_demo_test.cpp** (`examples/calculator/test/benchmarktest/common/`)

```cpp
#include "calculator.h"
#include <benchmark/benchmark.h>

static void BM_Calculator_Add(benchmark::State& state) {
    Calculator calc;
    for (auto _ : state) {
        benchmark::DoNotOptimize(calc.Add(1, 2));
    }
}
BENCHMARK(BM_Calculator_Add);

BENCHMARK_MAIN();
```

## 7.3 App Info 示例

### 7.3.1 概述

JS 测试示例，展示如何使用 deccjsunit 框架编写 FA 模型的 JavaScript 测试用例。

### 7.3.2 测试用例示例

**AppInfoTest.js**

```javascript
/*
 * Copyright (C) 2021 XXXX Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 */
import app from '@system.app'

import {describe, beforeAll, beforeEach, afterEach, afterAll, it, expect} from 'deccjsunit/index'

describe("AppInfoTest", function () {
    beforeAll(function() {
        // input testsuit setup step
        console.info('beforeAll called')
    })
    
    afterAll(function() {
        // input testsuit teardown step
        console.info('afterAll called')
    })
    
    beforeEach(function() {
        // input testcase setup step
        console.info('beforeEach called')
    })
    
    afterEach(function() {
        // input testcase teardown step
        console.info('afterEach called')
    })

    /*
     * @tc.name: appInfoTest001
     * @tc.desc: verify app info is not null
     * @tc.type: FUNC
     * @tc.require: issueNumber
     */
    it("appInfoTest001", 0, function () {
        // Call function to get result
        var info = app.getInfo()

        // Use assertion to compare expected and actual result
        expect(info != null).assertEqual(true)
    })
})
```

## 7.4 Detector 示例

### 7.4.1 概述

C++ 测试示例，展示基础测试用例的编写方式。

### 7.4.2 测试用例结构

```
examples/detector/
├── BUILD.gn
├── include/
│   └── detector.h
├── src/
│   └── detector.cpp
└── test/
    └── unittest/
        └── common/
            └── BUILD.gn
```

## 7.5 Sleep 示例

### 7.5.1 概述

性能测试示例，展示如何编写 PERF 类型测试。

### 7.5.2 测试用例结构

```
examples/sleep/
├── BUILD.gn
├── include/
│   └── sleep.h
├── src/
│   └── sleep.cpp
└── test/
    └── performance/
        └── common/
            └── BUILD.gn
```

## 7.6 Distributed DB 示例

### 7.6.1 概述

分布式测试示例，展示如何编写 DST 类型测试。

### 7.6.2 测试用例结构

```
examples/distributedb/
├── BUILD.gn
├── include/
│   └── distributedb.h
├── src/
│   └── distributedb.cpp
└── test/
    └── distributedtest/
        └── common/
            └── BUILD.gn
```

## 7.7 Stage Test 示例

### 7.7.1 概述

Stage 模型测试示例，使用 ACTS 测试框架。

### 7.7.2 测试用例结构

```
examples/stagetest/
└── actsbundlemanagerstagetest/
    └── unittest/
        └── BUILD.gn
```

## 7.8 示例运行

### 7.8.1 运行全部示例

```bash
# 在 user_config.xml 中设置
<build>
  <example>true</example>
</build>

# 然后运行
./start.sh
run -t UT
```

### 7.8.2 运行特定示例

```bash
# 运行 calculator 测试
run -t UT -tp PartName -tm calculator -ts CalculatorAddTest

# 运行 benchmark 测试
run -t BENCHMARK -tp PartName -tm calculator -ts benchmark_demo_test

# 运行 fuzz 测试
run -t FUZZ -tp PartName -tm calculator -ts Calculator_fuzzer
```

## 7.9 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 系统架构
- [03_Directory_Structure.md](03_Directory_Structure.md) - 目录结构
- [05_Build_System.md](05_Build_System.md) - 构建系统
- [06_Usage_Guide.md](06_Usage_Guide.md) - 使用指南
- [README_zh.md](../README_zh.md) - 原始中文文档

# 测试框架说明

## 概述

DCTS 使用 OpenHarmony 官方测试框架进行测试用例开发，涵盖多种测试框架以适应不同系统类型。

## 测试框架类型

### 框架对比

| 框架 | 系统类型 | 语言 | 基于 |
|------|----------|------|------|
| **HCTest** | Mini | C | Unity |
| **HCPPTest** | Small/Standard | C++ | GoogleTest |
| **HJSUnit** | Standard | JavaScript | Jest |

## HJSUnit 框架

### 框架结构

```
┌─────────────────────────────────────────────────────────┐
│                    HJSUnit 框架                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │                   Hypium                         │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐    │   │
│  │  │describe │ │ it      │ │ expect          │    │   │
│  │  │测试套件 │ │测试用例 │ │ 断言函数         │    │   │
│  │  └─────────┘ └─────────┘ └─────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │              测试运行器 (Test Runner)             │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │              测试报告生成器                       │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 核心 API

#### describe - 定义测试套件

```javascript
import { describe, beforeAll, beforeEach, afterEach, afterAll, it, expect } from '@ohos/hypium';

describe('TestSuiteName', function() {
  // 测试套件初始化
  beforeAll(function() {
    // 在所有测试用例前执行一次
  });
  
  beforeEach(function() {
    // 每个测试用例前执行
  });
  
  // 测试用例定义
  it('testCase001', function() {
    // 测试逻辑
  });
  
  afterEach(function() {
    // 每个测试用例后执行
  });
  
  afterAll(function() {
    // 所有测试用例后执行
  });
});
```

#### it - 定义测试用例

```javascript
// 基础测试用例
it('TestCaseName', function() {
  // 测试代码
});

// 带参数的测试用例
it('TestCaseWithParams', function() {
  // 可使用 Level, Size, TestType 标记
}, Level1, Size.SmallTest, TestType.Function);
```

#### expect - 断言函数

```javascript
// 相等断言
expect(actualValue).assertEqual(expectedValue);

// 条件断言
expect(value).assertTrue();
expect(value).assertFalse();

// 抛出异常
expect(() => {
  // 可能抛出异常的代码
}).assertThrow();

// 数组包含
expect(array).assertContain(item);
```

### 测试用例参数

#### Level - 测试级别

| Level | 说明 | 优先级 |
|-------|------|--------|
| `Level0` | Smoke冒烟测试 | 高 |
| `Level1` | Basic基本功能 | 高 |
| `Level2` | Major主要功能 | 中 |
| `Level3` | Regular常规测试 | 中 |
| `Level4` | Rare边缘测试 | 低 |

#### Size - 测试规模

| Size | 说明 |
|------|------|
| `SmallTest` | 单元测试 |
| `MediumTest` | 集成测试 |
| `LargeTest` | 系统测试 |

#### Type - 测试类型

| Type | 说明 |
|------|------|
| `Function` | 功能测试 |
| `Performance` | 性能测试 |
| `Power` | 功耗测试 |
| `Reliability` | 可靠性测试 |
| `Security` | 安全测试 |

### 完整示例

```javascript
import { describe, beforeAll, beforeEach, afterEach, afterAll, it, expect, Level, Size, TestType } from '@ohos/hypium';

describe('DistributedKvStoreTest', function() {
  // 设备管理器实例
  let deviceManager;
  
  beforeAll(async function() {
    // 初始化设备管理器
    deviceManager = await globalThis.distributedDeviceManager.createDeviceManager('bundleName');
  });
  
  beforeEach(function() {
    // 每个测试前清理状态
  });
  
  it('put_Sync_001', async function() {
    let kvStore = await factory.createKVStore(deviceManager);
    let key = 'test_key';
    let value = 'test_value';
    
    await kvStore.put(key, value);
    
    let ret = await kvStore.get(key);
    expect(ret).assertEqual(value);
  }, Level1, Size.MediumTest, TestType.Function);
  
  afterAll(function() {
    // 清理资源
  });
});
```

## HCPPTest 框架

### 框架结构

```
┌─────────────────────────────────────────────────────────┐
│                   HCPPTest 框架                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              GoogleTest (gtest)                  │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐    │   │
│  │  │TEST     │ │TEST_F   │ │ ASSERT_*        │    │   │
│  │  │基础测试 │ │带Fixture│ │ 断言宏           │    │   │
│  │  └─────────┘ └─────────┘ └─────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 核心 API

#### TEST - 定义测试用例

```cpp
#include "gtest/gtest.h"

TEST(TestSuiteName, TestCaseName) {
  // 测试代码
  EXPECT_EQ(1, 1);
  EXPECT_TRUE(true);
}
```

#### TEST_F - 带 Fixture 的测试

```cpp
class TestSuite : public testing::Test {
protected:
  static void SetUpTestCase() {
    // 套件级初始化
  }
  
  static void TearDownTestCase() {
    // 套件级清理
  }
  
  virtual void SetUp() {
    // 用例级初始化
  }
  
  virtual void TearDown() {
    // 用例级清理
  }
  
  // 共享成员
  int sharedValue;
};

TEST_F(TestSuite, TestCaseWithFixture) {
  // 使用 SetUp/TearDown
  EXPECT_EQ(sharedValue, expected);
}
```

#### 断言宏

```cpp
// 二值比较
EXPECT_EQ(val1, val2);  // 相等
EXPECT_NE(val1, val2);  // 不等
EXPECT_LT(val1, val2);  // 小于
EXPECT_LE(val1, val2);  // 小于等于
EXPECT_GT(val1, val2);  // 大于
EXPECT_GE(val1, val2);  // 大于等于

// 布尔条件
EXPECT_TRUE(condition);
EXPECT_FALSE(condition);

// 字符串比较
EXPECT_STREQ(str1, str2);
EXPECT_STRNE(str1, str2);

// 浮点数比较
EXPECT_DOUBLE_EQ(val1, val2);
EXPECT_NEAR(val1, val2, tolerance);

// 异常检测
EXPECT_THROW(statement, exception_type);
EXPECT_ANY_THROW(statement);
EXPECT_NO_THROW(statement);
```

### 完整示例

```cpp
#include "gtest/gtest.h"
#include "distributed_kv_store.h"

using namespace std;
using namespace testing::ext;

class KvStoreTest : public testing::Test {
protected:
  static void SetUpTestCase() {
    // 初始化
  }
  
  static void TearDownTestCase() {
    // 清理
  }
  
  virtual void SetUp() {
    kvStore = new DistributedKVStore();
  }
  
  virtual void TearDown() {
    delete kvStore;
  }
  
  DistributedKVStore* kvStore;
};

HWTEST_F(KvStoreTest, PutGet_001, Function | MediumTest | Level1) {
  string key = "test_key";
  string value = "test_value";
  
  kvStore->Put(key, value);
  string ret = kvStore->Get(key);
  
  EXPECT_EQ(ret, value);
}
```

## HCTest 框架

### 框架结构

```
┌─────────────────────────────────────────────────────────┐
│                   HCTest 框架                            │
│  ┌─────────────────────────────────────────────────┐   │
│  │                  Unity                          │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐    │   │
│  │  │TEST     │ │TEST_GP │ │ TEST_ASSERT_*   │    │   │
│  │  │基础测试 │ │测试组   │ │ 断言宏           │    │   │
│  │  └─────────┘ └─────────┘ └─────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### 核心 API

#### LITE_TEST_SUIT - 定义测试套件

```c
#include "hctest.h"

LITE_TEST_SUIT(test, example, IntTestSuite);
```

#### LITE_TEST_CASE - 定义测试用例

```c
LITE_TEST_CASE(IntTestSuite, TestCase001, Function | MediumTest | Level1) {
  // 测试代码
  TEST_ASSERT_EQUAL(1, 1);
};
```

#### RUN_TEST_SUITE - 注册测试套件

```c
RUN_TEST_SUITE(IntTestSuite);
```

### 断言宏

```c
// 相等断言
TEST_ASSERT_EQUAL(expected, actual);
TEST_ASSERT_EQUAL_INT(expected, actual);
TEST_ASSERT_EQUAL_STRING(expected, actual);
TEST_ASSERT_EQUAL_HEX(expected, actual);

// 条件断言
TEST_ASSERT_TRUE(condition);
TEST_ASSERT_FALSE(condition);
TEST_ASSERT_NULL(pointer);
TEST_ASSERT_NOT_NULL(pointer);

// 数值范围
TEST_ASSERT_GREATER_THAN(threshold, value);
TEST_ASSERT_LESS_THAN(threshold, value);
TEST_ASSERT_IN_RANGE(actual, min, max);

// 异常检测
TEST_ASSERT_THROWS(statement, exception);
TEST_ASSERT_THROWS_ANY(statement);
TEST_ASSERT_NOTHROW(statement);
```

## 测试用例注册模式

### JS 测试用例注册

```javascript
// List.test.js - 测试用例入口
import testsuite from './FeatureAbilityTest.test.js';

export default testsuite;
```

### 测试用例组织

```
entry/src/ohosTest/js/
├── test/
│   ├── List.test.js        # 测试用例入口
│   ├── FeatureAbilityTest.test.js  # 主测试文件
│   ├── DistributedDevice.test.js    # 分布式设备测试
│   └── ...
└── pages/
    └── index/              # 测试页面
```

## 测试运行

### 执行测试

```bash
# 编译测试套件
hb build -f -t test

# 推送到设备
hdc install ./out/xts/dcts/suites/dcts/DctsXxxTest.hap

# 执行测试
hdc shell aa test -b <bundleName> -p <packageName>
```

### 查看结果

```
# 串口日志输出
Start to run test suite: TestSuiteName
[TEST LOG] TestCase001: PASSED
[TEST LOG] TestCase002: PASSED
...
xx Tests xx Failures xx Ignored
```

---

## 最佳实践

### 测试用例设计

1. **独立性**：每个测试用例应独立运行
2. **可重复**：测试结果应一致
3. **原子性**：每个测试用例一个验证点
4. **可读性**：清晰的测试命名

### 命名规范

```javascript
// 测试套件命名
describe('ModuleNameFeatureTest', function() { });

// 测试用例命名
it('operation_State_001', function() { });
// 格式: 操作_状态_序号
```

### 断言策略

```javascript
// 推荐：单一断言
it('should_return_correct_value', function() {
  let result = calculate(1, 2);
  expect(result).assertEqual(3);
});

// 避免：多重断言
it('multiple_asserts_bad', function() {  // 不推荐
  expect(a).assertEqual(b);
  expect(c).assertEqual(d);
  expect(e).assertEqual(f);
});
```

---

## 相关文档

- [项目概览](01_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
- [模块详解](04_Modules.md)
- [构建系统](05_Build_System.md)

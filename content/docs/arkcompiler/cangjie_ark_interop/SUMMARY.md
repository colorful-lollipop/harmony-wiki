# 文档导航

## 快速索引

| 文档 | 说明 | 适用场景 |
|------|------|----------|
| [00_Overview.md](./00_Overview.md) | 项目定位与核心能力 | 新人入门 |
| [01_API_Reference.md](./01_API_Reference.md) | N-API 接口清单 | API 查阅 |
| [02_Architecture.md](./02_Architecture.md) | 内部模块与数据流 | 架构理解 |
| [03_Build_System.md](./03_Build_System.md) | GN 构建配置 | 构建调试 |
| [04_Compilation_Products.md](./04_Compilation_Products.md) | 产物清单与加载 | 运行时分析 |
| [05_Security_Review.md](./05_Security_Review.md) | 安全风险与修复 | 安全审计 |
| [06_CodeMap.md](./06_CodeMap.md) | 代码地图与导航 | 开发定位 |

## 新人阅读路线

```
1. 项目定位 → 00_Overview.md
   ↓
2. 核心概念 → 02_Architecture.md (模块职责、数据流)
   ↓
3. 接口使用 → 01_API_Reference.md (按需查阅)
   ↓
4. 构建部署 → 03_Build_System.md + 04_Compilation_Products.md
   ↓
5. 代码导航 → 06_CodeMap.md (开发时快速定位)
```

## 安全研究路线

```
1. 安全概览 → 05_Security_Review.md (信任边界、攻击面)
   ↓
2. 风险详情 → 05_Security_Review.md (7 大风险详解)
   ↓
3. 修复建议 → 05_Security_Review.md#修复建议
   ↓
4. 架构理解 → 02_Architecture.md (数据流、线程模型)
   ↓
5. 代码定位 → 06_CodeMap.md (漏洞代码路径)
```

## API 快速跳转

### 互操作核心
- [JSRuntime](./01_API_Reference.md#jsruntime) - ArkTS 运行时
- [JSContext](./01_API_Reference.md#jscontext) - 执行上下文
- [JSCallInfo](./01_API_Reference.md#jscallinfo) - 调用信息

### 类型转换
- [JSValue](./01_API_Reference.md#jsvalue-类型系列) - JS 值封装
- [JSObject](./01_API_Reference.md#jsobject) - JS 对象
- [JSArray](./01_API_Reference.md#jsarray) - JS 数组

### 工具与异常
- [BusinessException](./01_API_Reference.md#businessexception) - 业务异常
- [AsyncCallback](./01_API_Reference.md#asynccallback) - 异步回调

## 构建相关

- [编译产物清单](./04_Compilation_Products.md)
- [GN Targets 列表](./03_Build_System.md)
- [依赖关系图](./03_Build_System.md#模块依赖关系)

## 安全相关

- [攻击面清单](./05_Security_Review.md#攻击面分析)
- [风险修复建议](./05_Security_Review.md#修复建议)

## 代码导航

- [核心功能定位表](./06_CodeMap.md#核心功能代码定位)
- [API 快速跳转](./06_CodeMap.md#api-快速跳转)
- [错误码定位](./06_CodeMap.md#错误码定位)
- [快速搜索关键词](./06_CodeMap.md#快速搜索关键词)

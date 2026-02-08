# OpenHarmony Location 服务 Wiki

**项目**: base/location/location  
**版本**: OpenHarmony 位置服务组件  
**最后更新**: 2026-02-05  
**生成方式**: 自动从代码仓库生成

---

## 目录

- [概览](index.md)
- [系统架构](01_Architecture.md)
- [C/N-API 接口](02_C_NAPI.md)
- [JS API 接口](03_JS_API.md)
- [内部 API](04_Inner_API.md)
- [GN 构建配置](05_Build.md)
- [编译产物](06_Artifacts.md)
- [安全风险评审](07_Security.md)
- [常见问题](08_FAQ.md)

---

## 覆盖范围

本文档覆盖 OpenHarmony 位置服务组件的以下模块：

| 模块 | 状态 | 说明 |
|------|------|------|
| C API | ✅ 已覆盖 | `interfaces/c_api/` |
| JS API | ✅ 已覆盖 | `frameworks/js/napi/` |
| NDK | ✅ 已覆盖 | `frameworks/native/location_ndk/` |
| Inner API | ✅ 已覆盖 | `interfaces/inner_api/` |
| SA 服务 | ✅ 已覆盖 | `services/` |
| 框架层 | ✅ 已覆盖 | `frameworks/` |

### 未覆盖范围

- 测试代码 (`test/` 目录)
- 单元测试和模糊测试
- 文档中已明确排除的示例代码

---

## 文档更新方式

### 何时更新本文档

当发生以下变更时，应更新本文档：

1. 新增、删除或修改 N-API 接口
2. 新增或删除 SA 服务
3. 构建配置发生变更（`bundle.json`, `config.gni`）
4. 安全机制发生变更
5. 新增重大功能模块

### 更新步骤

1. **更新代码事实记录** (`_work/NOTES.md`)
   - 记录新增/修改的符号名和文件路径
   - 记录新的 API 接口签名
   - 记录权限或安全相关的变更

2. **更新对应章节**
   - API 接口变更 → 更新 `02_C_NAPI.md` 或 `03_JS_API.md`
   - 构建配置变更 → 更新 `05_Build.md`
   - 安全机制变更 → 更新 `07_Security.md`

3. **更新导航** (`SUMMARY.md`)
   - 确保新增章节已加入导航

4. **验证文档完整性**
   - 检查链接有效性
   - 验证代码证据引用正确
   - 确保术语一致性

---

## 代码证据引用规范

本文档所有关键结论均基于代码证据，引用格式如下：

- **文件路径**: `path/to/file:line`
- **符号名**: 类名/函数名/宏名
- **代码片段**: 最小必要代码片段

示例：
```cpp
// 文件: services/location_locator/locator/source/locator_ability.cpp:74
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    LocatorAbility::GetInstance());
```

---

## 反馈与贡献

如发现文档错误或遗漏，请通过以下方式反馈：

1. 在代码仓库提 Issue
2. 更新 `wiki/_work/NOTES.md` 记录发现
3. 提交 PR 修改对应文档

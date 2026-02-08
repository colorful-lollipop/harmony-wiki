# 常见问题 (FAQ)

> Sandbox Manager 构建、运行与调试常见问题解答

---

## 编译问题

### Q1: 编译时报错 "cannot find -l..."
```
error: cannot find -lrelational_store
```

**解决方案**：
确保依赖组件已先编译：
```bash
./build.sh --product-name {product} --build-target relational_store
./build.sh --product-name {product} --build-target sandbox_manager
```

---

### Q2: 头文件找不到
```
fatal error: 'sandbox_manager_kit.h' file not found
```

**解决方案**：
1. 检查 `bundle.json` 中 `inner_kits` 配置是否正确
2. 确认 include path 包含 `interfaces/inner_api/sandbox_manager/include`
3. 清理并重新构建：
```bash
rm -rf out/
./build.sh --product-name {product} --build-target sandbox_manager
```

---

### Q3: SA 注册失败
```
E/SAM_GR: RegisterToSaMap failed, result = -1
```

**解决方案**：
1. 检查 SA 配置文件是否存在：
```
system/etc/sandbox_manager/sandbox_manager_sa_profile.xml
```
2. 确认配置文件中的 SA ID 与代码一致
3. 检查 `sandbox_manager_sa_profile` 目标是否正确构建

---

## 运行问题

### Q4: 服务无法启动
```
SandboxManagerService::OnStart() failed
```

**排查步骤**：
1. 检查日志：`hilog | grep SandboxManager`
2. 确认 RDB 数据库初始化成功
3. 确认 MAC 设备 `/dev/dec` 存在且可访问
4. 检查权限：服务进程是否有足够权限

---

### Q5: IPC 调用超时
```
SandboxManagerKit::SetPolicy() timeout
```

**可能原因**：
1. MAC 层繁忙（大批量操作）
2. 服务处于延迟卸载状态
3. 系统资源不足

**解决方案**：
1. 使用 `CallProxyWithRetry()` 的重试机制
2. 减少单次批量操作的数量
3. 增加服务保活机制

---

## 调试问题

### Q6: 如何查看策略状态？

```cpp
// 检查 MAC 层策略状态
hilog | grep -i "sandbox"

 // 使用 CheckPolicy API
SandboxManagerKit::CheckPolicy(tokenId, policies, results);
```

---

### Q7: 如何开启调试日志？

```cpp
// 在代码中添加
SANDBOXMANAGER_LOG_DEBUG(LABEL, "Debug message: %{public}s", info.c_str());
```

---

### Q8: 如何进行单元测试？

```bash
# 构建单元测试
./build.sh --product-name {product} --build-target sandbox_manager_build_module_test

# 运行测试
# 测试二进制通常位于 out/.../tests/
```

---

## 使用问题

### Q9: 策略不生效？

**排查步骤**：
1. 检查权限：`CheckPermission()` 返回 true
2. 检查路径：`AdjustPath()` 规范化后是否符合要求
3. 检查策略类型：`SELF_PATH` 需要匹配 bundle name
4. 检查 MAC 状态：`IsMacSupport()` 返回 true

---

### Q10: 路径验证失败？

**常见原因**：
1. 路径不以 `/storage/Users/` 开头
2. 路径深度不足（过于顶层）
3. 包含非法字符
4. SELF_PATH 类型但 bundle name 不匹配

**解决方案**：
```cpp
// 使用规范化路径
std::string normalizedPath = AdjustPath(userInputPath);

// 检查 SELF_PATH 的 bundle name
if (policy.type == SELF_PATH) {
    if (!CheckPathWithinBundleName(path, bundleName, components)) {
        return INVALID_PATH;
    }
}
```

---

*文档更新时间: 2025-02-07*

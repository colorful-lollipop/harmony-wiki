# 安全风险分析

## CVE 状况

### 当前版本信息

| 信息项 | 值 |
|-------|-----|
| README.OpenSource 版本 | 3.7 |
| bundle.json 版本 | 3.1（疑似过时） |
| 上游最新版本 | 需要查询 |

### 已知 CVE 列表

> **TODO**: 需要进一步查询 CVE 数据库获取 SELinux 3.7 版本的已知漏洞

由于 CVE 数据库需要实时查询，以下为分析框架：

#### 查询建议

1. **NVD (National Vulnerability Database)**
   - 搜索关键词: `selinux`
   - 版本过滤: 3.7 及以下
   - URL: https://nvd.nist.gov/

2. **CVE Details**
   - URL: https://www.cvedetails.com/product/17169/SELinux-SELinux.html

3. **Red Hat Security Tracker**
   - URL: https://access.redhat.com/security/cve

### 历史 CVE 示例（仅供参考）

以下为 SELinux 历史上的 CVE 类型，供升级时参考：

| CVE 类型 | 影响 | 修复方式 |
|---------|------|---------|
| 策略绕过 | 恶意应用可能绕过 SELinux 限制 | 更新策略 + 库升级 |
| 提权漏洞 | 本地权限提升 | 库升级 |
| 信息泄露 | 获取敏感策略信息 | 库升级 |
| DoS | 拒绝服务 | 库升级 |

---

## OH Patch 引入的新攻击面

### 1. OHOS_FC_INIT 宏

**新增攻击面**：多文件 file_contexts 加载

**风险分析**：
- **风险等级**：低
- **原因**：
  - 文件路径通过配置传入，非外部可控
  - 文件解析逻辑与上游相同
  - 内存分配已做边界检查

**安全建议**：
- 确保 file_contexts 文件只可由 root 写入
- 验证文件在系统分区，防止被篡改

### 2. app_allow_config 模块

**新增攻击面**：白名单配置文件解析

**风险分析**：
- **风险等级**：中
- **潜在问题**：
  - 路径前缀匹配可能导致意外匹配
  - 配置文件格式错误可能导致越界

**代码审查**：

```c
// app_allow_config.c:90
if (strncmp(pathname, allow_path, strlen(allow_path)) == 0) {
    return true;
}
```

**潜在问题**：
- 如果 `allow_path` 是 `/data/app`，会匹配 `/data/app` 和 `/data/application`
- 实际配置中应使用完整路径或正确分隔

**安全建议**：
- 配置文件中路径应精确，避免过于宽泛的前缀
- 建议路径以 `/` 结尾（代码会自动去除）
- 配置文件权限应设置为 644 或 600

**配置文件安全示例**：
```
# ✅ 安全的配置（精确路径）
/data/app/com.example.app1
/data/app/com.example.app2
/data/vendor/specific-app

# ⚠️ 有风险的配置（过于宽泛）
/data/app          # 会匹配 /data/application 等
/data              # 会匹配所有 /data 下的路径
```

### 3. BUILD.gn 编译选项

**风险分析**：

| 选项 | 风险 | 说明 |
|-----|------|-----|
| `-w` | 中 | 禁用所有警告，可能隐藏潜在问题 |
| `-fno-lto` | 低 | 禁用 LTO，不影响安全性 |
| `-U__BIONIC__` | 低 | 正确行为，避免 Android 特定代码 |

**安全建议**：
- 考虑在调试版本中保留警告信息
- 发布版本可继续禁用警告以减少噪音

---

## 建议的安全升级策略

### 短期（1-3 个月）

1. **版本号同步**
   - 确认 bundle.json 版本号（3.1）与 README.OpenSource（3.7）不一致的原因
   - 更新 bundle.json 以反映真实版本

2. **CVE 扫描**
   - 查询 SELinux 3.7 已知 CVE
   - 评估对 OH 的影响

3. **配置文件审计**
   - 审计 `app_allow_cfg` 文件内容
   - 确保没有过于宽泛的路径规则

### 中期（3-6 个月）

1. **上游版本跟踪**
   - 关注上游 SELinux 发布
   - 评估 3.7 → 最新版 的升级成本

2. **安全加固**
   - 考虑启用编译器安全选项（如 -fstack-protector）
   - 审查 init 阶段 SELinux 初始化流程

### 长期（6-12 个月）

1. **版本升级**
   - 跟踪上游安全修复
   - 制定升级计划

2. **自动化安全检测**
   - 集成 CVE 扫描到 CI/CD
   - 静态代码分析

---

## 安全配置建议

### file_contexts 文件权限

```bash
# 建议权限设置
-rw-r--r-- root root /system/etc/file_contexts
-rw-r--r-- root root /system/etc/selinux/app_allow_cfg
```

### SELinux 模式建议

| 场景 | 建议模式 | 说明 |
|-----|---------|-----|
| 正式发布 | Enforcing | 强制访问控制 |
| 开发调试 | Permissive | 记录但不阻止 |
| 紧急恢复 | Disabled | 仅用于问题诊断 |

### 调试与生产环境差异

**开发版本**：
- 可启用 SELinux 日志
- 可临时切换 Permissive 模式
- 保留编译警告

**生产版本**：
- Enforcing 模式
- 最小化日志
- 禁用调试接口

---

## 安全测试建议

### 1. 策略一致性测试

```bash
# 使用 checkpolicy 验证策略
checkpolicy -c -M policy.conf -o policy.bin
```

### 2. 标签恢复测试

```bash
# 验证 restorecon 不修改白名单路径
restorecon -R /data/app/
# 检查白名单应用的数据标签是否保持不变
```

### 3. 权限边界测试

```bash
# 验证应用沙箱隔离
# 尝试跨应用访问数据目录（应被拒绝）
```

### 4. 模糊测试

- 使用上游 SELinux 的 fuzzing 工具
- 重点关注 file_contexts 解析
- 测试 app_allow_config 解析边界情况

---

## 参考资源

### CVE 查询

- NVD: https://nvd.nist.gov/
- CVE Details: https://www.cvedetails.com/
- Red Hat Security: https://access.redhat.com/security/

### SELinux 安全资源

- SELinux Project Security: https://github.com/SELinuxProject/selinux/security
- SELinux Wiki: https://github.com/SELinuxProject/selinux/wiki

### OH 安全文档

- OpenHarmony 安全指南
- OpenHarmony SELinux 策略文档

---

## TODO

- [ ] 查询 SELinux 3.7 版本的 CVE 列表
- [ ] 审计实际使用的 `app_allow_cfg` 配置
- [ ] 确认 bundle.json 版本号与 README.OpenSource 不一致的原因
- [ ] 制定 CVE 扫描自动化流程
- [ ] 评估升级到上游最新版本的计划


# ncurses 安全风险分析

## 已知 CVE 分析

### CVE-2023-29491 (已修复)

| 属性 | 内容 |
|------|------|
| **CVE ID** | CVE-2023-29491 |
| **公开日期** | 2023年4月 |
| **影响版本** | ncurses < 6.4-20230408 |
| **OH 影响版本** | 已修复 (backport-0002-CVE-2023-29491-env-access.patch) |
| **严重程度** | 中等 (CVSS 评分约 5.5) |
| **攻击向量** | 本地 |

#### 漏洞详情

**问题描述**:
`_nc_env_access()` 函数在处理 `TERMINFO` 和 `TERMINFO_DIRS` 环境变量时，对 root 用户有特殊处理逻辑。这种不一致的处理方式可能导致安全漏洞。

**漏洞代码** (修复前):
```c
// ncurses/tinfo/access.c
_nc_env_access(void)
{
    bool result = TRUE;
    // ... 其他检查 ...
    
#if !defined(USE_ROOT_ENVIRON)
    if ((getuid() == ROOT_UID) || (geteuid() == ROOT_UID)) {
        result = FALSE;  // root 用户不能使用环境变量
    }
#endif
    
    return result;
}
```

**安全风险**:
1. **权限绕过**: 攻击者可能利用 root 检查逻辑的缺陷
2. **环境变量注入**: root 用户在特定情况下可能加载恶意的 terminfo 数据库
3. **信息泄露**: 不一致的处理可能导致敏感信息泄露

**修复方案**:
```c
// 修复后 - 移除 root 特殊处理
_nc_env_access(void)
{
    bool result = TRUE;
    // ... 其他检查 ...
    
    // 移除了对 root 的特殊处理
    // 所有用户统一处理环境变量访问
    
    return result;
}
```

**修复效果**:
- ✅ 统一了环境变量访问策略
- ✅ 消除了 root 检查的安全隐患
- ✅ 符合最小权限原则

#### 修复状态

- **上游修复**: 已在 ncurses 6.4-20230408 及更高版本中修复
- **OH 修复**: 通过 `backport-0002-CVE-2023-29491-env-access.patch` 回迁到 6.5
- **当前状态**: ✅ **已修复**

---

## 历史 CVE 回顾

### ncurses 6.5 之前的其他 CVE

| CVE ID | 影响版本 | 严重程度 | 描述 | OH 6.5 状态 |
|--------|----------|----------|------|-------------|
| CVE-2022-29458 | < 6.3 | 中等 | 栈缓冲区溢出 | ✅ 已修复 (6.5 > 6.3) |
| CVE-2021-39537 | < 6.2 | 中等 | 缓冲区溢出 | ✅ 已修复 (6.5 > 6.2) |
| CVE-2019-17594 | < 6.1 | 低 | 堆缓冲区溢出 | ✅ 已修复 (6.5 > 6.1) |
| CVE-2019-17595 | < 6.1 | 低 | 堆缓冲区溢出 | ✅ 已修复 (6.5 > 6.1) |
| CVE-2018-19217 | < 6.1 | 低 | 空指针解引用 | ✅ 已修复 (6.5 > 6.1) |
| CVE-2017-10684 | < 6.0 | 高 | 缓冲区溢出 | ✅ 已修复 (6.5 > 6.0) |
| CVE-2017-10685 | < 6.0 | 高 | 缓冲区溢出 | ✅ 已修复 (6.5 > 6.0) |

### 版本升级优势

OpenHarmony 使用 **ncurses 6.5** (2024年4月发布)，具有以下安全优势：

1. **最新稳定版本**: 包含截至 2024 年的所有安全修复
2. **长期支持**: 上游活跃维护，持续修复安全漏洞
3. **成熟代码库**: 经过 30+ 年发展，安全性较高

---

## OH Patch 引入的新攻击面

### Patch 安全评估

| Patch | 是否引入新攻击面 | 风险等级 | 说明 |
|-------|-----------------|----------|------|
| ncurses-kbs.patch | ❌ 否 | 无 | 仅终端定义变更 |
| ncurses-urxvt.patch | ❌ 否 | 无 | 仅新增终端定义 |
| ncurses-libs.patch | ❌ 否 | 低 | 仅构建配置变更 |
| ncurses-config.patch | ❌ 否 | 低 | 仅配置脚本变更 |
| cross_compile_support_ohos.patch | ❌ 否 | 无 | 仅平台支持 |
| CVE-2023-29491.patch | ❌ 否 | 无 | 安全修复 |

**结论**: OpenHarmony 的 ncurses Patch **未引入新的安全风险**。

### 详细分析

#### 终端类型 Patch (kbs, urxvt)

**变更**: 修改 `misc/terminfo.src`

**安全分析**:
- ✅ 仅添加/修改终端能力定义
- ✅ 不涉及代码执行路径
- ✅ 无缓冲区操作
- ✅ 风险：无

#### 构建系统 Patch (libs, config)

**变更**: 修改 Makefile 和配置脚本

**安全分析**:
- ✅ 仅构建时影响
- ✅ 不修改运行时行为
- ✅ 移除而非添加功能
- ✅ 风险：极低

#### 平台适配 Patch (cross_compile)

**变更**: 添加 ohos 平台支持

**安全分析**:
- ✅ 仅添加平台识别
- ✅ 不修改安全相关代码
- ✅ 风险：无

---

## 安全使用建议

### 对于应用开发者

#### 1. 输入验证

```c
// 始终验证用户输入，避免格式字符串漏洞
char user_input[256];
// ... 获取输入 ...

// ✅ 安全: 使用固定格式字符串
printw("%s", user_input);

// ❌ 不安全: 直接使用用户输入作为格式字符串
// printw(user_input);  // 可能导致格式字符串攻击
```

#### 2. 环境变量处理

```c
// CVE-2023-29491 修复后，注意环境变量的使用
const char *term_info = getenv("TERMINFO");
if (term_info != NULL) {
    // 验证路径合法性
    if (strncmp(term_info, "/system/", 8) != 0) {
        // 非系统路径，可能需要额外检查
        log_warning("Using non-system terminfo path: %s", term_info);
    }
}
```

#### 3. 缓冲区管理

```c
// ncurses 函数处理多字节字符时，确保缓冲区足够大
wchar_t wbuf[256];
// 使用 mvwgetn_wstr 限制输入长度
mvwgetn_wstr(win, y, x, wbuf, sizeof(wbuf)/sizeof(wbuf[0]) - 1);
```

### 对于系统维护者

#### 1. 定期安全更新

```bash
# 监控 ncurses 安全公告
# 上游安全页面: https://invisible-island.net/ncurses/ncurses.faq.html

# 检查当前版本
strings /system/lib/libncurses.so | grep "ncurses 6"

# 验证 CVE 修复
readelf -s /system/lib/libncurses.so | grep _nc_env_access
```

#### 2. terminfo 数据库安全

```bash
# 确保 terminfo 数据库来源可信
ls -la /system/share/terminfo/

# 验证文件完整性
# 建议使用只读文件系统挂载 terminfo 目录
```

#### 3. 权限配置

```bash
# ncurses 库文件权限建议
chmod 644 /system/lib/libncurses.so*
chown root:root /system/lib/libncurses.so*

# terminfo 数据库权限
chmod 644 /system/share/terminfo/*/*
chown -R root:root /system/share/terminfo/
```

---

## 安全升级策略

### 升级检查清单

当上游发布 ncurses 新版本时：

- [ ] 检查上游安全公告
- [ ] 查看 ChangeLog 中的安全相关变更
- [ ] 评估升级影响
- [ ] 重新应用 OH 特有 Patch
- [ ] 在测试环境验证
- [ ] 监控升级后的异常行为

### 版本升级路径

```
当前版本: 6.5
    │
    ▼
监控上游发布 ──► 新版本发布 ──► 安全评估
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
                仅安全更新      功能更新         重大版本
                    │               │               │
                    ▼               ▼               ▼
                立即升级       评估后升级      详细测试后升级
```

### 应急响应

如发现新的 ncurses 安全漏洞：

1. **评估影响**: 确认漏洞是否影响 OH 版本
2. **临时缓解**: 如可能，实施临时防护措施
3. **准备补丁**: 获取或开发修复补丁
4. **测试验证**: 在测试环境验证修复
5. **部署更新**: 发布安全更新

---

## 安全监控

### 监控资源

| 资源 | 用途 | 链接 |
|------|------|------|
| MITRE CVE | CVE 数据库 | https://cve.mitre.org/ |
| NVD | 美国国家漏洞库 | https://nvd.nist.gov/ |
| ncurses 主页 | 官方公告 | https://invisible-island.net/ncurses/ |
| ncurses 邮件列表 | 安全通知 | https://lists.gnu.org/mailman/listinfo/bug-ncurses |

### 自动化监控建议

```bash
# 创建监控脚本
#!/bin/bash
# check_ncurses_security.sh

CURRENT_VERSION="6.5"
# 查询 NVD API 获取最新 CVE
# curl -s "https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch=ncurses"

# 检查本地库版本
LOCAL_VERSION=$(strings /system/lib/libncurses.so | grep -o "ncurses [0-9]\+\.[0-9]\+" | head -1)

if [ "$LOCAL_VERSION" != "ncurses $CURRENT_VERSION" ]; then
    echo "WARNING: Version mismatch detected"
fi
```

---

## 总结

### 安全状况评估

```
┌────────────────────────────────────────────────────────────┐
│                ncurses 6.5 安全状况总结                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  已知 CVE:                                                 │
│  ┌──────────────────────────────────────────────────┐    │
│  │  • CVE-2023-29491: 已修复 ✅                       │    │
│  │  • 历史 CVE: 全部已修复 ✅                         │    │
│  └──────────────────────────────────────────────────┘    │
│                                                            │
│  Patch 安全性:                                             │
│  ┌──────────────────────────────────────────────────┐    │
│  │  • 6 个 OH Patch 均未引入新攻击面 ✅               │    │
│  │  • 无新增安全风险                                  │    │
│  └──────────────────────────────────────────────────┘    │
│                                                            │
│  整体评级:                                                 │
│  ┌──────────────────────────────────────────────────┐    │
│  │  安全 ✅ 推荐使用                                  │    │
│  └──────────────────────────────────────────────────┘    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 关键结论

1. **当前版本安全**: ncurses 6.5 包含所有已知安全修复
2. **CVE-2023-29491 已修复**: 通过 Patch 成功回迁安全修复
3. **OH Patch 安全**: 未引入新的安全风险
4. **建议**: 持续监控上游安全公告，及时更新

OpenHarmony 的 ncurses 集成是**安全且维护良好的**，可以放心使用。

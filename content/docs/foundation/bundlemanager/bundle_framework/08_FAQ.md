# 常见问题

## 构建问题

### Q1: hb 工具未找到

**问题**: `hb: command not found`

**解决方案**:
```bash
# 安装 hb 工具
pip3 install hb

# 或从源码安装
cd /path/to/ohos-sdk
./build.sh --build_only_tools
```

### Q2: product name 无效

**问题**: 构建时提示 `product not found`

**解决方案**:
1. 检查可用的 product 列表:
```bash
hb list
```

2. 使用正确的 product name:
```bash
./build.sh --product-name rk3568 --build-target bundle_framework
```

### Q3: 依赖缺失

**问题**: 编译失败，提示依赖模块不存在

**解决方案**:
```bash
# 先构建依赖
./build.sh --product-name <product> --build-target <dependency>

# 然后构建 bundle_framework
./build.sh --product-name <product> --build-target bundle_framework
```

### Q4: 编译超时

**问题**: 编译时间过长

**解决方案**:
```bash
# 使用并行编译
./build.sh --product-name <product> -j <jobs>

# 只构建 bundle_framework
./build.sh --product-name <product> --build-target bundle_framework
```

---

## 运行时问题

### Q5: SA 401 启动失败

**问题**: BundleMgrService 无法启动

**日志检查**:
```bash
hdc shell hilog -T BMS
```

**常见原因**:
1. 依赖 SA 未启动 (SA 3503)
2. 配置文件损坏
3. 权限不足

**解决方案**:
```bash
# 检查 SA 状态
hdc shell hidumper -s samgr

# 重启 foundation 进程
hdc shell killall foundation
```

### Q6: SA 511 未自动启动

**问题**: InstalldService 延迟启动导致安装失败

**解决方案**:
InstalldService 配置为按需启动 (run-on-create: false)，首次调用时会自动启动。
如果启动失败，检查:
```bash
# 检查进程状态
hdc shell ps -A | grep installs

# 手动启动服务
hdc shell startserv installs
```

### Q7: IPC 调用超时

**问题**: API 调用返回超时错误

**常见原因**:
1. 服务端繁忙
2. 内存不足
3. Binder 线程池满

**解决方案**:
```bash
# 查看系统资源
hdc shell top
hdc shell memory

# 增加 Binder 线程数（需要系统修改）
```

---

## 权限问题

### Q8: 权限校验失败

**问题**: 调用 API 返回 `PERMISSION_DENIED`

**检查清单**:
1. 是否在 `module.json5` 中声明了所需权限
2. 是否是系统应用（某些权限仅限系统应用）
3. 权限名称是否正确

**示例**:
```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.GET_BUNDLE_INFO_PRIVILEGED",
        "reason": "Need to query privileged bundle info",
        "usedScene": {
          "abilities": ["MainAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

### Q9: 签名校验失败

**问题**: 安装 HAP 时签名验证失败

**解决方案**:
1. 检查 HAP 是否正确签名
2. 确认签名证书有效
3. 检查签名算法是否支持

```bash
# 查看签名信息
hdc shell bm dump -n <bundleName>
```

---

## 调试方法

### 日志查看

**BMS 日志**:
```bash
hdc shell hilog -T BMS
```

** Installd 日志**:
```bash
hdc shell hilog -T Installd
```

**所有相关日志**:
```bash
hdc shell hilog | grep -E "(BMS|Installd|Bundle)"
```

### 调试命令

**查看已安装应用**:
```bash
hdc shell bm dump -a
```

**查看应用信息**:
```bash
hdc shell bm dump -n <bundleName>
```

**清除应用数据**:
```bash
hdc shell bm uninstall -n <bundleName>
```

**重新安装应用**:
```bash
hdc shell bm install -p <hapPath>
```

### 系统能力检查

**查看 SA 列表**:
```bash
hdc shell hidumper -s samgr -a "-s"
```

**查看特定 SA**:
```bash
hdc shell hidumper -s 401
hdc shell hidumper -s 511
```

---

## 性能问题

### Q10: 安装速度慢

**优化建议**:
1. 启用并行解压 (如果支持)
2. 使用 SSD 存储
3. 减少日志级别

### Q11: 内存占用高

**检查方法**:
```bash
hdc shell cat /proc/<pid>/status | grep VmRSS
```

**优化建议**:
1. 启用 RDB 延迟写入
2. 定期清理缓存

---

## 其他问题

### Q12: 与其他子系统集成问题

**问题**: 无法查询其他子系统的能力

**解决方案**:
1. 检查依赖是否正确配置 (`bundle.json`)
2. 确认子系统是否已初始化
3. 查看 IPC 连接状态

### Q13: 多用户支持

**问题**: 在多用户环境下行为异常

**解决方案**:
```bash
# 查看当前用户
hdc shell bm get-hap-info <bundleName> | grep userId

# 指定用户操作
hdc shell bm dump -n <bundleName> -u <userId>
```

---

## 延伸阅读

- [架构说明](02_Architecture.md)
- [安全风险评审](07_Security_Review.md)
- [GN 构建](05_GN_Build.md)

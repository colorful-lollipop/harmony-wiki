# 常见构建/运行/调试问题与定位路径

## 文档信息

- **目的**：提供 vendor_hihope 仓库的常见问题、排查方法和调试技巧
- **适用范围**：vendor_hihope 仓库的所有产品线
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库主要涉及配置和适配层
  - 常见问题集中在构建配置、HDF 加载、权限配置等
  - 提供了基于代码的排查路径和日志分析方法

## 构建问题

### 1. 构建配置问题

#### 问题：找不到 target

**症状**：
```
ERROR at //line:column:column: message
ninja: error: unknown target 'xxx'
```

**可能原因**：
1. **target 名称错误**：BUILD.gn 中引用的 target 不存在
2. **依赖未定义**：deps 中引用的 target 未定义
3. **路径错误**：文件路径不正确

**排查方法**：
```bash
# 1. 检查 target 定义
gn desc out/{product} path/to/BUILD.gn

# 2. 检查依赖关系
gn check path/to/BUILD.gn --check

# 3. 查看所有 targets
gn ls out/{product}
```

**修复建议**：
- 检查 target 拼写是否正确
- 确保依赖的 target 已在其他地方定义
- 验证 include_dirs 是否包含必要的头文件

#### 问题：类型错误

**症状**：
```
ERROR at //vendor/hihope/xxx/BUILD.gn:line:column:column
Type "xxx" does not exist.
```

**可能原因**：
1. **OpenHarmony 模板拼写错误**
2. **自定义类型未声明**
3. **参数数量不匹配**

**排查方法**：
```bash
# 1. 检查 BUILD.gn 语法
gn format path/to/BUILD.gn

# 2. 检查模板使用
# 确认使用的是 ohos_* 模板而非原生 GN 模板
```

**修复建议**：
- 参考 OpenHarmony GN 编码规范
- 检查官方文档中的模板定义
- 确保 template 参数正确

#### 问题：链接错误

**症状**：
```
undefined reference to "symbol_name"
```

**可能原因**：
1. **依赖的库导出符号缺失**
2. **deps/external_deps 配置错误**
3. **头文件未找到**

**排查方法**：
```bash
# 1. 检查符号导出
nm -D out/{product}/libs/*.so | grep symbol_name

# 2. 检查库依赖
readelf -d out/{product}/libs/*.so
```

**修复建议**：
- 检查 external_deps 中的库路径是否正确
- 确保依赖库已正确安装
- 检查库的导出符号列表

#### 问题：配置文件错误

**症状**：
```
ERROR: Parse error in {product}/hals/audio/audio_paths.json
```

**可能原因**：
1. **JSON 语法错误**
2. **JSON 格式错误**
3. **必需字段缺失**

**排查方法**：
```bash
# 1. 验证 JSON 格式
python3 -m json.tool {product}/hals/audio/audio_paths.json

# 2. 检查 JSON 语法
cat {product}/hals/audio/audio_paths.json | python3 -m json.tool
```

**修复建议**：
- 使用 JSON 验证工具
- 参考示例配置文件格式
- 确保所有字段都正确闭合

#### 问题：HCS 配置错误

**症状**：
```
ERROR: Failed to load HDF config file
```

**可能原因**：
1. **HCS 语法错误**
2. **文件路径错误**
3. **缺少必需的 include**

**排查方法**：
```bash
# 1. 检查 HCS 语法
hcs -o device_info.hcs {product}/hdf_config/khdf/device_info.hcs

# 2. 检查文件存在
ls -l {product}/hdf_config/khdf/device_info.hcs

# 3. 查看 HDF 日志
hdc shell
hilog -T HDF | grep "error"
```

**修复建议**：
- 参考 HDF 配置文档
- 检查 include 路径是否正确
- 确保所有引用的 .hcs 文件都存在

## 运行时问题

### 2. HDF 服务未找到

**症状**：
```bash
# 尝试访问 HDF 服务
hdc shell
hdf -s service_name

# 错误信息
service not found
```

**可能原因**：
1. **服务未加载**：priority 设置错误导致服务未加载
2. **配置文件错误**：HCS 配置解析失败
3. **驱动 .so 文件缺失**：驱动库文件不存在

**排查方法**：
```bash
# 1. 检查 HDF 服务状态
hdf -h

# 2. 检查特定服务
hdf -s {service_name}

# 3. 查看加载的驱动
hdf -h

# 4. 查看 HDF 日志
hdc shell
hilog -T HDF
```

**修复建议**：
- 检查 device_info.hcs 中的 priority 设置
- 确认驱动库文件已安装
- 重新启动设备

### 3. 权限被拒绝

**症状**：
```
Permission denied
```

**可能原因**：
1. **应用未声明权限**：config.json 中缺少权限配置
2. **权限级别不足**：normal 级别应用请求了 system 级别权限
3. **签名不匹配**：应用签名与预安装配置不匹配

**排查方法**：
```bash
# 1. 检查应用权限
hdc shell
bm app -a bundle_name

# 2. 查看 HAP 包权限
bm dump -a bundle_name
```

**修复建议**：
- 确保应用在 install_list_permissions.json 中声明了所需权限
- 检查 app_signature 是否正确
- 参考 OpenHarmony 权限文档

## 调试技巧

### 1. 日志系统使用

#### OpenHarmony 日志（hilog）

```bash
# 查看 HDF 驱动日志
hdc shell
hilog -T HDF

# 查看应用日志
hdc shell
hilog -T App

# 持续监控日志
hdc shell hilog -T App | tail -f

# 按级别过滤
hdc shell hilog -T HDF | grep ERROR
hdc shell hilog -T HDF | grep WARN
```

#### 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|---------|
| **DEBUG** | 调试信息 | 开发调试 |
| **INFO** | 一般信息 | 正常运行状态 |
| **WARN** | 警告信息 | 可能的问题 |
| **ERROR** | 错误信息 | 需要处理 |
| **FATAL** | 致命错误 | 系统崩溃 |

### 2. HDF 调试命令

```bash
# 列出所有 HDF 服务
hdf -h

# 查看服务详细信息
hdf -s {service_name}

# 列出设备节点
hdf -d

# 强制重载服务
hdf -s {service_name}
```

### 3. GN 构建调试

```bash
# 分析依赖关系
gn analyze out/{product} path/to/target

# 输出依赖图
gn graph path/to/BUILD.gn --output=out/graph.dot

# 查看依赖路径
gn path out/{product} path/to/target
```

### 4. 常见调试场景

#### 场景 1：修改配置后未生效

**症状**：修改了 config.json 或 HCS 配置，但重启后未生效

**排查步骤**：
1. 检查配置文件语法
2. 确认配置文件位于正确位置
3. 清理构建产物：`hb clean`
4. 重新编译：`hb build`
5. 重新烧录镜像

#### 场景 2：HDF 服务启动失败

**症状**：系统启动后，某些 HDF 服务未正常运行

**排查步骤**：
1. 查看 HDF 日志：`hilog -T HDF`
2. 检查服务状态：`hdf -h`
3. 检查驱动 .so 文件：`ls -l /vendor/lib/`
4. 检查设备节点：`ls -l /dev/`
5. 验证 HCS 配置：`hcs -o device_info.hcs`

#### 场景 3：权限不足

**症状**：应用运行时提示权限不足

**排查步骤**：
1. 检查 install_list_permissions.json 配置
2. 验证应用签名：检查 app_signature
3. 查看应用权限：`bm app -a {bundle_name}`
4. 重新安装应用：如果需要修改权限

## 问题定位路径

### 1. 文档路径

| 问题类型 | 相关文档 | 说明 |
|---------|---------|------|
| **构建问题** | [GN Targets 说明](06_GN_Targets.md) | BUILD.gn 和依赖问题 |
| **HDF 配置** | [HDF 配置详解](04_HDF_Configuration.md) | HCS 配置和驱动加载 |
| **权限问题** | [安全风险评审](08_Security_Review.md) | 权限和安全配置 |
| **HAL 问题** | [HAL 实现说明](03_HAL_Implementation.md) | HAL 接口和实现 |

### 2. 代码证据定位

使用提供的代码证据路径和行号来定位问题：

**示例**：
- HDF 配置问题：参考 `rk3568/hdf_config/khdf/device_info.hcs:53`
- 权限配置问题：参考 `rk3568/security_config/high_privilege_process_list.json:1`
- 构建问题：参考 `{product}/BUILD.gn` 文件

### 3. 获取更多帮助

**官方文档**：
- OpenHarmony 构建文档：https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/subsystems/subsys-build-gn
- OpenHarmony HDF 开发文档：https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/driver/driver-overview-foundation.md

**社区支持**：
- Gitee Issue：https://gitee.com/openharmony/issues
- 开发者论坛

## 快速故障排查清单

### 构建问题清单

- [ ] 配置文件语法是否正确
- [ ] 依赖的 target 是否都存在
- [ ] include_dirs 是否正确
- [ ] OpenHarmony 模板是否正确使用
- [ ] external_deps 配置是否正确

### 运行时问题清单

- [ ] HDF 服务是否正常加载
- [ ] 设备节点是否正确创建
- [ ] 权限配置是否正确应用
- [ ] 日志中是否有错误信息
- [ ] 应用签名是否正确配置

### 安全问题清单

- [ ] 高权限进程配置是否合理
- [ ] SELinux 策略是否启用
- [ ] Seccomp 过滤是否启用
- [ ] 预安装应用权限是否符合最小权限原则
- [ ] 设备权限配置是否最小化

## 相关跳转

- [返回 Wiki 首页](SUMMARY.md)
- [产品定位详解](01_Product_Positioning.md)
- [目录结构详解](02_Directory_Structure.md)

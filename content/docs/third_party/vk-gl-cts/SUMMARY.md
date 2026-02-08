# VK-GL-CTS OpenHarmony Wiki - 阅读路线建议

## 快速导航

根据您的角色和目的，选择不同的阅读路线：

---

## 路线 1：初次了解（5分钟）

适合：首次接触该库的开发者

### 阅读顺序

1. **[README.md](./README.md)**
   - 库概览
   - 核心信息
   - 文档导航

2. **[01_Overview.md](./01_Overview.md)** 第 1-2 节
   - 基础信息
   - 在 OH 中的作用

### 关键收获

- VK-GL-CTS 是什么
- 为什么 OpenHarmony 需要它
- 与传统第三方库的不同之处

---

## 路线 2：适配开发（20分钟）

适合：需要理解或修改适配代码的开发者

### 阅读顺序

1. **[01_Overview.md](./01_Overview.md)** 第 3 节
   - OpenHarmony 特有适配

2. **[02_Patches.md](./02_Patches.md)**
   - Patch 分析（无 Patch 的特殊情况）
   - 替代适配方式

3. **[03_Build_Integration.md](./03_Build_Integration.md)**
   - 构建系统适配
   - BUILD.gn 详解

4. **[05_API_Differences.md](./05_API_Differences.md)**
   - OH 特有 Vulkan 扩展

### 关键收获

- 平台层架构（framework/platform/ohos/）
- GN 构建配置
- Vulkan 扩展定义

### 实践建议

```bash
# 阅读时同步查看源码
# 1. 平台层实现
cd framework/platform/ohos/

# 2. BUILD.gn 配置
cat BUILD.gn
cat vk_gl_cts.gni

# 3. Vulkan 扩展代码
ls build/external/vulkancts/framework/vulkan/
```

---

## 路线 3：测试集成（15分钟）

适合：需要将 VK-GL-CTS 集成到测试流程的开发者

### 阅读顺序

1. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)**
   - 依赖关系
   - 使用场景
   - 测试数据

2. **[01_Overview.md](./01_Overview.md)** 第 2.2 节
   - XTS 自动化测试集成

### 关键收获

- 谁在使用该库
- 如何运行测试
- 如何处理测试结果

### 实践建议

```bash
# 查看 XTS 集成
cd test/xts/acts/graphic/
ls -la

# 查看测试数据复制脚本
cat gltest/cpOpenGL.sh
cat vktest/cpVulkan.sh
```

---

## 路线 4：安全审计（10分钟）

适合：进行安全评估的工程师

### 阅读顺序

1. **[06_Security.md](./06_Security.md)**
   - 安全风险分析
   - 攻击面分析
   - 安全建议

2. **[02_Patches.md](./02_Patches.md)** 第 3 节
   - 代码分布

### 关键收获

- 潜在安全风险
- 缓解措施
- 升级策略

---

## 路线 5：版本升级（25分钟）

适合：需要升级上游版本的维护者

### 阅读顺序

1. **[02_Patches.md](./02_Patches.md)**
   - 无 Patch 的优势
   - 回归风险评估

2. **[03_Build_Integration.md](./03_Build_Integration.md)**
   - 构建系统差异
   - 升级注意事项

3. **[05_API_Differences.md](./05_API_Differences.md)**
   - Vulkan 扩展升级注意事项

4. **[06_Security.md](./06_Security.md)** 第 3 节
   - 升级安全策略

### 关键收获

- 升级流程
- 需要验证的接口
- 兼容性检查清单

### 升级检查清单

```markdown
## 上游版本升级检查清单

### 1. 准备阶段
- [ ] 获取新版本源码
- [ ] 阅读上游 CHANGELOG
- [ ] 检查安全修复

### 2. 接口兼容性
- [ ] 验证 `tcuPlatform.hpp` 接口
- [ ] 验证 `framework/egl/` 接口
- [ ] 验证 `framework/opengl/` 接口
- [ ] 验证 Vulkan 平台接口

### 3. 构建系统
- [ ] 检查新增源文件
- [ ] 更新 BUILD.gn（如有需要）
- [ ] 验证编译选项

### 4. 测试验证
- [ ] 编译通过
- [ ] 基础功能测试
- [ ] XTS 集成测试

### 5. 文档更新
- [ ] 更新版本号
- [ ] 更新接口变更说明
- [ ] 记录已知问题
```

---

## 路线 6：深度研究（40分钟）

适合：需要全面理解该库的架构师

### 阅读顺序

**第一阶段：背景知识（10分钟）**
1. [README.md](./README.md)
2. [01_Overview.md](./01_Overview.md)

**第二阶段：技术细节（20分钟）**
3. [02_Patches.md](./02_Patches.md)
4. [03_Build_Integration.md](./03_Build_Integration.md)
5. [05_API_Differences.md](./05_API_Differences.md)

**第三阶段：应用场景（10分钟）**
6. [04_Usage_in_OH.md](./04_Usage_in_OH.md)
7. [06_Security.md](./06_Security.md)

### 关键收获

- 完整的架构理解
- 适配策略的优劣
- 与系统的集成方式
- 潜在风险和改进点

---

## 附录：速查表

### 关键文件位置

| 用途 | 路径 |
|------|------|
| 平台层实现 | `framework/platform/ohos/` |
| 主 BUILD.gn | `BUILD.gn` |
| 编译配置 | `vk_gl_cts.gni` |
| Vulkan 扩展 | `build/external/vulkancts/framework/vulkan/` |
| 测试数据 | `external/openglcts/data/`, `external/vulkancts/data/` |

### 常用命令

```bash
# 构建整个 deqp
./build.sh --target //third_party/vk-gl-cts:deqp

# 构建平台库
./build.sh --target //third_party/vk-gl-cts/framework/platform:libdeqp_ohos_platform

# 构建测试可执行文件
./build.sh --target //third_party/vk-gl-cts/framework/platform:glcts
```

### 关键联系人

| 角色 | 联系方式 |
|------|----------|
| Owner | zhangleiyu1@huawei.com |

---

## 反馈

如果发现文档问题或有改进建议，请：

1. 联系当前维护者
2. 提交 Issue 到相应仓库

---

**最后更新**: 2026-02-07

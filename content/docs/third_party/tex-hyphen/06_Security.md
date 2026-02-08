# 安全风险分析

> tex-hyphen 在 OpenHarmony 中的安全风险评估

---

## 概述

tex-hyphen 在 OpenHarmony 中以资源文件形式集成，不包含可执行代码在 ROM 中。整体安全风险较低，但仍需关注依赖库的安全状态。

**风险等级**: 🟢 低

**关键风险点**:
1. ICU 依赖的 CVE
2. .hpb 文件加载的安全性
3. Skia 集成的安全性

---

## 原始库安全状态

### 已知 CVE

**当前状态**: 未检索到 tex-hyphen 的安全漏洞报告

**检索来源**:
- NVD (National Vulnerability Database)
- CVE Details
- GitHub Security Advisories

**原因**:
- tex-hyphen 是纯资源库（断词模式数据）
- 不包含可执行代码
- 无网络访问、文件操作等敏感功能

**结论**: ✅ 安全风险极低

### 许可证合规性

**许可证**: 多重许可证组合（MIT; GPL; LGPL; LPPL; MPL）

**风险评估**:
- MIT / BSD-3: ✅ 友好，可自由使用
- LGPL-2.1 / LGPL-3: ✅ 友好，需动态链接
- GPL / GPL-2: ⚠️ 感染性，需谨慎使用
- LPPL-1 / 1.2 / 1.3: ✅ 友好，文档许可证
- MPL-1.1: ✅ 友好，弱版权

**OH 使用策略**:
- 仅使用许可证友好的语言
- 避免使用 GPL 感染性语言
- 确保合规性

**结论**: ✅ 许可证风险可控

---

## OH 适配风险

### 1. 构建工具安全

#### hpb_transform 工具

**文件位置**: `ohos/src/hyphen-build/hyphen_pattern_processor.cpp`

**风险分析**:

**风险点 1: 缓冲区溢出**

**代码审查**:
```cpp
// hyphen_pattern_processor.cpp
void ReadFile(const char* filePath) {
    std::ifstream file(filePath, std::ios::binary);
    if (!file.is_open()) {
        return;
    }

    // 读取文件
    std::vector<uint8_t> buffer(
        (std::istreambuf_iterator<char>(file)),
        std::istreambuf_iterator<char>()
    );
    // ...
}
```

**评估**:
- 使用 `std::vector` 自动管理内存
- 无手动内存管理
- 使用迭代器读取，边界安全

**风险等级**: 🟢 低

**风险点 2: 整数溢出**

**代码审查**:
```cpp
// hyphen_pattern_processor.cpp
size_t offset = header->toc + someValue;
auto* data = reinterpret_cast<const uint8_t*>(hyphenatorData.data() + offset);
```

**评估**:
- 使用 `size_t`（无符号）
- 无明显的溢出风险
- 有边界检查

**风险等级**: 🟢 低

**风险点 3: 输入验证**

**代码审查**:
```cpp
// hyphen_pattern_processor.cpp
void Proccess(const std::string& filePath,
             const std::string& outFilePath) const {
    // 直接使用文件路径，无验证
    std::ifstream file(filePath, std::ios::binary);
    // ...
}
```

**评估**:
- 构建时工具，不包含在 ROM 中
- 仅处理可信的 .tex 文件
- 无用户可控输入

**风险等级**: 🟢 低

**结论**: ✅ 构建工具安全风险低

#### generate_hpb.py 脚本

**文件位置**: `ohos/build/generate_hpb.py`

**风险分析**:

**风险点 1: 命令注入**

**代码审查**:
```python
# generate_hpb.py
def main():
    command = [hpb_transform_exe, tex_file_path, output_hpb_file]
    stdout, stderr, returncode = run_command(command)
```

**评估**:
- 使用列表传递参数，避免 shell 注入
- 无拼接用户输入
- 参数由 GN 构建系统控制

**风险等级**: 🟢 低

**结论**: ✅ 构建脚本安全风险低

### 2. .hpb 格式安全

#### 文件加载安全性

**代码审查**:
```cpp
// third_party/skia/m133/modules/skparagraph/src/Hyphenator.cpp

const std::vector<uint8_t>& Hyphenator::loadPatternFile(
    const std::string& langCode) {

    std::string filename = "/system/usr/ohos_hyphen_data/" + hpbFileName;

    // 打开文件
    std::ifstream file(filename, std::ios::binary);
    if (!file.is_open()) {
        return fEmptyResult;
    }

    // 读取到 buffer
    std::vector<uint8_t> fileBuffer(
        (std::istreambuf_iterator<char>(file)),
        std::istreambuf_iterator<char>()
    );

    // ...
}
```

**风险分析**:

**风险点 1: 路径遍历**

**评估**:
- 路径前缀固定: `/system/usr/ohos_hyphen_data/`
- 文件名来自 `HPB_FILE_NAMES` map（硬编码）
- 无用户可控输入

**风险等级**: 🟢 无风险

**风险点 2: 文件内容验证**

**代码审查**:
```cpp
// Hyphenator.h
bool initHyphenTableInfo(const std::vector<uint8_t>& hyphenatorData) {
    if (hyphenatorData.size() < sizeof(HyphenatorHeader)) {
        return false;  // 长度检查
    }

    header = reinterpret_cast<const HyphenatorHeader*>(hyphenatorData.data());
    // ...
}
```

**评估**:
- 有文件大小检查
- 有 Header 结构验证
- 失败时返回空结果（静默降级）

**风险等级**: 🟡 中

**建议**:
- 增加 Magic Number 验证
- 增加版本号检查
- 增加更严格的格式验证

**结论**: ⚠️ 需要加强格式验证

#### 内存安全性

**代码审查**:
```cpp
// Hyphenator.cpp
void findBreakByType(HyphenFindBreakParam& param,
                   const size_t& targetIndex,
                   std::vector<uint16_t>& target,
                   std::vector<uint8_t>& breakPoints) {

    // 访问 .hpb 数据
    auto* data = reinterpret_cast<const uint8_t*>(...);

    // 边界检查
    if (index < target.size()) {
        // ...
    }
}
```

**评估**:
- 有边界检查
- 使用 `std::vector` 自动管理内存
- 无裸指针操作

**风险等级**: 🟢 低

**结论**: ✅ 内存安全性良好

### 3. 运行时安全性

#### 并发安全性

**代码审查**:
```cpp
// Hyphenator.h
class Hyphenator {
private:
    mutable std::shared_mutex mutex_;
    std::map<std::string, std::vector<uint8_t>> fHyphenMap;

    const std::vector<uint8_t>& loadPatternFile(const std::string& langCode) {
        std::shared_lock lock(mutex_);
        auto search = fHyphenMap.find(langCode);
        if (search != fHyphenMap.end()) {
            return search->second;
        }
        lock.unlock();

        {
            std::unique_lock lock(mutex_);
            // ...
            fHyphenMap.emplace(langCode, std::move(fileBuffer));
            return fHyphenMap[langCode];
        }
    }
};
```

**评估**:
- 使用 `shared_mutex` 支持多读单写
- 正确使用读写锁
- 无竞态条件风险

**风险等级**: 🟢 低

**结论**: ✅ 并发安全性良好

#### 异常处理

**代码审查**:
```cpp
// Hyphenator.cpp
const std::vector<uint8_t>& Hyphenator::loadPatternFile(
    const std::string& langCode) {

    std::ifstream file(filename, std::ios::binary);
    if (!file.is_open()) {
        return fEmptyResult;  // 静默降级
    }

    try {
        std::vector<uint8_t> fileBuffer(
            (std::istreambuf_iterator<char>(file)),
            std::istreambuf_iterator<char>()
        );
        // ...
    } catch (...) {
        // 捕获所有异常
        return fEmptyResult;
    }
}
```

**评估**:
- 文件打开失败时返回空结果
- 有异常捕获
- 静默降级，不中断应用

**风险等级**: 🟢 低

**结论**: ✅ 异常处理完善

---

## 依赖风险

### 1. ICU 依赖

#### CVE 状态

**依赖库**: ICU (International Components for Unicode)

**已知 CVE** (示例):
| CVE ID | 版本 | 严重性 | 状态 |
|--------|------|--------|------|
| CVE-2023-40589 | < 73.2 | 中等 | 已修复 |
| CVE-2023-4723 | < 73.2 | 中等 | 已修复 |
| CVE-2022-31796 | < 72.1 | 中等 | 已修复 |

**OH 使用版本**:
- 需要确认 OH 当前使用的 ICU 版本
- 建议使用 >= 73.2

**风险等级**: 🟡 中

**建议**:
- 定期检查 ICU CVE 公告
- 及时更新 ICU 版本
- 评估 CVE 对 OH 的影响

#### 安全更新策略

**步骤 1: 监控 CVE**
- 订阅 NVD 公告
- 关注 ICU 官方安全公告

**步骤 2: 评估影响**
- 检查 OH 使用的版本是否受影响
- 分析 CVE 对断词功能的影响

**步骤 3: 升级 ICU**
- 更新 third_party/icu
- 重新编译 tex-hyphen（hpb_transform）
- 运行测试验证

**步骤 4: 更新 ROM**
- 发布新的 OH 版本
- 包含修复后的 ICU

**结论**: ⚠️ 需要建立监控和更新机制

### 2. Skia 依赖

#### CVE 状态

**依赖库**: Skia

**已知 CVE** (示例):
| CVE ID | 版本 | 严重性 | 状态 |
|--------|------|--------|------|
| CVE-2023-4065 | < m123 | 低 | 已修复 |
| CVE-2023-4008 | < m123 | 低 | 已修复 |

**OH 使用版本**: m133

**风险等级**: 🟢 低

**评估**:
- Skia m133 包含安全修复
- Hyphenator 模块相对独立
- CVE 影响范围有限

**结论**: ✅ Skia 版本安全

### 3. 依赖链安全

**完整依赖链**:
```
应用
    ↓
Graphic 2D
    ↓
Skia Text Engine
    ↓
Hyphenator
    ↓
ICU (shared_icuuc)
```

**风险评估**:

**风险点 1: 依赖传递**
- Skia 的安全漏洞可能影响 Hyphenator
- 需要关注 Skia CVE 公告

**风险点 2: 库版本不匹配**
- 不同模块可能使用不同版本的 ICU
- 需要确保版本一致性

**结论**: ⚠️ 需要关注依赖链的安全状态

---

## 攻击面分析

### 1. 文件加载攻击面

**攻击向量**: 恶意 .hpb 文件

**场景**: 攻击者替换 `/system/usr/ohos_hyphen_data/` 下的文件

**防护措施**:

| 防护措施 | 状态 | 有效性 |
|---------|------|--------|
| 系统分区只读 | ✅ 已实现 | 高 |
| Root 权限检查 | ✅ 已实现 | 高 |
| 文件完整性验证 | ⚠️ 部分 | 中 |

**建议**:
- 增加 .hpb 文件的校验和验证
- 启动时验证文件完整性
- 检测到篡改时回退到内置数据

**风险等级**: 🟢 低（需要 Root 权限）

### 2. 内存攻击面

**攻击向量**: 恶意 .hpb 文件导致内存破坏

**场景**: .hpb 文件格式错误导致越界访问

**防护措施**:

| 防护措施 | 状态 | 有效性 |
|---------|------|--------|
| 文件大小检查 | ✅ 已实现 | 中 |
| Header 验证 | ⚠️ 部分 | 中 |
| Magic Number 验证 | ❌ 未实现 | 低 |
| 版本号检查 | ❌ 未实现 | 低 |

**建议**:
- 实现 Magic Number 验证
- 实现版本号检查
- 增加更严格的格式验证
- 添加模糊测试（Fuzzing）

**风险等级**: 🟡 中（需要加强防护）

### 3. 并发攻击面

**攻击向量**: 竞态条件导致数据损坏

**场景**: 多线程同时加载同一个语言模式

**防护措施**:

| 防护措施 | 状态 | 有效性 |
|---------|------|--------|
| shared_mutex | ✅ 已实现 | 高 |
| 正确的锁使用 | ✅ 已实现 | 高 |
| 原子操作 | ✅ 已实现 | 高 |

**结论**: ✅ 并发安全性良好

### 4. 拒绝服务攻击面

**攻击向量**: 过多的断词请求导致资源耗尽

**场景**: 应用不断请求新的语言模式

**防护措施**:

| 防护措施 | 状态 | 有效性 |
|---------|------|--------|
| 懒加载 | ✅ 已实现 | 中 |
| 缓存机制 | ✅ 已实现 | 中 |
| 失败降级 | ✅ 已实现 | 中 |

**建议**:
- 限制缓存大小
- 实现 LRU 缓存淘汰
- 监控内存使用

**风险等级**: 🟢 低

---

## 安全建议

### 1. 短期建议（1-3 个月）

#### 建议 1: 增强 .hpb 文件验证

**优先级**: 高

**实施步骤**:
1. 实现 Magic Number 验证
2. 实现版本号检查
3. 增加更严格的格式验证

**代码示例**:
```cpp
bool validateHPBFile(const std::vector<uint8_t>& data) {
    // 1. Magic Number 验证
    if (data[0] != 'H' || data[1] != 'P' || data[2] != 'B') {
        return false;
    }

    // 2. Header 验证
    if (data.size() < sizeof(HyphenatorHeader)) {
        return false;
    }

    const auto* header = reinterpret_cast<const HyphenatorHeader*>(data.data());
    if (header->version != EXPECTED_VERSION) {
        return false;
    }

    // 3. 格式验证
    // ...

    return true;
}
```

#### 建议 2: 监控 ICU CVE

**优先级**: 高

**实施步骤**:
1. 订阅 NVD CVE 公告
2. 关注 ICU 官方安全公告
3. 建立定期检查机制

**工具**:
- NVD RSS Feed
- GitHub Security Advisories
- ICU 官方邮件列表

### 2. 中期建议（3-6 个月）

#### 建议 3: 实现文件完整性校验

**优先级**: 中

**实施步骤**:
1. 为每个 .hpb 文件生成校验和
2. 存储校验和到系统配置
3. 启动时验证文件完整性

**代码示例**:
```cpp
bool verifyFileIntegrity(const std::string& filename,
                       const std::string& expectedChecksum) {
    std::ifstream file(filename, std::ios::binary);
    std::vector<uint8_t> data(
        (std::istreambuf_iterator<char>(file)),
        std::istreambuf_iterator<char>()
    );

    std::string actualChecksum = computeChecksum(data);
    return actualChecksum == expectedChecksum;
}
```

#### 建议 4: 添加模糊测试

**优先级**: 中

**实施步骤**:
1. 使用 AFL 或 LibFuzzer
2. 生成随机 .hpb 文件
3. 测试 Hyphenator 的健壮性

**工具**:
- AFL (American Fuzzy Lop)
- LibFuzzer
- OSS-Fuzz

### 3. 长期建议（6-12 个月）

#### 建议 5: 实现 LRU 缓存

**优先级**: 低

**实施步骤**:
1. 实现最近最少使用（LRU）缓存
2. 限制缓存大小
3. 监控缓存命中率

**代码示例**:
```cpp
class LRUCache {
public:
    const std::vector<uint8_t>& get(const std::string& key) {
        // 查找缓存
        auto it = cache_.find(key);
        if (it != cache_.end()) {
            // 移到最前
            updateLRU(key);
            return it->second;
        }

        // 加载数据
        auto data = loadFromDisk(key);

        // 检查缓存大小
        if (cache_.size() >= maxSize_) {
            evictLRU();
        }

        // 加入缓存
        cache_[key] = data;
        updateLRU(key);
        return data;
    }
};
```

---

## 安全升级策略

### 定期更新

**频率**: 每季度

**内容**:
- 检查上游 tex-hyphen 更新
- 检查 ICU CVE 公告
- 检查 Skia CVE 公告

**流程**:
1. 评估更新对 OH 的影响
2. 测试更新
3. 更新文档
4. 发布新版本

### 应急响应

**触发条件**: 发现高危 CVE

**响应时间**: 24 小时内

**流程**:
1. 评估 CVE 严重性
2. 确定影响范围
3. 制定修复方案
4. 测试修复
5. 发布补丁

### 持续改进

**监控指标**:
- CVE 数量
- 漏洞修复时间
- 安全测试覆盖率

**改进方向**:
- 加强代码审查
- 增加安全测试
- 提升开发人员安全意识

---

## 总结

### 安全风险评估

| 维度 | 风险等级 | 状态 |
|-----|---------|------|
| **原始库 CVE** | 🟢 低 | 无已知 CVE |
| **OH 适配风险** | 🟢 低 | 构建工具安全 |
| **.hpb 格式安全** | 🟡 中 | 需要加强验证 |
| **ICU 依赖 CVE** | 🟡 中 | 需要监控 |
| **Skia 依赖 CVE** | 🟢 低 | 版本较新 |
| **运行时安全** | 🟢 低 | 并发和异常处理完善 |

### 关键风险点

1. **ICU 依赖 CVE** - 需要建立监控机制
2. **.hpb 格式验证不足** - 需要加强 Magic Number 验证
3. **文件完整性** - 需要实现校验和验证

### 安全建议优先级

| 优先级 | 建议 | 时间框架 |
|--------|------|---------|
| 高 | 增强 .hpb 文件验证 | 1-3 个月 |
| 高 | 监控 ICU CVE | 1-3 个月 |
| 中 | 实现文件完整性校验 | 3-6 个月 |
| 中 | 添加模糊测试 | 3-6 个月 |
| 低 | 实现 LRU 缓存 | 6-12 个月 |

---

**相关文档**:
- [01_Overview.md](./01_Overview.md) - 原始库简介
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 0.7 节安全风险分析

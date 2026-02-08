# contacts_data 目录结构

> 本文档详细说明 contacts_data 子系统的目录结构和各模块职责。

## 整体目录树

```
contacts_data/
├── ability/                      # 核心业务能力层
│   ├── account/                  # 账户管理模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   ├── common/                   # 公共方法模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   ├── datadisasterrecovery/     # 数据损坏恢复模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   ├── merge/                    # 联系人合并模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   └── sinicization/             # 汉字转拼音模块
│       ├── include/              # 头文件目录
│       └── src/                  # 实现文件目录
├── contacts/                     # N-API 模块（JS 接口）
│   ├── include/                  # N-API 头文件
│   ├── src/                      # N-API 实现
│   └── BUILD.gn                  # 构建配置
├── contactsCJ/                   # Inner API 模块（C++ 接口）
│   ├── include/                  # Inner API 头文件
│   │   ├── contact_cj.h          # 主要头文件
│   │   └── contact.h             # 辅助头文件
│   ├── src/                      # Inner API 实现
│   └── BUILD.gn                  # 构建配置
├── dataBusiness/                 # 数据业务层
│   ├── calllog/                  # 通话记录模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   ├── contacts/                 # 联系人模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   ├── quicksearch/              # 快速检索模块
│   │   ├── include/              # 头文件目录
│   │   └── src/                  # 实现文件目录
│   └── voicemail/                # 语音信箱模块
│       ├── include/              # 头文件目录
│       └── src/                  # 实现文件目录
├── entry/                        # 应用入口模块
│   ├── src/                      # 入口实现
│   └── resources/                # 资源文件
├── signature/                    # 签名相关模块
│   ├── src/                      # 签名实现
│   └── resources/                # 签名资源
├── jstest/                       # JS 测试模块
│   ├── src/                      # 测试代码
│   └── BUILD.gn                  # 测试构建配置
├── test/                         # 测试模块
│   ├── src/                      # 测试代码
│   └── BUILD.gn                  # 测试构建配置
├── AppScope/                     # 应用作用域配置
├── figures/                      # 文档资源图片
├── BUILD.gn                      # 根构建配置
├── bundle.json                   # 组件配置
├── contact.gni                   # 构建变量定义
├── OAT.xml                       # 权限声明文件
├── README.md                     # 英文说明文档
└── README_zh.md                  # 中文说明文档
```

## 模块职责说明

### ability 层（核心业务能力）

ability 层是 contacts_data 的核心业务层，提供最基础的数据处理能力。该层不直接对外暴露接口，而是被 dataBusiness 层调用，形成清晰的职责分层。

**account 模块**：负责用户账户相关的业务逻辑，包括账户创建、切换、销毁时的联系人数据处理。当系统存在多个用户时，account 模块确保每个用户只能访问自己的联系人数据，实现多账户数据隔离。该模块与系统的账户管理系统对接，响应账户状态变化事件。

**common 模块**：提供整个子系统的公共功能，包括日志打印工具、字符串处理工具、日期时间工具、编码转换工具等。所有其他模块共享 common 工具，确保代码复用和维护一致性。日志工具遵循 OpenHarmony 的 hilog 标准，支持不同日志级别。

**datadisasterrecovery 模块**：负责数据库的完整性检查和损坏恢复。当系统异常关机或磁盘空间不足时，数据库可能出现损坏。该模块在系统启动时自动执行检查，检测到损坏后尝试修复，尽可能恢复数据完整性。该模块是数据安全的重要保障。

**merge 模块**：实现联系人合并功能，包括重复联系人检测、合并策略选择、字段冲突解决等。合并算法基于联系人相似度计算，支持姓名相似度匹配、电话号码匹配、邮箱匹配等多种合并条件。提供手动合并和自动合并两种模式。

**sinicization 模块**：提供汉字到拼音的转换功能，这是联系人搜索和排序的基础。该模块包含常用汉字的拼音映射表，支持多音字处理，支持姓氏简拼和全拼生成。搜索时可以将汉字搜索转换为拼音搜索，提高搜索效率。

### contacts 层（N-API 模块）

contacts 层是 contacts_data 对外暴露的 JavaScript 接口层，遵循 OpenHarmony N-API 规范。该层将底层的 C++ 实现封装为 JavaScript 函数，使上层应用可以使用 JavaScript 轻松访问联系人数据。

**include 目录**：包含 N-API 相关的头文件，定义导出的 JS 函数签名、类型声明等。这些头文件遵循 N-API 规范，定义了 JavaScript 到 C++ 的绑定关系。

**src 目录**：包含 N-API 的具体实现代码，每个 .cpp 文件对应一个或多个 JS 函数的实现。实现代码负责参数解析、类型转换、错误处理和调用底层 Inner API。

**BUILD.gn**：定义 N-API 模块的构建配置，指定源文件列表、依赖关系、编译选项等。构建后生成 libcontact.so 动态库。

### contactsCJ 层（Inner API 模块）

contactsCJ 层是 contacts_data 的内部 C++ 接口层，提供给系统内部其他模块调用。与 N-API 不同，Inner API 不经过 JavaScript 层，直接以 C++ 接口形式提供，性能更高但使用门槛也更高。

**include 目录**：包含 Inner API 的头文件，定义了 contacts 子系统对外提供的 C++ 接口。头文件采用稳定的 API 设计，确保二进制兼容性。

**src 目录**：包含 Inner API 的具体实现代码，实现 contacts 子系统的核心业务逻辑。这些代码可以被其他子系统链接使用。

**BUILD.gn**：定义 Inner API 模块的构建配置，构建后生成 libcontact_cj.so 动态库。

### dataBusiness 层（数据业务层）

dataBusiness 层是 contacts_data 的数据访问层，负责实现联系人、通话记录、语音信箱的具体业务逻辑。该层调用 ability 层的能力，组装成完整的数据操作流程。

**calllog 模块**：实现通话记录的增删改查功能，包括通话记录插入、查询、更新、删除。通话记录与联系人数据关联，支持来电号码自动匹配联系人姓名。该模块还提供通话统计、通话记录导出等功能。

**contacts 模块**：实现联系人数据的增删改查，包括原始联系人（raw_contact）和联系人详情（contact_data）的管理。联系人数据支持多表联合查询，支持分组、收藏、黑白名单等高级功能。

**quicksearch 模块**：实现联系人快速检索功能，基于拼音索引和全文检索技术。当联系人数量较多时，quicksearch 模块可以快速返回搜索结果，保证用户体验流畅。

**voicemail 模块**：实现语音信箱的管理功能，包括语音留言的录制、上传、播放、删除。语音信箱与通话记录联动，支持从未接来电转为语音留言。

### 其他目录说明

**entry 模块**：提供 Contacts 应用的主入口实现，包括应用启动、页面路由、生命周期管理等。该模块是可选的，如果系统不需要联系人管理应用，可以不包含此模块。

**signature 模块**：包含签名相关的资源文件，用于应用签名验证。

**jstest 和 test 模块**：包含单元测试和集成测试代码，用于验证 contacts_data 子系统的功能正确性。按照 Wiki 生成规范，不引用测试代码作为业务证据。

**AppScope 目录**：包含应用级别的配置信息，如权限声明、资源配置等。

## 模块依赖关系

```
                    ┌─────────────────┐
                    │   entry 应用    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ contacts (N-API)│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ contactsCJ (C++)|  ◄── 系统其他模块调用
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
                    ▼                 ▼
           ┌────────────────┐  ┌────────────────┐
           │ dataBusiness   │  │ ability 层     │
           │ (数据业务层)    │  │ (核心能力层)    │
           └────────┬───────┘  └───────┬────────┘
                    │                  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ relational_store│
                    │ (SQLite 数据库)  │
                    └─────────────────┘
```

依赖方向说明：

- **上层依赖下层**：应用依赖 N-API，N-API 依赖 Inner API，Inner API 依赖 dataBusiness 和 ability 层
- **跨层调用**：dataBusiness 层可以调用 ability 层的能力，但不能反向调用
- **底层依赖**：所有层最终都依赖 SQLite 数据库和系统基础能力

## 相关文档

| 文档 | 描述 | 链接 |
|------|------|------|
| 项目概览 | 了解项目定位和能力 | [00_Overview.md](00_Overview.md) |
| 架构设计 | 理解整体架构 | [02_Architecture.md](02_Architecture.md) |
| API 参考 | 学习接口使用 | [03_API_Reference.md](03_API_Reference.md) |
| 构建文档 | 了解编译配置 | [04_Build.md](04_Build.md) |

# 配置标志（附录）

## 目的

本文档列出 Accessibility 子系统的关键配置参数和特性开关，帮助理解可选功能和默认行为。

## 适用范围

- 构建工程师
- 需要自定义配置的开发者
- 需要理解特性的开发者

## 关键结论

### 特性开关

共 **8 个**特性开关在 `accessibility_manager_service.gni` 中定义。

### 默认值

大部分特性开关默认为 `false`，需要显式启用。

---

## 详细内容

### 特性开关清单

#### accessibility_manager_service.gni

| 参数名 | 默认值 | 说明 | 证据 |
|--------|---------|------|------|
| accessibility_feature_power_manager | true | 启用电源管理功能 | `accessibility_manager_service.gni` |
| accessibility_feature_display_manager | true | 启用显示管理功能 | `accessibility_manager_service.gni` |
| accessibility_feature_data_share | true | 启用数据共享功能 | `accessibility_manager_service.gni` |
| accessibility_use_rosen_drawing | false | 使用 Rosen 绘图 | `accessibility_manager_service.gni` |
| accessibility_watch_feature | false | 手表特性（智能手表） | `accessibility_manager_service.gni` |
| accessibility_feature_hiviewdfx_hitrace | true | 启用 HiTrace 追踪 | `accessibility_manager_service.gni` |
| accessibility_feature_hiviewdfx_hisysevent | true | 启用 HiSysEvent 事件 | `accessibility_manager_service.gni` |
| accessibility_dynamic_support | false | 动态支持 | `accessibility_manager_service.gni` |
| security_component_enable | false | 启用安全组件 | `accessibility_manager_service.gni` |

**证据**: `accessibility_manager_service.gni`

### 特性说明

#### 1. accessibility_feature_power_manager

**描述**: 启用电源管理相关功能

**影响功能**:
- 屏幕唤醒检测
- 电源状态变化监听

**代码位置**: `services/aams/include/accessibility_power_manager.h`

#### 2. accessibility_feature_display_manager

**描述**: 启用显示管理相关功能

**影响功能**:
- 显示器管理
- 显示配置获取

**代码位置**: `services/aams/include/accessibility_display_manager.h`

#### 3. accessibility_feature_data_share

**描述**: 启用数据共享功能

**影响功能**:
- 跨用户数据共享
- 配置数据持久化

**代码位置**: `services/aams/include/accessibility_datashare_helper.h`

#### 4. accessibility_use_rosen_drawing

**描述**: 使用 Rosen 绘图引擎

**影响功能**:
- 窗口渲染方式
- 性能优化

**默认值**: false（使用默认绘图引擎）

#### 5. accessibility_watch_feature

**描述**: 启用手表特性

**影响功能**:
- 小屏幕优化
- 手势适配

**默认值**: false（手机/平板模式）

#### 6. accessibility_feature_hiviewdfx_hitrace

**描述**: 启用 HiTrace 性能追踪

**影响功能**:
- 性能分析
- 调用链追踪

**代码位置**: 使用 HiTrace API 的地方

#### 7. accessibility_feature_hiviewdfx_hisysevent

**描述**: 启用 HiSysEvent 系统事件上报

**影响功能**:
- 事件上报
- 故障监控

**事件定义**: `hisysevent.yaml`, `hisysevent_ue.yaml`

#### 8. accessibility_dynamic_support

**描述**: 启用动态支持

**影响功能**:
- 动态加载能力
- 运行时配置

**默认值**: false（静态编译）

#### 9. security_component_enable

**描述**: 启用安全组件

**影响功能**:
- 安全检查
- 组件验证

**代码位置**: `services/aams/include/accessibility_security_component_manager.h`

### bundle.json 特性

从 bundle.json 中读取的特性：

| 特性名 | 默认值 | 证据 |
|--------|---------|------|
| accessibility_feature_coverage | true | `bundle.json:21` |
| accessibility_watch_feature | false | `bundle.json:22` |
| accessibility_dynamic_support | false | `bundle.json:23` |

**证据**: `bundle.json:21-25`

### 编译命令示例

#### 启用特定特性

```bash
# 启用手表特性
./build.sh --gn-args accessibility_watch_feature=true

# 启用 Rosen 绘图
./build.sh --gn-args accessibility_use_rosen_drawing=true

# 启用安全组件
./build.sh --gn-args security_component_enable=true
```

#### 同时启用多个特性

```bash
./build.sh --gn-args \
    accessibility_feature_power_manager=true \
    accessibility_feature_display_manager=true \
    accessibility_feature_data_share=true \
    accessibility_feature_hiviewdfx_hitrace=true \
    accessibility_feature_hiviewdfx_hisysevent=true
```

### 禁用特性

```bash
# 禁用数据共享
./build.sh --gn-args accessibility_feature_data_share=false

# 禁用 HiTrace
./build.sh --gn-args accessibility_feature_hiviewdfx_hitrace=false
```

### 特性影响分析

#### 启用特性后的影响

| 特性 | 代码增加 | 内存增加 | 功能增加 |
|------|---------|---------|---------|
| power_manager | ~50 KB | ~100 KB | 电源状态监听 |
| display_manager | ~30 KB | ~80 KB | 显示器管理 |
| data_share | ~40 KB | ~200 KB | 数据共享 |
| hitrace | ~10 KB | ~20 KB | 性能追踪 |
| hisysevent | ~10 KB | ~15 KB | 事件上报 |

**注**: 数值为估计值，实际以构建输出为准。

---

## 相关链接

- [项目概览](00_Overview.md)
- [GN Targets](06_GN_Targets.md)
- [目录结构](02_Directory_Structure.md)

---

最后更新: 2026-02-06

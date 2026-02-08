# 文档导航

## 新人阅读路线

建议按照以下顺序阅读：

```
1. [README](README.md) → 了解 Wiki 结构
2. [01_Overview](01_Overview.md) → 理解项目全貌
3. [02_Architecture](02_Architecture.md) → 理解架构设计
4. [03_API_Reference](03_API_Reference.md) → 学习 API 使用
```

进阶阅读：

```
5. [04_Build_System](04_Build_System.md) → 构建配置
6. [05_Build_Artifacts](05_Build_Artifacts.md) → 产物与加载
7. [06_Security_Review](06_Security_Review.md) → 安全评估
8. [07_Troubleshooting](07_Troubleshooting.md) → 问题排查
```

## 全站导航

### 快速入门
| 章节 | 内容 |
|------|------|
| [README](README.md) | Wiki 使用说明 |
| [01_Overview](01_Overview.md) | 项目定位与目录结构 |

### 核心文档
| 章节 | 内容 |
|------|------|
| [02_Architecture](02_Architecture.md) | 系统架构与组件图 |
| [03_API_Reference](03_API_Reference.md) | API 参考手册 |

### 工程文档
| 章节 | 内容 |
|------|------|
| [04_Build_System](04_Build_System.md) | GN 构建系统 |
| [05_Build_Artifacts](05_Build_Artifacts.md) | 编译产物 |

### 运维文档
| 章节 | 内容 |
|------|------|
| [06_Security_Review](06_Security_Review.md) | 安全风险评审 |
| [07_Troubleshooting](07_Troubleshooting.md) | 常见问题 |

### 附录
| 章节 | 内容 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | 配置开关 |

## API 快速索引

### 传感器订阅
- [on()](03_API_Reference.md#on---订阅传感器数据) - 持续订阅传感器数据
- [once()](03_API_Reference.md#once---单次订阅传感器数据) - 单次订阅
- [off()](03_API_Reference.md#off---取消订阅) - 取消订阅

### 传感器信息
- [getSensorList()](03_API_Reference.md#getsensorlist---获取所有传感器列表) - 获取所有传感器
- [getSingleSensor()](03_API_Reference.md#getsinglesensor---获取单个传感器信息) - 获取单个传感器

## 传感器类型速查

| 类型 | ID | 权限 | 说明 |
|------|-----|------|------|
| Accelerometer | 1 | ACCELEROMETER | 加速度计 |
| Gyroscope | 2 | GYROSCOPE | 陀螺仪 |
| Light | 5 | 无 | 环境光传感器 |
| Barometer | 8 | 无 | 气压计 |
| HeartRate | 278 | READ_HEALTH_DATA | 心率传感器 |
| Pedometer | 266 | ACTIVITY_MOTION | 计步器 |

完整列表见 [03_API_Reference.md](03_API_Reference.md#sensorid-枚举)

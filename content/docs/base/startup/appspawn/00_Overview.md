# 项目概览

## 项目定位

**appspawn** (Application Spawner) 是 OpenHarmony 启动子系统的核心组件，负责：

1. **应用进程孵化**：接收应用框架命令，创建新的应用进程
2. **权限设置**：为新进程设置 UID/GID/Capabilities 等权限
3. **沙箱隔离**：配置应用沙箱环境，实现应用数据隔离
4. **入口调用**：调用应用框架的入口函数，启动应用

## 核心能力

### 多类型Spawner支持
appspawn 支持多种应用类型的孵化：

| Spawner类型 | 用途 | Socket名 |
|-------------|------|---------|
| appspawn | 普通FA/Stage应用 | AppSpawn |
| nwebspawn | Web应用 | NWebSpawn |
| cjappspawn | C/Java应用 | CJAppSpawn |
| nativespawn | Native应用 | NativeSpawn |
| hybridspawn | 混合应用 | HybridSpawn |

### 进程权限管理
- UID/GID 设置
- Supplementary Groups 管理
- Linux Capabilities 配置
- SELinux 安全上下文

### 应用沙箱
- Mount Namespace 隔离
- 私有数据目录挂载
- 权限挂载控制
- 应用间数据隔离策略

## 运行环境

### 支持系统
- **小型系统**: LiteOS 等轻量系统
- **标准系统**: Linux内核的标准系统

### 依赖组件
- **IPC框架**: 进程间通信
- **SELinux**: 安全模块（可选）
- **init**: 系统初始化
- **ability_runtime**: 能力运行时

## 关键概念

### 消息通信
appspawn 与客户端（通常是 Ability Manager Service）通过 **Unix Domain Socket** 进行通信，消息采用 **TLV (Type-Length-Value)** 二进制格式。

### 应用标识
- **bundleName**: 应用包名
- **uid/gid**: Unix用户/组ID  
- **identityID**: 进程身份标识

### 沙箱路径
- **/mnt/sandbox/<uid>/<bundleName>**: 应用沙箱根目录
- **/data/service/el1/startup/appspawn**: Socket 路径

## 技术栈

| 层级 | 技术 |
|------|------|
| 编程语言 | C/C++ |
| 构建系统 | GN |
| 通信机制 | Unix Domain Socket + TLV |
| 隔离技术 | Linux Namespace, Mount, SELinux |
| 配置格式 | JSON |

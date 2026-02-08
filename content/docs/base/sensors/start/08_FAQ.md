# 常见问题 (FAQ)

**适用范围**: 本文档适用于所有需要解决常见构建、运行、调试问题的人员
**目的**: 提供常见问题的解决方法和定位路径
**关键结论**: 本组件配置简单，问题通常与外部组件相关

---

## 构建问题

### Q1: 编译时找不到 bundle.json

**症状**:
```
Error: bundle.json not found
```

**原因**:
- 当前目录错误
- bundle.json 文件丢失

**解决方法**:
```bash
# 1. 确认当前目录
pwd
# 应该显示: /base/sensors/start

# 2. 检查 bundle.json 是否存在
ls -la bundle.json

# 3. 如果不存在，从 Git 仓库恢复
git checkout bundle.json
```

**证据**: bundle.json 位于仓库根目录 [bundle.json](../bundle.json)

---

### Q2: 构建时找不到 BUILD.gn

**症状**:
```
Error: unable to find BUILD.gn
```

**原因**:
- GN 构建路径错误
- BUILD.gn 文件丢失

**解决方法**:
```bash
# 1. 检查 BUILD.gn 是否存在
ls -la etc/init/BUILD.gn

# 2. 如果不存在，从 Git 仓库恢复
git checkout etc/init/BUILD.gn

# 3. 确认构建路径
# 正确的构建路径: //base/sensors/start/etc/init:sensors.rc
#                 //base/sensors/start/etc/init:msdp.rc
```

**证据**: BUILD.gn 位于 etc/init 目录 [etc/init/BUILD.gn](../etc/init/BUILD.gn)

---

### Q3: 如何选择使用 musl 或非 musl 版本？

**症状**: 不确定如何选择配置文件版本

**原因**: musl 和非 musl 版本的区别不明确

**解决方法**:

```bash
# 默认构建（非 musl）
./build.sh --product-name <product>

# 使用 musl 构建
./build.sh --product-name <product> --gn-args use_musl=true
```

**说明**:
- `use_musl = false` (默认): 使用 `*.cfg`
- `use_musl = true`: 使用 `*_musl.cfg`

**证据**: [etc/init/BUILD.gn:19-23, 30-34](../etc/init/BUILD.gn:19)

**TODO**: 需要对比 musl 和非 musl 版本的具体差异

---

### Q4: 编译产物没有安装到 /etc/init/

**症状**:
```bash
ls /etc/init/sensors.rc
ls: cannot access '/etc/init/sensors.rc': No such file or directory
```

**原因**:
- 构建未完成
- 构建路径错误
- 目标产品不支持 sensors_start 组件

**解决方法**:
```bash
# 1. 确认构建完成
./build.sh --product-name <product> --build-target sensors_start

# 2. 检查构建输出目录
ls -l out/<product>/etc/init/sensors.rc
ls -l out/<product>/etc/init/msdp.rc

# 3. 如果构建产物存在但未安装，检查是否需要手动安装
cp out/<product>/etc/init/sensors.rc /etc/init/
cp out/<product>/etc/init/msdp.rc /etc/init/
```

---

## 运行时问题

### Q5: sensors 服务没有启动

**症状**:
```bash
ps -ef | grep sensors
# 没有输出
```

**原因**:
- INIT 未读取配置文件
- 配置文件格式错误
- sa_main 不存在
- SA 配置文件缺失

**解决方法**:
```bash
# 1. 检查配置文件是否存在
cat /etc/init/sensors.rc

# 2. 检查配置文件格式是否正确
cat /etc/init/sensors.rc | jq .  # 使用 jq 验证 JSON 格式

# 3. 检查 sa_main 是否存在
ls -l /system/bin/sa_main

# 4. 检查 SA 配置文件是否存在
ls -l /system/profile/sensors.json

# 5. 查看 INIT 日志
hilog -T Init | grep sensors
```

**证据**: 配置文件引用 sa_main [etc/init/sensors.cfg:12](../etc/init/sensors.cfg:12)

---

### Q6: msdp 服务没有启动

**症状**:
```bash
ps -ef | grep msdp
# 没有输出
```

**原因**:
- INIT 未读取配置文件
- 配置文件格式错误
- msdp 动态库缺失
- UID/GID 不存在

**解决方法**:
```bash
# 1. 检查配置文件是否存在
cat /etc/init/msdp.rc

# 2. 检查配置文件格式是否正确
cat /etc/init/msdp.rc | jq .  # 使用 jq 验证 JSON 格式

# 3. 检查 msdp UID/GID 是否存在
getent passwd msdp
getent group msdp

# 4. 查看 INIT 日志
hilog -T Init | grep msdp

# 5. 检查数据目录是否创建
ls -l /data/service/el1/public/msdp
```

**证据**: msdp 配置文件 [etc/init/msdp.cfg:11-54](../etc/init/msdp.cfg:11)

---

### Q7: 数据目录未创建

**症状**:
```bash
ls -l /data/service/el1/public/sensor
ls: cannot access '/data/service/el1/public/sensor': No such file or directory
```

**原因**:
- Boot job 未执行
- 目录创建失败（权限不足）
- 分区挂载问题

**解决方法**:
```bash
# 1. 检查 INIT 日志
hilog -T Init | grep sensor

# 2. 手动创建目录（测试）
mkdir -p /data/service/el1/public/sensor
chown sensor:sensor /data/service/el1/public/sensor

# 3. 检查分区挂载
df -h | grep data
```

**证据**: sensors boot job [etc/init/sensors.cfg:5-6](../etc/init/sensors.cfg:5)

---

### Q8: 服务启动后立即退出

**症状**:
```bash
ps -ef | grep sensors
# 进程存在但立即退出

# 或日志显示
hilog -T sensors
# "Service exited with code 1"
```

**原因**:
- SA 配置文件错误
- 动态库加载失败
- 初始化失败

**解决方法**:
```bash
# 1. 查看 INIT 日志
hilog -T Init

# 2. 查看服务日志
hilog -T sensors
hilog -T msdp

# 3. 检查 SA 配置文件
cat /system/profile/sensors.json
cat /system/profile/msdp.json

# 4. 检查动态库是否存在
ls -l /system/lib/libsensor_service.z.so
ls -l /system/lib/libmiscdevice_service.z.so
```

---

## 权限问题

### Q9: msdp 服务权限被拒绝

**症状**:
```bash
hilog -T msdp
# "Permission denied"
```

**原因**:
- 权限未声明
- 用户未授权
- 权限系统配置错误

**解决方法**:
```bash
# 1. 检查配置文件中的权限声明
cat /etc/init/msdp.rc | grep permission

# 2. 检查权限系统配置
# (需要查阅权限系统文档)

# 3. 检查用户授权状态
# (需要查阅权限管理文档)
```

**证据**: msdp 权限列表 [etc/init/msdp.cfg:16-42](../etc/init/msdp.cfg:16)

---

### Q10: sensors 服务权限被拒绝

**症状**:
```bash
hilog -T sensors
# "Permission denied"
```

**原因**:
- 权限未声明
- UID/GID 配置错误

**解决方法**:
```bash
# 1. 检查配置文件中的权限声明
cat /etc/init/sensors.rc | grep permission

# 2. 检查 UID/GID 是否存在
getent passwd sensor
getent group sensor
```

**证据**: sensors 权限配置 [etc/init/sensors.cfg:15-18](../etc/init/sensors.cfg:15)

---

## 调试问题

### Q11: 如何查看服务启动日志？

**方法**:
```bash
# 1. 查看 INIT 日志
hilog -T Init

# 2. 查看 sensors 服务日志
hilog -T sensors

# 3. 查看 msdp 服务日志
hilog -T msdp

# 4. 查看所有相关日志
hilog | grep -E "Init|sensors|msdp"
```

---

### Q12: 如何查看服务状态？

**方法**:
```bash
# 1. 查看进程状态
ps -ef | grep sensors
ps -ef | grep msdp

# 2. 查看 SA 状态
hidumper -s 3601  # 传感器服务 (SA 3601)
hidumper -s 3602  # 震动器服务 (SA 3602)
hidumper -s -a    # 列出所有 SA

# 3. 查看服务属性
systemctl status sensors
systemctl status msdp
```

---

### Q13: 如何手动重启服务？

**方法**:
```bash
# 方法 1: 使用 systemctl
systemctl restart sensors
systemctl restart msdp

# 方法 2: 使用 svc control
svc control restart sensors
svc control restart msdp

# 方法 3: 使用 kill (不推荐，可能影响系统稳定性)
killall sensors
killall msdp
# INIT 会自动重启服务
```

---

## 架构相关问题

### Q14: 为什么 sensors 和 msdp 服务需要单独的启动配置？

**原因**:
- 两个服务属于不同的子系统
- 两个服务有不同的权限需求
- 两个服务可能由不同的团队维护

**证据**:
- sensors subsystem: [etc/init/BUILD.gn:25](../etc/init/BUILD.gn:25)
- msdp subsystem: [etc/init/BUILD.gn:36](../etc/init/BUILD.gn:36)

---

### Q15: sensors_sensor 和 sensors_miscdevice 如何共享 sensors 进程？

**说明**:
- `sensors_sensor` (传感器服务) 和 `sensors_miscdevice` (震动器等服务) 共享 `sensors` 进程
- 通过本仓库的统一配置文件启动
- 避免两个组件各自启动进程导致重复

**证据**: [README.md:24](../README.md:24)

**实现方式**:
- 两个组件的 SA 配置文件 (`sensors.json`) 合并为一个
- 由 SA 框架加载两个动态库到同一进程
- SA ID 分别为 3601 (传感器) 和 3602 (震动器)

---

### Q16: msdp 服务的 26 个权限都是必需的吗？

**分析**:
- 部分权限可能不是必需的
- 建议审计并移除不必要的权限
- 遵循最小权限原则

**详细分析**: [07_Security_Audit.md](./07_Security_Audit.md)

**建议**:
1. 审计 msdp 服务实际需要的权限
2. 移除不必要的权限
3. 考虑将 msdp 服务拆分为多个服务

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位
- [目录结构](./01_Directory_Structure.md) - 文件组织
- [架构说明](./02_Architecture.md) - 服务启动流程
- [GN Targets](./05_GN_Targets.md) - 构建配置
- [编译产物](./06_Build_Artifacts.md) - 安装和加载
- [安全评审](./07_Security_Audit.md) - 权限和安全

---

**最后更新**: 2026-02-06

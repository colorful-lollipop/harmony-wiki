# 08_Common_Troubleshooting.md

## 目的

本文档提供 XTS Tools 仓库的常见构建/运行/调试问题与定位路径，帮助开发者快速解决问题。

## 适用范围

- 仅涉及 tools/ 仓库的构建与运行问题
- 不涉及 acts 测试用例调试

---

## 常见构建问题

### 1. GN 编译失败

- 症状：GN 报错 "unknown target"、"dependency cycle"
- 定位路径：
  - 检查 BUILD.gn/.gni 文件语法
  - 检查 deps/public_deps 是否存在循环依赖
  - 查看 gn gen --check 输出
- 解决方法：
  - 修正 BUILD.gn 语法
  - 移除循环依赖
  - 参考 [05_GN_Targets.md](05_GN_Targets.md)

### 2. Ninja 执行失败

- 症状：Ninja 报错 "undefined reference"、"multiple definition"
- 定位路径：
  - 检查 .cpp/.c 文件实现
  - 检查链接库路径与顺序
  - 查看 ninja -C out/xxx -v 详细输出
- 解决方法：
  - 补充缺失符号实现
  - 修改链接顺序
  - 参考 [06_Build_Artifacts.md](06_Build_Artifacts.md)

---

## 常见运行问题

### 1. 测试套件挂载失败（Small/Standard 系统）

- 症状：NFS 挂载失败，测试套件无法执行
- 定位路径：
  - 检查网络连接（PC 与设备同网段）
  - 检查 NFS 服务器配置
  - 检查 mount 命令参数
- 解决方法：
  - 配置 IP 地址与子网掩码
  - 启动 NFS 服务
  - 正确执行 mount 命令
  - 证据：README.md:488-504

### 2. 测试用例执行失败

- 症状：测试用例断言失败、崩溃
- 定位路径：
  - 检查串口日志输出
  - 检查测试用例实现
  - 检查依赖库版本
- 解决方法：
  - 修正测试用例逻辑
  - 更新依赖库（Unity/Googletest）
  - 参考 [02_Architecture.md](02_Architecture.md)

---

## 常见工具链问题

### 1. HVIGR 检查失败

- 症状：check_hvigor.py 报错
- 定位路径：
  - 检查 HVIGR 工具版本
  - 检查 Python 脚本路径
  - 查看 standard_check/check_hvigor.py 输出
- 解决方法：
  - 更新 HVIGR 工具
  - 修正 Python 脚本路径
  - 参考 [01_Directory_Structure.md](01_Directory_Structure.md)

### 2. 文档格式化失败

- 症状：format_tc_doc.py 报错
- 定位路径：
  - 检查文档格式
  - 检查 Python 脚本依赖
  - 查看 xts-project-tools/format-tc-doc/ 输出
- 解决方法：
  - 修正文档格式
  - 安装 Python 依赖
  - 参考 [01_Directory_Structure.md](01_Directory_Structure.md)

---

## 调试技巧

### 1. GN 调试

- 使用 gn args out/xxx 查看当前构建参数
- 使用 gn desc out/xxx <target> 查看目标详情
- 使用 gn check out/xxx //... 检查依赖关系

### 2. Ninja 调试

- 使用 ninja -C out/xxx -v 查看详细构建日志
- 使用 ninja -C out/xxx -t targets 查看目标列表

### 3. Python 脚本调试

- 使用 python -m pdb `<script>`.py 进入调试模式
- 添加 print/logging 输出调试信息

---

## 日志与输出

### 串口日志（Mini/Small/Standard 系统）

- 格式："Start to run test suite: ..." → "xx Tests xx Failures xx Ignored"
- 定位路径：
  - 连接串口工具
  - 保存日志至文件
- 证据：README.md:368-369

### Python 脚本日志

- 工具链日志：print/logging 输出
- 定位路径：查看终端输出或日志文件

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [05_GN_Targets.md](05_GN_Targets.md)
- [06_Build_Artifacts.md](06_Build_Artifacts.md)

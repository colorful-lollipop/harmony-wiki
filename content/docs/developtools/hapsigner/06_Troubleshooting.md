# 常见问题

## 目的

本文档整理 hapsigner 常见的构建、运行和调试问题，提供问题定位路径和解决方案。

## 适用范围

- 初次使用 hapsigner 的开发者
- 遇到问题的运维人员

---

## 构建问题

### Q1: Maven 构建失败，提示依赖无法下载

**现象**:
```
[ERROR] Failed to execute goal on project hap_sign_tool_lib: 
Could not resolve dependencies for project com.ohos:hap_sign_tool_lib:jar:4.0
```

**原因**:
- 网络连接问题
- Maven 仓库配置问题
- 依赖版本冲突

**解决方案**:
1. 检查网络连接，确保可以访问 Maven Central
2. 配置镜像仓库（如阿里云镜像）
   ```xml
   <!-- 在 ~/.m2/settings.xml 中添加 -->
   <mirror>
       <id>aliyunmaven</id>
       <mirrorOf>*</mirrorOf>
       <name>阿里云公共仓库</name>
       <url>https://maven.aliyun.com/repository/public</url>
   </mirror>
   ```
3. 清理本地仓库缓存
   ```bash
   rm -rf ~/.m2/repository/com/ohos
   mvn clean package
   ```

**代码证据**: `hapsigntool/pom.xml:26-61` 定义了依赖管理

---

### Q2: GN 构建找不到 OpenSSL 头文件

**现象**:
```
error: 'openssl/evp.h' file not found
#include <openssl/evp.h>
         ^~~~~~~~~~~~~~~~~
```

**原因**:
- OpenSSL 未安装
- include 路径配置错误
- 构建环境不完整

**解决方案**:
1. 确保 OpenSSL 开发包已安装
   ```bash
   # Ubuntu/Debian
   sudo apt-get install libssl-dev

   # macOS
   brew install openssl
   ```
2. 检查 BUILD.gn 中的 include 路径
   ```gn
   include_dirs = [
       "//third_party/openssl/include",
       "//third_party/openssl/crypto/pkcs12",
   ]
   ```
3. 确保在 OpenHarmony 源码环境中构建

**代码证据**: `hapsigntool_cpp/BUILD.gn:43-48`

---

### Q3: C++ 编译报错，提示 C++17 特性不支持

**现象**:
```
error: 'std::filesystem' is not available in C++14
```

**原因**:
- 编译器版本过低
- 编译标志未正确设置

**解决方案**:
1. 升级编译器到 GCC 7+ 或 Clang 5+
2. 检查 BUILD.gn 中的编译标志
   ```gn
   cflags_cc = [
       "-std=c++17",
       "-fno-rtti",
   ]
   ```

**代码证据**: `hapsigntool_cpp/BUILD.gn:97-100`

---

## 运行问题

### Q4: 运行 JAR 包提示 "找不到主类"

**现象**:
```
Error: Could not find or load main class com.ohos.hapsigntool.HapSignTool
```

**原因**:
- JAR 包 manifest 配置错误
- 依赖库未包含

**解决方案**:
1. 使用正确的 JAR 包（带依赖的 fat JAR）
   ```bash
   java -jar hap_sign_tool/target/hap-sign-tool.jar
   ```
2. 检查 pom.xml 中的打包配置
3. 使用 Maven  shade 插件打包

**代码证据**: `hapsigntool/hap_sign_tool/pom.xml` 定义了打包配置

---

### Q5: 签名时提示 "密钥库密码错误"

**现象**:
```
KEYSTORE_PASSWORD_ERROR: keyStore password error
```

**原因**:
- 密码输入错误
- Keystore 文件损坏
- 密码编码问题

**解决方案**:
1. 使用交互模式输入密码（避免命令行历史泄露）
   ```bash
   java -jar hap-sign-tool.jar sign-app \
       -pwdInputMode 1 \
       -keystoreFile keystore.p12 \
       ...
   ```
2. 验证 Keystore 文件完整性
   ```bash
   keytool -list -v -keystore keystore.p12
   ```
3. 检查密码编码（确保使用 UTF-8）

**代码证据**: `hapsigntool_cpp/utils/src/key_store_helper.cpp:505-527`

---

### Q6: 签名 HAP 时提示 "文件格式不支持"

**现象**:
```
NOT_SUPPORT_ERROR: Not support file: app.txt
```

**原因**:
- 文件扩展名不正确
- 文件内容不是有效的 ZIP/HAP

**解决方案**:
1. 确保文件扩展名为 `.hap`, `.hsp`, `.hqf`, `.zip`
2. 验证文件是否为有效的 ZIP 格式
   ```bash
   unzip -t app.hap
   ```
3. 使用正确的 `-inForm` 参数
   ```bash
   java -jar hap-sign-tool.jar sign-app \
       -inForm zip \
       -inFile app.zip \
       ...
   ```

**代码证据**: `hapsigntool/hap_sign_tool/src/main/java/com/ohos/hapsigntool/HapSignTool.java:85-89`

---

### Q7: 代码签名失败，提示 "FsVerity 生成错误"

**现象**:
```
Error: Failed to generate fsverity descriptor
```

**原因**:
- 文件过大
- 内存不足
- Merkle 树构建失败

**解决方案**:
1. 检查文件大小，确保在支持范围内
2. 增加 JVM 内存（Java 版本）
   ```bash
   java -Xmx4g -jar hap-sign-tool.jar ...
   ```
3. 检查磁盘空间是否充足

**代码证据**: `hapsigntool_cpp/codesigning/fsverity/src/fs_verity_generator.cpp`

---

## 验证问题

### Q8: 验证签名时提示 "证书链验证失败"

**现象**:
```
VERIFY_ERROR: Certificate chain verification failed
```

**原因**:
- 证书过期
- 证书链不完整
- 根证书不受信任

**解决方案**:
1. 检查证书有效期
   ```bash
   openssl x509 -in cert.pem -noout -dates
   ```
2. 确保证书链完整（包含中间 CA）
3. 使用正确的根证书进行验证

**代码证据**: `hapsigntool_cpp/hap/verify/src/verify_hap.cpp`

---

### Q9: 验证 Profile 时提示 "PKCS7 解析失败"

**现象**:
```
Error: PKCS7 parse failed
```

**原因**:
- Profile 文件损坏
- 不是有效的 p7b 格式
- 编码问题

**解决方案**:
1. 检查文件是否为有效的 PKCS7 格式
   ```bash
   openssl pkcs7 -in profile.p7b -inform DER -noout
   ```
2. 重新签名 Profile
3. 检查文件传输过程中是否损坏

---

## 调试技巧

### 启用详细日志

**Java 版本**:
```bash
java -Dlog4j.configurationFile=log4j2.xml -jar hap-sign-tool.jar ...
```

**C++ 版本**:
```bash
# 设置日志级别环境变量
export SIGNATURE_TOOLS_LOG_LEVEL=DEBUG
./hap-sign-tool ...
```

### 检查签名块内容

```bash
# 提取 HAP 签名块
unzip -p app.hap signature_block > sign_block.bin

# 使用 OpenSSL 查看 PKCS7 内容
openssl pkcs7 -in sign_block.bin -inform DER -print_certs -noout
```

### 验证证书链

```bash
# 验证证书链完整性
openssl verify -CAfile root.cer -untrusted sub.cer app.cer
```

---

## 性能优化

### 大文件签名优化

1. 使用 C++ 版本替代 Java 版本
2. 禁用代码签名（如不需要）
   ```bash
   -signCode 0
   ```
3. 使用 SSD 存储临时文件

### 批量签名优化

1. 使用并行处理
2. 复用 Keystore 连接
3. 考虑使用远程签名服务

---

## 相关链接

- [API 参考](03_API_Reference.md) - 查看接口详情
- [构建系统](04_Build_System.md) - 了解构建配置
- [安全风险分析](05_Security_Analysis.md) - 了解安全注意事项

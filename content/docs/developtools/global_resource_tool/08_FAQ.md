# 常见问题

## 目的与适用范围

本文档汇总 `global_resource_tool` 的常见问题及解决方案，帮助开发者快速定位和解决问题。

**适用对象**: 工具使用者、构建工程师、开发者

---

## 构建问题

### Q1: 编译失败，提示缺少依赖库

**错误信息**:
```
ERROR: //third_party/cJSON:cjson_static not found
```

**原因**: 缺少第三方依赖库

**解决方案**:
1. 确保已同步所有子仓库:
   ```bash
   repo sync -c
   ```

2. 检查 `bundle.json` 中的依赖配置:
   ```json
   "deps": {
       "third_party": [
           "bounds_checking_function",
           "cJSON",
           "libpng"
       ]
   }
   ```

3. 确认第三方库路径正确:
   ```
   //third_party/cJSON
   //third_party/libpng
   //third_party/bounds_checking_function
   ```

---

### Q2: Windows 下编译失败，链接错误

**错误信息**:
```
undefined reference to `__imp_WideCharToMultiByte'
```

**原因**: Windows API 库未正确链接

**解决方案**:
1. 确保使用 MinGW 工具链
2. 检查 `BUILD.gn` 中的链接选项:
   ```gn
   if (is_mingw) {
       ldflags = [
           "-static",
           "-lws2_32",
           "-lshlwapi",
       ]
   }
   ```

---

## 运行问题

### Q3: 执行 restool 提示权限不足

**错误信息**:
```
Failed to create the directory or file '...', Permission denied.
```

**原因**: 当前用户对目标目录没有写权限

**解决方案**:
1. 检查输出目录权限:
   ```bash
   ls -la {output_dir}
   ```

2. 修改目录权限:
   ```bash
   chmod 755 {output_dir}
   ```

3. 或使用 `-f` 选项强制覆盖（谨慎使用）:
   ```bash
   restool -i {input} -o {output} -f ...
   ```

---

### Q4: 资源编译失败，提示无效的资源路径

**错误信息**:
```
Error: invalid input '...'
```

**原因**: 
- 输入路径不存在
- 路径包含非 ASCII 字符（Windows）
- 路径格式不正确

**解决方案**:
1. 确认输入路径存在:
   ```bash
   ls -la {input_path}
   ```

2. Windows 下确保路径为 ASCII 字符

3. 使用绝对路径:
   ```bash
   restool -i $(pwd)/resources ...
   ```

---

### Q5: 输出目录已存在，如何覆盖

**错误信息**:
```
Error: output path already exists
```

**解决方案**:
使用 `-f` 或 `--forceWrite` 选项强制覆盖:
```bash
restool -i {input} -o {output} -f ...
```

**注意**: 使用 `-f` 会删除原有输出目录，请确保已备份重要数据。

---

## 资源编译问题

### Q6: 资源 ID 冲突

**错误信息**:
```
Error: resource id duplicate
```

**原因**: 
- 多个资源使用相同名称
- 手动指定的 ID 与自动分配冲突

**解决方案**:
1. 检查资源文件命名，确保唯一性

2. 使用 `--defined-ids` 指定 ID 定义文件:
   ```bash
   restool -i {input} --defined-ids id_defined.json ...
   ```

3. 使用 `-e` 指定不同的起始 ID:
   ```bash
   restool -i {input} -e 0x02000000 ...
   ```

---

### Q7: JSON 资源格式错误

**错误信息**:
```
Failed to parse the JSON file: incorrect format
```

**原因**: JSON 文件格式不正确

**解决方案**:
1. 检查 JSON 语法:
   - 确保使用双引号
   - 删除多余的逗号
   - 确保括号匹配

2. 使用 JSON 校验工具验证

3. 参考示例格式:
   ```json
   {
       "string": [
           {
               "name": "app_name",
               "value": "MyApp"
           }
       ]
   }
   ```

---

### Q8: 资源引用无法解析

**错误信息**:
```
Error: ref not defined
```

**原因**: 引用的资源不存在或路径错误

**解决方案**:
1. 检查引用语法:
   ```json
   {
       "name": "bg_color",
       "value": "$color:primary"  // 正确
   }
   ```

2. 确保被引用的资源已定义

3. 检查资源类型匹配

---

## 纹理压缩问题

### Q9: 纹理压缩失败

**错误信息**:
```
Failed to load the library '...'
```

**原因**: 未找到纹理压缩库

**解决方案**:
1. 确认压缩库已安装:
   ```bash
   ls {SDK}/toolchains/libimage_transcoder_shared.so
   ```

2. 检查配置文件路径:
   ```json
   {
       "context": {
           "extensionPath": "/absolute/path/to/libimage_transcoder_shared.so"
       }
   }
   ```

3. 使用绝对路径指定库位置

---

### Q10: 如何禁用纹理压缩

**解决方案**:
不提供 `--compressed-config` 参数，或在配置中禁用:
```json
{
    "compression": {
        "media": {
            "enable": false
        }
    }
}
```

---

## 性能问题

### Q11: 资源编译速度慢

**解决方案**:
1. 使用多线程编译:
   ```bash
   restool -i {input} --thread 8 ...
   ```

2. 排除不需要的文件:
   ```bash
   restool -i {input} --ignored-file '\.git:\.svn' ...
   ```

3. 使用增量编译:
   ```bash
   # 1. 生成中间件
   restool -x {resources} -o {intermediate}
   
   # 2. 编译中间件
   restool -i {intermediate} -o {output} -z ...
   ```

---

### Q12: 内存不足

**错误信息**:
```
std::bad_alloc
```

**解决方案**:
1. 减少线程数:
   ```bash
   restool -i {input} --thread 2 ...
   ```

2. 分批编译资源

3. 增加系统可用内存

---

## 调试技巧

### 查看详细错误信息

使用 FAQ 链接获取详细帮助:
```bash
# 错误信息中会包含解决方案链接
Error: ...
Solutions:
> https://developer.huawei.com/consumer/...
```

### 启用调试输出

当前版本不支持详细调试日志，可通过以下方式排查:

1. 检查输入文件:
   ```bash
   ls -laR {input_path}
   ```

2. 验证 JSON 格式:
   ```bash
   python -m json.tool {config.json}
   ```

3. 逐步执行:
   ```bash
   # 先测试简单资源
   restool -i simple_resources -o test_out ...
   ```

---

## 版本兼容性

### Q13: 不同版本的 restool 兼容性

**API 版本对应**:
| restool 版本 | 支持 API | 特性 |
|--------------|----------|------|
| 6.1.0.003 | 18-23+ | 多线程、TS 头文件 |

**升级建议**:
1. 升级 SDK 时同步更新 restool
2. 检查新版本的命令行参数变化
3. 测试现有构建脚本兼容性

---

## 集成问题

### Q14: DevEco Studio 中资源编译失败

**排查步骤**:

1. 检查 SDK 配置:
   ```
   File -> Settings -> SDK -> HarmonyOS
   ```

2. 验证 restool 存在:
   ```bash
   ls {SDK}/toolchains/restool
   ```

3. 检查构建日志:
   ```
   Build -> Build Output
   ```

4. 清理重建:
   ```
   Build -> Clean Project
   Build -> Rebuild Project
   ```

---

### Q15: CI/CD 集成问题

**Docker 环境**:
```dockerfile
FROM ubuntu:20.04

# 安装依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    python3

# 复制 restool
COPY restool /usr/local/bin/
RUN chmod +x /usr/local/bin/restool

# 验证
RUN restool -v
```

**Jenkins 集成**:
```groovy
stage('Build Resources') {
    steps {
        sh '''
            ${RESTOOL_PATH}/restool \
                -i ${WORKSPACE}/entry/src/main \
                -j ${WORKSPACE}/entry/src/main/module.json \
                -p com.example.app \
                -o ${WORKSPACE}/build/res \
                -r ${WORKSPACE}/build/ResourceTable.h \
                -f
        '''
    }
}
```

---

## 错误码速查

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 11210001 | 未知选项 | 检查命令行参数拼写 |
| 11210004 | 无效输入 | 检查输入路径是否存在 |
| 11210007 | 无效输出 | 检查输出路径权限 |
| 11210013 | 无效起始 ID | 确保 ID 在有效范围内 |
| 11210026 | 无效线程数 | 使用正整数 |
| 11211001 | 输出已存在 | 使用 `-f` 强制覆盖 |
| 11211002 | 缺少配置 | 提供 `-j` 参数 |
| 11211117 | 资源重复 | 检查资源命名 |

完整错误码列表参见: `include/restool_errors.h`

---

## 获取帮助

### 官方文档
- [OpenHarmony 资源开发指南](https://gitee.com/openharmony/docs)

### 社区支持
- [OpenHarmony Issues](https://gitee.com/openharmony/developtools_global_resource_tool/issues)

### 报告问题
提供以下信息:
1. restool 版本 (`restool -v`)
2. 操作系统版本
3. 完整命令行
4. 错误信息
5. 最小复现步骤

---

## 相关文档

- [对外接口](03_Public_API.md) - 命令行参数详解
- [安全风险评审](06_SecurityReview.md) - 安全注意事项
- [GN 构建目标](07_GN_Targets.md) - 构建配置

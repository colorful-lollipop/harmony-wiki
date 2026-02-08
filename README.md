# Harmony Wiki

本项目使用 AI 辅助对 OpenHarmony 开源代码进行梳理（模块/架构/N-API/构建/安全等，尽量做到结论可追溯到源码），目前已启动约 480 个代码仓的整理并完成初稿。

欢迎在该 Wiki 平台留言反馈/补充内容一起共建（如有疏漏以源码与官方文档为准）。

## 🌐 在线访问

**GitHub Pages**: https://colorful-lollipop.github.io/harmony-wiki/

## 🚀 技术栈

本项目使用 [Hugo](https://gohugo.io/) + [hugo-book](https://github.com/alex-shpak/hugo-book) 主题构建。

## 📝 本地开发

### 安装 Hugo

```bash
# macOS
brew install hugo

# Linux
sudo apt install hugo

# Windows
choco install hugo-extended
```

### 启动开发服务器

```bash
hugo server -D
```

访问 http://localhost:1313/harmony-wiki/

### 构建

```bash
hugo --minify
```

构建结果在 `public/` 目录。

## 🤝 参与共建

欢迎通过以下方式参与：

- 在 Issues 中提交反馈或建议
- 提交 Pull Request 补充内容
- 在文档页面底部评论区留言

## 📄 许可证

本项目文档内容遵循相应开源协议，转载请注明出处。

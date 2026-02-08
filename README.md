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

本项目采用 [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)（署名 4.0 国际许可协议）。

[![CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

您可以自由地：
- **共享** — 在任何媒介以任何形式复制、发行本作品
- **演绎** — 修改、转换或以本作品为基础进行创作，在任何用途下，甚至商业目的

只要您遵循以下条款：
- **署名** — 您必须给出适当的署名，提供许可协议的链接，并标明是否进行了修改
- **无附加限制** — 您不得使用法律条款或技术措施限制他人做许可协议允许的事情

详见 [LICENSE](LICENSE) 文件。

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Harmony Wiki** is a documentation site for OpenHarmony source code analysis. It uses [Hugo](https://gohugo.io/) with the [hugo-book](https://github.com/alex-shpak/hugo-book) theme.

- **Live site**: https://colorful-lollipop.github.io/harmony-wiki/
- **Language**: Chinese (zh-CN)
- **License**: CC BY 4.0

## Development Commands

**Start development server:**
```bash
hugo server -D
```
Access at http://localhost:1313/harmony-wiki/

**Build for production:**
```bash
hugo --minify
```
Output goes to `public/` directory.

## Project Structure

```
content/docs/          # Documentation content (Markdown)
├── _index.md          # Home page
├── about/             # Project documentation
│   ├── _index.md
│   ├── readme.md      # Project overview
│   ├── structure.md   # Repository structure
│   ├── content.md     # Content system
│   ├── build.md       # Build & deployment
│   ├── contribute.md  # Contribution guide
│   └── summary.md     # Site navigation
├── applications/      # Application modules
│   ├── _index.md
│   ├── standard/      # Standard system apps
│   │   ├── settings/
│   │   ├── launcher/
│   │   └── ...
│   └── ...
├── arkcompiler/       # ArkCompiler documentation
├── base/              # Base system modules
├── build/             # Build system
├── commonlibrary/     # Common libraries
├── developtools/      # Development tools
├── device/            # Device development
├── drivers/           # Device drivers
├── foundation/        # Foundation system services
├── kernel/            # Kernel documentation
├── napi_generator/    # N-API generator
├── third_party/       # Third-party components
└── ...

layouts/partials/docs/inject/  # Custom layout injections
├── footer.html        # CC BY 4.0 license footer

themes/hugo-book/      # Hugo theme (Git submodule)
```

## Content Organization

Each module documentation typically includes:
- `00_Overview.md` - Module overview and positioning
- `01_Directory_Structure.md` - Code directory structure
- `02_Architecture.md` - Architecture design
- `03_API_Reference.md` or `04_NAPI_API.md` - API documentation
- `05_Build_System.md` or `06_GN_Targets.md` - Build configuration
- `06_Security_Analysis.md` or `08_Security_Audit.md` - Security review
- `09_FAQ.md` or `07_Troubleshooting.md` - FAQ and troubleshooting
- `appendix/` - Additional reference materials (callgraphs, config flags)

## Hugo Configuration

Key settings in `config.toml`:
- `baseURL`: https://colorful-lollipop.github.io/harmony-wiki/
- `theme`: hugo-book
- `BookSection`: '*' (all sections are book sections)
- `unsafe = true` in goldmark renderer (allows raw HTML)
- `enableGitInfo = false`: Disables Git info to improve build speed (5600+ files)
- `paginate = 20`: Limits pagination for faster builds

## Performance Optimization

With 5600+ Markdown files, Hugo server startup can be slow. Optimizations applied:

1. **config.toml changes (most effective):**
   - `BookSearch = false` - **Disables search** (hugo-book builds index on-the-fly for 5600+ files!)
   - `enableGitInfo = false` - Skips Git history reading for each file
   - `enableRobotsTXT = false` - Disables robots.txt generation in dev
   - Removed `defaultContentLanguage` - Site uses single language (zh-CN)

2. **Enable search for production builds:**
   ```bash
   # Build for production with search enabled
   hugo --minify  # uses config.toml which has BookSearch = true in production
   ```
   Or temporarily edit config.toml to set `BookSearch = true` before building.

3. **Development command with flags:**
   ```bash
   # Fastest startup - disables Git info (search already disabled in config)
   hugo server -D

   # Even faster - disable drafts too
   hugo server -D --buildDrafts=false
   ```

## Content Front Matter

Standard front matter for documentation pages:
```yaml
---
title: "Page Title"
type: docs
bookCollapseSection: true  # For section index pages
---
```

### Section Index Files (`_index.md`)

For Hugo to display nested menu structure, **every directory must have an `_index.md`** file:

```yaml
---
title: "applications"      # Use original directory name
type: docs
weight: 20                 # Controls menu order (smaller = first)
bookCollapseSection: true  # Makes section collapsible in menu
---
```

**Important fields:**
- `title`: Use the **original directory name** (e.g., `applications`, `arkcompiler`, `base`)
- `weight`: Controls the menu order. Root sections use weights 10, 20, 30... `about` uses 999 to appear last
- `bookCollapseSection: true`: All sections are collapsed by default

Without `_index.md`, Hugo won't recognize the directory as a section, and subdirectories won't appear in the left sidebar.

To batch create missing `_index.md` files:
```bash
# Run the helper script
python3 /tmp/create_index.py
```

### Menu Order

Root sections are sorted alphabetically with weights:
- `appendix`: 10
- `applications`: 20
- `arkcompiler`: 30
- `base`: 40
- ... (increment by 10)
- `vendor`: 190
- `about`: 999 (always last)

Subdirectories within each section are sorted alphabetically by their `title`.

## Custom Layouts

The project uses a custom footer injection at `layouts/partials/docs/inject/footer.html` to display CC BY 4.0 license information on all pages.

## Deployment

Automated via GitHub Actions (`.github/workflows/deploy.yml`):
- Triggers on push to `main` branch
- Uses `peaceiris/actions-hugo@v3` with extended version
- Deploys to GitHub Pages

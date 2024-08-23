今天分享的主题是使用 VitePress 搭建并自动化部署个人网站。

## VitePress 是什么
[VitePress](https://vitepress.dev/zh/guide/what-is-vitepress) 是一个静态站点生成器 (SSG)，专为构建快速、以内容为中心的站点而设计。简而言之，VitePress 获取用 Markdown 编写的内容，对其应用主题，并生成可以轻松部署到任何地方的静态 HTML 页面。

## 真实案例
https://chenlei0608.github.io/blog/

## 快速搭建一个 Demo
```bash
# 创建文件目录
$ mkdir vitepress-demo

# 进入文件目录
$ cd vitepress-demo

# 安装 vitepress
$ npm add -D vitepress

# 初始化 vitepress
$ npx vitepress init

# 回答几个简单的问题
┌  Welcome to VitePress!
│
◇  Where should VitePress initialize the config?
│  ./docs
│
◇  Site title:
│  My Awesome Project
│
◇  Site description:
│  A VitePress Site
│
◇  Theme:
│  Default Theme
│
◇  Use TypeScript for config and theme files?
│  Yes
│
◇  Add VitePress npm scripts to package.json?
│  Yes
│
└  Done! Now run npm run docs:dev and start writing.
```
![vitepress-demo-terminal](/vitepress-demo-terminal.png)

## 语法
VitePress 的语法不算多，下面只介绍容易引起疑惑的几个语法点

### base
- 类型：`string`
- 默认值： `/`

站点将部署到的 base URL。如果计划在子路径例如 GitHub 页面下部署站点，则需要设置此项。如果计划将站点部署到 `https://foo.github.io/bar/`，那么应该将 `base` 设置为 `'/bar/'`。它应该始终以 `/` 开头和结尾。

base 会自动添加到其他选项中以 `/` 开头的所有 URL 前面，因此只需指定一次。

```ts
export default {
  base: '/base/'
}
```

### 项目根目录

项目根目录是与 `.vitepress` 特殊目录同级的地方。`.vitepress` 目录是 VitePress 配置文件、开发服务器缓存、构建输出和可选主题自定义代码的预留位置。

### 源目录

源目录是 Markdown 源文件所在的位置。默认情况下，它与项目根目录相同。但是，可以通过 [`srcDir`](../reference/site-config#srcdir) 配置选项对其进行配置。

`srcDir` 选项是相对于项目根目录解析的。例如，对于 `srcDir: 'src'`，文件结构将如下所示：

```
.                          # 项目根目录
├─ .vitepress              # 配置目录
└─ src                     # 源目录
   ├─ getting-started.md
   └─ index.md
```

生成的源代码到 HTML 的映射：

```
src/index.md            -->  /index.html (可以通过 / 访问)
src/getting-started.md  -->  /getting-started.html
```

### srcDir
- 类型：`string`
- 默认值： `.`

相对于项目根目录的 markdown 文件所在的文件夹。

```ts
export default {
  srcDir: './src'
}
```

### frontmatter 配置
frontmatter 支持基于页面的配置。在每个 markdown 文件中，可以使用 frontmatter 配置来覆盖站点级别或主题级别的配置选项。此外，还有一些配置选项只能在 frontmatter 中定义。

示例用法：

```md
---
title: Docs with VitePress
editLink: true
---
```

> 详细：https://vitepress.dev/zh/reference/frontmatter-config

### 本地搜索
得益于 [minisearch](https://github.com/lucaong/minisearch/)，VitePress 支持使用浏览器内索引进行模糊全文搜索。要启用此功能，只需在 `.vitepress/config.ts` 文件中将 `themeConfig.search.provider` 选项设置为 `'local'` 即可：

```ts
import { defineConfig } from 'vitepress'

export default defineConfig({
  themeConfig: {
    search: {
      provider: 'local'
    }
  }
})
```

默认显示语言是英文，如果想要使用中文，请使用下面的写法：

```ts
import { defineConfig } from 'vitepress'

export default defineConfig({
  themeConfig: {
    search: {
      provider: 'local',
      options: {
        translations: {
          button: {
            buttonText: '搜索文档',
            buttonAriaLabel: '搜索文档'
          },
          modal: {
            noResultsText: '无法找到相关结果',
            displayDetails: '显示详细列表',
            resetButtonTitle: '清除查询条件',
            footer: {
              selectText: '选择',
              navigateText: '切换',
              closeText: '关闭'
            }
          }
        }
      }
    }
  }
})
```

## 使用 GitHub Pages 部署
1. 在项目的 `.github/workflows` 目录中创建一个名为 `deploy.yml` 的文件，其中包含这样的内容：

```yaml
# 构建 VitePress 站点并将其部署到 GitHub Pages 的示例工作流程
name: Deploy VitePress site to Pages

on:
  # 在针对 `main` 分支的推送上运行
  push:
    branches: [main]

  # 允许你从 Actions 选项卡手动运行此工作流程
  workflow_dispatch:

# 设置 GITHUB_TOKEN 的权限，以允许部署到 GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# 只允许同时进行一次部署，跳过正在运行和最新队列之间的运行队列
# 但是，不要取消正在进行的运行，因为我们希望允许这些生产部署完成
concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  # 构建工作
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # 如果未启用 lastUpdated，则不需要
      # - uses: pnpm/action-setup@v3 # 如果使用 pnpm，请取消注释
      # - uses: oven-sh/setup-bun@v1 # 如果使用 Bun，请取消注释
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm # 或 pnpm / yarn
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: npm ci # 或 pnpm install / yarn install / bun install
      - name: Build with VitePress
        run: npm run docs:build # 或 pnpm docs:build / yarn docs:build / bun run docs:build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs/.vitepress/dist # 构建输出目录

  # 部署工作
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

> [!NOTE]
> - 如果你使用的是 Github Pages 并部署到 `https://user-name.github.io/repo-name/`，请将 base 设置为 `/repo-name/`
> - 构建输出目录，如果按照上文的快速初始化的目录，是 `docs/.vitepress/dist`

2. 在存储库设置中的“Pages”菜单项下，选择“Build and deployment > Source > GitHub Actions”
![vitepress-demo-github-pages](/Snipaste_2024-08-07_16-24-24.png)

3. 将更改推送到 `main` 分支并等待 GitHub Action 工作流完成。你应该看到站点部署到 `https://user-name.github.io/repo-name/`。你的站点将在每次推送到 `main` 分支时自动部署。
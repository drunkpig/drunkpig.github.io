tags: 开发, 教程

[TOC]

## 为什么要写 mdwiki

`mdwiki` 是我自己写的一个极简静态博客生成器，最核心的想法只有一句话：

> 目录就是路由，Markdown 就是页面。

也就是说，你平时怎么整理文件，网站最终就怎么组织。

比如下面这棵目录树：

```text
post/
├── mdwiki博客使用教程.md
├── 2024/
│   └── 04/
│       └── 03/
│           └── hello.md
└── 2024-05/
    └── demo.md
```

构建后会自然变成：

```text
/mdwiki博客使用教程.html
/2024-04-03/hello.html
/2024-05/demo.html
```

这套思路的好处很直接：

- 不需要再单独学习一套路由规则
- 不需要写 front matter 去声明 slug、permalink、layout
- 文章放在哪个目录下，心里就知道它最后会出现在什么 URL
- 本地文件结构和线上网站结构高度一致，排查问题很省脑子

`mdwiki` 不是想做一个“功能最全”的博客系统，而是想做一个“把写作和发布过程压到最简单”的工具。

## mdwiki 的特色

### 1. 目录就是网站结构

很多静态站点生成器都会提供一层额外的抽象，例如：

- 配置 permalink
- 指定 content type
- 在 front matter 里写发布日期、分类、模板名
- 再通过主题规则把这些映射成最终页面

`mdwiki` 则反过来，尽量少引入概念。  
你只需要维护目录和文件名，生成器负责把这些 Markdown 变成网页。

### 2. 零配置就能生成完整博客

`mdwiki` 默认会帮你生成：

- 文章详情页
- 首页文章列表
- 分页
- 标签页
- 代码高亮
- 数学公式支持
- 本地图片复制

对个人博客来说，很多时候这就够了。

### 3. 更接近“写完就发布”的工作流

你写文章时关注的是内容，不是工程配置。  
一篇文章通常只需要：

```text
新建一个 md 文件
写内容
git push
```

剩下的构建和部署，可以交给 GitHub Actions。

### 4. 仓库历史更干净

现在这套博客已经改成：

- 仓库里只保存 Markdown 源文件和模板源码
- GitHub Actions 在线构建 `dist/`
- GitHub Pages 直接部署构建产物

这样就不会像早期“把生成后的 html 再提交回仓库”那样，导致仓库历史越来越胖。

## 和 Jekyll、Hugo 这类工具的对比

这里重点拿 Jekyll 来对比，因为 GitHub Pages 生态里它最经典。

### mdwiki 更轻的地方

和 Jekyll 相比，`mdwiki` 的优点主要是“简单、直接、可控”：

- Jekyll 有比较完整的站点模型，适合做功能丰富的网站；`mdwiki` 更像一个专门为 Markdown 博客准备的轻量生成器
- Jekyll 很依赖 front matter 和主题约定；`mdwiki` 更依赖目录结构本身
- Jekyll 的生态更大，但心智负担也更高；`mdwiki` 的概念更少，上手更快
- Jekyll 默认是 Ruby 生态；`mdwiki` 是 Python 工具，对 Python 用户更顺手

如果你只是想维护一个个人技术博客，`mdwiki` 的体验会非常干脆：

- 不需要先理解 Liquid 模板语言
- 不需要熟悉 `_posts`、`_layouts`、`_includes`
- 不需要折腾 Gem、Bundler
- 不需要在“生态很大”和“我其实只想发文章”之间做妥协

### mdwiki 适合什么人

它尤其适合下面这些场景：

- 你本来就习惯用目录整理资料
- 你希望文章就是普通 Markdown 文件
- 你想完全掌控生成逻辑
- 你希望工具链尽量短，方便自己改源码
- 你不想为了一个个人博客引入太重的框架

### Jekyll/Hugo 更强的地方

也要实话实说，`mdwiki` 并不是在所有方向都比 Jekyll/Hugo 更强。

如果你的需求是下面这些：

- 丰富主题生态
- 大量现成插件
- 多语言站点
- 内容模型复杂
- SEO、feed、菜单、taxonomy 都要非常成熟

那 Jekyll 或 Hugo 这类成熟工具会更省事。

所以 `mdwiki` 的优势，不是“功能更大”，而是：

> 在个人博客这个问题上，用更少的概念，解决足够多的事情。

## 安装与命令行用法

现在 `mdwiki` 已经改成 `uv` 和标准 Python 包管理方式，可以直接安装。

### 方式一：直接安装已发布版本

```bash
uv tool install mdwiki
```

如果你已经装过，升级也很简单：

```bash
uv tool upgrade mdwiki
```

### 方式二：从源码运行

```bash
git clone https://github.com/drunkpig/mdwiki.git
cd mdwiki
uv sync
uv run mdwiki_exec --help
```

帮助信息和版本信息都可以直接查看：

```bash
mdwiki_exec --help
mdwiki_exec --version
```

最基本的构建命令是：

```bash
mdwiki_exec <source_dir> <dist_dir>
```

例如：

```bash
mdwiki_exec ./post ./dist
```

这条命令的含义就是：

- 从 `post/` 读取 Markdown 源文件
- 把生成好的静态站点输出到 `dist/`

## 写作方式

`mdwiki` 的使用方式很朴素，就是往目录里放 Markdown。

### 一篇最简单的文章

```md
tags: Python, 教程

# 这是标题

这里是正文。
```

其中：

- 文件名会成为页面标题的一部分
- `tags:` 会被读取出来，自动进入标签页
- 本地图片如果用相对路径引用，构建时会自动复制到输出目录

### 图片与目录建议

一个很自然的写法是把图片放在文章旁边：

```text
post/
└── 2024/
    └── 04/
        └── 03/
            ├── hello.md
            └── image.png
```

Markdown 中直接这样引用：

```md
![demo](image.png)
```

构建时 `mdwiki` 会自动把它复制到对应页面输出目录里。

### 标签、目录和分页

默认主题会自动生成：

- 首页分页列表
- 标签聚合页
- 文章详情页的目录

如果你平时写技术文档，这套默认能力基本就够用。

## GitHub Pages 的推荐部署方式

以前很多个人博客会用这种方式部署：

1. 本地生成 html
2. 把生成产物提交到仓库
3. 再让 GitHub Pages 直接读取这些 html

这种做法最大的问题是：

- 每次发布都会把新的构建结果写进 Git 历史
- 仓库会越来越大
- 生成产物和源码混在一起，不方便维护

现在这套博客已经改成更合理的方式：

1. 仓库只保存源码
2. GitHub Actions 在线安装 `mdwiki`
3. 构建生成 `dist/`
4. GitHub Pages 直接部署 artifact

这也是现在官方更推荐的用法。

## 怎么配置 GitHub

下面这套流程，适合直接部署到 GitHub Pages。

### 1. 准备博客仓库

仓库里至少要有：

```text
.github/workflows/pages.yml
post/
post/CNAME      # 可选
```

其中 `post/` 是你的文章源目录。

### 2. 新建 GitHub Actions workflow

在仓库里创建：

```text
.github/workflows/pages.yml
```

内容可以参考我现在博客正在使用的这份：

```yaml
name: Build and Deploy Pages

on:
  push:
    branches: ["master", "source"]
  pull_request:
    branches: ["master", "source"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v6

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Install mdwiki
        run: |
          python -m pip install --upgrade pip
          pip install "mdwiki==0.3.3"
          mdwiki_exec --version

      - name: Build site
        run: |
          rm -rf dist
          mdwiki_exec ./post ./dist
          if [ -f CNAME ]; then cp CNAME dist/CNAME; elif [ -f post/CNAME ]; then cp post/CNAME dist/CNAME; fi
          touch dist/.nojekyll

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v4
        with:
          path: ./dist

  deploy:
    if: github.event_name == 'push'
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

### 3. 打开 GitHub Pages

进入仓库：

```text
Settings -> Pages
```

把发布方式设成：

```text
Build and deployment -> Source -> GitHub Actions
```

这一步很关键。  
如果还是老的 branch 模式，workflow 虽然能跑，但不会用 artifact 来发布。

### 4. 配置自定义域名

如果你有自己的域名，只需要在源码目录里放一个 `CNAME` 文件。

比如：

```text
mkmerich.com
```

在上面的 workflow 里，这个文件会在构建后被复制进 `dist/`，GitHub Pages 会自动识别。

然后到你的域名服务商后台，把 DNS 指到 GitHub Pages 即可。

### 5. 后续发布流程

以后写文章，流程就很简单：

```bash
git add .
git commit -m "add new post"
git push origin master
```

推送之后：

- GitHub Actions 自动安装 `mdwiki`
- 自动构建整站
- 自动部署到 GitHub Pages

整个过程中，不需要再把生成后的 html 提交回仓库。

## 一个最小可用目录示例

如果你想从零开始，可以先准备这样一个结构：

```text
my-blog/
├── .github/
│   └── workflows/
│       └── pages.yml
└── post/
    ├── CNAME
    ├── mdwiki博客使用教程.md
    └── 2026/
        └── 04/
            └── 03/
                └── hello.md
```

然后本地先试跑：

```bash
mdwiki_exec ./post ./dist
```

确认 `dist/` 生成正常后，再提交到 GitHub。

## 总结

`mdwiki` 最适合的，不是超复杂的网站，而是个人博客、知识记录和技术文章归档。

它的价值不在于做了多少“花活”，而在于下面这几点：

- Markdown 到网页的路径非常短
- 目录结构和网站结构天然一致
- 默认功能够用，不必上来就堆一大堆配置
- 很容易自己改源码
- 和 GitHub Actions 组合后，部署链路非常干净

如果你想要的是一个能快速写、快速发、还能完全掌控源码的博客工具，`mdwiki` 这条路其实很舒服。

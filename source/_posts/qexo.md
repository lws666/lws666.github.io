---
abbrlink: ''
categories: []
date: '2026-03-11T17:37:04.047225+08:00'
tags:
- hexo
- github
title: github自动化部署 hexo
updated: '2026-03-11T21:45:12.038+08:00'
---
**Hexo 自动化部署最推荐的方式：GitHub Actions + GitHub Pages**

这是目前 Hexo 官方推荐的自动化部署方案。

每次你把文章推送到 GitHub 主分支（`main` 或 `master`），GitHub Actions 就会**自动**：

1. 安装依赖
2. 执行 `hexo generate` 生成静态文件
3. 部署到 GitHub Pages

整个过程无需服务器、无需手动 `hexo d`，完全免费（GitHub Actions 每月 2000 分钟免费额度足够个人使用）。

### 前提条件（必须先完成）

1. 在 GitHub 创建仓库，名称必须是 **`你的用户名.github.io`**（例如 `ws.github.io`）。
2. 把你的 Hexo 项目整个文件夹 push 到该仓库的默认分支（`main`）。

   - 确保 `.gitignore` 里有一行 `public/`（静态文件不要上传）。
3. 本地运行 `node --version`，记下主版本号（例如 v20.x.x → 记 `20`）。

### 详细部署步骤

#### 步骤 1：在 GitHub 设置 Pages 来源

1. 进入你的仓库 → **Settings** → **Pages**
2. 在 **Source** 下拉菜单选择 **GitHub Actions**
3. 点击 **Save** 保存。

#### 步骤 2：创建 GitHub Actions 工作流文件

在项目根目录新建文件夹和文件：

```

.github/workflows/pages.yml

```

把下面**完整内容**复制进去（把 `20` 改成你本地 Node 主版本号yaml

YAML

name: Pages



on:

  push:

```
branches:
```


```
- main # 如果你的默认分支是 master，改成 master
```




jobs:

  build:

```
runs-on: ubuntu-latest
```


```
steps:
```


```
- uses: actions/checkout@v4
```


```
with:
```


```
token: ${{ secrets.GITHUB_TOKEN }}
```


```
submodules: recursive   # 如果用了主题子模块，保留这行
```




```
- name: Use Node.js 20
```


```
uses: actions/setup-node@v4
```


```
with:
```


```
node-version: "20"      # ← 这里改成你的 Node 主版本
```


```
cache: 'npm'
```




```
- name: Cache NPM dependencies
```


```
uses: actions/cache@v4
```


```
with:
```


```
path: node_modules
```


```
key: ${{ runner.OS }}-npm-cache
```


```
restore-keys: |
```


```
${{ runner.OS }}-npm-cache
```




```
- name: Install Dependencies
```


```
run: npm install
```




```
- name: Build
```


```
run: npm run build          # 等同于 hexo generate
```




```
- name: Upload Pages artifact
```


```
uses: actions/upload-pages-artifact@v3
```


```
with:
```


```
path: ./public
```




  deploy:

```
needs: build
```


```
runs-on: ubuntu-latest
```


```
permissions:
```


```
pages: write
```


```
id-token: write
```


```
environment:
```


```
name: github-pages
```


```
url: ${{ steps.deployment.outputs.page_url }}
```


```
steps:
```


```
- name: Deploy to GitHub Pages
```


```
id: deployment
```


```
uses: actions/deploy-pages@v4
```




#### 步骤 3：提交并触发部署

```bash

git add .

git commit -m "Add GitHub Actions for auto deploy"

git push

```

提交后，进入仓库 → **Actions** 标签页，就能看到正在运行的 workflow。

成功后，访问 `https://你的用户名.github.io` 即可看到博客！

### 常见注意事项

- **自定义域名**：在 `source/` 文件夹新建 `CNAME` 文件，里面写你的域名（例如 `blog.example.com`），然后重新 push 一次。
- **主题是子模块**：上面的 `submodules: recursive` 已自动处理。
- **加速构建**：使用了 npm cache，第二次及以后构建会更快。
- **私有仓库**：同样支持，Pages 页面仍是公开的。

### 其他自动化部署方案（可选）

- **Vercel / Netlify**（最简单）：

  直接在 Vercel/Netlify 网站上用 GitHub 一键导入仓库，推送代码就自动部署，无需写任何 YAML。
- **部署到自己服务器/VPS**：

  在 workflow 里加 SSH 密钥（Secrets），用 `rsync` 或 `scp` 把 `public/` 传上去，适合有独立域名+服务器的用户。
- **传统 hexo-deployer-git**：仍可使用，但需要配置 SSH 密钥 + Actions 运行 `hexo deploy`，不如官方 Actions 优雅。

### 快速验证是否成功

1. 改一篇文章，`git push`
2. 去 Actions 看流水线（1-2 分钟完成）
3. 刷新你的 `.github.io` 页面，看到更新就成功了！

照着上面操作，基本 10 分钟内就能搞定自动化部署。

现在就去试试吧！推送一次代码，以后写文章再也不用手动部署了～ 🚀

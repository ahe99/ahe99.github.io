---
title: Hexo + Github Actions，自動部署Markdown文章
comments: false
cover: /img/posts/Hexo-Github-Actions，自動化部署Markdown文章/cover.png
date: 2022-10-04 21:45:21
updated:
tags:
  - 前端
  - Hexo
  - Github Actions
  - Github Pages
  - Markdown
  - CI/CD
categories:
  - 技術
keywords:
description:
top_img:
toc:
toc_number:
toc_style_simple:
copyright:
copyright_author:
copyright_author_href:
copyright_url:
copyright_info:
mathjax:
katex:
aplayer:
highlight_shrink:
aside:
---

---

## 前言

在上一篇文章[你好世界]中有提到，

為了好好專心寫文章，於是使用了 [Hexo] + [Github Actions]

這樣一個組合來快速搭建一個自動化部署的 Blog

![flow](/img/posts/Hexo-Github-Actions，自動化部署Markdown文章/flow.png)

今天就來說要怎麼做吧～

---

<!-- ## 目錄

- [事前準備](#事前準備)
- [Hexo 介紹](#hexo-介紹)
- [Github Actions 介紹](#github-actions-介紹)

--- -->

## 事前須知

- 安裝 [Node.js、npm]
- 安裝 [Git]
- 基礎 Git、Github (pages) 概念
- 基礎 Markdown 概念

---

## Hexo int

`Hexo`，簡單來說，就是讓使用者能夠省略許多建置過程、寫`Markdown`當作文章的框架

且有許多強大的`Plugins`和`Themes`可以做選擇

### 安裝

我們可以透過`npm`去安裝`Hexo-cli`

這個工具可以幫我們創建 `Hexo`專案、編譯...等

```sh
npm install hexo-cli -g
```

### 創建專案

```sh
hexo init (blog名稱)
cd (blog名稱)
```

### 專案架構

```console
├── _config.landscape.yml：Hexo主題的設定檔
├── _config.yml：Hexo的設定檔
├── node_modules
├── package-lock.json
├── package.json
├── scaffolds：Markdown模板
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source：專案主要內容（文章、頁面）
│   └── _posts：文章目錄
│       └── hello-world.md Hexo送你的教學文章（？
└── themes
```

這邊可以特別說明一下，這裡有兩個`config`

- \_config.landscape.yml
- \_config.yml

因為**Hexo 預設的 Theme**是`landscape`

像本站用的`butterly`，就會是：

- \_config.butterly.yml
- \_config.yml

以此類推，

設定的詳細內容，因為篇幅問題

所以應該之後會再拿來水文章 XD

### 新增文章

新增文章除了可以直接在`_posts`底下新增 Markdown 檔案，

也可以透過指令

```sh
hexo new post (文章名稱)
```

這個指令會從`scaffold`底下，找到你指定的模板

去創一個新的`Markdown`在`source`底下，

以這裡來說就是`post`，也就是你的文章

![new-post](/img/posts/Hexo-Github-Actions，自動化部署Markdown文章/new-post.png)

這邊我們就可以使用`Markdown`撰寫文章在`test.md`：

```markdown
---
title: test
date: 2022-09-04 21:15:21
tags:
---

---

## 測試

大家好，這裡是**測試**的文章。
```

### 本地預覽

文章也寫好了，那當然在正式上去之前

還是會希望能夠看到文章在網站上呈現的樣子

這時候就可以使用：

```sh
hexo server
```

來在本機架起網站環境

(預設會是`http://localhost:4000`)

![preview](/img/posts/Hexo-Github-Actions，自動化部署Markdown文章/preview.png)

---

## Github Actions 介紹

在新增完文章過後，準備 **Push 到 Github** 之前，

需要注意的是，**沒有編譯過的程式碼，沒辦法直接變成網站**

所以我們需要對現在的 code 進行一些額外處理

有兩種方式：

![two-way-deploy](/img/posts/Hexo-Github-Actions，自動化部署Markdown文章/two-way-deploy.png)

前者的流程代表本地編譯之後上傳 Github

而後者會直接上傳 Github，**由 Github 幫我們編譯**

兩種方式都可以，今天要介紹的是後者的方式

會使用到 `Github Actions`，也就是 `Github CI`

這檔事情，簡單來說就是叫 Github 的 server 去做一些指令操作

像這邊我們希望將`編譯(build)`的這個動作交給 Github 去做

就可以依照 [Hexo 官方建議的方式]

在專案目錄底下創建 `.github/workflows/pages.yml`

```yml
# .github/workflows/pages.yml
name: Pages

on:
  push:
    branches:
      - main # 你會推上的Github分支

jobs:
  pages:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

可以注意的是這一段

```yml
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./public
```

這邊的作用就是會將 build 好的 code

推到你 Github repo 的另外一個 branch `gh-pages`

於是最後在我們 Giuhub pages repository 設定上

![github-pages](/img/posts/Hexo-Github-Actions，自動化部署Markdown文章/github-pages.png)

這裡有有畫綠底線代表說大多數Github pages

repo跟網址的關係

```text
repo: 你的帳號/你的帳號.github.io

url: https://你的帳號.github.io/
```

這裡就不多贅述

而在右下角綠框設定

- Deploy from a branch
- gh-pages

這樣就大功告成了！

每次文章寫好後，Git push上到Github時

都會跑Github actions將網站更新

---

[你好世界]: https://ahe99.github.io/2022/08/28/%E4%BD%A0%E5%A5%BD%E4%B8%96%E7%95%8C/
[hexo]: https://hexo.io/zh-tw/
[github actions]: https://github.com/features/actions
[butterfly theme]: https://github.com/jerryc127/hexo-theme-butterfly
[node.js、npm]: https://nodejs.org/zh-tw/
[git]: https://git-scm.com/
[hexo 官方建議的方式]: https://hexo.io/zh-tw/docs/github-pages.html

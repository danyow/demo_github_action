[![WebGL Demo](https://img.shields.io/badge/demo-WebGL-orange.svg?style=flat&logo=google-chrome&logoColor=white&cacheSeconds=2592000)](https://danyow.github.io/demo_github_action/)

# 如何用 `Github Action` 来完成自己的 '4399' 在线游戏库!

## 原理

1. `Github Action` 自动构建工作流
2. `Github Pages` 

## 手动确保自己的项目可以构建出 `WebGL` 项目并运行

1. 在工程根目录新建 `docs`, 并建立一个名为 `.nojekyll` 文件

    ```tree
    docs
    └── .nojekyll
    ```

   > 其中 `.nojekyll` 作用是告诉 `Github Pages` 不是 `jekyll` 项目

2. 在 `Unity Editor` 中构建并运行, 目录选择 `docs` 得到下列目录结构

    ```tree
    docs
   ├── Build
   │   ├── website.data.gz
   │   ├── website.framework.js.gz
   │   ├── website.loader.js
   │   └── website.wasm.gz
   ├── index.html
   └── TemplateData
       ├── favicon.ico
       ├── fullscreen-button.png
       ├── progress-bar-empty-dark.png
       ├── progress-bar-empty-light.png
       ├── progress-bar-full-dark.png
       ├── progress-bar-full-light.png
       ├── style.css
       ├── unity-logo-dark.png
       ├── unity-logo-light.png
       └── webgl-logo.png
    ```
3. 推送项目, 并且启用, `Github Pages`
   - 在仓库页面点击 `Settings`
   - 左侧倒数第二(左右) 点击`Pages`
   - 看到一个 `None` 的下拉框, 选中 `main` 分支, (可以是任意分支, 类似 `master` 或者 `gh-pages`)
   - 修改后面的 `/(root)` 为 `docs`
   - 点击 `save` 保存即可
   - 等待片刻可以看到自己的在线游戏页面

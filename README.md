[![WebGL Demo](https://img.shields.io/badge/demo-WebGL-orange.svg?style=flat&logo=google-chrome&logoColor=white&cacheSeconds=2592000)](https://danyow.github.io/demo_github_action/)

# 如何用 `Github Action` 来完成自己的 '4399' 在线游戏库!

## 原理

1. `Github Action` 自动构建工作流
2. `Github Pages` 

## 手动确保自己的项目可以构建出 `WebGL` 项目并运行

1. 在工程根目录新建 `website`, 并建立一个名为 `.nojekyll` 文件

    ```tree
    website
    └── .nojekyll
    ```

   > 其中 `.nojekyll` 作用是告诉 `Github Pages` 不是 `jekyll` 项目

2. 在 `Unity Editor` 中构建并运行, 目录选择 `website` 得到下列目录结构

    ```tree
    website
    └── .nojekyll
    ```
3. 推送项目
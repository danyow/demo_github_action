# 如何用 `Github Action` 来完成自己的 '4399' 在线游戏库!

[![构建🍳 → 发布🍻](https://github.com/danyow/demo_github_action/actions/workflows/main.yml/badge.svg?branch=main)](https://github.com/danyow/demo_github_action/actions/workflows/main.yml)

[![WebGL Demo](https://img.shields.io/badge/demo-WebGL-orange.svg?style=flat&logo=google-chrome&logoColor=white&cacheSeconds=2592000)](https://danyow.github.io/demo_github_action/)


## 原理

其实就是用上了 `Github Action` 自动构建和 `Github Pages`

## 首先手动确保自己的项目可以构建出 _WebGL_ 项目并运行

1. 在工程根目录新建 `docs`, 并建立一个名为 `.nojekyll` 文件

```tree
docs
└── .nojekyll
```

> 其中 `.nojekyll` 作用是告诉 `Github Pages` 不是 `jekyll` 项目

2. 构建之前确保自己项目中的发布压缩格式选择为 `已禁用`

3. 在 `Unity Editor` 中构建并运行, 目录选择 `docs` 得到下列目录结构

```tree
docs
├── Build
│   ├── docs.data
│   ├── docs.framework.js
│   ├── docs.loader.js
│   └── docs.wasm
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

4. 推送项目, 并且启用, `Github Pages`
    - 在仓库页面点击 `Settings`
    - 左侧倒数第二点击 `Pages`
    - 看到一个 `None` 的下拉框, 选中 `gh-pages` 主分支, (当然可以是任意分支, 类似 `master` 或者 `main`)
    - 修改后面的 `/(root)` 为 `docs`
    - 点击 `save` 保存即可
    - 等待片刻可以看到自己的在线游戏页面

以上就算是完成了如何发布 _WebGL_ 游戏到 `Github Pages`

## 利用 `Github Pages` 完成以上复杂的种种操作

首先在根目录创建以下内容

```tree
.github
└── workflows
    ├── getManualLicenseFile.yml
    └── main.yml
```

### 在 `main.yml` 内填入

```yaml
name: 构建🍳 → 发布🍻
on:
  workflow_dispatch: { }
  push:
    branches:
      - main
    paths:
      - "Assets/**"
      - "Packages/**"
      - "ProjectSettings/**"

jobs:
  buildWithLinux:
    name: 为 ${{ matrix.targetPlatform }} 平台构建 🍳
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        targetPlatform:
          # - Android
          - WebGL
    outputs:
      targetPlatform: ${{ matrix.targetPlatform }}
    steps:
      - name: 获取代码存储库
        uses: actions/checkout@v2
        with:
          fetch-depth: 0
          lfs: true
      - name: 缓存Unity库文件夹
        uses: actions/cache@v2
        with:
          path: Library
          key: Library-build-${{ matrix.targetPlatform }}-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: |
            Library-build-${{ matrix.targetPlatform }}-
            Library-build-
      - name: 构建Unity工程
        uses: game-ci/unity-builder@main
        env:
          ### 以下按需注释或打开
          # 个人免费版
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
          # 订阅版
          # UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          # UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
          # UNITY_SERIAL: ${{ secrets.UNITY_SERIAL }}
        with:
          targetPlatform: ${{ matrix.targetPlatform }}
      - name: 上传构建结果
        uses: actions/upload-artifact@v2
        with:
          name: ${{ github.event.repository.name }}-${{ matrix.targetPlatform }}
          path: build/${{ matrix.targetPlatform }}

  releaseToGitHubPages:
    name: 通过GitHubPages发布WebGL🍻
    runs-on: ubuntu-latest
    needs: buildWithLinux
    steps:
      - name: 获取代码存储库
        uses: actions/checkout@v2
        with:
          ref: gh-pages
      - name: 下载WebGL构建包
        uses: actions/download-artifact@v2
        with:
          name: ${{ github.event.repository.name }}-WebGL
          path: tmp
      - name: 移动并覆盖所有下载文件
        run: |
          cp -rf tmp/WebGL/* docs
      - name: 查看下载后的内容
        run: |
          ls
          tree
      - name: 部署到GitHubPages
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          branch: gh-pages
          file_pattern: docs/**
          commit_message: 部署到GitHubPages
```

### _secrets_ 内容的获取和填写

#### UNITY_LICENSE 的获取

1. 在 `getManualLicenseFile.yml` 填入以下内容

```yml
name: 获取激活文件
on:
  workflow_dispatch: { }
jobs:
  activation:
    name: 请求手动激活文件 🔑
    runs-on: ubuntu-latest
    steps:
      - name: 请求手动激活文件
        id: getManualLicenseFile
        uses: game-ci/unity-request-activation-file@v2
      - name: 导出
        uses: actions/upload-artifact@v2
        with:
          name: ${{ steps.getManualLicenseFile.outputs.filePath }}
          path: ${{ steps.getManualLicenseFile.outputs.filePath }}
 ```

2. 推送该文件并手动运行该 `Action` 等待片刻得到一个 `Unity_v20XX.X.XXXX.alf` 并下载保存解压
3. 访问 [手动激活 Unity 许可证](https://license.unity3d.com/manual)
4. 上传刚刚得到的 `.alf` 文件
    - 可能会有 _serial has reached the maximum number of activations._ 这个问题的出现, 目前没有好的解决方案.
    - 我的解决方案是创建了一个新的账号, 这个账号不在个人电脑上操作.
5. 得到一个 `Unity_v20XX.x.ulf` 文件, 里面的内容就是一会儿要填入的内容 `UNITY_LICENSE`.

- 在仓库页面点击 `Settings`
    - 左侧倒数第二点击 `Secrets`
    - 点击 `New repository secret`
    - _Name_ 填入 `UNITY_LICENSE`
    - _Value_ 填入 `Unity_v20XX.x.ulf` 文件里的内容

#### 如果是订阅用户(付费的, 但是需要席位吧, 自己并不是没法测试)

一共有三个要填写

- `UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}`  : 你的 _Unity_ 的电子邮件地址
- `UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}` : 你的 _Unity_ 的密码
- `UNITY_SERIAL: ${{ secrets.UNITY_SERIAL }} ` : 从 [Unity 订阅页面](https://id.unity.com/en/subscriptions) 获取密钥.

### 进阶内容

构建的 `Github Pages` 也仅仅只是仅限于当前仓库的页面, 如果有个人主页的话, 可以一并同步过去岂不美哉

#### 首先获取 _Github Personal Access Token_

就是能推送构建内容到你其他仓库的一个凭证

1. 点击右上角个人头像 `Settings`
2. 点击右侧有个单独的一栏 `Developer settings`
3. 继续点击右侧第三个 `Personal access tokens`
4. `Generate new token`
5. 按需勾选得到一个 `ghp_` 开头的内容, 复制保存(只会在这里显示一次)

在 `secrets` 内加入新的值. 命名为 `GH_PERSONAL_ACCESS_TOKEN_FULL` (可以为任意值, 但不能以`GITHUB` 开头).

在 `main.yml` 后面补充以下内容

```yaml
  releaseToSite:
    name: 同时发布到个人站点🍻
    runs-on: ubuntu-latest
    needs: buildWithLinux
    steps:
      - name: 获取代码存储库
        uses: actions/checkout@v2
        with:
          repository: danyow/danyow
          token: ${{ secrets.GH_PERSONAL_ACCESS_TOKEN_FULL }}
          ref: gh-pages
      - name: 下载WebGL构建包
        uses: actions/download-artifact@v2
        with:
          name: ${{ github.event.repository.name }}-WebGL
          path: tmp
      - name: 移动并覆盖所有下载文件
        run: |
          mkdir ${{ github.event.repository.name }} -p
          cp -rf tmp/WebGL/*  ${{ github.event.repository.name }}
      - name: 查看下载后的内容
        run: |
          ls
          tree
      - name: 发布到个人站点
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          branch: gh-pages
          file_pattern: ${{ github.event.repository.name }}/**
          commit_message: ${{ github.event.repository.name }}发布到个人站点  
```
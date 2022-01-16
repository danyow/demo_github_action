[![WebGL Demo](https://img.shields.io/badge/demo-WebGL-orange.svg?style=flat&logo=google-chrome&logoColor=white&cacheSeconds=2592000)](https://danyow.github.io/demo_github_action/)

# 如何用 `Github Action` 来完成自己的 '4399' 在线游戏库!

## 原理

1. `Github Action` 自动构建工作流
2. `Github Pages`

## 手动确保自己的项目可以构建出 _WebGL_ 项目并运行

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
    - 左侧倒数第二(左右) 点击`Pages`
    - 看到一个 `None` 的下拉框, 选中 `main` 主分支, (当然可以是任意分支, 类似 `master` 或者 `gh-pages`)
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


### 获取 `Unity Editor` 个人版许可

1. 在 `getManualLicenseFile` 填入以下内容

   ```yml
   name: 获取激活文件
   on:
     workflow_dispatch: {}
   jobs:
     activation:
       name: 请求手动激活文件 🔑
       runs-on: ubuntu-latest
       steps:
         # 请求手动激活文件
         - name: 请求手动激活文件
           id: getManualLicenseFile
           uses: game-ci/unity-request-activation-file@v2
         # Upload artifact (Unity_v20XX.X.XXXX.alf)
         - name: 导出
           uses: actions/upload-artifact@v2
           with:
             name: ${{ steps.getManualLicenseFile.outputs.filePath }}
             path: ${{ steps.getManualLicenseFile.outputs.filePath }}
   ```

2. 推送该文件并手动运行该`Action`等待片刻得到一个`Unity_v20XX.X.XXXX.alf`并下载保存解压
3. 访问[手动激活 Unity 许可证](https://license.unity3d.com/manual)
4. 上传刚刚得到的`.alf`文件
   - 可能会有 _serial has reached the maximum number of activations._ 这个问题的出现, 目前没有好的解决方案. 
   - 我的解决方案是创建了一个新的账号, 这个账号不在个人电脑上操作.
5. 得到一个 `Unity_v20XX.x.ulf` 文件, 里面的内容保存.
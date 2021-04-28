---
title: hexo自动同步部署GitHub和Gitee pages
comments: true
date: 2021-03-23 13:55:12
tags:
    - hexo
categories:
    - front_end
---

这大概是最简单的hexo同步部署github和gitee pages的方案了。使用Github Actions + MarketPlace中的两个Actions，[Hexo Action](https://github.com/marketplace/actions/hexo-action)和[Gitee Pages Action](https://github.com/marketplace/actions/gitee-pages-action)，这样就可以做到配置一次，以后每次推送代码即可自动部署到两个pages中。

<!-- more -->

### 配置Hexo Action

  + 先决条件

    首先在安装了```hexo-deployer-git```的前提下，在`_config.yml`中设置

```
deploy:
  type: git
  repo: <repository url>	# 项目地址
  branch: [branch]		# 分支名
  message: [message]		#自定义提交信息

示例：

deploy:
  type: git
  repo:
    github:
      url: git@github.com:lxd0619/hexo_blog.git
      branch: gh-pages
  message: 版本发布
```

    其实这样就可以实现`hexo deploy`命令手动发布了，但是想要推送代码自动发布请接下向下看。

+ 生成SSH keys并设置Secrets。

    ``` $ ssh-keygen -t rsa -C "username@example.com" ```

​		生成SSH可参考[Connecting to GitHub with SSH](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)。

​		生成之后将私钥添加到所在项目的``` Settings > Secrets ```中，命名为``` Deploy Keys```（可更改，但需与下文中的```secrets.DEPLOY_KEY```相匹配）。

+  然后需要在项目的Actions中添加工作流（workflows）。

    可以直接使用示例，

```yml
name: Deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
    - name: Cache node modules
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install Dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: sma11black/hexo-action@v1.0.3
      with:
        deploy_key: ${{ secrets.DEPLOY_KEY }} # 此处则为secrets中设置的ssh私钥，如命名为DEPLOY_KEY则无需更改
        user_name: your github username  # (or delete this input setting to use bot account)
        user_email: your github useremail  # (or delete this input setting to use bot account)
        commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
    # Use the output from the `deploy` step(use for test action)
    - name: Get the output
      run: |
        echo "${{ steps.deploy.outputs.notify }}"
```

​		以上就是部署到github pages的步骤。

### 配置Gitee Pages Action

​		配置可直接参考[Gitee Pages Action](https://github.com/marketplace/actions/gitee-pages-action)说明文档，步骤与上文类似，纯中文，十分友好。

+ 准备工作

    生成SSH ksys（可参考[Gitee帮助——生成/添加SSH公钥](https://gitee.com/help/articles/4181#article-header0)），之后在`Settings > Secrets `中添加生成的私钥并命名为`GITEE_RSA_PRIVATE_KEY `，以及Gitee的密码`GITEE_PASSWORD `。

+ 添加Github Actions工作流

    示例代码（仔细阅读注释）

```yml
name: Sync

on: page_build

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Sync to Gitee
        uses: wearerequired/git-mirror-action@master
        env:
          # 注意在 Settings->Secrets 配置 GITEE_RSA_PRIVATE_KEY
          SSH_PRIVATE_KEY: ${{ secrets.GITEE_RSA_PRIVATE_KEY }}
        with:
          # 注意替换为你的 GitHub 源仓库地址
          source-repo: git@github.com:doocs/advanced-java.git
          # 注意替换为你的 Gitee 目标仓库地址
          destination-repo: git@gitee.com:Doocs/advanced-java.git

      - name: Build Gitee Pages
        uses: yanglbme/gitee-pages-action@main
        with:
          # 注意替换为你的 Gitee 用户名
          gitee-username: yanglbme
          # 注意在 Settings->Secrets 配置 GITEE_PASSWORD
          gitee-password: ${{ secrets.GITEE_PASSWORD }}
          # 注意替换为你的 Gitee 仓库，仓库名严格区分大小写，请准确填写，否则会出错
          gitee-repo: doocs/advanced-java
          # 要部署的分支，默认是 master，若是其他分支，则需要指定（指定的分支必须存在）
          branch: main
```

注：如果运行Actions报错可参考官方[错误及解决方案](https://github.com/marketplace/actions/gitee-pages-action#%E9%94%99%E8%AF%AF%E5%8F%8A%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)；我就遇到了`Error: Need phone captcha validation, please follow gitee wechat subscription and bind your account.`的错误，参考方案关注了 Gitee 微信公众号，并绑定帐号，即可正常运行。

参考文献

+ [Hexo官方文档——一键部署](https://hexo.io/zh-cn/docs/one-command-deployment)
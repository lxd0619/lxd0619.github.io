---
title: hexo自动同步部署GitHub和Gitee pages
date: 2021-03-23 13:55:12
tags:
    - hexo
categories:
    - front_end
---

​		这大概是最简单的hexo同步部署github和gitee pages的方案了。使用Github Actions + MarketPlace中的两个Actions，[Hexo Action](https://github.com/marketplace/actions/hexo-action)和[Gitee Pages Action](https://github.com/marketplace/actions/gitee-pages-action)，这样就可以做到配置一次，以后每次推送代码即可自动部署到两个pages中。

<!-- more -->

### 配置Hexo Action

  + 先决条件

    生成SSH keys并设置Secrets。

``` $ ssh-keygen -t rsa -C "username@example.com" ```，生成SSH可参考[Connecting to GitHub with SSH](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh)。

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









参考文献
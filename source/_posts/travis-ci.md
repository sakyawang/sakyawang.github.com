---
title: gor-travis-ci
date: '2017-11-13'
description: 使用travis-ci自动编译github page
categories:
- ci

tags:

- travis
- gor

---

# 使用travis-ci自动更新github page

## 配置github pages仓库

master分支为github pages发布分支。

blog-source分支为blog源文件分支。

## blog使用gor引擎

[gor使用](http://sakyawang.me/golang/gor%E5%88%9B%E5%BB%BA%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2/)

linux下编译gor生成gor可执行二进制文件。

## 配置travis-ci

参考：

[手把手教你使用Travis CI自动部署你的Hexo博客到Github上](http://blog.csdn.net/woblog/article/details/51319364) 

## .travis.yml文件配置如下：

```yml
language: bash
                                                                            ```
env:

  global:

    - GH_REF: github.com/sakyawang/sakyawang.github.com.git

branches:

  only:

    - blog-source

script:

  - chmod +x gor

  - ./gor compile

after_script:

  - cp -rf CNAME ./compiled

  - cd compiled

  - git init

  - git config user.name "user@gmail.com"

  - git config user.email "user@gmail.com"

  - git add .

  - git commit -m "update blog"

  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}"  master:master
```

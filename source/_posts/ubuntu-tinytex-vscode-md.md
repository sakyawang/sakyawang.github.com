---
title: Ubuntu18.4下搭建latex环境
date: 2018-11-26 11:34:38
description: Ubuntu18.4安装Tinytex，使用vscode+latex workshop搭建latex环境
categories:
- latex

tags:
- latex
- vscode
---

## 环境

* Ubuntu18.4
* Vscode + latex workshop
* TinyTeX

> 原本打算安装`latex live`但是安装需要5G存储空间,后来看到轻量级的[TinyTeX](https://yihui.name/tinytex/)，所以放弃使用`Latex Live`转而使用`TinyTeX`。

## 安装

* VSCode 安装配置自行谷歌
* Latex Workshop 直接在VSCode插件安装配置
    ```shell
    # setting中新增
    "latex-workshop.latex.magic.args": [
        "-shell-escape",
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "%DOC%"
    ],
    ```
* TinyTeX安装
    ```shell
    wget -qO- "https://yihui.name/gh/tinytex/tools/install-unx.sh" | sh
    ```
* TinyTeX添加到path
    ```shell
    vim ~/.bashrc
    # add
    export PATH=$PATH:/home/usename/.TinyTeX/bin/x86_64-linux
    # reload
    source ~/.bashrc
    ```

## 使用

1. 新建hello.tex
2. 添加内容
    ```latex
    % hello.tex
    % !TEX program = xelatex
    % !BIB program = bibtex
    \documentclass[a4paper,11pt]{article}
    % 中文支持
    \usepackage{xeCJK}
    \setCJKmainfont{WenQuanYi Micro Hei}
    % define the title
    \author{Skayawang}
    \title{Minimalism}
    \begin{document}
    % generates the title
    \maketitle
    % insert the table of contents
    \tableofcontents
    \section{Start}
    Well, and here begins my lovely article.
    \section{End}
    \ldots{} and here it ends.
    \end{document}
    ```
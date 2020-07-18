---
title: 使用Github Action来自动化部署Web应用
categories:
  - Technology
tags:
  - DevOps
  - Web应用
  - Github Action
---

作为个人开发者，用来跑 Side Project 的机器一般是从云服务器厂商中租的机器，
或者是自己替换下来不用的机器。这些机器一般配置比较低，
特别是内存容量难以满足像 Jenkins 这样的大型 CI/CD 系统的需要。
比如我最近在使用 Jenkins 构建一个由 Create React App 生成的前端项目的 Docker 镜像时，
Jenkins 轻易地就吃掉了整台机器的仅有的 2GB 内存，导致整台机器陷入无法响应的状态。
难道我只能舍弃 Jenkins 了吗？如果不是内存不够还挺好用的......
于是，我有了一个想法，如果我们把内存需求比较高的像构建镜像这类工作切出来，
放到另外一台机器上去做。在生产环境上，Jenkins 只需要部署构建好的镜像就行了。
Github 的自动化工作流服务 [Github Action](https://github.com/features/actions) 满足了我这个需求。

<!--more-->

## Github Action 的优势

吸引我使用 Github Action 作为 CI/CD 的一部分的理由有两点，
第一是 Github Action 提供的运行实例的配置比较高——双核 CPU 与 7GB 内存
特别对于构建过程这种内存要求较高的任务非常合适。
第二是，它居然还免费。对于公开仓库，Github Action 是完全免费的，
对于私有仓库，每个免费账号每月有 2000 分钟的免费额度，
Pro 账号每月有 3000 分钟的免费额度。
按每 Push 一次触发一个 1 分钟的 Workflow，每个月能 push 2000 次。
emmm，用量应该是完全够的。

## 使用 Github Action 的自动化部署流程

我理想中的一个构建与发布过程是这样的：

1. （测试）当更改推送到项目的 repo 上时，CI/CD 服务会自动触发测试，如果测试用例出错则停止；
2. （构建）根据项目中的`Dockerfile`构建镜像，然后推送到一个 Docker Registry 上；
3. （通知部署）通知远端 Jenkins 服务器下载最新的镜像，然后部署到生产环境中。

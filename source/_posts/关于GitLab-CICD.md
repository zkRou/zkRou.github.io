---
title: 关于GitLab CI/CD
author: Kairou Zeng
date: 2018/9/17
tags: 
    - git
    - 持续集成
categories:
    - CICD
---

## 持续集成

> 持续集成(Continuous Integration,简称CI)，指频繁将代码集成到主干。

- 目的：

    更快速地发布更新，让产品快速迭代，同时还能保持高质量，提高开发效率。

- 核心措施：

    代码集成到主干前，必须通过自动化测试。只要有测试用例不通过，就不能集成。

- 组成：

    - 一个自动构建的过程，包括自动编译、分发、部署和测试等；
    - 一个代码存储库，即需要版本控制来保障代码的可维护性，同时作为构建过程的素材库；
    - 一个持续集成服务器

## 持续交付

> 持续交付(Continuous Delivery)，指频繁地将软件的新版本交付给质量团队或用户，以供评审。如果评审通过，代码就进入生产阶段。持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。

## 持续部署

> 持续部署(Continuous Deployment)，是持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境。持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。
> 与`持续交付`的区别在于：部署到生产环境的过程是否自动化(`持续交付`需要手动部署到生产环境)。

## Gitlab CI/CD

> GitLab CI是GitLab提供的持续集成服务，只要在仓库根目录创建一个.gitlab.yml文件(定义GitLab runner要做哪些操作)，并为该项目指派一个Runner，当有Merge Request或push的时候就会触发build.

- .gitlab-ci.yml文件

    `.gitlab-ci.yml`是用来配置CI在我们的项目中做些什么工作。位于项目的根目录。

    任何的push操作，GitLab都会寻找`.gitlab-ci.yml`文件，并对此次commit开始jobs，jobs的内容来源于`.gitlab-ci.yml`文件。

- GitLab  Runner 

    `GitLab Runner`是`.gitlab-ci.yml`脚本的运行器，在GitLab中，Runners将会运行你在`.gitlab-ci.yml`中定义的jobs，Runner可以是虚拟机、VPS、裸机、docker容器，或者是容器。`GitLab Runner`不需要和`GitLab`安装在同一台机器上，但是考虑到`GitLab Runner`的资源消耗问题和安全问题， 也不建议这两者安装在同一台机器上。GitLab和Runners通过API通信，所以唯一的要求就是运行Runners的机器可以联网。

    `GitLab Runner`分为两种：

        - Shared Runners:可以运行所有开启`Allow shared runners`选项的项目
        
        - Specific Runners:只能被指定的项目使用

- Pipelines

    `Pipelines`是定义于`.gitlab-ci.yml`中的不同阶段的不同任务。一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。每个`Pipelines`包含有多个`Stages`,每个`Stages`包含有一个或多个`jobs`。每一次Merge Request或push都要经过流水线之后才可以合格出厂。而`.gitlab-ci.yml`正是定义了`Pipelines`有哪些`Stages`，每个`Stages`要做什么事。

- Stages

    `Stages`表示构建阶段，可以在一次`Pipelines`中定义多个`Stages`.默认`Pipelines`有三个`Stages`：`build`、`test`、`deploy`

    - 所有`Stages`会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始；

    - 只有当所有`Stages`完成后，该构建任务`(Pipeline)`才会成功

    - 如果任何一个`Stages`失败，那么后面的`Stages`不会执行，该构建任务`(Pipeline)`失败

- Jobs

    `Jobs`表示构建工作，表示某个`Stage`里面执行的工作。

| 关键字 | 是否必须 | 描述 |
| --- | --- | --- |
| script | 是 | 定义`Runner`需要执行的脚本或命令 |
| extends | 否 | 定义此`job`将继承的配置条目 |
| image | 否 | 需要使用的docker镜像 |
| services | 否 | 定义了所需的docker服务 |
| stage | 否 | 定义了`Job`的阶段，默认是`test` |
| type | 否 | `stage`的别名，不赞成使用 |
| except | 否 | 定义了哪些git分支不适合该`job` |
| vaiables | 否 | 在`job`级别上定义的变量 |
| only | 否 | 定义那些git分支适合该`job` |
| tages | 否 | 定义了哪些runner适合该job(runner在创建时会要求用户输入标签名来代表该runner) |
| allow_failure | 否 | 允许任务失败，但是如果失败，将不会改变提交状态 |
| when | 否 | 定义`job`什么时候能被执行，可以是`on_success`,`on_failure`,`always`或者`manual` |
| dependencies | 否 | 定义了该`job`依赖哪一个`job`，如果设置该项，你可以通过artifacts设置 |
| cache | 否 | 定义需要被缓存的文件、文件夹列表 |
| before_script | 否 | 覆盖在根元素上定义的`before_script` |
| after_script | 否 | 覆盖在根元素上定义的`after_script` |
| environment | 否 | 定义让job完成部署的环境名称 |
| coverage | 否 | 定义代码覆盖率设置 |
| retry | 否 | 定义`job`失败后的自动充重试次数 |


- script

    `script`是一段由`Runner`执行的shell脚本，例如：
    
    ```YAML
    job:
        script: "bundle exec rspec"
    ```
    
    这个参数也可以使用数组包含几条命令：
    
    ```YAML
    job:
        script:
            - uname -a
            - bundle exec rspec
    ```

- 其他

    - 从8.0版本开始，GitLab 持续集成（CI）完全集成到GitLab中，且默认所有的项目开启。

    - [文档地址](https://docs.gitlab.com/ee/ci/quick_start/README.html)

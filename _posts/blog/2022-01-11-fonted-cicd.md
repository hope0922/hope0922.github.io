---
layout: post
title: 前端 CI/CD 规范
category: blog
description: 前端 CI/CD 在自己项目中的尝试
---

### 前端 CI/CD 规范

#### 没有 CI/CD 的开发的流程

- 1.本地开发代码
- 2.本地命令行运行单元测试
- 3.提交代码 push 到 git 远程仓库
- 4.构建前端和后端代码 打包发布到测试服务器

如果每个步骤都是认为的操作会浪费大量的重复操作的时间，也不能保证流程的准确，而且十分繁琐

#### 如果有了 CI/CD

- 1.本地开发代码
- 2.提交代码 push 到 git 远程仓库
- 3.gitlab webhook 自动触发 jenkins 的构建 job
- 4.jenkins 自动拉去代码运行单元测试并构建，如果一步失败就给予提示
- 5.前后端都构建成功后运行部署脚本发布到对应服务器

这是一个非常标准化的流程， 除了 1、2 两个步骤需要人为操作，其他都是自动运行，减少了人工操作的风险，省去了重复性工作，增强了项目的可见性

#### CI/CD 配置概要

##### 项目代码及提交规范化

统一 cicd 脚手架包含了初始化项目的能力，可以新建标准规范的项目，也可以在已有项目根目录上运行配置初始化。
根据项目规范化的模板文件初始化以下文件

- .commitlintrc.js(提交格式规范)
- .eslintignore .eslintrc(代码规范)
- .prettierignore .prettierrc(格式化代码)
- tsconfig(ts 配置)
- package.json
  保证所有项目代码风格和格式的统一，方便团队合作开发。

##### 自动化集成

- 1.在 jenkins 安装 Gitlab Hook Plugin 以支持 gitlab 的 webhook 功能
- 2.开发了一个统一的 cicd 脚手架 npm 包，先本地通过 cicd 包跑一遍整体的测试安装部署流程
- 3.本地流程构建成功通过后 根据本地配置的 cicdConfig.js 配置文件，生成对应的 JenkinsFile 文件并对应提示生成 webhook 地址
- 3.复制 webhook 地址，在 gitlab-》Setting-》Integrations 在 URL 选项粘贴刚才复制的地址，并配置 trigger 条件为 Push events
- 4.在 jenkins 上新建多分支流水线 job，配置 job 的 git 信息，选择 Pipeline script from SCM 模式，即通过项目内的 Jenkinsfile 文件配置流水线，方便进行管理

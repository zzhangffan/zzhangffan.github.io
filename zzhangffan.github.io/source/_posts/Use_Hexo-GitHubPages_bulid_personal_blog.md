---
title: 用 Hexo + GitHub Pages 搭建个人博客
date: 2017-12-31 11:39:56
categories: Hexo
tags: Hexo
---
> ## 准备工具

* Node.js（必须）用于支持Hexo，生成静态页面
* Git（必须）用于推送本地内容
* 注册一个GitHib账号（必须）存放文件，服务器

> ## 搭建环境

1. ## _Node.js_
# _Windows_
[官网](https://nodejs.org/en/download/)下载.msi文件到本地，双击安装即可
# _Linux(CentOS)_
```
	$ sudo yum install nodejs
```

2. ## _Git_
# _Windows_
[官网](https://git-scm.com/)下载安装软件包到本地，双击安装即可
# _Linux(CentOS)_ 
```
	$ yum install git
```
&emsp;&emsp;&emsp;__[Git教程——廖雪峰](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)__

3. ## _GitHub_
[去注册一个GitHub账号](https://github.com/join?source=header-home)
### 关联Git和GitHub
生成SSH密匙 输入下面命令后，一路回车即可，[GitHub 配置SSH Key](https://github.com/settings/keys)
```
	$ ssh-keygen -t rsa -C "你的邮箱地址"
```

4. ## _创建一个仓库做GitHub Pages（[What is GitHub Pages?](https://help.github.com/articles/what-is-github-pages/)）_
## 注意点：
&emsp;&emsp;_1.第一次需要验证邮箱_
&emsp;&emsp;_2.仓库名称固定格式为： 你的GitHub用户名.github.io_
### 创建成功后，现在可以用你的仓库名称访问你的GitHub Pages了
![GitHub Pages.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/call_GitHub-Pages.png)

> ## 安装Hexo

```
	$ npm install -g hexo
```
### 查看版本
```
	$ hexo -v
```
![see_Hexo_version.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/see_Hexo_version.png)
### 创建一个目录用于存放Hexo相关文件，如： ` $ mkdir Hexo ` 
### 使用命令 ` hexo init [目录名] ` 或进入目录执行命令 ` hexo init `
![command_hexo-init.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/command_hexo-init.png)
### Hexo会自动在这个目录下建立需要的文件夹和文件，[目录介绍](https://hexo.io/zh-cn/docs/setup.html)
![hexo-inited_dirStructure.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/hexo-inited_dirStructure.png)
### 进入目录使用命令 ` npm install ` 安装依赖
![command_npm-install.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/command_npm-install.png)

> ## 本地预览

### 生成静态页面和资源文件
```
	$ hexo generate	简写：hexo g
```
### 启动服务
```
	$ hexo server 简写：hexo s
```
### 命令可以组合使用，如 ` $ hexo s -g ` [更多命令查看Hexo文档](https://hexo.io/zh-cn/docs/index.html)
### 现在你可以通过 ` http://localhost:4000/ ` 进行访问了
![Hexo_index.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/Hexo_index.png)

> ## 部署至GitHub

修改Hexo根目录下的 ` _config.yml ` 主站配置文件中的 ` deploy ` 参数：
```
	deploy:
  	  type: git
  	  repo: <仓库地址>
  	  branch: [分支名称]
  	  message: [提交信息]
```
### 部署
```
	$ hexo deploy 简写： hexo d
```
![command_hexo-d.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/command_hexo-d.png)
> ### 部署提示找不到：git
>> 3.0 后 type需改为 git ，deploy部分被单独出来了，需要单独安装

## 安装 hexo-deployer-git
```
	$ npm install hexo-deployer-git --save
```
## 再 ` hexo d ` ，部署成功后快用你的GitHub Pages仓库名称访问你的博客吧~
![GitHub-Pages_index.png](/images/Use_Hexo-GitHubPages_bulid_personal_blog/GitHub-Pages_index.png)

（完）
=====
---
title: 利用Hexo搭建GitHub
typora-root-url: ../
date: 2020-12-24 09:51:53
tags:
 - Hexo
 - GitHub
 - Blog
categories: GitHub
---









本文基于Windows环境配置。

<!--more-->

## 搭建前提

- 安装好Node.js、npm
- 保证主机上有Git并同自己的GitHub账号通过SSH建立联系
- 简单了解GitHub和Git

## 关于GitHub的部署

- 在GitHub上新建一个仓库，其命名保证为以下要求

  ![image-20201224100650328](/images/image-20201224100650328.png)

  `username`要求为自己GitHub上面的账户名

  

- 进入自己新建的仓库，在setting中向下拖，找到GitHub Pages，选择主分支保存

  ![image-20201224101348108](/images/image-20201224101348108.png)

  成功时，一般上方会显示`Your site is published at https://username.github.io/`

   

  到此GitHub的配置完成，已经可以通过https://username.github.io访问你的GitHub Pages界面，需要利用一些工具优化你的博客部署，这里我们采用Hexo进行优化。



## 关于Hexo的部署

- 利用`npm install -g hexo-cli`指令**安装Hexo**，可在Git、cmd上进行，推荐使用Git

- 安装后在电脑选择一个文件夹作为今后放置自己文章的地方，打开Git命令行`hexo init `命令用于**初始化本地文件夹为网站的根目录**

  这里简单了解下这些目录作用：

  Source文件夹用来存放文章、图片草稿

  scaffolds文件夹可以针对文章进行一些全局部署

  themes文件夹存放主题

  _config.yml文件对博客进行全局配置

- **博客主题**

  此步可以略过，博客即是默认主题

  - 首先可以在[这个地址](https://hexo.io/themes/)选择自己喜欢的主题，并进入GitHub发现该主题项目

  - 前面说到themes文件夹存放主题，所以在该文件夹中打开Git将该项目复制下来

    ![image-20201224105030661](/images/image-20201224105030661.png)

  - 最后在前面说到的_config.yml文件对博客主题进行修改，找到theme选项，修改主题名为下载主题的文件夹名，注意空格

    ![image-20201224105351858](/images/image-20201224105351858.png)

- **本地测试**

  - 在根目录下，利用`hexo g`生成静态文件
  - 再利用`hexo s`可以启动本地服务器，即可在http://localhost:4000 上查看自己的博客情况

  这里的两部操作对应就是保存修改生成静态文件、在本地查看修改看是否满意，一般用来测试用。

- **上传博客**

  - 上传博客需要利用`hexo-deployer-git`,所以第一次操作时需要利用`npm install hexo-deployer-git --save`指令进行安装

    生成.deploy_git文件夹

  - 安装后修改前面说到的_config.yml文件，建立与GitHub Pages的联系，这一步也是第一次操作需要修改
  
    ```
    deploy:
      type: git
      repository: git@github.com:zhu-ym/zhu-ym.github.io.git
    branch: main
    ```

    这里注意空格，以及 repository后面地址就是对应上面GItHub部署的SSH key（克隆时用的SSH地址），分支那里之前见人写master,但是考虑到GitHub2020年的修改，我改为了main，没有影响，可以成功。

  - 最后即是将文件部署到GitHub上，执行`hexo g -d`即可
  
    一般Git末尾出现如下图类似情况即成功
    
    ![image-20201224111652154](/images/image-20201224111652154.png)
    
    这步实质是生成静态文件，并将.deploy_git文件夹的内容上传到main（master）分支上去
  
  最后，以后写文章，主要就是执行`hexo g -d`上传


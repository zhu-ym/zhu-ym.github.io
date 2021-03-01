---
title: Maven
typora-root-url: ../
date: 2021-02-27 13:16:04
tags:
categories:
---

介绍Maven基础知识点，并用一实例体现其在IDEA上的用途

<!--more-->

# Maven基础

　	理想的项目构建是高度自动化，跨平台，可重用的组件，标准化的，使用Maven就可以帮我们完成上述所说的项目构建过程。

​	 Java 框架基本上都依赖于 Maven，其是跨平台项目管理工具，主要服务于基于Java平台的项目构建、依赖管理和项目信息管理：

- 项目构建：项目构建经过清理项目、编译项目、测试项目、生成测试报告、打包项目、部署项目几个阶段，可以使用Maven完成
- 依赖管理：指jar包之间的相互依赖，Maven可以自动下载项目所需要的jar包，统一管理jar包之间的依赖关系。

就最直观的依赖管理来说，Maven 可以自动化地解决依赖问题，也就是说只要你写清楚你需要什么包，那么 Maven 可以自动把这个包准备好，同时也会把所有的依赖包准备好，而不需要自己手动下载再一个一个添加。



# POM

POM即项目对象模型，是一个包含项目信息的XML文件，每新建一个项目会有一个POM文件。

执行任务或目标时，Maven 会在当前目录中查找 POM。它读取 POM，获取所需的配置信息，然后执行目标。

在POM中可以指定一些配置，比如项目依赖、插件、项目构建profile、以及一些项目信息。

Maven与项目依赖：

![image-20210228153514059](/images/image-20210228153514059.png)

Maven的项目信息：

![image-20210228153647972](/images/image-20210228153647972.png)

所有 POM 文件都需要 project 元素和三个必需字段：groupId，artifactId，version。

groupId：项目组Id，全球唯一；groupId：项目组Id，全球唯一；version为项目版本

**父POM**

父（Super）POM是 Maven 默认的 POM。所有的 POM 都继承自一个父 POM（无论是否显式定义了这个父 POM）父POM 包含了一些可以被继承的默认设置。因此，当 Maven 发现需要下载 POM 中的依赖时，它会到 Super POM 中配置的默认仓库下载，如下面的仓库内容

# 仓库

仓库是一个位置，用来管理构件（依赖、插件或者项目构建的输出），是放置所有JAR文件的地方。

仓库可以分本地仓库，中央仓库、远程仓库，也是依赖的搜索顺序

- 本地仓库：用来存储项目的依赖库

  默认地址在`${user.home}/.m2/repository`,可以在conf的setings.xml文件中定义另一个路径

  ![img](/images/20151215145450769)

- 中央仓库：用来下载依赖库的默认位置

- 远程仓库：中央仓库依赖库的补充，或者弥补中央仓库速度慢的问题

- 依赖的搜索顺序：本地仓库，中央仓库、远程仓库

# 生命周期

Maven 构建生命周期定义了一个项目构建跟发布的过程。生命周期把每个项目构建跟发布按阶段顺序的执行

而Maven有三套独立的生命周期：clean、default（build）、site，每个生命周期又会分成几个阶段，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行！

- **clean：**在真正的构造之前进行项目清理工作，其阶段划分如下：

  - pre-clean：执行一些需要在clean之前完成的工作
  - clean：移除上一次构建生成的文件
  - post-clean：执行一些需要在clean之后立刻完成的工作

  这时来理解”在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行“这句话，如果输入`mvn clean`，代表着已经运行了pre-clean，clean两个生命周期阶段。

- **default：**项目构建的核心，主要生命周期，阶段包括编译，测试，打包，部署等等。default生命周期有23个阶段

  ```
  validate
  initialize
  generate-sources
  process-sources
  generate-resources
  process-resources     复制并处理资源文件，至目标目录，准备打包。
  compile     编译项目的源代码。
  process-classes
  generate-test-sources 
  process-test-sources
  generate-test-resources
  process-test-resources     复制并处理资源文件，至目标测试目录。
  test-compile     编译测试源代码。
  process-test-classes
  test     使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
  prepare-package
  package     接受编译好的代码，打包成可发布的格式，如 JAR 。
  pre-integration-test
  integration-test
  post-integration-test
  verify
  install     将包安装至本地仓库，以让其它项目依赖。
  deploy     将最终的包复制到远程的仓库，以让其它开发人员与项目共享。
  ```

  在default生命周期，主要又是以下阶段序列

  ![image-20210228162151444](/images/image-20210228162151444.png)

- **site：**生成项目报告，站点，发布站点。

  - pre-site：执行一些需要在生成站点文档之前完成的工作

  - site：生成项目的站点文档

  - post-site：执行一些需要在生成站点文档之后完成的工作，并且为部署做准备

  - site-deploy：将生成的站点文档部署到特定的服务器上

    

# 内置命令

Maven有一些内置命令，可以结合前面的生命周期一起使用，注意使用时需要转到对于的项目目录下

- 创建Maven的普通java项目：
  `mvn archetype:create -DgroupId=packageName -DartifactId=projectName`
- 创建Maven的Web项目：
  `mvn archetype:create -DgroupId=packageName -DartifactId=webappName-DarchetypeArtifactId=maven-archetype-webapp`
- 编译源代码：` mvn compile`
- 编译测试代码：`mvn test-compile`
- 运行测试：`mvn test`
- 产生site：`mvn site`
- 打包：`mvn package`
- 在本地Repository中安装jar：`mvn install`
- 清除产生的项目：`mvn clean`
- 生成eclipse项目：`mvn eclipse:eclipse`
- 生成idea项目：`mvn idea:idea`
- 组合使用goal命令，如只打包不测试：`mvn -Dtest package`
- 编译测试的内容：`mvn test-compile`
- 只打jar包: `mvn jar:jar`
- 只测试而不编译，也不测试编译：`mvn test -skipping compile -skipping test-compile`( -skipping 的灵活运用，当然也可以用于其他组合命令)
- 清除eclipse的一些系统设置:`mvn eclipse:clean`



# Nexus私服

私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的用户使用。当Maven需要下载构件的时候，它从私服请求，如果 私服上不存在该构件，则从外部远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。

# IDEA与Maven

未完待续
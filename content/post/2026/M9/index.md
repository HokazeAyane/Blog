--- 
title: "Maven 配置与 IDEA、命令行的区别"
description: "Maven" 
date: 2026-09-29T20:52:00+08:00 
draft: false         
categories:     
    - "Java" 
tags:     
    - "Java"
--- 
## Maven 配置与 IDEA、命令行的区别
### 1. Maven 有三类配置
- 用户配置：C:\Users\<username>\.m2\settings.xml
- IDEA 配置：IDEA 设置中指定的 settings.xml，例如 D:\config\settings.xml
- 项目配置：项目自己的 pom.xml
在 IDEA 的 Terminal 中输入 mvn install，仍然是命令行 Maven，不会读取 IDEA Maven 设置页面。
### 2. 两种运行方式

```
命令行：使用默认用户配置
mvn install
命令行：明确使用 F 盘配置和指定本地仓库
mvn -s "D:\config\settings.xml" "-Dmaven.repo.local=D:\maven-repository" install
```
IDEA Maven 侧边栏使用的是 IDEA Maven 设置中配置的文件和本地仓库。
### 3. 仓库配置的关键区别
项目 pom.xml 中可能定义多个远程仓库，例如：
<br>
public-repo  → public-repository
<br>
internal-repo → internal-repository
<br>
如果 settings.xml 中配置了：
```
<mirrorOf>*</mirrorOf>
```
那么所有 POM 中的仓库都会被强制转到这个镜像仓库。
本次问题就是：
```
默认配置 → public-repository  → 有 1.0.0-SNAPSHOT
IDEA 配置 → internal-repository → 只有 0.9.0-SNAPSHOT
Maven 不会用 0.9.0 替代项目要求的 1.0.0。
```

### 4. 常见误区
- 本地仓库目录存在，不代表依赖下载成功。
- .lastUpdated 是下载失败标记，不是真正的 jar。
- mvn clean 只删除项目的 target，不会删除 Maven 本地仓库。
- mvn install 会下载依赖，并把当前项目安装到本地仓库。
- 修改或补充本地 jar 后，需要在 IDEA 中重新加载 Maven 项目。

### 5. 排查口诀
先确认四件事：
1. 当前到底是命令行 Maven，还是 IDEA Maven 侧边栏？
2. 使用了哪个 settings.xml？
3. 本地仓库路径是哪一个？
4. 当前远程仓库是否真的有项目所需的版本？
不要只看目录名，重点看目录中是否存在真正的 .jar 和 .pom 文件。
### 6. 个人复盘
最主要的问题不是 Maven 本身，而是我之前形成了路径依赖：
- 误以为 pom.xml 只负责声明依赖，settings.xml 只是补充配置，没有意识到镜像配置可能覆盖 POM 中的远程仓库。
- 因为之前几个项目都使用同一份 settings.xml，就想当然地认为当前项目也应该使用它，没有先确认项目 POM 和仓库版本是否匹配。
- 没有区分命令行 Maven 和 IDEA Maven 侧边栏。之前遇到类似问题时，只是在命令行反复执行 mvn clean，实际上没有解决配置和仓库不一致的问题。
这次真正需要先确认的是：运行入口、使用的 settings.xml、本地仓库路径、镜像仓库以及依赖版本。配置不能靠“以前这样能用”来判断，要以当前项目的实际配置和 Maven 输出为准。
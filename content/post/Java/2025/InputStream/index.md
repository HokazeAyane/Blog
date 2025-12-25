---
title: "Java文件上传中的Stream Closed问题"
description: "Java不安desu"
date: 2025-12-25T22:47:00+08:00
image: "yrlv.jpg"  
draft: false         
categories:
    - "Java"
tags:
    - "开发"
    - "Java"
---
## ❌问题描述
在现有项目添加图片上传功能，测试时发现：
- ✅ 小图片（100KB）上传成功
- ❌ 大图片（4MB）上传失败，报错 `java.io.IOException: Stream closed`

### 错误信息（修复前）
![](/error1.png)

### 错误信息（添加检查后）
![](/error2.jpg)

## 问题分析

### 核心原因

**XSS Filter 和 Tomcat 的 InputStream 竞争问题**

现有项目为了防止注入攻击,重写doFliter方法，把请求参数封装进InputStream,且Tomcat默认最大文件限制为2MB,如果超出2MB, Tomcat 为了验证大小，提前读取 InputStream，读取完成后关闭 InputStream， XSS Fliter自然按无法读取InputStream。

**第一次次错误信息**

第一次错误是因为当图片超过Tomcat默认最大2MB时，Tomcat提前读取了InputStream,导致流关闭，XSS FLiter尝试读取关闭的InputStream，日志打印`Stream CLosed`错误。

**第二次错误信息**

当我在重写的doFilter方法中添加了Multipart分支判断，不会提前读取导致流关闭，真正的错误才显现出来


### 问题根源
**责任划分不清**：
- XSS Filter 尝试处理所有类型的请求（包括 multipart）
- 文件上传的 InputStream 应该由 Spring/MultipartResolver 独占管理
- XSS Filter 和 Tomcat 竞争读取同一个 InputStream

## 小结
明明之前能跑通的，然后被抓去别的项目写Vue2，前端苦手喵。 跑回来后让豆包生成了几张图片测试，发现测试失败，天塌了喵。



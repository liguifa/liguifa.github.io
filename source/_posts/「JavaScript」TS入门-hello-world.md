---
title: 「Javascript」TS入门-hello-world
date: 2023-07-27 20:14:00
tags: [前端, JavaScript, TypeScript]
---
### [](#一、简介 "一、简介")一、简介

TypeScript是微软开发的开源编程语言，它是Javascript的超集。  

### [](#二、开发环境 "二、开发环境")二、开发环境

1.安装node.js，TypeScript需要编译成Javascript才能运行，Node.js提供了编译环境。
    
2.安装TypeScript编译工具，打开cmd，输入npm命令：
    
    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">npm install -g typescript</span><br></pre></td></tr></tbody></table>
    

安装成功如图：  
![Typescript安装](2019010101.png)

如果安装的过程中出现以下错误  
![Typescript安装](2019010102.png)

请执行以下命令，将npm镜像地址切换到国内

npm –registry [http://registry.cnpmjs.org](http://registry.cnpmjs.org)

### [](#三、Hello-World "三、Hello World")三、Hello World

新建文件Hello.ts，用记事本打开，并输入以下代码：  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">function hello(){</span><br><span class="line">	return 'Hello World'</span><br><span class="line">}</span><br><span class="line">console.log(hello());</span><br></pre></td></tr></tbody></table>

这段代码将在console中输出Hello World

ts文件为TypeScipt的后缀名，我们需要将TypeScript编译成Js，在命令行中输入以下命令执行编译:  

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">tsc hello.ts</span><br></pre></td></tr></tbody></table>

命令执行后会在同目录生成Hello.js，为TypeScript编译文件

我们用Node.js运行该js文件，如图：  
![Typescript运行](2019010103.png)

一个简单的Hello World程序就出来了
---
title: 「JavaScript」JavaScript作用域
date: 2023-07-27 14:02:14
tags: [JavaScript, 前端]
---
JaveScript 是Web前端开发的最重要的语言，在JavaScript中作用域是一个重要的基础知识点。  
  
在JavaScript中共有三种作用域，分别是全局作用域、函数左右域和块级作用域。  
  
本文会先对三种作用域做介绍，之后举一个常见的例子做整体介绍。  

#### [](#1、全局作用域 "1、全局作用域")1、全局作用域

全局作用域，故名思议，全局作用域就是整个JavaScript代码范围。如下代码所示:

```
var a = "Hello";  
console.log(a);  //Hello  
for(var i in a){  
    console.log(a[i]); //H、e、l、l、o  
}  
console.log(i);  // 4  
function func(){  
    console.log(a); //Hello  
    console.log(i) // 4  
}  
func();
```
  
在这段代码中，我们先声明了变量a，并对a赋值Hello，a是我们在全局上声明的变量，此时a的作用域为全局作用域，我们之后在直接输出、循环中输出、函数中输出都没有错误，说明此时a在任何位置均可以访问。同时在for循环中我们声明了变量i，for循环结束之后变量i并没有释放，而是输出了最后的值4，说明此时变量i的作用域为全局作用域。

### [](#2、函数作用域 "2、函数作用域")2、函数作用域

函数作用域，故名思议，函数作用域就是生个函数范围。如下代码所示:

```
function fun() {  
   var a = "Hello";  
   var i = 0;  
   switch (i) {  
           case 0: console.log(a); //hello  
    }  
    console.log(a); //hello  
}  
func();  
console.log(a); //Error  
var t = "hello2";  
function fun2() {  
    var t = "helllo3";  
    console.log(t); //hello3  
}  

```
  
在这段代码中，我们先声明了函数fun，在函数内部我们声明了变量a，此时a的作用域为函数作用域，之后我们在switch中输出，尽管我们的switch被大括号包裹，但是我们依然可以访问，说明函数作用域可以在函数的任意位置范围，之后在函数之外，我们输出变量a，此时就会出错，因为变量a的作用域为函数作用域，在函数为无法访问。在fun2中，我们声明了一个与全局变量同名的变量t，此时函数作用域中的t为覆盖全局作用域的t，这里的规则是子作用域覆盖父作用域，而全局作用域做为所有作用域的顶层作用域，一定会被子作用域的同名变量覆盖。

### [](#3、块级作用域 "3、块级作用域")3、块级作用域

在ES6中，JavaScript引入了块级作用域，块级作用域的作用域范围为代码块。如下代码所示：

```
"use strict"  
{      
    var a = 0;  
    console.log(a);  
}  
console.log(a) //Error
```
  
在上面代码中，我们使用use strict启用ES6，此时块级作用域起效，在之后的代码块中，变量a为块级作用域，在代码中访问正常，而在代码我们无法访问，与函数作用域类似，我们依然可以访问父作用域的变量，当发生变量重名是会覆盖父作用域的变量。

### [](#4、例子 "4、例子")4、例子

与作用域相关的问题中，大多出在for中，如下代码所示：

```
var funcs = [];  
for(var i = 0;i<4;i++) {  
    funcs.push(function() {  
        console.log(i);  
    });  
}  
funcs.forEach(function() {  
    fun();  
});  

```
  
我们执行这段代码，此时我们期望输出0、1、2、3，但是在实际运行时，我们输出了4、4、4、4，这是因为var声明的变量只能是全局作用域或函数作用域，如果我们这段代码运行在全局作用域的，那么变量i的作用域就是全局作用域，在整个作用域中变量i只有一个值，也就是在funs中我们引用的变量i和全局作用域的i是同一个并且只有一个，当循环结束后，变量的值变成了4，我们在执行funs，那么输出的就是变量i的值4。那么很明显这个问题是作用域没有独立引起的，解决办法就是新建一样作用域。

#### [](#4-1-方法1 "4.1 方法1")4.1 方法1

```
var funcs = [];for (var i = 0; i < 4; i++) {  
    funcs.push((function (in_i) {  
        return function () {  
            console.log(in_i);  
        }  
    })(i));  
}  
funcs.forEach(function () {  
    fun();  
});
```
  
如上面的代码所示，我们使用闭包，闭包就是一个立即执行的函数，此时我们使用闭包新建了一个函数作用域，并把变量i参入，在闭包内部会拷贝一份变量i到in_i，此时for内部不再是单一作用域，做个作用域有多个in_i。

#### [](#4-2-方法2 "4.2 方法2")4.2 方法2

```
var funcs = [];for(let i = 0;i<4;i++) {  
    funcs.push(function() {  
        console.log(i);  
    });  
}  
funcs.forEach(function() {  
    fun();  
});
```
  
如上代码所示，我们将var替换成了let，let是ES6声明变量的方法，此时let声明后i变成了块级作用域变量，也就是新建了一个块级作用域，在for内部i不再是同一个，而是在一个作用域中一个变量i。
---
title: 一道JS闭包面试题
date: 2019-02-22 22:35
tags:
  - javascript
  - 闭包
---

#### 说明
最近看到这样一段代码

```javascript
	 function fun(n,o){
	    console.log(o);
	    return {
	        fun:function(m){
	            return fun(m,n);
	        }
	    };
	 }

	 var a = fun(0);a.fun(1);a.fun(2);a.fun(3);
	 var b = fun(0).fun(1).fun(2).fun(3);
	 var c = fun(0).fun(1);c.fun(2);c.fun(3);
	 //问:三行a,b,c的输出分别是什么？
```

觉得有点意思，和大家一起来聊聊。 
<!--more--> 
我相信如果你不是非常理解JavaScript中的闭包，一定是不想看这段代码的。  

#### 解释
好的，我们暂时先不去想这段代码，先看点简单的  

```javascript
	function fun0(){
	    var a=1;
	    console.log(a);
	}
	function fun1(){
	    console.log(a);
	}
	fun0();    //1
	fun1();    //报错 a is not defined
```

这段代码，我相信大家应该知道最后为什么结果会是 1 和 报错 的，在函数内声明的变量只在函数体内定义，它们是局部变量，作用域是局部的，所以 函数 fun1 调用后，找不到a，就报错了，JavaScript采用词法作用域，函数的执行依赖于变量作用域，这个作用域是在函数定义时决定的，所以我们只要改改上面函数 fun1的位置，它就不会报错了。

```javascript
	function fun0(){
	    var a=1;
	    console.log(a);

	    //把fun1放在fun0中，就不报错了
	    function fun1(){
	        console.log(a);
	    }
	    fun1();   //1
	}
	fun0();   //1
```

代码改成这样，只是把fun1 放在 fun0 中就不报错了，函数调用后都输出1  
好了，我们来看最开始提到的代码，先简化一下

```javascript
	function fun(n,o){
	    return {
	    }
	}
```

我们先看这段代码，fun 调用后会怎么样？  
很明显会返回一个空对象，记住，fun调用后会返回对象，这点很重要。

```javascript
	function fun(n,o){
	    console.log(o);
	    return {
	        fun:function(m){
	            return fun(m,n);
	        }
	    };
	 }

	 var a = fun(0);
```

这里提一句，当调用函数的时候传入的实参比函数声明时指定的形参个数要少，剩下的形参都将设置为undefined值。  
`console.log(o);` 输出undefined  
`var a = fun(0);` 那a是值是什么，是fun(0)，返回的那个对象  

```javascript
	{
	    fun:function(m){
	        return fun(m,0);
	    }
	}
```

这个对象，有一个fun的方法，方法返回的结果就是最外面 fun 调用的结果。  
![结果](/img/asda.png)

`var a=fun(0)`，传入一个参数0，那就是说，函数fun中参数 n 的值是0了，而返回的那个对象中，需要一个参数n，而这个对象的作用域中没有n，它就继续沿着作用域向上一级的作用域中寻找n，最后在函数fun中找到了n，n的值是0，这段话是本文的重点， 明白这段，那问题就容易解决了。  
<br/>
说到这里，这道题基本上可以解决了，希望大家能听明白我上面说的话，下面的就简单了。我们一步一步看。  
<br/>
现在我们知道 a 是  

```javascript
{
    fun:function(m){
        return fun(m,0);
    }
}
```

这样的一个对象 
`a.fun(1);` 会怎么样？看代码

```javascript
{
    fun:function(1){
        return fun(1,0);
    }
}
```

`a.fun(1);` 返回的结果，就是 fun(1,0)，返回的结果

```javascript
 function fun(n,o){ //n的值为1，o的值为0
        console.log(o);
        return {
            fun:function(m){
                return fun(m,n);//n的值为1
            }
        };
}
fun(1,0);  //输出0，并返回一个对象，这个对象有一个fun的方法,这个方法调用后，会返回外层fun函数调用的结果，并且外层函数的第二个参数是 n 的值，也就是1  
```

`a.fun(2);` 会怎么样？看代码

```javascript
{
    fun:function(2){
        return fun(2,0);
    }
}
```

`a.fun(2);` 返回的结果，就是 fun(2,0)，返回的结果  

```javascript
function fun(n,o){ //n的值为2，o的值为0
        console.log(o);
        return {
            fun:function(m){
                return fun(m,n); //n的值为2
            }
        };
}
fun(2,0);  //输出0，并返回一个对象，这个对象有一个fun的方法,这个方法调用后，会返回外层fun函数调用的结果，并且外层函数的第二个参数是 n 的值，也就是2  
```

`a.fun(3);` 就不说了，一样的。

```javascript
var a = fun(0); a.fun(1); a.fun(2); a.fun(3);
var b = fun(0).fun(1).fun(2).fun(3);
```

把返回的对象，重新赋值给a，这样两行的结果就是一样的了。 
`var c = fun(0).fun(1); c.fun(2); c.fun(3);`  
c 与他们的不同，只是`var c = fun(0).fun(1);` 之后用的是同一个对象罢了。

#### 总结

说下结果

```javascript
var a = fun(0); a.fun(1); a.fun(2); a.fun(3);
//undefined 0 0 0 
  
var b = fun(0).fun(1).fun(2).fun(3);
//undefined 0 1 2
  
var c = fun(0).fun(1); c.fun(2); c.fun(3);
//undefined 0 1 1
```

[转载来自http://www.cnblogs.com/xxcanghai/p/4991870.html](http://www.cnblogs.com/xxcanghai/p/4991870.html)
顺便推荐几篇讲解闭包的文章  
<br/>
[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
[Javascript闭包——懂不懂由你，反正我是懂了 ](http://kb.cnblogs.com/page/110782/)
[JS闭包可被利用的常见场景](http://blog.csdn.net/yanghua_kobe/article/details/6780181)
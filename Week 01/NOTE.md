# 学习笔记

> 泪雨澜风

 ---

## TicTacToe

### 小知识点
- 块级作用域：在《重学前端》中提及的块级作用域，以下语句会产生 let 使用的作用域：
> 1. for；
> 2. if；
> 3. switch；
> 4. try/catch/finally；
> 5. {}

- 三元操作符
- `label`语句：在看到老师使用之前我完全不知道这个语法（立马加入到脑图中），后在《JavaScript高级程序设计》(第三版)的第三章**基本概念3.6.6 label 语句**(P58)中找到，原文摘抄如下：

  > 使用`label`语句可以在代码中添加标签，以便将来使用。以下是`label`语句的语法:
  > `label:statement`
  > 下面是一个示例:
  > ```javascript
  > start:for(var i=0;i<count;i++){
  >    alert(i);
  > }
  > ```
  > 这个例子定义的start标签可以在将来由`break`或`continue`语句引用。加标签的语句一般都要与`for`语句等循环语句配合使用。

- [Object.create()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)用法：`Object.create()`方法创建一个新对象，使用现有的对象来提供新创建的对象的`__proto__`。该方法创建的新对象继承原对象的所有数据以及方法，**完美复制！**

### 极小极大算法（Minimax）
   &nbsp;&nbsp;老师在编写TicTacToe的AI使用的算法是MiniMax算法，Minimax算法又名极小极大算法，是一种找出失败的最大可能性中的最小值的算法。Minimax算法常用于棋类等由两方较量的游戏和程序，这类程序由两个游戏者轮流，每次执行一个步骤。我们众所周知的五子棋、象棋等都属于这类程序，所以说Minimax算法是基于搜索的博弈算法的基础。该算法是一种零总和算法，即一方要在可选的选项中选择将其优势最大化的选择，而另一方则选择令对手优势最小化的方法。说白了就是，我们在决策当中可以做到，**对方所有决策的结果是我方最差情况中最好的结果（假设你的每一步决策是局部最好的），从而使自己的损失最小。**（感觉是一种防守战术）

## async异步编程

所谓异步编程，就是不确定顺序执行，可以通过触发条件执行，不确定时间执行，可以同时执行多个任务，这里不展开细说。

> **回调地狱:**
> 我们使用回调函数实现异步操作时，很容易出现回调地狱问题，比如实现一个红绿灯，1s红灯，200ms黄灯,500ms绿灯，示例如下
> ```html
> 
> <!DOCTYPE html>
> <html lang="en">
> <head>
>    <meta charset="UTF-8">
>    <meta name="viewport" content="width=device-width, initial-scale=1.0">
>    <title>redgreen</title>
>     <style>
>         div{
>             background-color: grey;
>             display: inline-block;
>             margin:30px;
>             width:100px;
>             height:100px;
>             border-radius:50px;
>         }
> 
>         .green.light{
>             background-color: green;
>         }
> 
>         .yellow.light{
>             background-color: yellow;
>         }
> 
>         .red.light{
>             background-color: red;
>         }
>     </style>
> </head>
> <body>
>     <div class="green"></div>
>     <div class="yellow"></div>
>     <div class="red"></div>
> </body>
> <script>
>     function green(){
>         var lights=document.getElementsByTagName('div');
>         for(var i=0;i<3;i++)
>             lights[i].classList.remove("light")
>         document.getElementsByClassName('green')[0].classList.add('light')
>     }
>     function red(){
>         var lights=document.getElementsByTagName('div');
>         for(var i=0;i<3;i++)
>             lights[i].classList.remove("light")
>          document.getElementsByClassName('red')[0].classList.add('light')
>     }
>     function yellow(){
>         var lights=document.getElementsByTagName('div');
>         for(var i=0;i<3;i++)
>             lights[i].classList.remove("light")
>         document.getElementsByClassName('yellow')[0].classList.add('light')
>     }
> 
>     function go() {
>         green();
>         setTimeout(() => {
>             yellow();
>             setTimeout(() => {
>                 red();
>                 setTimeout(() => {
>                     go();
>                 }, 1000);
>             }, 200);
>         }, 500);
>     }
> </script>
> </html>
> ```
> 该示例当中的
> ```javascript
> function go() {
>     green();
>     sleep(1000).then(()=>{
>         yellow()
>         return sleep(200)
>     }).then(()=>{
>         red()
>         return sleep(500)
>     }).then(go);
> }
> ```
> 就是一种回调地狱，回调地狱容易造成逻辑混乱，代码很难维护。

那如何去进行异步编程，并且避免回调地狱问题呢？就目前的标准以及草案来看，主要有下面的几种方式：

- Promise（ES2015）
- Generator（ES2015）
- await/async（ES2016）

直接上代码！

### Promise
```javascript
...
function sleep(t) {
        return new Promise((resolve,reject)=>{
            setTimeout(resolve, t);
        })
    }

function go() {
    green();
    sleep(1000).then(()=>{
        yellow()
        return sleep(200)
    }).then(()=>{
        red()
        return sleep(500)
    }).then(go);
}
...
```

### Generator
```javascript
...
function sleep(t) {
        return new Promise((resolve,reject)=>{
            setTimeout(resolve, t);
        })
    }

function* go() {
    while(true){
        green()
        yield sleep(1000)
        yellow()
        yield sleep(200)
        red()
        yield sleep(500)
    }
}

function run(iterator) {
    let {value,done}=iterator.next();
    if(done)return;
    if(value instanceof Promise){
        value.then(()=>{
            run(iterator)
        })
    }
}

function co(generator) {
    return function() {
        return run(generator())
    }
}
...
```

### await/async
```javascript
...
function sleep(t) {
        return new Promise((resolve,reject)=>{
            setTimeout(resolve, t);
        })
    }

async function go() {
    while(true){
        green()
        await sleep(1000)
        yellow()
        await sleep(200)
        red()
        await sleep(500)
    }
}
...
```
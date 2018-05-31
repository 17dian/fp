
## 1.1 函数式编程（FP）思想

面对对象（OOP）可以理解为是对数据的抽象，比如把一个人抽象成一个Object，关注的是数据。
函数式编程是一种过程抽象的思维，就是对当前的动作去进行抽象，关注的是动作。

    举个例子：如果一个数a=1 ，我们希望执行+3（f函数），然后在*5（g函数），最后得到结果result是20

    数据抽象，我们关注的是这个数据：a=1 经过f处理的到  a=4 , 在经过g处理的到 a = 20

    过程抽象，我们关注的是过程：a要执行两个f,g两操作，先将fg合并成一个K操作，然后a直接执行K，的到 a=20

**问题：f和g合并成了K，那么可以合并的函数需要符合什么条件呢？下面就讲到了纯函数的这个概念。**


## 1.2 纯函数

定义：一个函数如果输入参数确定，输出结果是**唯一确定**的，那么他就是纯函数。    
特点：无状态，无副作用，无关时序，幂等（无论调用多少次，结果相同）

**下面哪些是纯函数 ?**

``` javascript              
let arr = [1,2,3];                                            
arr.slice(0,3);                                               //是纯函数
arr.splice(0,3);                                              //不是纯函数，对外有影响

function add(x,y){                                           // 是纯函数   
   return x + y                                              // 无状态，无副作用，无关时序，幂等
}                                                            // 输入参数确定，输出结果是唯一确定

let count = 0;                                               //不是纯函数 
function addCount(){                                         //输出不确定
    count++                                                  // 有副作用
}

function random(min,max){                                    // 不是纯函数     
    return Math.floor(Math.radom() * ( max - min)) + min     // 输出不确定
}                                                            // 但注意它没有副作用


function setColor(el,color){                                  //不是纯函数 
    el.style.color =  color ;                                 //直接操作了DOM，对外有副作用
}                                                             
```

是不是很简单，接下来我们家一个需求？       
如果最后一个函数，你希望批量去操作一组li并且还有许多这样的需求要改，写一个公共函数？

``` javascript 
function change (fn , els , color){
    Array.from(els).map((item)=>(fn(item,color)))
}
change(setColor,oLi,"blue")
```

**那么问题来了这个函数是纯函数吗？**   

首先无论输入什么，输出都是undefined,接下来我们分析一下对外面有没有影响，我们发现，在函数里并没有直接的影响，但是调用的setColor对外面产生了影响。那么change到底算不算纯函数呢？   

答案是当然不算，这里我们强调一点，纯函数的依赖必须是无影响的，也就是说，在内部引用的函数也不能对外造成影响。    

**问题：那么我们有没有什么办法，把这个函数提纯呢？**

## 1.3 柯里化（curry）

定义：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。    
``` javascript 
function add(x, y) {
     return x + y;
}
add(1, 2)
 
//******* 柯里化之后 *************
  
function addX(y) {
   return function (x) { 
    return x + y;
   }; 
}
var newAdd =  addX(2) 
 newAdd (1)  
```

现在我们回过头来看上一节的问题？    
如果我们不让setColor在change函数里去执行，那么setColor不就是纯函数了吗？

``` javascript    
function change (fn , els , color){
    Array.from(els).map((item)=>(fn(item,color)))
}
change(setColor,oLi,"blue")

//******* 柯里化之后 *************

function change(fn){
    return function(els,color){
        Array.from(els).map((item)=>(fn(item,color)))
    }
}
var newSetColor = change(setColor);
newSetColor(oLi,"blue")
```

    
- 我们先分析柯里化（curry）过程。在之前change函数中fn , els , color三个参数，每次调用的时候我们都希望参数fn值是 setColor，因为我们想把不同的颜色給到不同的DOM上。我们的最外层的参数选择了fn，这样返回的函数就不用在输入fn值啦。    
- 接下来我们分析提纯的这个过程，改写后无论fn输入是什么，都return出唯一确定的函数，并且在chang这个函数中，只执行了return这个语句，setColor函数并未在change上执行，所以chang对外也不产生影响。显然chang这时候就是一个纯函数。    
- 最后如果我们抛弃柯里化的概念，这里就是一个最典型的闭包用法而已。而change函数的意义就是我们可以通过它把一类setColor函数批量去改成像newSetColor这样符合新需求的函数。

上面那个例子是直接重写了chang函数，能不能直接在原来change的基础上通过一个函数改成 newSetColor呢？

``` javascript    
function change (fn , els , color){
    Array.from(els).map((item)=>(fn(item,color)))
}
change(setColor,oLi,"blue")

//******* 通过一个curry函数*************

var changeCurry = curry(change);
var newSetColor = changeCurry(setColor);
newSetColor(oLi,"blue")
```

哇！真的有这种函数吗？当然作为帮助函数（helper function），lodash 或 ramda都有啊。我们在深入的系列的课程中会动（chao）手（xi）写一个。


**问题：处理上一个问题时，我们将一个函数作为参数传到另一个函数中去处理，这好像在函数式编程中很常见，他们有什么规律吗？**

## 1.4 高阶函数

定义：函数当参数，把传入的函数做一个封装，然后返回这个封装函数,达到更高程度的抽象。  

很显然上一节用传入fn的chang函数就是一个高阶函数，显然它是一个纯函数，对外没有副作用。可能这么讲并不能让你真正去理解高阶函数，那么我就举几个例子！

###1.4.1 等价函数

定义 ：调用函数本身的地方都可以其等价函数；

``` javascript    
function __equal__(fn){
        return function(...args){
            return fn.apply(this,args);
        }
    }
//第一种
function add(x,y){
    return x + y
}
var addnew1 = __equal__(add);
console.log(add(1,2));
console.log(addnew1(1,2));

//第二种
let obj = {
      x : 1,
      y : 2,
      add : function (){
        console.log(this)
        return this.x + this.y  
      }
   }
   
var addnew2 = __equal__(obj.add);

console.log( obj.add() ) ;           //3
console.log( addnew2.call(obj));      //3

```

第一种不考虑this

- __equal__(add):让等价（__equal__）函数传入原始函数形成闭包，返回一个新的函数addnew1
- addnew1(1,2):addnew1中传入参数，在fn中调用，fn变量指向原始函数

第二种考虑this

- addnew2.call(obj): 让__equal__函数返回的addnew2函数在obj的环境中执行,也就是fn.apply(this,args);中的父级函数中this,指向obj
- fn.apply(this,args)中，this是一个变量，继承父级, 父级指向obj，所以在obj的环境中调用fn
- fn是闭包形成指向obj.add

好了，看懂代码后，我们发现，这好像和直接把函数赋值给一个变量没啥区别，那么等价函数有什么好处呢？

等价函数的拦截和监控：

``` javascript    
function __watch__(fn){
        //偷偷干点啥
         return function(...args){
            //偷偷干点啥
            let ret = fn.apply(this,args);
            //偷偷干点啥
            return ret
         }
}
```
我们知道，上面本质就是等价函数，fn执行结果没有任务问题。但是可以在执行前后，偷偷做点事情，比如consle.log("我执行啦")。

**问题：等价函数可以用于拦截和监控，那有什么具体的例子吗？**

###1.4.2 节流（throtle）函数

前端开发中会遇到一些频繁的事件触发，为了解决这个问题，一般有两种解决方案：

- throttle 节流
- debounce 防抖

``` javascript 

function throttle(fn,wait){
     var timer;
     return function(...args){
        if(!timer){
            timer = setTimeout(()=>timer=null , wait);
            console.log(timer)
            return fn.apply(this,args)
        }
     }
}

const fn  = function(){ console.log("btn clicked")}
const btn = document.getElementById('btn');
btn.onclick = throttle(fn , 5000);

```

分析代码

- 首先我们定义了一个timer
- 当timer不存在的时候，执行if判断里函数
- setTimeout给timer 赋一个id值，fn也执行
- 如果继续点击，timer存在，if判断里函数不执行
- 当时间到时，setTimeout的回调函数清空timer，此时再去执行if判断里函数

所以，我们通过对等价函数监控和拦截很好的实现了节流（throtle）函数。而对函数fn执行的结果丝毫没有影响。

**问题：既然我们实现了等价函数，不是还有一种方案吗？**

###1.4.3 防抖（debounce）函数




<meta http-equiv="refresh" content="2">


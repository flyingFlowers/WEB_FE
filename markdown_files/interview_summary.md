### 面试题目总结

#### 浏览器方面的问题

##### 主流浏览器的内核问题

| 浏览器         | 内核           |
| ------------- |:-------------: |
| IE            | trient         |
| chrome        | webkit         |
| fireFox       | gecko          |
| opera         | presto->blink  |
| safari        | webkit->blink  |

##### 浏览器的两种模式

1. 标准模式：所有浏览器都支持。HTML页面开头有<DOCTYPE html>即此页面为标准模式。
2. 怪异模式：又称为混杂模式。是为了兼容以前老版本浏览器的语法。与标准模式正好相对，HTML页面开头没有<DOCTYPE html>就是混杂模式。
> 可以使用document.compatMode来查看当前页面的模式类型。
标准模式："CSS1Compat"
怪异模式："BackCompat"

##### 针对兼容性问题封装了哪些函数

1. 获取元素的样式getStyle(ele, prop)

```
    function getStyle(ele, prop) {
        if(ele.currentStyle) {//兼容IE
            return ele.currentStyle[prop];
        }else {//getComputedStyle第二个参数可以填写伪元素，这也是唯一获取伪元素样式的方法
            window.getComputedStyle(ele, null)[prop];
        }
    }
```

2. 事件绑定addEvent(ele, type, handle)

```
    function addEvent(ele, type, handle) {
        if(ele.addEventListener) {
            ele.addEventListener(type, handle, false);//第三个参数用来表示是否开启事件捕获，默认开启事件冒泡
        }else if(ele.attachEvent) {
            ele['temp' + type + handle] = handle;
            ele[type + handle] = function () {//将回调函数保存在ele中，用于取消事件
                ele['temp' + type + handle].call(ele); //由于attachEvent的this默认指向window,所以改变this指向ele
            }
            ele.attachEvent('on' + type, ele[type + handle]);
        }else {
            ele['on' + type] = handle;
        }
    }
```

3. 对应绑定事件，取消事件绑定removeEvent(ele, type, handle)

```
    function removeEvent(ele, type, handle) {
        if(ele.removeEventListener) {
            ele.removeEventListener(type, handle, false);
        }else if(ele.detachEvent) {
            ele.detachEvent('on' + type, ele[type + handle]);//使用的是上面封装好的ele[type + handle]函数
        }else {
            ele['on' + type] = null;//false或者''都可以
        }
    }
```
4. 取消冒泡stopBubble(e)

```
    function stopBubble(e) {
        var event = e || window.event;//兼容IE的会将事件传到window.event上面
        if(event.stopPropagation) {
            event.stopPropagation();//IE9 以下版本不兼容
        }else {
            event.cancelBubble = true;//IE独有，现在高版本的浏览器可能也存在
        }
    }
```

5. 阻止默认事件 cancelHandler(e)：表单提交，a标签跳转，右键菜单等

```
    function cancelHandler(e) {
        var event = e || window.event;
        if(event.preventDefault) {//W3C标准
            event.preventDefault();
        }else if(event.returnValue) {
            event.returnValue = false;//兼容IE
        }
    }
```
对于用对象属性方式注册的时间来说，可以return false

```
    document.oncontextmenu = function (e) {//右键菜单失效
        return false;
    }
```


6. 事件源对象：chrome都有下面两个对象属性

* event.target        fireFox独有

* event.srcElement     IE独有

```
    var target = event.target || event.srcElement;
```

7. 封装ajax

```
    function ajax(method, url, data, success) {
        var xhr = null;
        try {
            xhr = new XMLHttpRequest();
        }catch(e) {
            xhr = new ActiveXObject('Microsoft.XMLHTTP');//IE
        }
        if(method == 'get') {
            url += '?' + data;
        }
        xhr.open(method, url, true);//第三个参数为是否开启异步
        if(method == 'get') {
            xhr.send();
        }else if(method == 'post') {
            xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');//设置请求头部
            xhr.send(data);
        }
        xhr.onreadystatechange = function () {
            if(xhr.readyState == 4) {
                if(xhr.state == 200) {
                    success(xhr.responseText);
                }else {
                    console.log('err');
                }
            }
        }
    }
```

##### 标准的盒子模型与IE的盒子模型

* 标准盒子模型： margin + border + padding + content
这是W3C标准的盒子模型，即box-sizing: conten-box(默认)

* IE的盒子模型：margin + content（border + padding + content）。其中content为三个之和，即box-sizing: border-box;

#### HTML和CSS方面的

##### CSS选择器的权重问题

| 选择器            | 权重           |
| -------------    |:-------------: |
| !important       | 无穷大          |
| 行间样式          | 1000           |
| id               | 100            |
| class\|属性\|伪类 | 10             |
| 标签\|伪元素      | 1              |
| 通配符*           | 0              |

##### 消除浮动的方式

* 在外层wrapper上面使用伪元素

```
    .wrapper::after {
        content: '';
        clear: both;
        display: block;//消除浮动必须是block
    }
```

* 将外层wrapper的display变成inline-block

* 将外层的wrapper也float

* 将外层的wrapper里面添加overflow:hidden

* 将外层的wrapper的position设为fixed或者absolute

* 对于IE来说没有伪元素，通过hasLayout出发bfc
``` 
    *zoom: 1;//ie6和ie7
```

```
    _zoom: 1;//只有ie6能识别
```

##### 说一下margin合并和margin塌陷问题

margin合并问题： 当一个元素在另一个元素上面的时候，他们的margin会有重合的部分。可以把他们包裹在一个大的wrapper里面，设置wrapper的overflow为hidden。还可以将这两个元素float一下，但是这样可能会带来上面的问题。

margin塌陷问题：对于父块DIV内含子块DIV的情况，就会按另一条CSS惯例来解释了，那就是：对于有块级子元素的元素计算高度的方式,如果元素没有垂直边框和填充,那其高度就是其子元素顶部和底部边框边缘之间的距离。常用的解决方式：在父级上方添加一个看不到的border或者overflow设置为hidden。

##### 单行文字截断打点

```
    {
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }
```
这种设置的兼容性不好，最牛逼的还是强写，就像百度那样。。。

#### javascript方面的

##### 说一下闭包问题

每一个javascript函数都是一个对象，它在每一次执行的时候都会生成一个独一无二的执行期上下文，并且在函数执行完毕之后就释放这个资源，即外部环境是无法访问函数内部的成员。闭包就是函数内部的部分资源占着内存没有释放，导致外部可以访问这部分资源。闭包会导致内存泄漏。闭包既有优势又有劣势，但是总体来说劣势大于优势，所以要避免使用。但是也有很多的好处，例如jquery就是利用了闭包的特点。（下面开始吹jquery）

jquery本身就是一个立即执行函数，它利用了闭包的优势，将自己的成员变量$挂在到了window上面，这样即使立即执行函数执行完毕之后，仍然能够访问到$。
jquery最大的特点就是将所有的成员变量即方法写到了立即执行函数的参数里面。其中主函数只是把参数的factory函数执行一下。
在factory函数中首先定义了jquery的初始化函数，这个函数只是调用了jquery.fn.init方法而已。所以绝大部分的方法是在jquery.fn.init中定义的。而jquery.fn = jquery.prototype,也就是说jquery中调用的函数就是其父级上上面的函数。jquery中y偶jquery.fn.prototype = jquery.fn,也就是说在父级的init函数中又把它的父级指向了jquery的父级，这样的话jquery.prototype可以访问init函数，而jquery.prototype.init.prototype又指向了jquery.prototype,这样就是无线循环的指向了。(同学们可以输出jquery.prototype看一下)。
自己也可以仿照jquery的这种格式写自己的"jquery",我就暂且命名为xquery了
```
    (function (global, factory) {
        factory(global);
    } (window, function (window) {
        var xquery = function () {
            return new xquery.fn.init();
        }
        xquery.fn = xquery.prototype;
        var init = xquery.fn.init = function () {
            return {
                name: 'xujian',
                test: function () {
                    console.log('I am xquery');
                }
            }
        }   
        init.prototype = xquery.fn;
        window.$ = window.xquery = xquery();
        return xquery;
    }));
```
以上就是jquery对闭包的应用(可以吹个3分钟吧)，如果面试官没有问题jquery的问题，那你可以继续吹嘛。

（意犹未尽的脸）当然在编程中还是要注意闭包问题，比如最经典的闭包问题就是一个数组（假设10个），每个元素绑定一个function,要求输出他们的index值，最后执行的时候发现输出了10个10。这个就是闭包带来的。

```
    function retB() {
        var arr = [];
        for(var i = 0; i < 10; i ++) {
            arr[i] = function () {
                console.log(i);
            }
        }
    }
    var testArr = retB();
    for(var j = 0; j < testArr.length; j ++) {
        testArr[i]();//输出10 个 10
    }
```
造成这种现象的原因就是数组中的每个元素要访问原来函数中的i只，经过一番循环之后，i已经变成了10，所以会输出10个10。解决的方法就是把下面这段代码换掉

```
    arr[i] = function () {
        console.log(i);
    }
    换成
    arr[i] = (function (n) {
        console.log(n);
    } (i));
```
或者使用es6中的let

```
    for(let i = 0; i < 10; i ++) {
        arr[i] = function () {
            console.log(i);
        }
    }
```
到这里的话闭包就讲的差不多了，下面就刚才这个例子继续给他将预编译的过程，或者将es6。





























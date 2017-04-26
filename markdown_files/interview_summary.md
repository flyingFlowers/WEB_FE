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























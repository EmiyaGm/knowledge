## 尽量少用 innerHTML
* 原因：因为 innerHTML 会调用一个沉重且高消耗的HTML解析器。

## 缓存DOM操作
* `NodeList` 或 `HTMLCollection` 由于这两个集合的实时性原因，每次访问此集合都会重新查询文档。这将导致性能问题，应该做缓存处理
* JavaScript的DOM操作可以说是JavaScript最重要的功能，我们经常要根据用户的操作来动态的增加和删除元素，或是通过AJAX返回的数据动态生成元素。比如我们获得了一个很多元素的数组data[]，需要将其每个值生成一个li元素插入到一个id为container的ul元素中，最简单（最慢）的方式是：
```javascript
    var liNode, i, m;
    for (i = 0, m = data.length; i < m; i++) {
        liNode = document.createElement("li");
        liNode.innerText = data[i];
        document.getElementById("container").appendChild(liNode);
    }
```
这里每一次循环都会去查找id为container的元素，效率自然非常低，所以我们需要将元素在循环前查询完毕，在循环中仅仅是引用就行了，修改代码为：
```javascript
    var ulNode = document.getElementById("container");
    var liNode, i, m;
    for (i = 0, m = data.length; i < m; i++) {
        liNode = document.createElement("li");
        liNode.innerText = data[i];
        ulNode.appendChild(liNode);
    }
```
缓存DOM对象的方式也经常被用在元素的查找中，查找元素应该是DOM操作中最频繁的操作了，其效率优化也是大头。在一般情况下，我们会根据需要，将一些频繁被查找的元素缓存起来，在查找它或查找它的子孙元素时，以它为起点进行查找，就能提高查找效率了。

## 在内存中操作元素
* 由于DOM操作会导致浏览器的重排（回流），重排（回流）需要花费大量的时间进行样式计算和节点重绘与渲染，所以应当尽量减少重排（回流）次数。一种可靠的方法就是加入元素时不要修改页面上已经存在的元素，而是在内存中的节点进行大量的操作，最后再一并将修改运用到页面上。DOM操作本身提供一个创建内存节点片段的功能:document.createDocumentFragment()，我们可以将其运用于上述代码中：
```javascript
    var ulNode = document.getElementById("container");
    var liNode, i, m;
    var fragment = document.createDocumentFragment();
    for (i = 0, m = data.length; i < m; i++) {
        liNode = document.createElement("li");
        liNode.innerText = data[i];
        fragment.appendChild(liNode);
    }
    ulNode.appendChild(fragment);
```
这样就只会触发一次重排（回流），效率会得到很大的提升。如果需要对一个元素进行复杂的操作（删减、添加子节点），那么我们应当先将元素从页面中移除，然后再对其进行操作，或者将其复制一个（cloneNode()），在内存中进行操作后再替换原来的节点。

## 通过事件委托（代理）批量操作事件
* 还是之前那个ul和添加li，如果我们需要给每个li都绑定一个click事件，就可能写出类似如下代码：
```javascript
    var ulNode = document.getElementById("container");
    var fragment = document.createDocumentFragment();
    var liNode, i, m;
    var liFnCb = function(evt){
        //do something
    };
    for (i = 0, m = data.length; i < m; i++) {
        liNode = document.createElement("li");
        liNode.innerText = data[i];
        liNode.addEventListener("click", liFnCb, false);
        fragment.appendChild(liNode);
    }
    ulNode.appendChild(fragment);
```
这里每个li元素都需要执行一次addEventListener()方法，如果li元素数量一多，就会降低效率。所以我们可以通过事件代理的方式，将事件绑定在ul上，然后通过event.target来确定被点击的元素是否是li元素，同时我们也可以使用innerHTML属性一次性创建节点了，修改代码为：

```javascript
    var ulNode = document.getElementById("container");
    var fragmentHtml = "", i, m;
    var liFnCb = function(evt){
        //do something
    };
    for (i = 0, m = data.length; i < m; i++) {
        fragmentHtml += "<li>" + data[i] + "</li>";
    }
    ulNode.innerHTML = fragmentHtml;
    ulNode.addEventListener("click", function(evt){
        if(evt.target.tagName.toLowerCase() === 'li') {
            liFnCb.call(evt.target, evt);
        }
    }, false);
```
这样事件绑定的代码就只要执行一次，可以监听所有li元素的事件了。当然如果需要移除事件回调函数，我们也不需要循环遍历所有的li元素，只需要移除ul元素上的事件处理就行了。

## 使用原型链扩展我们需要的功能
```javascript
    /**                          
    * 方法:Array.contains(element)     
    * 功能:确定某个元素是否在数组中.        
    * 参数:要查找的Object对象(简单对象)
    * 返回:找到返回true,否则返回false;
    */                                                
    Array.prototype.contains=function(element){
        for(var i=0;i<this.length;i++){
            if(this[i]== element){
                return true;
            }
        }
        return false;
    }
    //判断是否存在,通过元素的id
    Array.prototype.containsById=function(elementId){
        for(var i=0;i<this.length;i++){
            if(this[i].id== elementId){
                return true;
            }
        }
        return false;
    }
    
    //删除Array的元素
    Array.prototype.remove=function(element){
        for(var i=0;i<this.length;i++){
            if(this[i] == element){
                this.splice(i,1);
                //break;
            }
        }
    }
    
    //删除Array的元素,通过元素的Id
    Array.prototype.removeById=function(elementId){
        for(var i=0;i<this.length;i++){
            if(this[i].id == elementId){
                this.splice(i,1);
                //break;
            }
        }
    }
```

## JavaScript默认是同步解析的
* 当DOM在解析时遇到 `<script>` 标签，将停止解析文档，并执行JavaScript脚本，如果是外部脚本，必须要下载后再解析，这将导致性能问题。

## 处理JavaScript事件时的性能问题
* 过多的绑定事件处理函数，每个函数都是对象，都会占用内存
* 在绑定事件的时候务必要访问DOM元素，如果访问的过多，这也将导致性能问题
* 内存中留有废掉的事件处理函数：
    * 使用 removeChild() 或 replaceChild() 函数移除或替换节点时，如果被移除或替换的节点有绑定事件函数，那么该函数不会被当做垃圾回收。另外使用 innerHTML 替换DOM时也会出现这种情况

使用事件委托解决以上问题：减少DOM元素与事件函数的链接数，减少DOM访问次数，减少事件函数数量以减少内存占用。

## 最小化重排和重绘

改变元素多种样式的时候，最好用className，一次性完成操作，这样只会修改一次DOM。

[重排和重绘的概念及触发条件查看这里](/note/performance/reflow-repaint)

## FOUC (无样式内容闪烁)

该问题主要出现在 IE 浏览器，原因有两个：

* 1、使用@import方法导入CSS

```html
<style>
    @import "../reset.css";
</style>
```

`IE` 加载HTML文档后，会先解析文档，然后再去加载由 `import` 导入的外部CSS文件，在CSS没有被加载的这段时间内，页面是无样式的。

* 2、零散的添加样式引用

将样式表链接放在页面不同位置时，在IE5/6下某些页面会无样式显示内容且瞬间闪烁，这现象就是文档样式短暂失效（Flash Of Unstyled Content），即FOUC。

解决方案：

* 避免使用 `@import` 引入外部样式
* 将样式表引入放在 `<head>` 标签内

## 懒加载和延迟加载
### 什么是懒加载？
懒加载也就是延迟加载。
当访问一个页面的时候，先把img元素或是其他元素的背景图片路径替换成一张大小为1*1px图片的路径（这样就只需请求一次，俗称占位图），只有当图片出现在浏览器的可视区域内时，才设置图片正真的路径，让图片显示出来。这就是图片懒加载。
### 为什么要使用懒加载？
很多页面，内容很丰富，页面很长，图片较多。比如说各种商城页面。这些页面图片数量多，而且比较大，少说百来K，多则上兆。要是页面载入就一次性加载完毕。估计大家都会等到黄花变成黄花菜了。
### 懒加载的原理是什么？
页面中的img元素，如果没有src属性，浏览器就不会发出请求去下载图片，只有通过javascript设置了图片路径，浏览器才会发送请求。
懒加载的原理就是先在页面中把所有的图片统一使用一张占位图进行占位，把正真的路径存在元素的“data-url”（这个名字起个自己认识好记的就行）属性里，要用的时候就取出来，再设置；
### 懒加载的实现步骤？
1)首先，不要将图片地址放到src属性中，而是放到其它属性(data-original)中。
2)页面加载完成后，根据scrollTop判断图片是否在用户的视野内，如果在，则将data-original属性中的值取出存放到src属性中。
3)在滚动事件中重复判断图片是否进入视野，如果进入，则将data-original属性中的值取出存放到src属性中。
### 懒加载的优点是什么？
页面加载速度快、可以减轻服务器的压力、节约了流量,用户体验好

### 什么是预加载？
提前加载图片，当用户需要查看时可直接从本地缓存中渲染
### 为什么要使用预加载？
图片预先加载到浏览器中，访问者便可顺利地在你的网站上冲浪，并享受到极快的加载速度。这对图片画廊及图片占据很大比例的网站来说十分有利，它保证了图片快速、无缝地发布，也可帮助用户在浏览你网站内容时获得更好的用户体验。
### 实现预加载的方法有哪些？
方法一：用CSS和JavaScript实现预加载 html中img标签最初设置为display:none。
方法二：仅使用JavaScript实现预加载 js脚本中使用image对象动态创建好图片。
```html
    <img class="backImg" src="" alt="背景图片">
```
```javascript
    $(function () {
        preloadImg("images/backImg1.jpg",$(".backImg"),callback);
    });
    
    function preloadImg(url,$selector,callback) {
        var img = new Image();
        img.src = url;
    
        if(img.complete) {  
            //已有缓存时直接赋值                  
            $selector.attr("src",url);
            callback(img);
            return;
        } 
    
        img.onload = function() {
            img.onload = null;
            $selector.attr("src",url);
            callback(img);
        };
    }
```
方法三：使用Ajax实现预加载 使用XMLHttpRequest对象可以更加精细的控制预加载过程，缺点是无法跨域。

```javascript
    var xmlhttprequest = new XMLHttpRequest(); 
    xmlhttprequest.open("GET",src,true);
```

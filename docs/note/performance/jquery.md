## 适当使用原生JS
* 创建jQuery对象会带来一些开销。所以，如果比较注重性能的话，尽可能使用原生的javascript。在某些方面可能会更容易理解和写更少的代码。例如：
```html
    <div id="colors">
        <ul>
           <li id="a"></li>
           <li id="0"></li>
           <li id="mm"></li>
        </ul>
    </div>
```
```javascript
    // 打印list中的li的id
    $('#colors li' ).each(function(){
        //将$(this).attr('id')方法替换为直接通过ID属性访问
        console.log(this.id);
    })
```
## 选择器优化
* 如果你需要更好的性能，但是仍然要用到jQuery，你可以在jQuery选择器优化做一些尝试。以下是一个测试程序，通过浏览器的控制台console.time 和console.timeEnd 方法来记录不同选择器执行时间。

```html
    <div id="peanutButter" >
        <div id="jelly" class="jellyTime" ></div>
    </div>
```
```javascript
    //测试程序
    var iterations = 10000,i;
    
    //--------------------------------------------
    //Case 1: 很慢
    console.time('Fancy');
    for (i = 0; i < iterations; i++) {
        $('#peanutButter div:first');
    }
    console.timeEnd('Fancy'); //648.849ms
    
    //--------------------------------------------
    //Case 2: 比较好，但仍然很慢
    console.time('Parent-child');
    for (i = 0; i < iterations; i++) {
        $('#peanutButter div');
    }
    console.timeEnd('Parent-child'); //23.901ms
    
    //--------------------------------------------
    //Case 3: 一些浏览器会比较快
    console.time('Parent-child by class');
    for (i = 0; i < iterations; i++) {
        // 通过后代Class选择器
        $('#peanutButter .jellyTime');
    }
    console.timeEnd('Parent-child by class'); //11.010ms
    
    //--------------------------------------------
    //Case 4: 更好的方式 
    console.time('By class name');
    for (i = 0; i < iterations; i++) {
        // 直接通过Class选择器
        $('.jellyTime');
    }
    console.timeEnd('By class name'); //10.476ms
    
    //--------------------------------------------
    //Case 5: 推荐的方式 ID选择器
    console.time('By id');
    for (i = 0; i < iterations; i++) {
        $('#jelly');
    }
    console.timeEnd('By id'); //15.164ms
```

## 缓存jQuery对象
* 每次通过选择器构建一个新的jQuery对象时，jQuery的核心部分的Sizzle引擎会遍历DOM然后通过对应的选择器来匹配真正的dom元素。
* 这种方式比较低效，在现代浏览器中可以通过document.querySelector方法通过传入对应的Class参数来匹配对应的元素，不过IE8以下版本不支持此方法。一个提高性能的实践是通过变量缓存jQuery对象。例如：

```html
    <ul id="pancakes" >
             <li>first</li>
             <li>second</li>
             <li>third</li>
             <li>fourth</li>
             <li>fifth</li>
    </ul>
```
```javascript
    // 不好的方式:
    // $('#pancakes li').eq(0).remove();
    // $('#pancakes li').eq(1).remove();
    // $('#pancakes li').eq(2).remove();
    // ------------------------------------
    // 推荐的方式:
    var pancakes = $('#pancakes li');
    pancakes.eq(0).remove();
    pancakes.eq(1).remove();
    pancakes.eq(2).remove();
    // ------------------------------------
    // 或者:
    // pancakes.eq(0).remove().end()
    //  .eq(1).remove().end()
    //  .eq(2).remove().end();
```

## 扩展我们需要的功能
```javascript
    $.extend({
        min: function(a, b){return a < b?a:b; },
        max: function(a, b){return a > b?a:b; }
    }); 
    //为jquery扩展了min,max两个方法
```
使用扩展的方法（通过“$.方法名”调用）：
```javascript
    var m=10,
    　　n=30;
    　　console.log("m=10,n=30,max="+$.max(m,n)+",min="+$.min(m,n));
```

## 图片预加载
* 如果你的网页使用了很多隐藏图片文件（例如：鼠标悬停展示的图片），那么图片的预加载是有意义的：
```javascript
    $.preloadImages = function () {
      for (var i = 0; i < arguments.length; i++) {
        $('<img>').attr('src', arguments[i]);
      }
    };
     
    $.preloadImages('img/hover-on.png', 'img/hover-off.png');
```

### 总是从ID选择器开始继承
* 在jQuery中最快的选择器是ID选择器，因为它直接来自于JavaScript的getElementById()方法。
当然 这只是对于单一的元素来讲。如果你需要选择多个元素，这必然会涉及到 DOM遍历和循环，
为了提高性能，建议从最近的ID开始继承。

### 在class前使用tag(标签名)
* 在jQuery中第二快的选择器是tag(标签)选择器( 比如：$(“head”) )。跟ID选择器累时，因为它来自原生的getElementsByTagName() 方法。
在使用tag来修饰class的时候，我们需要注意以下几点：
（1） 不要使用tag来修饰ID
（2）不要画蛇添足的使用ID来修饰ID

### 推迟到 $(window).load
* jQuery对于开发者来说有一个很诱人的东西, 可以把任何东西挂到$(document).ready下。尽管$(document).rady 确实很有用， 它可以在页面渲染时，其它元素还没下载完成就执行。如果你发现你的页面一直是载入中的状态，很有可能就是$(document).ready函数引起的。你可以通过将jQuery函数绑定到$(window).load 事件的方法来减少页面载入时的cpu使用率。它会在所有的html(包括iframe)被下载完成后执行。一些特效的功能，例如拖放, 视觉特效和动画, 预载入隐藏图像等等，都是适合这种技术的场合。

### 给选择器一个上下文
* jQuery选择器中有一个这样的选择器，它能指定上下文。
  jQuery( expression, context );
  通过它，能缩小选择器在DOM中搜索的范围，达到节省时间，提高效率。
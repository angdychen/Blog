这是JavaScript设计模式的第四篇，这次我们来认识一下外观模式（the Facade Pattern）。不管你有没有听说过它，我敢保证，你用的任何一种编程语言，哪怕只写过几行代码你都会用到外观模式。（这么说虽然有些夸张，但是你一定不介意的对吧？）。你可以通过外观模式把一堆复杂接口定义成一小段代码，之后就随便你怎么调用了。难道你没在一个命名函数里写过代码？对，这就是外观模式的一个例子。
![facade_structure.png][facade_structure]
在我们进一步探讨之前，我想提醒一下，我们从JavaScript设计模式系列一步步走来。先是[单例][cn1]然后是[桥接][cn2]，最后是上一篇的[组合][cn3]。

## 几个现成的例子
言归正传，我会给你看几个非常不错的例子。我们从[jQuery][jq]开始，如果你还不知道什么是jQuery的话，那你真是井底之蛙了，看来你并没有读我的前几篇文章（你真应该花点时间去读一下）。jQuery和其他如[Prototype](http://prototypejs.org/)和[YUI](http://yuilibrary.com/)等JavaScript库本质上是一组外观，它们让像我们这样的程序员能活得轻松点。它们把复杂代码（通常包含一些浏览器特性检测，为了让不同浏览器都能正确执行）转换成单个方法供你调用。

另一个不错的例子就是桌面的快捷方式，你只需通过单（双）击它就能跳过好几层文件夹打开或者执行一个文件。

## JavaScript代码示例
我知道外观模式的概念很容易掌握，你都不一定需要一个JavaScript代码的例子，但是总有些人更在乎代码，会觉得那样才更容易理解。更何况，没有代码示例的JavaScript文章根本就不具说服力，就应该从网上删掉。
我们从一个简单的事件监听器例子开始。大家都知道要添加一个事件监听器并不一件容易的事，除非只想让代码运行在少数几个浏览器上。你不得不测试很多方法以确保针对不同浏览器的代码都能正常运行。在这个代码示例中我们只是把特性检测添加到这个方法中：
```javascript
function addEvent(element, type, func) {
    if (window.addEventListener) {
        element.addEventListener(type, func, false);
    }
    else if (window.attachEvent) {
        element.attachEvent('on'+type, func);
    }
    else {
        element['on'+type] = func;
    }
}
```
简单吧！我真希望我可以不用写那些不必要的代码，让它们越简单越好，但是如果真是这样就没什么意思了，你也不会想读下去了，对吧？所以我不这么认为，我想我要给你看点更复杂的东西。我只是想说，你的代码原本看起来会有些像下面这样：
```javascript
var foo = document.getElementById('foo');
    foo.style.color = 'red';
    foo.style.width = '150px';
     
var bar = document.getElementById('bar');
    bar.style.color = 'red';
    bar.style.width = '150px';
     
var baz = document.getElementById('baz');
    baz.style.color = 'red';
    baz.style.width = '150px';
```
太蹩脚了！你对每个元素做了一模一样的事！我认为我们可以让它变得更简单点：
```javascript
function setStyle(elements, property, value) {
    for (var i=0, length = elements.length; i < length; i++) {
        document.getElementById(elements[i]).style[property] = value;
    }
}
 
// 现在你可以这么写：
setStyle(['foo', 'bar', 'baz'], 'color', 'red');
setStyle(['foo', 'bar', 'baz'], 'width', '150px');
```
是不是觉得咱们NB坏了？你快算了吧！咱们可是JavaScript程序员呀！能不能用点脑子，来点真格的。也许我们可以只调用一次就能设置所有的样式。看下这个：
```javascript
function setStyles(elements, styles) {
    for (var i=0, length = elements.length; i < length; i++) {
        var element = document.getElementById(elements[i]);
         
        for (var property in styles) {
            element.style[property] = styles[property];
        }
    }
}
 
//现在你只要这样写：
setStyles(['foo', 'bar', 'baz'], {
    color: 'red',
    width: '150px'
});
```
如果我们有很多元素想设置相同的样式，那这段代码真是为我们节省了不少时间。

## 总结
这就是这一节JavaScript设计模式的全部内容了。我很荣幸能在评论中看到你的经验之谈。如果你真心觉得这篇文章不错请你用下方的分享按钮把我这个小小的博客分享到网上。说真的，我全凭它活呢。


[facade_structure]: http://www.codingserf.com/wp-content/uploads/2015/05/facade_structure.png
JavaScript设计模式系列：
- [单例模式][cn1]（[Singleton Pattern][en1]）
- [桥接模式][cn2]（[Bridge Pattern][en2]）
- [组合模式][cn3]（[Composite Pattern][en3]）
- [外观模式][cn4]（[Facade Pattern][en4]）
- [适配器模式][cn5]（[Adapter Pattern][en5]）
- [装饰者模式][cn6]（[Decorator Pattern][en6]）
- [工厂模式（一）][cn7]（[Factory Pattern Part 1][en7]）
- [工厂模式（二）][cn8]（[Factory Pattern Part 2][en8]）
- [代理模式][cn9]（[Proxy Pattern][en9]）
- [观察者模式][cn10]（[Observer Pattern][en10]）
- [命令模式][cn11]（[Command Pattern][en11]）
- [责任链模式][cn12]（[Chain of Responsibility Pattern][en12]）


<hr/>
转载请注明：
英文作者：[Joe Zim](http://www.joezimjs.com/authors/joe-zimmerman/)
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-facade/][en4]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-facade/][cn4]

[cn1]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-singleton/
[cn2]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-bridge/
[cn3]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-composite/
[cn4]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-facade/
[cn5]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-adapter/
[cn6]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-decorator/
[cn7]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-factory-part-1/
[cn8]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-factory-part-2/
[cn9]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-proxy/
[cn10]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-observer/
[cn11]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-command/
[cn12]: http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-chain-of-responsibility/

[en1]: http://www.joezimjs.com/javascript/javascript-design-patterns-singleton/
[en2]: http://www.joezimjs.com/javascript/javascript-design-patterns-bridge/
[en3]: http://www.joezimjs.com/javascript/javascript-design-patterns-composite/
[en4]: http://www.joezimjs.com/javascript/javascript-design-patterns-facade/
[en5]: http://www.joezimjs.com/javascript/javascript-design-patterns-adapter/
[en6]: http://www.joezimjs.com/javascript/javascript-design-patterns-decorator/
[en7]: http://www.joezimjs.com/javascript/javascript-design-patterns-factory/
[en8]: http://www.joezimjs.com/javascript/javascript-design-patterns-factory-part-2/
[en9]: http://www.joezimjs.com/javascript/javascript-design-patterns-proxy/
[en10]: http://www.joezimjs.com/javascript/javascript-design-patterns-observer/
[en11]: http://www.joezimjs.com/javascript/javascript-design-patterns-command/
[en12]: http://www.joezimjs.com/javascript/javascript-design-patterns-chain-of-responsibility/
[jq]: http://jquery.com/








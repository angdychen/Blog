又一篇博客，又一种JavaScript设计模式。这次我们着重介绍适配器模式（the Adapter Pattern）。如果你想阅读之前的文章，可以在文章底部找到它们的链接。适配器就像一台接口转换机一样。不过它并非完全改变一个接口，它只是创建一个新的对象或函数，去适配现有对象或函数的接口，去兼容那我些我们已经在使用的代码。
这么说有点让人费解，你需要一个更好的上情境帮助理解。至少，如果是我第一次听别人跟我说一个东西的工作原理的时候，就会有种困惑的感觉。所以我们需要更多的信息。首先，这个模式是为兼容现有代码而设计的。我们现有的代码已经在使用一些对象和函数，但是我们想替换它们。问题的关键是新创建的对象和函数会用到和之前完全不同的接口。相比修改之前的代码，我们不如引入适配器。
对于即将投入使用的新对象，为了能兼容旧对象建立的接口，适配器要么去包装它，要么就是像一个媒介一样位于中间作调解，这取决于我们如何使用这个对象。如果对象总是通过`new`操作符来实例化，那么适配器就应该包装它，内部有一个该对象的实例，然后调用这个实例上的方法。如果对象是“静态”的，只有一个实例，就不需要包装。
![addapter_structure.png][addapter_structure]

## 场景示例
我能给出好多例子来展示如何以及何时会使用适配器模式。一个最常见的例子就是在项目中使用一个框架或者一个库：例如jQuery。问题随之而来，因为某种原因，我们决定用其他的库（例如YUI）。在一个庞大的应用程序里我们绝对没办法把所有的代码和方法调用都从jQuery改成YUI。你必须创建一个适配器，虽然相当困难，但是总比改代码要强。
许多企业级应用里都会使用日志库，在不同的日志库之间切换会非常常见。在JavaScript应用程序里可能并不会那么常见，因为浏览器内建了日志功能，不过这种切换也是有可能出现的。

## JavaScript代码示例
一件事情有可能发生时，它就一定会发生。首先让我们来看一下这个小小的`LoggerFactory`，它让我们能更容易地修改我们使用的日志接口。
```javascript
var LoggerFactory = {
    getLogger: function() {
        return window.console;
    },
    ...
};
 
/* 用法示例 */
var logger = LoggerFactory.getLogger();
logger.log("something to log");
```
在我们调用`getLogger`时它给我们返回了控制台对象(console)。为了这个练习我们假装console对象只有一个方法——`log`，并且它只能接收一个字符串类型的参数。
接下来，我们有另一个日志接口，这个会复杂些，因为1）它是用JavaScript实现的，不像console那样是浏览器本身就有的；2）它会把日志通过AJAX发送到服务器，这也意味着我们要对URL数据进行编码（代码里不会具体实现URL编码相关的事，因为它和我们的要讲的适配器模式毫不相干）。当然，它会使用一个和控制台不同的接口。
```javascript
var AjaxLogger = {
    sendLog: function() {
        var data = this.urlEncode(arguments);
         
        jQuery.ajax({
            url: "http://example.com/log",
            data: data
        });
    },
     
    urlEncode: function(arg) {
        ...
        return encodedData;
    },
    ...
};
```
我们使用了jQuery的AJAX请求，主要是为了节省时间，忽略那些和适配器模式不想干的事情。
我们现在要做的事情就是创建一个适配器，并且改变之前的`LoggerFactory`让其返回这个适配器而不是控制台对象。
```javascript
var AjaxLoggerAdapter = {
    log: function(arg) {
        AjaxLogger.sendLog(arg);
    }
};
 
/* 调整 LoggerFactory */
 
var LoggerFactory = {
    getLogger: function() {
        // 改变返回值
        return AjaxLoggerAdapter;
    },
    ...
};
```
我们对现有代码只做了一行更改，整个程序就可以使用这个新的日志接口了。

## 复杂适配器
日志接口是个很简单的例子，它只有一个方法，把它直接映射到旧的方法上也没什么难的。大多数情况下并不是如此。你可能会碰到这样的问题，即这些互相映射的函数的参数是完全不同的，旧接口可能根本没有这些参数，你必须自己处理它们。某些情况下，你又必须删掉一些参数，因为新的接口根本用不上它们。如果两个对象之间的接口映射太难，我们就要想想别的办法了，反正我不希望查找和修改数千行旧代码。

## 总结
适配器很好用，也很容易在代码层面实现。你可以直接在你用来创建对象的工厂类里面做替换。变化是必然会发生的，尤其是在那些庞大的项目上。所以一定要记住这个模式，总有一天你会用得上。

本文底部有JavaScript设计模式系列文章的列表。你可以把这篇文章分享给你Facebook，Twitter或其他地方的朋友。如果你有什么想对我或是读者说的，就在下面留言吧。通过我们的努力让更多的人来这里学习JavaScript。


[addapter_structure]: http://www.codingserf.com/wp-content/uploads/2015/05/adapter_structure.png
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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-adapter/][en5]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-adapter/][cn5]

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








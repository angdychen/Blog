这是关于JavaScript设计模式的系列文章的第一篇。1995年，Erich Gamma, Richard Helm, Ralph Johnson和John Vlissides（有名的四人组，译注：The Gang of Four, 后文统称GoF）出版了《设计模式：可复用面向对象软件的基础》，一本为软件架构和设计难题提供一系列解决方案的书。书中同时为这些解决方案（译注：即各种设计模式）提供了一份词汇参照。如果你感兴趣可以在[维基百科](http://en.wikipedia.org/wiki/Design_Patterns)上了解更多。
那本书里对于每种解决方案的实现是用C++和Smalltalk写的，它们与JavaScript大相径庭。另一本书《[Pro JavaScript Design Patterns](http://www.amazon.com/dp/159059908X/ref=as_li_ss_tl?tag=jozisjabl-20)》则是用JavaScript来实现那些设计模式的。我本希望从这本书里能得到很多知识，然而事与愿违……他们可能只是想调起你的胃口让你去买这本书罢了。如果你真的买了这本书一定要让他们知道是我推荐你买的。或许他们会给我一些补偿（也可能不会，但是至少我是这么希望的）。

## 孤独的单例模式
在JavaScript里，单例模式（the Singleton Pattern）非常简单，简单到可以被忽略，但是它在技术层面是如何工作的，我们还是有必要了解一下的。单例的代码写在一个单独的对象里，因此你不需要去实例化一个新对象就可以在任何你需要的时候使用它的资源，单例允许在全局范围内访问它的资源。
在JavaScript里，我们常在管理命名空间时使用单例模式，它可以降低你在代码中创建全局变量的数量。单例模式在JavaScript里要比在其他语言中更有用，因为它可以用命名空间来降低全局变量所带来的风险。

## 一个基本的单例模式
这是一个最基本最简单的用JavaScript实现的单例模式。它就是一个简单的有一些方法和属性的对象字面量，假想它们是因为某种关系才被放到一起。
```javascript
    var Singleton = {
    attr: 1,
    another_attr: 'value',
     
    method: function() {...},
    another_method: function() {...}
};
```
因为它是一个对象字面量，所以我不需要实例化它，并且这个对象仅此一个。也就是说我们可以通过一个全局变量的入口访问所有的方法和属性，如下所示：
```javascript
Singleton.attr += 1;
Singleton.method();
...
```

## JavaScript命名空间
单例模式在JavaScript中的一种使用情形就创建命名空间。像Java和C#这样的语言，命名空间这种特性已经被内建到语言本身了，并且命名空间在这些语言中是必须的。通过创建命名空间或者包来把代码组织到逻辑块中。这也是在JavaScript中使用单例模式最重要的原因，当你通过使用命名空间把你的代码从全局作用域移动到一个新的单例中的时候，可以避免全局变量被意外覆盖（overwrite）以及全局变量引起其他的bug。
使用单例模式管理命名空间非常简单。同样你可以仅仅通过创建一个对象字面量就搞定：
```javascript
    var Namespace = {
    Util: {
        util_method1: function() {...},
        util_method2: function() {...}
    },
    Ajax: {
        ajax_method: function() {...}
    },
    some_method: function() {...}
};
```
如你所见，现在如果你想使用一个实用方法，可以在`Namespace。Util`下面找到它，就像下面代码中所示的那样。当然，下面所示的`some_method`方法因为并没有在单例中嵌套多层。
```javascript
Namespace.Util.util_method1();
Namespace.Ajax.ajax_method();
Namespace.some_method();
```
通常你有可能会把这些方法作为全局函数，这也就意味着它们有非常有可能会被覆盖，尤其是像`get`这样简单的名字，诸如此类的名字再平常不过了。你只需要把全部的变量和函数添加到一个单例中，就可以完全不给别人篡改你代码的机会。

## 具体页面的JavaScript代码
在很多情况下，一个网站的某些页面运行的JavaScript代码和其他页面是不同的。你可以用单例命名空间的技术来为每个页面组织专门的代码，然后在页面加载完成后运行它们。
```javascript
Namespace.homepage = {
    init: function() {...},
    method1: function() {...},
    method2: function() {...}
}
 
Namespace.contactpage = {
    init: function() {...},
    method1: function() {...},
    method2: function() {...}
}
 
Namespace.pageutil = {
    getPageName: function() {
        // 返回当前页面的标识符
    }
}
 
var pageName = Namespace.pageutil.getPageName();
window.onload = Namespace[pageName].init;
```
这对于针对不同页面上出现的不同表单添加验证代码非常有用。你甚至可以提取出一个新的命名空间来封装些实用功能，比如我这里的`Namespace.pageutil.getPageName`。这和我之前提到的稍有不同，因为`getPageName`方法并非某个页面专有的代码，但是却可以用它来正确映射到页面专有代码。

## 再说下《Pro JavaScript Design Patterns》
《Pro JavaScript Design Patterns》这本书中对单例模式介绍了更多，然而事实上我把书中整整6页的内容压缩成这篇短小的博客，书中也提及了通过闭包，被动初始化和分支创建私有变量。就像我在文章开头所说，关于这本书我不想复制太多的内容过来，只是想勾起你的兴趣让你去买它。这么做的好处是，你既可以给他们经济援助，同时也能避免我让人家给告了。更何况，一篇博客文章不应该硬塞下一本书中一章的内容。

如果你认为本文对你有所帮助或者你只是纯粹地喜欢这篇文章，请用文章下方的社交分享按钮把它传播出去。像我这种从后山来的从来不敢想像能有人像你这样帮我。谢谢！

请继续关注JavaScript设计模式系列的更多文章：
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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-singleton/][en1]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-singleton/][cn1]

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



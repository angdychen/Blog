又一篇博客，又一种JavaScript设计模式。这次我们着重介绍适配器模式（the Adapter pattern）。如果你想阅读之前的文章，可以在文章底部找到它们的链接。适配器就像一台接口转换机一样。不过它并非完全改变一个接口，它只是创建一个新的对象或函数，去适配现有对象或函数的接口，去匹配那我些我们已经在使用的代码。
这么说有点让人费解，你需要一个更好的上下文环境。至少，如果是我第一次听到别人跟我说一个东西的工作原理时就会有种困惑的感觉。所以我们需要更多的信息。首先，这个模式是为兼容现有代码而设计的。我们现有的代码已经在使用一些对象和函数，但是我们想替换它们。问题的关键是新创建的对象和函数会用到和之前完全不同的接口。相比修改之前的代码，我们不如引入适配器。
对于即将投入使用的新对象，为了能兼容旧对象建立的接口，适配器要么去包装它，要么就是像一个媒介一样位于中间作调解，这取决于我们如何使用这个对象。如果对象总是通过`new`操作符来实例化，那么适配器就应该包装它，内部有一个该对象的实例，然后调用这个实例上的方法。如果对象是“静态”的，只有一个实例，就不需要包装。
![addapter_structure.png][p1]

## 场景示例
我能给出好多例子来展示如何以及何时会使用适配器模式。一个最常见的例子就是在项目中使用一个框架或者一个库：例如jQuery。问题随之而来，因为某种原因，我们决定用其他的库（例如YUI）。在一个庞大的应用程序里我们绝对没办法把所有的代码和方法调用都从jQuery改成YUI。你必须创建一个适配器，虽然相当困难，但是总比改代码要强。
许多企业级应用里都会使用日志库，在不同的日志库之间切换会非常常见。在JavaScript应用程序里可能并不会那么常见，因为浏览器内建了日志功能，不过这种切换也是有可能出现的。

## JavaScript代码示例







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
中文翻译：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-adapter/][cn5]

[cn1]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-singleton/
[cn2]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-bridge/
[cn3]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-composite/
[cn4]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-facade/
[cn5]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-adapter/
[cn6]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-decorator/
[cn7]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-factory-part-1/
[cn8]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-factory-part-2/
[cn9]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-proxy/
[cn10]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-observer/
[cn11]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-command/
[cn12]: http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-chain-of-responsibility/

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








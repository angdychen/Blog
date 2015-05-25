是给大家介绍观察者模式(the Observer Pattern，译注：也有叫发布-订阅模式)的时候了。如果你关注本博有一段时间了，那你应该已经通过[这篇博客][whyplugin]的介绍，对我的jQueryr插件[JZ Publish/Subscribe][plugin]有所了解了。接下来我们会讨论观察者模式的几种实现方式，所以你会知道哪一个是最适合你和你的应用程序的方式。
![observer_structure.png][observer_structure]
开讲之前还是要说一下，这是JavaScript设计模式系列中的一篇，文章最后有本系列的一个列表，希望它们对你有用。

## 什么是观察者模式
观察者模式是相当简单的一个概念。一个观察者（observer，也叫订阅者：subscriber）订阅一个可观察对象（observable，也叫发布者：publisher），期待发生些有趣的事。观察者也可以退订。这种行为依赖你对模式的具体实现。对于观察者有两个基本的方式可以获取数据：push和pull。push方式是一但发生变化，发布者会立即触发订阅者的提醒事件。pull方式是只要订阅都觉得有必要，随时可以检查发布者的变化。
我打赌你一定想看个例子。当然没问题，对一个程序员来说，代码比文字能做更多有意义的事，对吧？我们先来看一个push方式的例子：
```javascript
var Observable = function() {
    this.subscribers = [];
}
 
Observable.prototype = {
    subscribe: function(callback) {
        // 大多数情况下，你会想要检查订阅者数组里是否已经存在这个回调(callback)了。
        // 不过我们现在没有必要关注这些旁枝末节的东西。
        this.subscribers.push(callback);
    },
    unsubscribe: function(callback) {
        var i = 0,
            len = this.subscribers.length;
         
        // 遍历数组，如果找到这个回调(callback)，就删除它。
        for (; i < len; i++) {
            if (this.subscribers[i] === callback) {
                this.subscribers.splice(i, 1);
                // 一旦我们找到了，就没必要继续执行后面的代码，直接return。
                return;
            }
        }
    },
    publish: function(data) {
        var i = 0,
            len = this.subscribers.length;
         
        // 遍历整个订阅者数组，执行每一个回调。
        for (; i < len; i++) {
            this.subscribers[i](data);
        }        
    }
};
 
var Observer = function (data) {
    console.log(data);
}
 
// 这里是具体的使用
observable = new Observable();
observable.subscribe(Observer);
observable.publish('我们发布了！');
```
这里我们有几个东西要讨论一下。首先，所有与观察者模式相关的方法都是在`Observable`里实现的（译注：这也就是所谓的push，是发布者推给订阅者）。基于JavaScript的灵活性，你也可以让`Observer`来做订阅和退订的工作，但是我相信让发布者来实现这些功能更有意义更易理解。另一点要注意的是，订阅者只是一个被当作回调的函数。而在像Java这样的语言里，一个订阅者应该是实现了指定接口的对象，然后整个对象会被订阅，并且发布者只是通过订阅者的接口简单的调用指定的方法。最后，在这个例子里，`Observable`本身可以做为一个类来用，所以其他对象可以通过继承变成发布者。
现在我们来实现观察者模式里的pull方式。pull方式，更多用在信息交换的场合：
```javascript
Observable = function() {
    this.status = "constructed";
}
Observable.prototype.getStatus = function() {
    return this.status;
}
 
Observer = function() {
    this.subscriptions = [];
}
Observer.prototype = {
    subscribeTo: function(observable) {
        this.subscriptions.push(observable);
    },
    unsubscribeFrom: function(observable) {
        var i = 0,
            len = this.subscriptions.length;
         
        // 遍历数组，如果找到这个发布者(observable)，就删除它。
        for (; i < len; i++) {
            if (this.subscriptions[i] === observable) {
                this.subscriptions.splice(i, 1);
                // 一旦我们找到了，就没必要继续执行后面的代码，直接return。
                return;
            }
        }        
    },
    doSomethingIfOk: function() {
        var i = 0;
            len = this.subscriptions.length;
         
        // 遍历subscriptions确定每个元素的状态是否变成了ok，
        // 如果是ok的话就处理
        for (; i < len; i++) {
            if (this.subscriptions[i].getStatus() === "ok") {
                // 做些处理，因为observable已经变成我们想要的状态
            }
        }
    }
}
 
var observer = new Observer(),
    observable = new Observable();
observer.subscribeTo(observable);
 
// 因为状态并没有改变所以什么都不会发生
observer.doSomethingIfOk();
 
// 把状态变为ok，现在你才会处理
observable.status = "ok";
observer.doSomethingIfOk();
```
这和push方式非常不一样，不是吗？现在无论何时，只要订阅者觉得应该（或者像上面的情况，是我告诉它应该）去检查发布者的状态时，它就会去检查。一般情况下会有个计时器，但是为了例子的简单明了我就手动调用了它。从技术的角度来说，我们并不应该像上面这样直接使用`Observable`，它应该被继承，应该有内置的状态改变机制，并不是像例子中那样是我手动去改变的。

## 你已经见识过观察者模式了
上面我给出的例子都很简单，一般情况下，一个发布者对象会有不止一种类型的事件会被订阅。说到事件，你可能已经意识到DOM元素的事件处理就是一个观察者模式的实现。观察者模式无处不在，非常有用。另外，许多用到动画的jQuery插件里也包含观察者模式，你可以在一个动画的不同位置注入你的功能。

## 我观察(Observe)到临近结尾了
观察者模式是一个了不起的工具，可维护和组织大型的基于操作的应用程序，甚至仅仅为了让你的jQuery插件更方便和灵活。它把代码抽象到了一个不错的水平，以帮助解耦你的代码，并保持它的整洁和可维护。显然观察者模式并不是放之四海皆准的，不过它确实可以用在无数场景中。
你应该看看[JZ Publish/Subscribe][plugin]里观察者模式的另一种实现。你更应该看看[如何和为什么应该使用它][whyplugin]。如果你想了解其他模式，下面就有JavaScript设计模式系列的文章列表。欢迎留言，也希望你能把这篇文章分享出去。Happy Coding!


[plugin]: http://www.joezimjs.com/projects/publish-subscribe-jquery-plugin/
[whyplugin]: http://www.joezimjs.com/javascript/how-and-why-jz-publish-subscribe-should-be-used/
[observer_structure]: http://www.codingserf.com/wp-content/uploads/2015/05/observer_structure.png


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-observer/][en10]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-observer/][cn10]

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








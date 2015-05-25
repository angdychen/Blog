我今天我会展示另一种JavaScript设计模式：装饰者(the Decorator Pattern)，它是一种**不**通过子类或添加额外属性的方式就可以给对象增加新的功能的手段。这篇博客是JavaScript设计模式系列中的一篇，如果你第一次接触这个系列，你可以在本文底部找到前几篇的链接。

## 回归博客
在开始我的[新插件](http://www.joezimjs.com/projects/publish-subscribe-jquery-plugin/)开发后，我似乎很难回到正常写博客的状态。如果你不知道我在说什么，你应该看看我的[这篇声明](http://www.joezimjs.com/javascript/new-jquery-plugin-publish-subscribe/)。然而不管如何，从这个月开始，我承诺每个月都会写两篇“教程”。我之所以把“教程”打上双引号，是因为我不确定通常它们会不会被人们这么叫，不过我在那些以传授知识为目的的博客里都是这么叫的。

## 关于装饰者模式
让我们回到这篇博客的初衷：学习装饰者模式。就像我说的，这个模式允许我们不通过子类继承的方式给对象添加新功能。相反我们用另一个已经添加了功能的并有相同接口的对象来装饰（包裹）它。为了更好地理解我所说的，我们先假设有一个不了解装饰者模式，并且还是有面向对象编程背景的人。
```javascript
// 父类
var Car = function() {...};
 
// 有不同功能的子类
var CarWithPowerLocks = function() {...};
var CarWithPowerWindows = function() {...};
var CarWithPowerLocksAndPowerWindows = function() {...};
var CarWithAC = function() {...};
var CarWithACAndPowerLocks = function() {...};
var CarWithACAndPowerWindows = function() {...};
var CarWithACAndPowerLocksAndPowerWindows = function() {...};
...
```
如你所见，每一种功能都用一个新的“类(class)”代表。功能不多的情况下还好，一但功能开始增加，它就会变成你的恶梦。当然，如果你想当个土鳖，你就可以用这种方式来开发你的应用，然后把它留给别人维护，但是我不知道人家在想添加新功能（不多，也就5个）时会不会冲上来打你的脸。

## 使用装饰者模式
谢天谢地，装饰者模式能让事情变得简单，也让后面维护的哥们轻松。首先我们要创建一个基类，只是一个没有任何功能的`Car`。装饰者就是以它为蓝本来构建接口的。
```javascript
var Car = function() {
    console.log('装配(assemble)：组建车架，添加主要部件');
}
 
// 装饰者也需要实现这些方法，遵守Car的接口
Car.prototype = {
    start: function() {
        console.log('伴随着引擎的轰鸣声，车子发动了！');
    },
    drive: function() {
        console.log('走起!');
    },
    getPrice: function() {
        return 11000.00;
    }
}
```
接下来我们会创建装饰者“类”，每一个装饰者都会继承它。你会发现装饰者的每个方法只是简单地包装了一下`Car`上的同名方法。在下面的例子里，只有`assemble`（译注：指的是构造函数里log打印的内容，也就是构造函数）和`getPrice`被重写了。
```javascript
// 你需要传递一个Car（或者是CarDecorator）才能为它添加功能。
var CarDecorator = function(car) {
    this.car = car;
}
 
// CarDecorator 实现相同的接口
CarDecorator.prototype = {
    start: function() {
        this.car.start();
    },
    drive: function() {
        this.car.drive();
    },
    getPrice: function() {
        return this.car.getPrice();
    }
}
```
接下来我们要为每一个功能创建一个装饰者对象，重写父级方法，添加我们想要的功能。
```javascript
var PowerLocksDecorator = function(car) {
    // 这是JavaScript里调用父类构造函数的方式
    CarDecorator.call(this, car);
    console.log('装配：添加动力锁');
}
PowerLocksDecorator.prototype = new CarDecorator();
PowerLocksDecorator.prototype.drive = function() {
    // 你可以这么写
    this.car.drive();
    // 或者你可以调用父类的drive方法：
    // CarDecorator.prototype.drive.call(this);
    console.log('车门自动上锁');
}
 
var PowerWindowsDecorator = function(car) {
    CarDecorator.call(this, car);
    console.log('装配：添加动力表盘');
}
PowerWindowsDecorator.prototype = new CarDecorator();
 
var ACDecorator = function(car) {
    CarDecorator.call(this, car);
    console.log('装配：添加空调');
}
ACDecorator.prototype = new CarDecorator();
ACDecorator.prototype.start = function() {
    this.car.start();
    console.log('冷风吹起来');
}
```
注意我们在包装对象里总是调用同名方法。这和[组合模式][cn3]多少有些相似，但是这两种模式的相似性也仅限于此。在这个例子中，我们总是先调用装饰者的方法（如果存在这个方法的话）才去执行后面的代码。也就是说有些核心的方法要先执行，但是在另外一些程序中可能并不想按这个顺序来执行，也有可能根本不会调用包装对象的方法，比如就是想完全改变这个功能，而不是在它上面又附加功能。
![decorator_structure.png][decorator_structure]

## 看看我们的代码能做些什么
那么我们花这么时间写的代码怎么用呢？实际使用的代码在下面，但我有必要先解释一下。当然如果你认为自己已经明白了，那么就可以直接跳过这段去看代码了。
首先我们创建了一个`Car`对象，接着我们创建了装饰者，为的是给传递进构造函数里的`Car`添加我们想要的功能。装饰者的构造函数返回的对象又赋值给了之前用来保存`Car`对象的那个变量，因为装饰者使用了相同的接口，所以它们（装饰者）也可以被看做是`Car`对象。我们可以继续添加更多的功能直到我们满意为止，我们拥有了自己梦想中的汽车(car)，现在我们可以做自己想做的事了。
```javascript
var car = new Car();                    // log打印 "装配(assemble)：组建车架，添加主要部件"
 
// 给车装上动力表盘
car = new PowerWindowDecorator(car);    // log打印 "装配：添加动力表盘"
 
// 现在加装动力锁和空调
car = new PowerLocksDecorator(car);     // log打印 "装配：添加动力锁"
car = new ACDecorator(car);             // log打印 "装配：添加空调"
 
// 让我们发动这个坏小子出去兜兜风吧!
car.start(); // log打印 '伴随着引擎的轰鸣声，车子发动了！' 和 '冷风吹起来'
car.drive(); // log打印 '走起!' 和 '车门自动上锁'
```

## 总结
装饰者模式是保持对象功能差异性的一种很好的方式，从长远来看有助于提高代码的可维护性。你可能已经注意到了，我没有添加任何代码逻辑来避免我们意外添加相同功能。别担心，下一篇博客我们会给出一个不用修改任何现有代码的完美答案。在装饰者里添加检测代码有点恼人。

如果你想讨论下装饰者模式和这篇博客抑或是纯粹想讨论JavaScript，你都可以在下面留言。我很高兴能看到你的留言，就算你说我是个白痴也好（当然最好是有点建设性的，别只说一句“you're an idiot”）。无论如何我们都要成长。最后，非常感谢你能用文章下方的分享按钮把它传播出去。像我这种从后山来的从来不敢想像能有人像你这样帮我。祝编码快乐！


[decorator_structure]: http://www.codingserf.com/wp-content/uploads/2015/05/decorator_structure.png
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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-decorator/][en6]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-decorator/][cn6]

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








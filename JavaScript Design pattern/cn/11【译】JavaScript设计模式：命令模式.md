在面向对象的世界里，命令模式(the Command Pattern)是一只奇怪的野兽。和大多数对象不同，一个命令对象相当于一个动词(verb)而不是一个名词(noun)，但是命令模式又不同于函数。

## 什么是命令模式
就像我说的，一个命令对象相当于一个动词。还有一种说法是，命令模式是一种对象方法的封装方式。 简单来说，它会作为一个**方法实现对象**和一个**方法调用对象**中间的抽象层。这对用户界面来说非常有用。像往常一样，直接给出一个代码示例更直观。
假设我们要做一个闹钟的应用，就和你手机上的那种非常相似。你可以有一个闹钟列表，数量可以是从0到无穷大，不像我的手机上最多只能设置4个闹钟。为了这个应用我需要一个`Alarm`对象，它包含闹钟的状态和设置选项。现在，我们只关心几个特定方法的实现：`enable`，`disable`，`reset`和`set`。
我们为第一个方法创建一个命令对象来封闭它：
```javascript
var EnableAlarm = function(alarm) {
    this.alarm = alarm;
}
EnableAlarm.prototype.execute = function () {
    this.alarm.enable();
}
 
var DisableAlarm = function(alarm) {
    this.alarm = alarm;
}
DisableAlarm.prototype.execute = function () {
    this.alarm.disable();
}
 
var ResetAlarm = function(alarm) {
    this.alarm = alarm;
}
ResetAlarm.prototype.execute = function () {
    this.alarm.reset();
}
 
var SetAlarm = function(alarm) {
    this.alarm = alarm;
}
SetAlarm.prototype.execute = function () {
    this.alarm.set();
}
```
注意每个命令对象只关注一个接口。在这个例子里，每个接口只定义一个方法，每个方法本身只调用一个函数。在这种情况下，你可能会忽略正在做的事情，只是使用这些回调函数，本质上其实就是使用命令对象本身。在这个例子中你当然还是用的命令模式，只是你自己不知道，因为它们全都被当做回调函数来用。
现我们要使用这些命令对象了。我们把这些命令对象传进一个UI对象里，作为一个按钮添加到屏幕上，当按钮被点击时就执行刚才传入的命令对象上的`execute`方法。按钮点击时当然知道调用哪个方法，因为所有的命令对象都实现相同的接口。
```javascript
var alarms = [/* 闹钟列表 */],
    i = 0, len = alarms.length;
 
for (; i < len; i++) {
    var enable_alarm = new EnableAlarm(alarms[i]),
        disable_alarm = new DisableAlarm(alarms[i]),
        reset_alarm = new ResetAlarm(alarms[i]),
        set_alarm = new SetAlarm(alarms[i]);
     
    new Button('enable', enable_alarm);
    new Button('disable', disable_alarm);
    new Button('reset', reset_alarm);
    new Button('set', set_alarm);
}
```

## 命令模式的4个部分
命令模式由四个主要部分组成。第一个也是最明显的一个就是命令对象。从前面讲的你应该知道它是什么了。另外三个部分分别是：委托者(client)，调用者(invoker)和接受者(receiver)。委托者是指创建命令对象并把它传递给调用者的那段代码。那么上面的例子里也就是`for`循环里的代码片段。调用者是使用这个命令对象并调用它上面（那些）方法的那个对象。最后，接受者是被调用方法所在的对象，也就是上例中的`alarms`数组元素。
没有这4部分就不能称作命令模式。认识到这一点，你可能会认为我说的命令模式不同于回调函数似乎有些偏颇，对吗？我并不认为。因为我相信JavaScript足够灵活，我们可以在操作一个函数时就像在操作命令对象。我们上面发生在命令对象里的4件事（译注：指enable，disable，reset和set），都是包含在接受者里的。再没有其他的抽象层。委托者现在需要知道接受者的函数的名字，然而之前委托者并不需要知道这些，此外它需要知道是命令对象代替了它们。你损失了抽象层和模块化，但是你却获得了更容易理解和运行更快的代码。
如果你想在命令对象和回调函数之间妥协，那么请看下面这个例子：
```javascript
var makeEnableCommand = function (alarm) {
    return function() {
        alarm.enable();
    }
}
 
var makeDisableCommand = function (alarm) {
    return function() {
        alarm.disable();
    }
}
 
var makeResetCommand = function (alarm) {
    return function() {
        alarm.reset();
    }
}
 
var makeSetCommand = function (alarm) {
    return function() {
        alarm.set();
    }
}
```
因为不用创建对象再去调用方法，所以代码没有那么多。换成直接创建方法，再把它当回调函数来用。这其实没什么用，除非你打算用它来做比现在更多的事。它可以作为一种增加代码安全性的手段，假设调用者的代码是第三方的，它可能添加，修改或替换接受者的属性。虽然这种可能性比较小。

## 我命令你结束
以上就是本篇的全部。我并没有提及命令模式可以用来实现撤销的操作，我只是不想让这篇文章太长，所以我也没有展示具体怎么实现。除了在使用简单的回调时命令模式显得有些局限性，在其他地方他非常好用，并且你也会因为使用了一个设计模式而感觉很棒。
最后，JavaScript设计模式系列文章都列在了下面。我非常感谢你能把它们分享出去。Happy Coding!


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-command/][en11]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-command/][cn11]

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








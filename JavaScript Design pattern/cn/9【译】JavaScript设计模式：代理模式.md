这是JavaScript设计模式系列的第9篇，今天我们主要介绍代理模式(the Proxy Pattern)。“代理”这个词可以理解成代替者，这其实也解释了什么是代理。代理对象与被代理对象有相同的接口。唯一的问题就是我们为什么要使用一个代理去代替原始对象。
在我回答这个问题之前，我想提醒各位这是一个长篇系列。你可以在本文底部访问这个系列的其他文章。学习代理模式并不依赖其他模式的知识，所以你可以等看完这篇再去看之前的文章也不迟。如果你已经从这个系列的其他文章里得到些灵感，那真应该给你加分，可惜我没什么东西奖励你。

## 为什么用代理
回到我们的问题上来——为什么要使用代理。在几种不同的情况下，代理可以派上用场：延迟一个大对象的实例化，访问远程对象，访问控制。
在讨论每种情况之前，我们先来看一个通用代理的例子，明白下什么是代理。首先我们需要创建一个`CarList`的类。然后我们创建一个代理类包装一下它。
```javascript
/*  为了清楚，我们不展示具体的实现代码 */
var CarList = function() {
    //创建
};
 
CarList.prototype = {
    getCar: function(...) {
        // 用传进来的参数创建车
    },
     
    search: function(...) {
        // 用传进来的条件查询车
    },
     
    addCar: function(...) {
        // 往数据库里添加一辆车
    },
    .
    .
    .
};
 
var CarListProxy = function() {
    this.carList = new CarList();
};
 
CarListProxy.prototype = {
    getCar: function(...) {
        return this.carList.getCar(...);
    },
     
    search: function(...) {
        return this.carList.search(...);
    },
     
    addCar: function(...) {
        return this.carList.addCar(...);
    },
    .
    .
    .
};
```

## 虚拟代理
开动大家的想象力，我们假设`CarList`有10倍之多的方法，并且大多数都庞大而复杂。我知道这种情况有点极端，但我只是想夸张一下突出重点。重点就是我们有一个庞大的对象，为了实例化它我们会占用很多的CPU。那当我们确定要使用这个对象的时候再去实例化它不是更好么？所以这就是虚拟代理的关键。通过实例化一个代理，我们可以在代理的方法被调用的时候再去实例化相关的对象。让我们把上面那个差劲的代理转换成虚拟代理。
```javascript
var CarListProxy = function() {
    // 先不初始化CarList.
    this.carList = null;
};
CarListProxy.prototype = {
    // 其他方法在任何时候都可以调用这个方法，
    // 为的是在必要的时候初始化CarList
    _init: function() {
        if (!this.carList) {
            this.carList = new CarList();
        }
    },
     
    getCar: function(...) {
        // 每个方法都会调用 _init()
        this._init();
        return this.carList.getCar(...);
    },
     
    search: function(...) {
        this._init();
        return this.carList.search(...);
    },
     
    addCar: function(...) {
        this._init();
        return this.carList.addCar(...);
    },
    .
    .
    .
}
```
当然，这可能并不是延迟初始化一个大对象的最好方法。让我们（再）想像一下，`CarList`并没有那么多的复杂方法，只是它包含的数据量比较大，比如包含现有的所有车的品牌和型号的整个数据库。在这种情况下，我们可以创建一个初始化所有数据的方法，但是只在我们需要这些数据的时候才去调用这个方法。我不是说代理模式不好，我只是想通过告诉你代理模式不是万能钥匙帮你成为一名更好的程序员。

## JavaScript的远程代理
我们之前提到的第二种使用场景是访问远程对象。这种情况对使用Java和SOAP（简单对象访问协议）更有意义，因为，大多数情况下，当我们讨论一些远程(remote)的东西时，通常是指网线的另一端，而我们访问服务器端的JavaScript的可能性并不大，尽管[Node.js](http://nodejs.org/)越来越受欢迎，这种可能性也越来越大。对于我们的JavaScript代理，我们打算以一种简便的方法用一个对象来访问web服务的API。这有点违背代理的定义，它本来是通过实现和对象相同接口的手段来代替这个对象的。但是，此时此刻我们就需要它访问远程对象。
我个人感觉这更像是[外观模式](cn4)，但是很早人们就叫它代理模式。
这次我们的`CarListProxy`没有实现任何对象的接口，而是从web服务获取数据，这个web服务我们假设它部署在www.WeSeriouslyListEverySingleVehicleModelEver.com上。
```javascript
// 真正的CarList放在服务器上，
// 因此CarListProxy是从服务器上获取数据的
var CarListProxy = function (){};
 
CaListProxy.prototype = {
    getCar: function(...) {
        // 我们还是不会展示具体的实现代码
        // 希望你不会受此影响
        ajax('http://www.weseriouslylisteverysinglevehiclemodelever.com/getCar/'
            + args);
    },
     
    search: function(...) {
        ajax('http://www.weseriouslylisteverysinglevehiclemodelever.com/search/'
            + args);
    },
     
    addCar: function(...) {
        ajax('http://www.weseriouslylisteverysinglevehiclemodelever.com/addCar/'
            + args);
    },
    .
    .
    .
}
```
这就是我要向你展示的JavaScript的远程代理的全部内容。那么最后一种情况：访问控制（对原始对象）是什么样的呢？下面就是！

## 通过代理控制JavaScript对象的访问
有很多原因导致我们不想让别人访问原始对象，在这个例子里，我们假设正在等一个特殊的日子，因为使用这段脚本的网站还没有上线。当然我也并不清楚为什么开发人员要把所有JavaScript文件放在一个“正在建设中”的页面里加载，不过和一个“假设”争论似乎没什么意义。
为了真正控制访问，我们一定要确保除了用代理，不能通过任何途径访问原始对象。所以我们会把它包裹在一个自调用匿名函数里，但是把要window对象上挂一个属性，就是我们的代理，它是外界访问这个对象的唯一通路。
```javascript
// 自调用匿名函数
(function() {
// 我们已经知道CarList的代码是什么，所以我这里就不再写了
var CarList = ...
 
var CarListProxy = function() {
    // 先不要初始化CarList.
    this.carList = null;
    this.date = new Date();
};
CarListProxy.prototype = {
    // 其他方法在任何时候都可以调用这个方法，
    // 为的是在必要的时候初始化CarList。
    // 如果时间未到，CarList也不会被初始化。
    _initIfTime: function() {
        if (this._isTime() && !this.carList) {
            this.carList = new CarList();
            return true;
        }
        return false;
    },
    // 检查是否已经到了指定日期
    _isTime() {
        return this.date > plannedReleaseDate;
    },
     
    getCar: function(...) {
        // 如果_initIfTime返回false，那getCar也会返回false，
        // 否则它会继续执行 && 操作符右边的表达式，并返回执行结果
        return this._initIfTime() && this.carList.getCar(...);
    },
     
    search: function(...) {
        return this._initIfTime() && this.carList.search(...);
    },
     
    addCar: function(...) {
        return this._initIfTime() && this.carList.addCar(...);
    },
    .
    .
    .
}
 
// 让CarListProxy变成公共的
window.CarListProxy = CarListProxy;
 
// 你也可以执行下面的语句，
// 这样别人甚至不知道他们使用的是一个代理：
window.CarList = CarListProxy;
}());
```
至此，三种不同场合的代理模式你都了解了。我个人并没有见过代理模式被大规模应用，除了可能作为一个外观(facade)用于web服务。我希望你能找到个使用它的理由，毕竟知识就是力量，不过只有你用它的时候它才是。
像往常一样，非常感谢你的留言和分享。事实上我们的JavaScript设计模式系列也快接近尾声，我也发愁接下来该写点什么。如果你有什么感兴趣的JavaScript话题请告诉我。我会考虑你的建议。也请你用下面的分享按钮分享这篇文章。最后，如果你还有点时间，你可能想要读下JavaScript设计模式系列的其他文章。它们就列在下面：


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-proxy/][en9]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-proxy/][cn9]

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








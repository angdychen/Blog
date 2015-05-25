上一篇文章，我们讨论了工场模式(the Factory Pattern)如何创建那些遵循相同接口的对象。目前为止我们讲到的工厂模式只涵盖了简单工厂，它可以通过一个单例对象来创建新功能，用单例来构造一个工厂是最简单的办法，简单工厂也因此得名。这次我会给你们展示真正的工厂模式。

## 什么是真正的工厂模式
真正的工厂模式不同于简单工厂，因为它不是使用一个独立对象来创建汽车(car)的（见[装饰者][cn6]示例），它使用的是子类(subclass)。工厂模式的官方定义是：在子类中对一个类的成员对象进行实例化。

## 汽车店工厂示例
这个例子里我们继续汽车主题，我会延用`Car`和我在[装饰者设计模式][cn6]中为它创建的装饰者的例子。但是我会增加一些车型，为的是帮助我们看到标准工厂是怎么工作的。别担心，我们其实也并没有做什么，这些车型只是`Car`的子类。为了保持代码的简洁，况且这些子类也确实没影响到什么，所以你甚至看不到这些子类的具体实现。
我们从一个汽车店开始（就叫`CarShop`吧）。我们会从汽车店里买到车，因为没人会直接去工厂买车（尽管在这个例子里我们的`CarShop`用了工厂模式）。`CarShop`并不是一个可以直接使用的对象，它本质上是一个抽象类。因为它只是实现了一些功能接口，但是它并没有被实例化，因为这些功能是留给子类去实现的。我们来看一下：
```javascript
/* 抽象 CarShop “类” */
var CarShop = function(){};
CarShop.prototype = {
    sellCar: function (type, features) {
        var car = this.manufactureCar(type, features);
         
        getMoney(); // 一个函数调用
         
        return car;
    },
    decorateCar: function (car, features) {
        /*
            给车加装新功能使用和CarFactory里相同的技术，
            请看我的上篇文章：
            http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-factory-part-1/
        */
    },
    manufactureCar: function (type, features) {
        throw new Error("manufactureCar必须由子类开实现");
    }    
};
```
看到`decorateCar`方法了吧？它和上篇文章中的`CarFactory.makeCar`是同一个方法，只不过这里是接受一个`Car`对象的参数，而不是自己去初始化一个car。注意这里定义的`manufactureCar`只是抛出一个错误，它是由子类来实现的方法，它其实就是一个工厂方法。现在我们就来建一个实现`manufactureCar`的具体的汽车店吧。
```javascript
/* 继承CarShop，同时创建工厂方法 */
var JoeCarShop = function() {};
JoeCarShop.prototype = new CarShop();
JoeCarShop.prototype.manufactureCar = function (type, features) {
    var car;
     
    // 根据用户的指定创建不同的汽车
    switch(type) {
        case 'sedan':
            car = new JoeSedanCar();
            break;
        case 'hatchback':
            car = new JoeHatchbackCar();
            break;
        case 'coupe':
        default:
            car = new JoeCoupeCar();
    }
     
    // 给汽车加装指定的功能
    return this.decorateCar(car, features);
};
```
这家店只出售Joe牌汽车，所以工厂方法和销售其他类型汽车的店是不同的，比如和下面这个就不一样，下面这家店只卖Zim牌的车。
```javascript
/* 另一个CarShop和工厂方法 */
var ZimCarShop = function() {};
ZimCarShop.prototype = new CarShop();
ZimCarShop.prototype.manufactureCar = function (type, features) {
    var car;
     
    // 根据用户的指定创建不同的汽车
    // 这些全是Zim牌的
    switch(type) {
        case 'sedan':
            car = new ZimSedanCar();
            break;
        case 'hatchback':
            car = new ZimHatchbackCar();
            break;
        case 'coupe':
        default:
            car = new ZimCoupeCar();
    }
     
    // 给汽车加装指定的功能
    return this.decorateCar(car, features);
};
```

## 你的汽车店开张了
下面你会看到怎么使用你刚刚创建的这些汽车店。从我个人来说我还是觉得简单工厂会更酷，你不妨尝试一下用简单工厂来创建你的汽车店。这么多工厂实现方式会让你看上去很专业！
```javascript
// Joe店
var shop = new JoeCarShop();
var car = shop.sellCar("sedan", ["powerlocks"]);
 
// Zim店怎么办，同样
shop = new ZimCarShop();
car = shop.sellCar("sedan", ["powerlocks"]);
 
// 不同的店决定我会拿到不同品牌的车
// 就算我们给它相同的参数
```

## 工厂模式：真正的结尾
到此我们涵盖了所有的工厂模式（这次是真的）。我希望你能学到些东西，否则我既损失自己的睡眠，你也浪费了你的时间。如果你真学到些什么，就在下面留言告诉我，或者你可以把它分享给你的朋友们。Happy Coding!


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-factory-part-2/][en8]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-factory-part-2/][cn8]

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








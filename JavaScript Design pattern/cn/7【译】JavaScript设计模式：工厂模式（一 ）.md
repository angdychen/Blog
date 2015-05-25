今天我们主要介绍工厂模式(the Factory Pattern)。工厂模式是我最喜欢的模式之一，尤其是我稍后会讲的“简单工厂”。工厂——在现实生活以及在程序世界里——都是用来创建object的（译注：作者这里一语双关，object有物品和对象两个意思，分别对应“现实生活”和“程序世界”）。通过省下所有的`new`操作符，它可以让你的代码变得干净利落。
像往常一样我，我在文章底部列出了JavaScript设计模式系列的所有文章。我认为你花点时间去读下它们还是值得的。

## 简单工厂
有两种类型的工厂：简单工厂和标准工厂。我们会从简单工厂开始，因为...它确实更简单。今天，我们不打算搞个新例子出来，我会直接用[装饰者模式][cn6]那篇文章里的例子，把它修复的更棒。如果你还不理解装饰者模式，那你真应该先[回过头去读一下上篇文章][cn6]。
那么，工厂模式会做些什么事让装饰者模式那个例子变得更好呢？如果你还记得那个例子的最终实现代码，那么当你想要一个有3种功能的车，就得用`new`操作符创建4个对象。这么做单调乏味又烦人，所以我们打算只调用一个方法就能创建出一部拥有所有功能的车。
简单工厂就像一个单例（或者是大多数语言中的静态类，然而在JavaScript里，它们本质上是一样的），它有一个或多个方法来创建或者返回对象。在下面的代码中你会看到对象是如何创建和使用的。
```javascript
var CarFactory = {
    // 用只一个方法就能制造一辆可任意组合功能的汽车
    makeCar: function (features) {
        var car = new Car();
         
        // 如果指定了功能就把功能加到car上
        if (features && features.length) {
            var i = 0,
                l = features.length;
             
            // 遍历所有的功能并添加到car上
            for (; i < l; i++) {
                var feature = features[i];
                 
                switch(feature) {
                    case 'powerwindows':
                        car = new PowerWindowsDecorator(car);
                        break;
                    case 'powerlocks':
                        car = new PowerLocksDecorator(car);
                        break;
                    case 'ac':
                        car = new ACDecorator(car);
                        break;
                }
            }
        }
         
        return car;
    }
}
 
// 调用工厂方法，传一个字符串列表进去 
// 这些字符串标识了你想这辆车拥有的功能
var myCar = CarFactory.makeCar(['powerwindows', 'ac']);
 
// 如果你只想一辆普通的老款车，那就什么参数都不用传
var myCar = CarFactory.makeCar();
```

## 让工厂模式变得更好
虽然简单工厂能让解决问题变得简单，但是，上面对装饰者模式改进的代码还是有些问题。第一个问题是，无法保证一个功能只被添加了一次。也就是说你可能对同一辆车包装了多次`PowerWindowDecorator`，而你却浑然不知。另一个问题是，如果这些功能的添加需要遵循特定的顺序的话，我们并没有一种强制的规则来约束。
我们可以用工厂模式来修复这两个问题。最让人称道的是，所有的逻辑并不是包含在Car或者是装饰者对象里的，而是只放在一个地方：工厂，从现实世界来看就应该是这样。难道你见过一辆车自己知道怎么添加功能么？或者你听说过这些功能自己知道要按什么样的顺序加到车上么？当然没有，这些都是工厂控制的。
```javascript
var CarFactory = {
    makeCar: function (features) {
        var car = new Car(),
            // 创建一个所有功能的列表，都设置成0，表示它还没有被添加
            featureList =  {
                powerwindows: 0,
                powerLocks: 0,
                ac: 0
            };
         
        // 如果指定了功能就把功能加到car上
        if (features && features.length) {
            var i = 0,
                l = features.length;
             
            // 遍历所有的功能并添加到car上
            for (; i < l; i++) {
                // 在功能列表里标记哪个功能会被添加，
                // 这样我们只获得每个功能中的一个
                featureList[features[i]] = 1;
            }
             
            // 现在我们可以按一定的顺序添加功能了
            if (featureList.powerwindows) {
                car = new PowerWindowsDecorator(car);
            }    
            if (featureList.powerlocks) {
                car = new PowerLocksDecorator(car);
            }    
            if (featureList.ac) {
                car = new ACDecorator(car);
            }
        }
         
        return car;
    }
}
 
// 现在那些粗心和程序员可以这么调用
var myCar = CarFactory.makeCar(['ac', 'ac', 'powerlocks', 'powerwindows', 'ac']);
// 而且，它只会给你的车加一个ACDecorator，并且是以正确的顺序
```
现在你明白为什么简单工厂模式是我最喜欢的模式之一了吧？它省下创建对像时多余的字节。使用工厂创建一个对象时，除工厂自身之外不需要依赖任何接口，这太傻瓜式了。

## 另一种途径
工厂模式的强大不止限于对装饰者。基本上对共享一个接口的任何对象都可以用工厂来创建，它可以帮我们从代码里剥离独立对象。你肯定知道自己从工厂接收到的是什么类型的对象，所以你唯一需要依赖的只有工厂和一个接口，而无所谓有多少不同的对象实现了那个接口。
我给你们展示一个没有装饰者的工厂例子怎么样？下面这个工厂我们假设它是一个MVC（Model View Controller）框架的一部分。工厂获取一个特定类型的model对象，并把它传回到controller。
不同的controller使用不同的model，甚至有可能同一个contoller里不同的方法使用不同的model。相比把具体的model“类”名硬编码到controller里，我们选择用工厂来获取model。使用这种方法时，如果我们想用一个新的model类（可能我们决定使用不同类型的数据库了），我们只需要在在那个工厂里做修改就行了。我不会去实现具体的细节，因为这是浪费时间。我们只展示它是怎么使用的，你自己负责想像代码的实现。下面展示的就是用工厂模式实现的一个控制器的代码。
```javascript
// 这个controller使用两种不同的model：car和cars
var CarController = {
    getCars: function () {
        var model = ModelFactory.getModel('cars');
        return model.get('all');
    },
    getCar: function (id) {
        var model = ModelFactory.getModel('car');
        return model.get(id);
    },
    createCar: function () {
        var model = ModelFactory.getModel('car');
        model.create();
        return model.getId();
    },
    deleteCars: function (carIds) {
        var model = ModelFactory.getModel('cars');
        model.delete(carIds);
    },
    .
    .
    .
}
```

## 终结这种疯狂
结束了吗？那标准工厂模式呢？我相信这篇博客会变得很长，你不觉得吗？我们周三用另一整篇文章来介绍标准工厂模式怎么样？应该又会写很多的代码。再说，在我用更多的东西填满你脑子之前，可以给你点时间消化你在这学到的知识
所以，如果你以前使用过装饰者模式，或者是其他有相同接口的一组对象，你就要考虑用工厂模式完成为这些对象的创建，这样可以避免对这些对象的过分依赖。你可以在下面给我留言，是你的留言让这里有吸引力，而不是博客里这些静止的文字。


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-factory/][en7]
中文译者：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-patterns-factory-part-1/][cn7]

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








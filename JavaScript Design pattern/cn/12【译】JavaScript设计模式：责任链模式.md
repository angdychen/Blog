这是JavaScript设计模式系列的最后一篇。今后每周一你不用再期待我发一篇新博客了。那么今天我会讨论责任链模式(the Chain of Responsibility Pattern)。这个模式会解耦一个请求的发送者(sender)和接收者(receiver)。这是通过一个对象链完成的，每一个对象本身都可以处理这个请求或将其传递到下一个对象。是不是有些困惑？继续往下看。

## 责任链结构
责任链模式有3部分：发送者(sender)，接收者(receiver)和请求(request)。发送者会创建请求。接收者是指一个或多个对象组成的对象链，这些对象会选择处理这个请求，或是把它传递下去。请求本身可以是一个对象，它封装了所有相关的数据。
一个发送者发送一个请求给对象链里的第一个接收者。发送者只知道第一个接收者对象，并不知道其他的。第一个接收者处理这个请求并且（或者）把它传给链路的下一个。在这条链路上每一个接收者只知道下一个接收者是谁。请求会一顺着链路一直传递下去直到被处理，或者没有接收者再可传递。至此，要么什么也不会发生，要么就是抛出一个错误，这取决于你想要怎么处理。

## 现实中的责任链
DOM的事件处理用了一个责任链的实现（是不是很惊奇DOM里使用了这么多模式）。一个事件一旦被触发，在它向上冒泡到不同级别的DOM时，调用每层的事件处理函数，直到到达链路的末端或者事件处理函数主动阻止冒泡。

## 责任链示例
我们今天的例子是创建一个ATM。链路由不同面值的钞票组成。当你想取钱时，机器会先吐大面值的钞票再吐小面值的。这个例子非常简单，这有助于更清晰地展示概念，不需要实现那些过于具体的代码。
![chain_of_responsibility_atm.png][chain_of_responsibility_atm]
我们从创建接收者类：`MoneyStacks`开始。通常它只是个抽象类或者是个接口，再通过继承和实现来创建多个不同的接收者。不过在本例中，每个接收者唯一的不同之处就是钞票的面值，所以我们可以通过构造函数的一个参数来设置这个数值。
```javascript
var MoneyStack = function(billSize) {
    this.billSize = billSize; // 译注：钞票面值
    this.next = null;
}
MoneyStack.prototype = {
    withdraw: function(amount) {  //译注：提现方法
        var numOfBills = Math.floor(amount / this.billSize);
         
        if (numOfBills > 0) {
            // 吐钞
            this._ejectMoney(numOfBill);
            // 减去已吐钞票
            amount = amount - (this.billSize * numOfBills);
        }
         
        // 是否还有钱没有取出，如果链路上还有别的面值的钞票，把请求传递下去
        amount > 0 && this.next && this.next.withdraw(amount);
    },
    // 设置链路上下一个面值的钞票
    setNextStack: function(stack) {
        this.next = stack;
    },
    // 私有的用来吐钞的方法
    _ejectMoney: function(numOfBills) {
        console.log(numOfBills + "张 $" + this.billSize 
            + " 已经吐钞"); 
            //译注：多少张billSize面值的钞票已经吐钞
    }
}
```
上面都是非常简单的数学运算。`withdraw`会使用链路上的对象按需吐钞并且在满足条件的情况下把请求传给下一个链路对象。
现在，我们已经造好了ATM。它的构造孙女创建了所有面值的钞票，并把它们按级别顺序排列。当有人调用了`withdraw`方法，它就会把责任(responsibility)传给钞票链路。
```javascript
var ATM = function() {
    // 创建不同面值的钱
    var stack100 = new MoneyStack(100),
        stack50 = new MoneyStack(50),
        stack20 = new MoneyStack(20),
        stack10 = new MoneyStack(10),
        stack5 = new MoneyStack(5),
        stack1 = new MoneyStack(1);
     
    // 钞票层级顺序
    stack100.setNextStack(stack50);
    stack50.setNextStack(stack20);
    stack20.setNextStack(stack10);
    stack10.setNextStack(stack5);
    stack5.setNextStack(stack1);
     
    // 把顶层钞票设置为一个属性
    this.moneyStacks = stack100;
}
 
ATM.prototype.withdraw = function(amount) {
    this.moneyStacks.withdraw(amount);
}
 
// 用法
var atm = new ATM();
atm.withdraw(186);
/* 输出:
    1张 $100 已经吐钞
    1张 $50 已经吐钞
    1张 $20 已经吐钞
    1张 $10 已经吐钞
    1张 $5 已经吐钞
    1张 $1 已经吐钞
*/
atm.withdraw(72);
/* 输出:
    1张 $50 已经吐钞
    1张 $20 已经吐钞
    2张 $1 已经吐钞
*/
```

## 结束我的责任
这就是这个设计模式的全部内容。它很简单。像[命令模式][cn11]和[观察者模式][cn10]一样，其目的是解耦发送者和接收者，但出于不同的原因会有不同的取舍。由于这种层级结构，它也有点类似[组合模式][cn3]，也可以结合组合模式让一些方法更高效。
JavaScript设计模式之旅因你而变得有趣。我希望你一路走来已经学到些东西。如果你还没读其他的文章，我非常推荐你读一下，下面就是文章列表。记住，不能因为你了解了一个械就觉得应该在你手头的项目里用起来（古话说得好“当你手上有一把锤子的时候，看所有的东西都是钉子”）。
如果你找到任何关于设计模式的文章，请你留言告诉我，也请你把它分享出来，帮助更多像你我这样的JavaScript程序员成长。你也可以分享下面的链接。Happy Coding!


[chain_of_responsibility_atm]: http://www.codingserf.com/wp-content/uploads/2015/05/chain_of_responsibility_atm.png
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








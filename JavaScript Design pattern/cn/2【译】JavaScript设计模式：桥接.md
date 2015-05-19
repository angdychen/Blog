在上一篇博客中我讨论了[单例设计模式][cn1]以及如何在JavaScript中使用它。这次我们围绕桥接模式（the Bridge Pattern）展开讨论，为了强调它的重要性我们把它安排在JavaScript设计模式系列的第二篇。
我读过的所有关于桥接模式的解释几乎都是对GoF（the Gang of Four）的直接引用，所以我在想要不我也这么写吧？桥接模式应该是“把抽象概念和具体实现分离开来，让这两部分可以完全独立地变化”。桥接模式在像JavaScript这种事件驱动的应用程序中相当有用。而事实上它是一个完全未被充分利用的设计模式。

## 事件监听器示例
下面的例子里我会用一些[jQuery][jq]，所以如果你不知道某个函数的功能或用法，你可以查看jQuery的[文档](http://api.jquery.com/)。
下面你将看到的这段代码是使用一个名叫`getXById`的API方法，当然这个方法的实现很糟糕。我们用一个click事件来确认是哪个元素的id会被发送。`getXById`回调本身会获取click元素的id，然后用这个获取到的id通过Ajax向服务端请求X。
```javascript
getXById = function() {
    var id = this.id;
     
    $.ajax({
        url:'/getx?id=' + id,
        success: function(response) {
            console.log(response);
        }
    });
}
 
$('someElement').bind('click', getXById);
```
这段代码如果仅仅只是用在某个特定页面的特定用途倒也没什么，然而它是（据说是）一个API的一部分，所以需要做个大修改。我们要把`getXById`从事件监听器和处理Ajax结果中剥离出来：
```javascript
getXById = function(id, callback) {
    $.ajax({
        url:'/getx?id=' + id,
        success: callback
    }
}
 
getXBridge = function() {
    var id = this.id;
    getXById(this.id, function() {
        console.log(response);
    });
}
 
$('someElement').bind('click', getXBridge);
```
现在你可以在任何地方使用`getXById`并且你可以用返回的X做任何事情。

## 经典示例
我所说的“经典”有双重含义：这个例子是比较常见的面向对象编程语言的案例，并且它使用了类（译注：即classical里的class）。
JavaScript本身并没有类的概念，但是你可以模拟接口，并使用原型来模拟类。这个例子最早见于《[Head First Design Pattern](http://www.amazon.com/gp/product/0596007124/ref=as_li_ss_tl?tag=jozisjabl-20)》，那是一个Java版本。然而它其实是个无关紧要到在书的后面都没有提供代码实例的模式，所以我只能使用几张示意图（此外我还重建了这些图，因为我太聪明了）。

### 我们的首发产品
![original_remote.png][p1]
首先我们从`RemoteControl`接口开始。`ToshibaRemote`和`SonyRemote`都实现了这个接口并且工作于各自的电视。通过这些代码你可以在任意遥控器上调用`on()`，`off()`或者是`setChannel()`，尽管所有的电视都不一样，但是它们都能正常工作。当你想用桥接模式对控制器做些改进时会发生些什么呢：
![bridge_remote.png][p2]
现在这些电视继承了一个接口，其他遥控器继承了另一个接口（事实上遥控器只有一个类，因为它只需要一个实现），我们可以通过继承来创建变量，并且保持兼容性。是不是想看看代码是怎么写的？我会给你看一个新版本的桥接模式的代码 ，因为我觉得你没有必要看最原始那个版本。其实我认为你根本不需要看任何代码，但是我敢肯定有些人无论如何就是想看代码。谁让我们是程序员呢？那就给我们看代码！
```javascript
var RemoteControl = function(tv) {
    this.tv = tv;
  
    this.on = function() {
        this.tv.on();
    };
  
    this.off = function() {
        this.tv.off();
    };
  
    this.setChannel = function(ch) {
        this.tv.tuneChannel(ch);
    };
};
  
/* 更新，更好的遥控器 */
var PowerRemote = function(tv) {
    this.tv = tv;
    this.currChannel = 0;
  
    this.setChannel = function(ch) {
        this.currChannel = ch;
        this.tv.tuneChannel(ch);
    };
  
    this.nextChannel = function() {
        this.setChannel(this.currChannel + 1);
    };
  
    this.prevChannel = function() {
        this.setChannel(this.currChannel - 1);
    };
};
PowerRemote.prototype = new RemoteControl();
  
  
/** TV 接口
    因为JavaScript里没有接口的概念，
    所以我只是使用一些注释来定义应该实现些什么
  
    function on
    function off
    function tuneChannel(channel)
*/
  
/* Sony TV */
var SonyTV = function() {
    this.on = function() {
        console.log('Sony TV is on');
    };
  
    this.off = function() {
        console.log('Sony TV is off');
    };
  
    this.tuneChannel = function(ch) {
        console.log('Sony TV tuned to channel ' + ch);
    };
}
  
/* Toshiba TV */
var ToshibaTV = function() {
    this.on = function() {
        console.log('Welcome to Toshiba entertainment');
    };
  
    this.off = function() {
        console.log('Goodbye Toshiba user');
    };
  
    this.tuneChannel = function(ch) {
        console.log('Channel ' + ch + ' is set on your Toshiba television');
    };
}
  
/* 让我们看看它是如何操作的 */
var sony = new SonyTV(),
    toshiba = new ToshibaTV(),
    std_remote = new RemoteControl(sony),
    pwr_remote = new PowerRemote(toshiba);
  
std_remote.on();            // prints "Sony TV is on"
std_remote.setChannel(55);  // prints "Sony TV tuned to channel 55"
std_remote.setChannel(20);  // prints "Sony TV tuned to channel 20"
std_remote.off();           // prints "Sony TV is off"
  
pwr_remote.on();            // prints "Welcome to Toshiba entertainment"
pwr_remote.setChannel(55);  // prints "Channel 55 is set on your Toshiba television"
pwr_remote.nextChannel();   // prints "Channel 56 is set on your Toshiba television"
pwr_remote.prevChannel();   // prints "Channel 55 is set on your Toshiba television"
pwr_remote.off();           // prints "Goodbye Toshiba user"
```
上面就是JavaScript里的桥接模式。如果你还不过瘾，可以回过头去看[单例模式][cn1]。或者也可以关注这个系列的下一篇：[组合模式][cn3]。如果你认为此文对你有所帮助或者你只是纯粹地喜欢这篇文章，请你用文章下方的分享按钮把它分享出去。谢谢！


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-bridge/][en2]
中文翻译：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-bridge/][cn2]

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








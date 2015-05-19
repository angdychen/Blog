我上一篇博客继JavaScript设计模式系列的第一篇[单例][cn1]之后介绍了[桥接设计模式][cn2]。今天我们继续介绍组合模式（the Composote Parttern）。组合模式非常有用。通过“组合”这个词的定义我们就能知道组合模式是把很多部分组合起来创建一个整体。
组合模式有两个优点：
你可以像对待单个独立的对象一样对待这些对象的集合。在组合模式中，函数的执行会被分派到各个子对象去执行。这对一个大的集合非常有用（这里可能有些似是而非，因为你并不知道多大的集合才算大，所以也就无从知晓会遭受多大的性能影响）。
组合模式把对象组织成一棵树的结构，并且，因为每一个组合对象都包含一个获取其子对象（children）的方法，所以你可以隐藏具体的实现，并且随便你怎么组织这些子对象。

## 组合模式的结构
在组合模式的层级里有两类对象：叶子(leaf)和组合(composite)。下面图片展示的是一个组合结构的例子。它是递归的，这也是这个模式的强大之处。叶子对象下面不能再有任何子对象，组合对象下面一定有子对象，这就是他们的不同之处。
![composite_structure.png][p1]

## 组合模式的例子
组合模式有一些常见的例子。如果你使用过PC，你应该能常常见到这种模式的一个具体实现：文件结构。每个硬盘(disk)或驱动器(driver)以及文件夹都可以当作是组合对象，每个文件可以当作叶子对象。当你删除一个文件夹的时候，不仅仅是删除这个文件夹，也包括这个文件夹下包含的所有其他文件夹和文件。你不会找到比这个更好的例子了。
另一个比较特殊的例子是二叉树(Binary Tree)。如果你不知道它是什么的话，可以看一下[维基百科上二叉树的文章][http://en.wikipedia.org/wiki/Binary_search_tree]。它之所以特殊是因为第个节点只能最多包含两个子节点。另外，叶子和组合是完全一样的。组合可以像叶子那样表示一个最终值，叶子也可以通过添加子节点变成组合。它也是优化搜索排序数据的主要手段。
如果你看看周围，我敢肯定你能看到更多的例子。你甚至能发现用JavaScript实现的版本。

## 组合模式的JavaScript例子
我不想用上面提到的那些例子来实现一个组合模式的JavaScript版本。我想做一个图片画廊(image gallery)。实事上它和文件系统的例子很像，除了不能映射到每个图片具体的磁盘存储位置。这个例子仅仅只是有几层而已。请看下面的图片：
![gallery_structure.png][p2]
注意，在这张图里，所有的image都包含在“Gallery”这一层的组合节点下，这当然不是必须的，它只是示意一下这些图片是如何组织的。
每一个对象都需要实现一个接口，然而JavaScript里没有接口的概念，为了确保我们实现的这些方法是对的，我们把它们列了出来：

<table>
    <tr><th colspan="2">组合节点的方法</th></tr>
    <tr><td>add</td><td>为这个组合添加一个子节点</td></tr>
    <tr><td>remove</td><td>从这个组合中删除一个子节点（或者更深层级的）</td></tr>
    <tr><td>getChild</td><td>返回一个子对象</td></tr>
    <tr><th colspan="2">Gallery节点的方法</th></tr>
    <tr><td>hide</td><td>隐藏这个组合及其所有子节点</td></tr>
    <tr><td>show</td><td>显示这个组合及其所有子节点</td></tr>
    <tr><th colspan="2">工具方法</th></tr>
    <tr><td>getElement</td><td>获取节点的HTML元素</td></tr>
</table>
首先我会展示实现这些接口的GalleryCompsite类。我会用[jQuery][jq]来完成一些DOM操作。
```javascript
var GalleryComposite = function (heading, id) {
    this.children = [];
     
    this.element = $('<div id="' + id + '" class="composite-gallery"></div>')
        .append('<h2>' + heading + '</h2>');
}
 
GalleryComposite.prototype = {
    add: function (child) {
        this.children.push(child);
        this.element.append(child.getElement());
    },
     
    remove: function (child) {    
        for (var node, i = 0; node = this.getChild(i); i++) {
            if (node == child) {
                this.children.splice(i, 1);
                this.element.detach(child.getElement());
                return true;
            }
             
            if (node.remove(child)) {
                return true;
            }
        }
         
        return false;
    },
     
    getChild: function (i) {
        return this.children[i];
    },
     
    hide: function () {
        for (var node, i = 0; node = this.getChild(i); i++) {
            node.hide();
        }
         
        this.element.hide(0);
    },
     
    show: function () {    
        for (var node, i = 0; node = this.getChild(i); i++) {
            node.show();
        }
         
        this.element.show(0);
    },
     
    getElement: function () {
        return this.element;
    }
}
```
接下来我们会用GalleryImage类实现image节点：
```javascript
var GalleryImage = function (src, id) {
    this.children = [];
     
    this.element = $('<img />')
        .attr('id', id)
        .attr('src', src);
}
 
GalleryImage.prototype = {
    // 由于这是叶子节点，所以用不着这些方法,
    // 但是必须实现他们，就当作是实现Composite接口
    add: function () { },
     
    remove: function () { },
     
    getChild: function () { },
     
    hide: function () {
        this.element.hide(0);
    },
     
    show: function () {    
        this.element.show(0);
    },
     
    getElement: function () {
        return this.element;
    }
}
```
注意，GalleryImage类并没有在add，remove和getChild函数里做任何事情。因为它是个叶子类，它不包含任何子节点，所以它不需要在这些方法里做任何事。但是我们需要罗列这些方法，hhrr是遵守我们之前定下的接口。毕竟，组合对象并不知道自己是不是叶子，所以有可能尝试调用这些方法。
现在我们创建好了这些类，接下来我们用它们写点JavaScript代码创建一个真正的图片画廊。这个例子只展示了上面三个方法的使用，你可以自己扩展代码可以让它能创建一个动态的画廊，让用户可以轻松地添加和删除图片和文件夹。
```javascript
var container = new GalleryComposite('', 'allgalleries');
var gallery1 = new GalleryComposite('Gallery 1', 'gallery1');
var gallery2 = new GalleryComposite('Gallery 2', 'gallery2');
var image1 = new GalleryImage('image1.jpg', 'img1');
var image2 = new GalleryImage('image2.jpg', 'img2');
var image3 = new GalleryImage('image3.jpg', 'img3');
var image4 = new GalleryImage('image4.jpg', 'img4');
 
gallery1.add(image1);
gallery1.add(image2);
 
gallery2.add(image3);
gallery2.add(image4);
 
container.add(gallery1);
container.add(gallery2);
 
// 确保把container添加到了body下面，
// 否则它永远不会显示.
container.getElement().appendTo('body');
container.show();
```
你可以在[组合样式Demo页][http://www.joezimjs.com/composite-pattern-demo-page/]看到代码的运行效果。你可以查看页面的源代码，你会发现我们并没有提前用HTML创建任何图片，我们是用JavaScript代码添加的DOM。

## 组合模式的优缺点
优点就是，只调用一个顶层对象的方法就可以影响到组合结构里的所有节点，这招太神奇了。代码变得易用了。此外，因为组合里的对象实现了相同的接口，所以它们之间是松耦合的。最后一点，组合为对象提供了一种更好的结构，要比把对象放入分散的变量或者是一个数组里好得多。
然而，正如我们前面提到的，组合模式有点似是而非。当一个组合变得非常大时，你可以轻而易举地只调用单个方法，然而你可能并不了解这么做会带来多大的性能损耗。
组合模式是一个强大的工具，但是，就像人们常说的“能力越大责任越大”。明智地使用JavaScript的组合模式能让你向JavaScript大师靠近一步。

如果你认为此文对你有所帮助或者你只是纯粹地喜欢这篇文章，请你用文章下方的分享按钮把它分享出去。谢谢！


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
英文原文：[http://www.joezimjs.com/javascript/javascript-design-patterns-composite/][en3]
中文翻译：[David @CodingSerf](http://www.codingserf.com)
中文译文：[http://www.codingserf.com/index.php/2015/05/javascript-design-partterns-composite/][cn3]

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








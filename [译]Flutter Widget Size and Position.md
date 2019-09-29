#### [译]Flutter: Widget Size and Position

> **[原文链接](https://medium.com/@diegoveloper/flutter-widget-size-and-position-b0a9ffed9407)** 
> **</br>作者: Diego Velasquez**
> **</br>喜欢理由：可以更好的调试布局问题**

![](D:\f\github\Writing\imgs\[译]Flutter Widget Size and Position\1.png)



我遇到过很多人问，如何在屏幕上获取Widgets 的 size 和 position 的问题。在某些情况下，我们会发现我们总是会有一些原因需要去实现这个功能。但是 Widget本身是没有 size 和 position。为了实现这一点，我们必须获得与Widget上下文相关联的RenderBox。

> How do we do this?

我们可以创建一个demo，在Column中画3个不同颜色的面板，分别是红、紫、绿，再在底部写两个按钮分别是 Get Sizes, Get Position.</br></br>
demo 代码片段
```Dart
   _getSizes() {
   }

   _getPositions(){
   }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
      ),
      body: Column(
        children: <Widget>[
          Flexible(
            flex: 2,
            child: Container(
              color: Colors.red,
            ),
          ),
          Flexible(
            flex: 1,
            child: Container(
              color: Colors.purple,
            ),
          ),
          Flexible(
            flex: 3,
            child: Container(
              color: Colors.green,
            ),
          ),
          Spacer(),
          Padding(
            padding: const EdgeInsets.only(bottom: 8.0),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceAround,
              crossAxisAlignment: CrossAxisAlignment.center,
              children: <Widget>[
                MaterialButton(
                  elevation: 5.0,
                  padding: EdgeInsets.all(15.0),
                  color: Colors.grey,
                  child: Text("Get Sizes"),
                  onPressed: _getSizes,
                ),
                MaterialButton(
                  elevation: 5.0,
                  color: Colors.grey,
                  padding: EdgeInsets.all(15.0),
                  child: Text("Get Positions"),
                  onPressed: _getPositions,
                )
              ],
            ),
          )
        ],
      ),
    );
  }
```
效果图：
![](D:\f\github\Writing\imgs\[译]Flutter Widget Size and Position\2.png)
现在的问题是：我如何得到每个面板的 size 和 position？</br></br>
让我们先关注上面的红面板，当我们知道如何获取它的 size 和 position，那么对于其他面板也就迎刃而解了。

#### 获取 Widget 的 Size
为了实现这个，我们需要让我们的Widget有一个Key，所以我们要创建一个GlobalKey并赋值给我们的Widget
```Dart
//creating Key for red panel
GlobalKey _keyRed = GlobalKey();
...
//set key
    Flexible(
             flex: 2,
             child: Container(
               key: _keyRed,
               color: Colors.red,
             ),
     ),
```
一旦我们Widget 有了 Key，我们就能通过下面的方式根据Key去获取size。
```Dart
_getSizes() {
    final RenderBox renderBoxRed = _keyRed.currentContext.findRenderObject();
    final sizeRed = renderBoxRed.size;
    print("SIZE of Red: $sizeRed");
 }
```
如果我们点击 Get Sizes 按钮，在控制台将显示
```Dart
flutter: SIZE of Red: Size(375.0, 152.9)
```
现在我们知道 红色面板的宽高分别是 375.0 152.9</br></br>
很简单对吧，让我们继续获取Widget所处的位置信息
#### 获取Widget的位置
与之前的方式一样，我们的Widget必须有一个Key值，我们更新一下获取Widget位置的方法，来获取Widget 相对于定义位置左上角的位置（在这个例子中，我们使用 0.0来表示当前屏幕的左上角）
```Dart
_getPositions() {
    final RenderBox renderBoxRed = _keyRed.currentContext.findRenderObject();
    final positionRed = renderBoxRed.localToGlobal(Offset.zero);
    print("POSITION of Red: $positionRed ");
  }
```
如果点击 Get Positions 按钮，将在控制台打印出：
```Dart
flutter: POSITION of Red: Offset(0.0, 76.0)
```
这说明我们的Widget 在X轴上是0.0 在Y轴上是76.0（在屏幕的左上角）</br></br>
为什么是76.0？那是因为在这个例子中上面有个高度为76.0的AppBar

![](D:\f\github\Writing\imgs\[译]Flutter Widget Size and Position\3.png)
哇~ 到这里，我们已经知道如何去获取Widget的 size 和 position</br></br>
But~ 如果我想在开始的时候就得到 size 和 position，而不是通过按钮的的点击获取，该如何？

![](D:\f\github\Writing\imgs\[译]Flutter Widget Size and Position\4.png)
那么试试在构造函数中调用我们的方法

```Dart
_MainSizeAndPositionState(){
     _getSizes();
     _getPositions();
   }
```
运行一下demo，会出现错误信息
```
flutter: The following NoSuchMethodError was thrown building Builder:
flutter: The method 'findRenderObject' was called on null.
flutter: Receiver: null
flutter: Tried calling: findRenderObject()
```
那么我们试试从InitState中调用会怎么样
```Dart
@override
  void initState() {
    _getSizes();
    _getPositions();
    super.initState();
  }
```
运行一下demo，与刚才有点不一样，但还是报错
```
flutter: Another exception was thrown: NoSuchMethodError: The method 'findRenderObject' was called on null.
```
那我们要在开始的时候获取大小和位置该如何去做呢？

![](D:\f\github\Writing\imgs\[译]Flutter Widget Size and Position\5.png)
上面报错的原因是因为我们的state还没有得到相应的context。

> 以下网址可以找到更多的信息关于Widget的生命周期  [https://medium.com/flutter-community/widget-state-buildcontext-inheritedwidget-898d671b7956](https://medium.com/flutter-community/widget-state-buildcontext-inheritedwidget-898d671b7956)，掘金上有一篇翻译的，但是作者与来源不一样，内容差不多，可以参考[https://juejin.im/post/5c768ad2f265da2dce1f535c](https://juejin.im/post/5c768ad2f265da2dce1f535c)

因此我们必须等待Widget完成渲染后，再执行。但是该怎么做？</br></br>
下面有个简单的实现方式：
```Dart
@override
  void initState() {
    WidgetsBinding.instance.addPostFrameCallback(_afterLayout);
    super.initState();
  }
 _afterLayout(_) {
    _getSizes();
    _getPositions();
  }
```
这样就可以确保布局渲染完成后去调用你的方法</br>
再运行一次demo，将会得到：
```
flutter: SIZE of Red: Size(375.0, 152.9)
flutter: POSITION of Red: Offset(0.0, 76.0)
```
最后！！！</br></br>
你也可以回顾一下我朋友**Simon Lightfoot** 创建的package [https://pub.dartlang.org/packages/after_layout](https://pub.dartlang.org/packages/after_layout)</br>
#### 总结
很多次我们都会把简单的事情搞得非常复杂。所以仔细阅读Flutter提供的官方文档其实很有必要，毕竟我们每天都在学习新的东西。

你可以在我的 flutter-samples repo中查看我的源代码
[https://github.com/diegoveloper/flutter-samples](https://github.com/diegoveloper/flutter-samples)

#### 参考文章
- [https://docs.flutter.io/flutter/rendering/RenderBox-class.html](https://docs.flutter.io/flutter/rendering/RenderBox-class.html)
- [https://medium.com/flutter-community/widget-state-buildcontext-inheritedwidget-898d671b7956](https://medium.com/flutter-community/widget-state-buildcontext-inheritedwidget-898d671b7956)
- [https://pub.dartlang.org/packages/after_layout](https://pub.dartlang.org/packages/after_layout)

#### 译者语
文章中有些链接需要~~你懂的~

这是我第一篇译文，全靠个人的语感翻译的，欢迎指正~~~
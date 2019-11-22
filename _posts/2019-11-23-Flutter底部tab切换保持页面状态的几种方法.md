---
layout: post
title: 'Flutter底部tab切换保持页面状态的几种方法'
date: 2019-11-23
author: 李童
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Flutter
---

# Flutter底部tab切换保持页面状态的几种方法

#### 参考链接
[Flutter底部tab切换保持页面状态的几种方法](https://cloud.tencent.com/developer/article/1507094)

**第一种方式：采用IndexdStack**

IndexdStack和Stack一样，都是层布局控件，可以在一个控件上面放置另一个控件，但唯一不同的是，IndexdStack在同一时刻只能显示子控件中的一个控件，通过index属性来设置显示的控件。

配置底部导航的核心代码如下：

```import 'package:flutter/material.dart';
import 'package:flutter_jdshop/pages/tabs/CategoryPage.dart';
import 'package:flutter_jdshop/pages/tabs/HomePage.dart';
import 'package:flutter_jdshop/pages/tabs/ShoppingCartPage.dart';
import 'package:flutter_jdshop/pages/tabs/UserPage.dart';

class Tabs extends StatefulWidget {
  Tabs({Key key}) : super(key: key);

  _TabsState createState() => _TabsState();
}

class _TabsState extends State<Tabs> {

  int _currentIndex = 0;//记录当前选中哪个页面

  List<Widget> _pages = [
    HomePage(),
    CategoryPage(),
    ShoppingCartPage(),
    UserPage()
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("JDShop")),
      body: this._pages[this._currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        fixedColor: Colors.red,//底部导航栏按钮选中时的颜色
        type: BottomNavigationBarType.fixed,//底部导航栏的适配，当item多的时候都展示出来
        currentIndex: this._currentIndex,
        onTap: (index){
          setState(() {
            this._currentIndex = index;
          });
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), title: Text("首页")),
          BottomNavigationBarItem(icon: Icon(Icons.category), title: Text("分类")),
          BottomNavigationBarItem(icon: Icon(Icons.shopping_cart), title: Text("购物车")),
          BottomNavigationBarItem(icon: Icon(Icons.people), title: Text("我的"))
        ],
      ),
    );
  }
}

```

此时还是不可以保持页面状态的。

这里我们将body由

`body: this._pages[this._currentIndex],`

替换成
```
body: IndexedStack(
        index: this._currentIndex,
        children: this._pages,
      ),
```


这样就能够**实现保持页面状态**了，效果如下：

我们可以看到，此时，页面的数据只在最开始进来的时候进行加载，然后就保持住这个页面的状态了，并不会每次进来都进行数据的加载刷新了。

**使用IndexedStack来保持页面状态的优点就是配置简单，但是它也有很大的缺点：IndexedStack中管理的子页面在一开始就全部一次性加载出来了，不管有没有显示出来，然后通过index属性来确定到底显示哪一个页面**。

**第二种方式：AutomaticKeepAliveClientMixin**

如果所有的页面都需要保持页面状态，那么就使用indexdStack；如果有些页面需要保持页面状态，有些页面需要进来就刷新，那么我们就需要使用AutomaticKeepAliveMixin这个类来单独控制某个页面的状态。

AutomaticKeepAliveClientMixin相比IndexdStack，配置起来要复杂一些。

AutomaticKeepAliveClientMixin结合底部BottomNavigationBar来保持页面状态的时候，其配置步骤如下：
```
import 'package:flutter/material.dart';
import 'package:flutter_jdshop/pages/tabs/CategoryPage.dart';
import 'package:flutter_jdshop/pages/tabs/HomePage.dart';
import 'package:flutter_jdshop/pages/tabs/ShoppingCartPage.dart';
import 'package:flutter_jdshop/pages/tabs/UserPage.dart';

class Tabs extends StatefulWidget {
  Tabs({Key key}) : super(key: key);

  _TabsState createState() => _TabsState();
}

class _TabsState extends State<Tabs> {

  int _currentIndex = 0;//记录当前选中哪个页面

  //第1步，声明PageController
  PageController _pageController;

  @override
  void initState() { 
    super.initState();
    //第2步，初始化PageController
    this._pageController = PageController(initialPage: this._currentIndex);
  }

  List<Widget> _pages = [
    HomePage(),
    CategoryPage(),
    ShoppingCartPage(),
    UserPage()
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("JDShop")),
      //第3步，将body设置成PageView,并配置PageView的controller属性
      body: PageView(
        controller: this._pageController,
        children: this._pages,
      ),
      bottomNavigationBar: BottomNavigationBar(
        fixedColor: Colors.red,//底部导航栏按钮选中时的颜色
        type: BottomNavigationBarType.fixed,//底部导航栏的适配，当item多的时候都展示出来
        currentIndex: this._currentIndex,
        onTap: (index){
          setState(() {
            //第4步，设置点击底部Tab的时候的页面跳转
            this._currentIndex = index;
            this._pageController.jumpToPage(this._currentIndex);
          });
        },
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), title: Text("首页")),
          BottomNavigationBarItem(icon: Icon(Icons.category), title: Text("分类")),
          BottomNavigationBarItem(icon: Icon(Icons.shopping_cart), title: Text("购物车")),
          BottomNavigationBarItem(icon: Icon(Icons.people), title: Text("我的"))
        ],
      ),
    );
  }
}
```

以上前4步都是在tabs.dart中进行配置的，此时所有的页面还是不可保持页面状态的。然后**第5步就是在需要保持页面状态的页面里面混入AutomaticKeepAliveClientMixin类，并将wantKeepAlive方法返回为true**，如下所示：
```
//首页页面
class _HomePageState extends State<HomePage> with AutomaticKeepAliveClientMixin{

  @override
  bool get wantKeepAlive => true;

  //分类页面
  class _CategoryPageState extends State<CategoryPage> with AutomaticKeepAliveClientMixin{

  @override
  bool get wantKeepAlive => true;
```

这样，**首页页面和分类页面就实现了页面状态的保持，页面数据只在首次进入该页面的时候进行刷新；而其他没有实现页面保持的页面在每次进入该页面的时候，数据都会刷新**。
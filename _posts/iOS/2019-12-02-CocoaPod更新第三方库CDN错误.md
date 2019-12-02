---
layout: post
title: 'CocoaPod更新第三方库CDN错误'
date: 2019-12-02
author: 李童
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: iOS
---

#### 解决办法：

##### 1. podfile文件中指定source源为master：

```
source 'https://github.com/CocoaPods/Specs.git'
```

##### 2.执行`pod repo remove trunk`移除trunk源

执行完后，`pod search`就都正常了！

#### 参考链接

[CDN: trunk Repo update failed](https://www.jianshu.com/p/bf1cbe49cb5d)


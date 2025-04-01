---
title: 'CSS flex 布局实现最后一行居左对齐'
date: 2025-04-01
description: "CSS flex 布局中，如果使用 `justify-content: space-between` 两端对齐，最后一行元素不满的情况下会出现垂直布局方向混乱，本文介绍了几种解决这种问题的方法。"
tags: ["CSS"]
---

在 CSS flex 布局中，要实现元素水平方向上的平均布局，通常我们会使用 `justify-content: space-between` 实现两端对齐，但如果最后一行元素不满，就会出现下面这种问题。

如下代码：

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <div class="item-wrapper">
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
  </div>
</body>
</html>
```

```CSS
.item-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.item {
  height: 50px;
  width: 50px;
  background: #51C3F1;
  margin: 10px;
}
```
可以看到最后一行只有四个元素，垂直方向没有对齐。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202504012049794.png)


那么怎么去处理这种尴尬的情况呢，下面有几种解决办法。

## 当列数固定，可以模拟 space-between

如果列数是确定的，可以通过 `margin` 来模拟 gap 间隙，比如固定4列，可以这样写：

```CSS
.item-wrapper {
  display: flex;
  flex-wrap: wrap;
}
.item {
  height: 100px;
  width: 24%;
  background: #51C3F1;
  margin-top: 10px;
}

.item:not(:nth-child(4n)) {
  margin-right: calc(4%/3);
}
```

效果如下：

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/202504012103474.png)

## 当元素间隙不确定时

当元素宽度不一，水平两端对齐时，间隙大小也不一，可以简单的撑开最后一个元素右侧的空间，下面提供两种方法：

### 1. 可以为最后一项加 `margin-right: auto`

当元素的宽度不确定，列数不确定时，元素间的间隙大小也不确定，这时可以直接给最后一个子元素加 `margin-right: auto` 属性来实现左对齐。

代码如下：

```CSS
.item-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.item {
  height: 100px;
  background: #51C3F1;
  margin: 10px;
}

.item:last-child {
  margin-right: auto;
}
```

效果如下：

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/Kapture%202025-04-01%20at%2021.38.04.gif)

### 创建伪元素并设置 `flex:auto` 或 `flex:1`

原理和上面的方法相同，给 `.item-wrapper` 加伪元素辅助左对齐，代码如下：

```CSS
.item-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.item {
  height: 100px;
  background: #51C3F1;
  margin: 10px;
}

.item-wrapper::after {
  content: '';
  flex: 1;
}
```

## 当列数不固定时

当列数不固定时，上面这些方法均不适用，可以用特殊的技巧来实现。

用足够的空白标签来占位，比如该布局最多7列，就写7个空白标签填充占位，最多10列，就写10个空白标签。

```HTML
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>JS Bin</title>
</head>
<body>
  <div class="item-wrapper">
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <div class="item"></div>
    <i></i><i></i><i></i><i></i><i></i>
  </div>
</body>
</html>
```

```CSS
.item-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.item {
  height: 100px;
  width: 100px;
  background: #51C3F1;
  margin: 10px 10px 0 0;
}

.item-wrapper > i {
  width: 100px;
  margin-right: 10px;
}
```

原理也很简单，占位的 `<i>` 元素宽度与 `item` 一致，由于 `<i>` 元素高度为0，所以不会影响垂直方向上的布局。

![](https://cyl-blog-image.oss-cn-shenzhen.aliyuncs.com/img/Kapture%202025-04-01%20at%2021.32.57.gif)
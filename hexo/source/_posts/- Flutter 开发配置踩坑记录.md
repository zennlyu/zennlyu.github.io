---
title: Flutter 开发配置踩坑记录
categories: [devissue]
tags: [devissue]
---

## Flutter 动画

| https://github.com/diegoveloper/flutter-samples |             |                       |
| ----------------------------------------------- | ----------- | --------------------- |
| ![1](1.gif)                                     | ![2](2.gif) | ![lianyi](lianyi.gif) |

使用引导高亮：https://juejin.cn/post/6894108615540473870

Dialog 组件 https://juejin.cn/post/7036915754607837192

### 实现涟漪和雷达效果

https://blog.csdn.net/mengks1987/article/details/108232372

https://cloud.tencent.com/developer/article/1883252

此动画通过 **CustomPainter** 绘制配合 **AnimationController** 动画控制实现，定义动画控制部分

```dart
class WaterRipple extends StatefulWidget {
  final int count;
  final Color color;

  const WaterRipple({Key key, this.count = 3, this.color = const Color(0xFF0080ff)}) : super(key: key);

  @override
  _WaterRippleState createState() => _WaterRippleState();
}

class _WaterRippleState extends State<WaterRipple>
    with SingleTickerProviderStateMixin {
  AnimationController _controller;

  @override
  void initState() {
    _controller =
        AnimationController(vsync: this, duration: Duration(milliseconds: 2000))
          ..repeat();
    super.initState();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return CustomPaint(
          painter: WaterRipplePainter(_controller.value,count: widget.count,color: widget.color),
        );
      },
    );
  }
}
```

**count** 和 **color** 分别代表水波纹的数量和颜色。

**WaterRipplePainter** 定义如下：

```dart
class WaterRipplePainter extends CustomPainter {
  final double progress;
  final int count;
  final Color color;

  Paint _paint = Paint()..style = PaintingStyle.fill;

  WaterRipplePainter(this.progress,
      {this.count = 3, this.color = const Color(0xFF0080ff)});

  @override
  void paint(Canvas canvas, Size size) {
    double radius = min(size.width / 2, size.height / 2);

    for (int i = count; i >= 0; i--) {
      final double opacity = (1.0 - ((i + progress) / (count + 1)));
      final Color _color = color.withOpacity(opacity);
      _paint..color = _color;

      double _radius = radius * ((i + progress) / (count + 1));

      canvas.drawCircle(
          Offset(size.width / 2, size.height / 2), _radius, _paint);
    }
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
}
```

重点是 **paint** 方法，根据动画进度计算颜色的透明度和半径。使用如下：

```dart
class WaterRipplePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
          child: Container(height: 200, width: 200, child: WaterRipple())),
    );
  }
}
```



### Flutter iOS 联调踩坑

![1](1.png)
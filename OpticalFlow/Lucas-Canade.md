# Lucas-Canade 光流

## 前向加性算法

Lucas-Canade 本质上是为了最小化目标函数（前向加性算法）：

$\sum{[I(W(x;p+\Delta p))-T(x)]}^2$

现简化目标函数为：

$\varepsilon(v_x, v_y)=\sum\sum{[T(x,y)-I(x+v_x,y+v_y)]}^2$

上式线性化的结果为：

$\varepsilon(v_x, v_y)=\sum\sum{[T(x,y)-I(x,y)-(\frac{\partial I}{\partial x},\frac{\partial I}{\partial y}) \cdot (v_x,v_y)]}^2$

上式求偏导的结果为：

$\frac{\partial \varepsilon(v_x, v_y)}{\partial v} \approx -2 (\frac{\partial I}{\partial x},\frac{\partial I}{\partial y})\sum\sum{[T(x,y)-I(x,y)-(\frac{\partial I}{\partial x},\frac{\partial I}{\partial y}) \cdot (v_x,v_y)]}$

现考虑最开始的目标函数，其线性化的结果为：

$\sum{[I(W(x;p)) + \nabla I\frac{\partial W}{\partial p} \Delta p-T(x)]}^2$

其中，$\nabla I\frac{\partial W}{\partial p} \Delta p = \frac{\partial I}{\partial W} \frac{\partial W}{\partial p} \Delta p$，对于 $\frac{\partial I}{\partial W}$ 的计算有两种方法：

1. 先对原图像 $I$ 求梯度，之后再对梯度 $\nabla I$ 进行 warp ，其过程可以表示为 $(\nabla I)(W(x;p))$；
2. 先对原图像 $I$ 进行 warp，之后再对 warp 后的结果求梯度，其过程可以表示为 $\nabla(I(W(x;p)))$

值得注意的是前面两者的结果是不一样的，这也是理解论文《Lucas-Kanade 20 Years On: A Unifying Framework》中前向加性算法、前向复合算法之间区别的关键。

## 前向复合算法

现考虑前向复合算法的目标函数：

$\sum{[I(W(W(x;0);p))-T(x)]}^2$

上式线性化的结果为：

$\sum{[I(W(x;p))+\nabla I(W) \frac{\partial W}{\partial p} \Delta p-T(x)]}^2$

其中，$\nabla I(W) \frac{\partial W}{\partial p} \Delta p = \frac{\partial I(W)}{\partial W} \frac{\partial W}{\partial p} \Delta p$

### [For English Version](https://github.com/xiaoqiangjun/Deepnest/blob/master/main/readme.md)  
---
# ![SVGNest](http://svgnest.com/github/logo2.png)  

**SVGNest**：一个基于浏览器的矢量排样工具。  

**Demo:** http://svgnest.com  

需要SVG和Webworker支持；对移动设备的警告：运行演示会占用大量CPU。  


参考文献 (PDF):
- [López-Camacho *et al.* 2013](http://www.cs.stir.ac.uk/~goc/papers/EffectiveHueristic2DAOR2013.pdf)
- [Kendall 2000](http://www.graham-kendall.com/papers/k2001.pdf)
- [E.K. Burke *et al.* 2006](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.440.379&rep=rep1&type=pdf)  

## 什么是排样？  

给定一块正方形的材料和一些待激光切割的字母：  

![letter nesting](http://svgnest.com/github/letters.png)  
  
我们希望使用尽可能少的正方形材料将所有字母打包。 如果一个正方形还不够，我们还希望减少使用的正方形数量。  

这在CNC术语中叫做**排样**，而[实现](http://www.autodesk.com/products/trunest/overview)这种功能的[软件](http://www.mynesting.com/)通常是针对[工业客户](http://www.hypertherm.com/en/Products/Automated_cutting/Nesting_software/)并且[十分昂贵](http://www.nestfab.com/pricing/)。

SVGnest是一个解决此问题的免费开源方案，使用[E.K. Burke *et al.* 2006](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.440.379&rep=rep1&type=pdf)提出的轨迹方法和全局优化的遗传算法。它适用于任意容器和凹形边缘的情况，并且可以与现有的商业软件媲美。

![non-rectangular shapes](http://svgnest.com/github/shapes.png)

它支持有孔零件排样，即可以把零件排样到其他零件的孔洞中。

![non-rectangular shapes](http://svgnest.com/github/recursion.png)

## 使用方法

确保所有零件都已转换为轮廓，并且轮廓没有重叠。 导入SVG文件，并且选择一个文件作为需排样材料的轮廓。

其他的文件会自动转换为排样的零件。


## 算法概述

尽管对于矩形排样问题有良好的[启发式方法](http://cgi.csc.liv.ac.uk/~epa/surveyhtml.html)，但是在实际情况中我们更关注不规则的图形。

排样策略包括两个部分：

- 定位策略（例如：我应该怎样放置每个零件)
- 定序策略（例如：最佳的放置顺序是什么）

### 放置零件

这里关键的概念是“临界多边形”。

给定多边形A和B，我们希望B在A周围“绕动”，使它们始终接触但不相交。

![No Fit Polygon example](http://svgnest.com/github/nfp.png)

由此产生的轨迹就是NFP。 NFP包含零件B所有可能与先前放置的零件A相接触的放置位置。 然后，我们可以使用一些启发式算法在NFP上选择一个点作为放置位置。

类似地，我们可以为零件和排样材料构造一个“内临界多边形”。 这与NFP相同，不同之处在于内临界多边形位于固定多边形内部。

当已经放置了两个或更多零件时，我们可以将先前放置的零件的NFP合并。

![No Fit Polygon example](http://svgnest.com/github/nfp2.png)

这意味着我们需要计算O(nlogn)个NFP来完成第一次排样。 虽然有一些方法可以缓解这种情况，但我们仍然采取了暴力方法，因为这种方法对优化算法具有良好的效果。

### 优化

现在我们可以放置零件了，我们需要优化放置顺序。 这是一个错误的放置顺序示例：

![Bad insertion order](http://svgnest.com/github/badnest.png)

如果大的“C”放在最后，它里面的凹入空间就不入能被利用，因为所有本来可以填充它的零件都已经被放置好了。

为了解决这个问题，我们第一次排样使用了“面积递减”的启发式方式。首先放置较大的零件，然后放置较小的零件。 这是非常直观的，因为较小的零件往往充当“沙子”以填补较大的零件所留下的空隙。

![Good insertion order](http://svgnest.com/github/goodnest.png)

尽管这个方法为我们提供了良好的开端，但我们希望探索更多的解决方案空间。 我们可以简单地将放置顺序随机化，但是使用[遗传算法](http://www.ai-junkie.com/ga/intro/gat1.html)可以做得更好。

## 评估适应度

在我们的遗传算法中，基因由零件的放置顺序和旋转构成。适应度函数遵循以下规则：

1. 减少不可放置零件的数量（由于旋转而无法放置的零件）
2. 用于排样的材料最少
3. 所有放置零件的**宽度**最小

第三条规则比较武断，因为我们也可以优化矩形边界或者最小的凹壳。但是在现实世界中，要切割的材料往往是矩形的，这些规则会导致长条形的未使用材料。

## 性能

![SVGnest comparison](http://svgnest.com/github/comparison1.png)

在都运行约5分钟后，其性能与商业软件相近。

## 参数配置

- **零件间距：** 零件之间最小的间距（例如激光切割缝隙，CNC的偏离量等）
- **曲线拟合容差：** Bezier曲线路径和圆弧的线性近似所允许的最大误差，以SVG单位或“像素”为单位。 如果曲线部分看起来略微重叠，则减小此值。
- **零件旋转：** 每个零件的可能旋转次数。例如：4次包括基本方向。较大的值可能会改善结果，但收敛速度会较慢。
- **GA种群大小：** 遗传算法的种群大小。
- **GA变异概率：** 每个基因的变异概率，数值为1-50。
- **带孔零件支持：** 启用后，允许将零件放置在其他零件的孔中。默认情况下处于关闭状态，因为它可能会占用大量资源。
- **探索凹边区域：** 启用后，以牺牲一些性能和鲁棒性为代价来解决凹边情况：

![Concave flag example](http://svgnest.com/github/concave.png)

## To-do

- ~~递归放置（将零件放置在其他零件的孔洞中）~~
- 自定义适应度函数（重心方向等）
- 单击停止按钮时取消工作线程
- 修复NFP生成中的某些极端情况

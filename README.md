# Welcome to plothelper
# 帮助你happy地画沙雕图表的R包plothelper

# 这个包可用来绘制渐变条形图、形状不随坐标系变化的（椭）圆形和矩形，或用来批量生成坐标点。
# 编写者：Jiang Wu (吴江，首都师范大学)，E-MAIL：textidea %%% sina.com （请把%%%换成@）。
# 本文只是展示一下这个包的功能，具体的参数设置和使用方法还请看详细的英文手册。

# ----------------------------------
# 华丽地提示：使用本包前必须手动library(ggplot2)，因为本包不会自动加载ggplot2 !
# ----------------------------------

# 一、使用方法
# 要保证R的版本>=3.5.0；同时要保证ggplot2是最新版本（如果不知道是什么版本请重新安装一回）。

安装：

```R
install.packages("plothelper")
library(plothelper)
library(ggplot2)
```

## 再次提示：在library本包的同时，一定要library一下ggplot2 ！

# 二、函数介绍

plothelper里的函数分成三类，第一类用于画图，第二类用于生成图形的坐标，第三类用于线性变换。

## 第一类，用于画图的函数

### gg_shading_bar用来画渐变条形图。生成的图象对可以和ggplot的其他图层叠加。但注意不要再往上加ggplot()了，也不要使用coord_fixed()。另外，gg_shading_bar跟geom_bar的区别在于前者只接受计算好的数值向量。

先放一个效果图吧。

每个条形都可以有自己的渐变色。

```R
v=c(2, 3, 4, 8, 10, 15)
r=list(
	c("lightseagreen", "orangered2"),
	c("grey", "khaki2"), 
	c("slateblue4", "red"), 
	c("olivedrab2", "orange1"), 
	c("darkorchid2", "yellow"), 
	c("forestgreen", "brown2")
)
p=gg_shading_bar(v, raster=r, smooth=40, flip=TRUE)
p=p+geom_text(aes(x=1: length(v), y=v+1, label=v), size=6)
p+scale_x_discrete(name="")+scale_y_continuous(name="number")+
	theme(
		panel.grid=element_blank(), 
		panel.background=element_blank(), 
		plot.background=element_rect(fill="#F2F1ED"), 
		axis.title=element_text(size=16, color="black"), 
		axis.text=element_text(size=16, color="black"),
		axis.ticks.y=element_blank(), 
		axis.line.x=element_line(color="black")
	)	

注意渐变方式有两种：

第一种是所有条形都使用一个渐变范围。

```R
v=c(-2, -3, 4, 8, 10, 15)
gg_shading_bar(v, flip=TRUE)
```

第二种是每个条形根据它所代表的数值的大小使用部分颜色。请注意，下图与上图使用的都是从蓝色到红色的渐变，但它们之间是有区别的。

```R
v=c(-2, -3, 4, 8, 10, 15)
gg_shading_bar(v, flip=TRUE, equal_scale=TRUE)
```

### geom_rect_cm用来画（因为以厘米为单位所以）形状不随坐标系和aspect ratio改变的矩形。

下图中，用geom_polygon画的蓝色矩形本来是正方形，但由于aspect ratio的问题看起来却是长方形，而且其形状会随窗口改变而改变；然而，用geom_rect_cm画的矩形的形状则不会发生变化。

```R
ggplot()+xlim(-0.5, 10.5)+ylim(0, 3)+
	geom_rect_cm(aes(x=1: 10, y=rep(2, 10)), fill="red", width=rep(1, 10), height=rep(1: 2, each=5))+
	geom_polygon(aes(x=c(0, 1, 1, 0), y=c(0, 0, 1, 1)), fill="blue")
```

### geom_circle_cm用来画不随坐标系aspect ratio变化的圆形。适合用来对图表的某个区域作标注。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot(dat)+coord_fixed()+xlim(0, 11)+ylim(1, 9)+
  geom_circle_cm(aes(x=x, y=y, rcm=R))
```

即使使用极坐标，形状也不会改变。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot() + coord_polar()+xlim(0, 11)+ylim(1, 9)+theme_void()+theme(plot.background=element_rect(fill="black"))+
  geom_circle_cm(data=dat, show.legend=FALSE, aes(x=x, y=y, rcm=R, alpha=x), fill="red")+
  geom_line(aes(x=c(0, 11), y=c(8, 8)), color="grey")
```

### geom_ellipse_cm用来画形状不随坐标系和aspect ratio改变的椭圆形。需要注意的是：需要用参数rcm来调整大小，用ab来调整圆被压扁的程度，但这两个数都无法严格指定椭圆的长半径和短半径（也就是说，无法像a=2, b=1这样指定半径）。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot(dat) + coord_fixed()+xlim(0, 11)+ylim(1, 9)+
  geom_ellipse_cm(aes(x=x, y=y, rcm=R, linetype=factor(x)), fill="red", alpha=0.5, size=2)
ggplot(dat) + coord_fixed()+xlim(0, 11)+ylim(1, 9)+
  geom_ellipse_cm(aes(x=x, y=y, rcm=R), ab=2)  
```

## 第二类函数：批量生成矩形、椭圆形的坐标，但并不画图；而生成的坐标适于用geom_polygon或geom_path等来画图。

### ellipsexy生成椭圆形和圆形（用a和b控制长半径和短半径，用angle控制旋转角度）。

```R
dat1=ellipsexy(
	x=1, y=1, a=seq(1, 4, length.out=8), angle=seq(0, pi, length.out=8), 
	xytype="middleleft", n=30, todf=TRUE
)		
ggplot()+
	coord_fixed()+
	geom_polygon(show.legend=FALSE, data=dat1, aes(x=x, y=y, group=g, fill=factor(g)), alpha=0.3)
```

### rectxy用来生成矩形的坐标（用a和b控制宽和高，用angle控制旋转角度）。

```R
dat1=rectxy(x=1: 5, y=1: 5, a=2, b=1, angle=seq(0, pi, length.out=5), xytype="middle", todf=TRUE)
ggplot()+coord_fixed()+
	geom_polygon(data=dat1, aes(x, y, group=g, fill=factor(g)), alpha=0.5)
```

### ANYxy，给出能够画出图形（任何图形！）的函数，ANYxy就能批量生成坐标，所以特别适合画各种沙雕图形。

举个例子，以下函数x_square给出了画抛物线y=A*(x^2)+B的方式，然后把它套进ANYxy，就可以批量生成曲线了。

```
x_square=function(start, end, A, B){
	x=seq(start, end, 0.1)
	data.frame(x=x, y=A*(x^2)+B)
}
dat=ANYxy(myfun=x_square, 
	start=-1, end=1, A=c(1, 2, 3, 4), MoreArgs=list(B=1), 
	group=TRUE, todf=TRUE)
ggplot(dat)+geom_polygon(show.legend=FALSE, aes(x, y, group=g, fill=factor(g)), alpha=0.3)
```

## 第三类函数用来做线性变换，包括：对称、旋转、拉伸。

### ABCxy，关于Ax+By+C=0对称；也可给出过这条线的两个点p1、p2。

```R
dat1=data.frame(x=c(0, 2, 2, 0), y=c(0, 0, 1, 1)) # 原图形
# 假设对称轴是y=-x+3，转化后是-x-y+3=0
dat2=ABCxy(dat1, -1, -1, 3) # 对称的图形
ggplot()+
	coord_fixed()+
	geom_polygon(data=dat1, aes(x=x, y=y), fill="red")+
	geom_polygon(data=dat2, aes(x=x, y=y), fill="blue")+
	geom_abline(intercept=3, slope=-1)	
```

### rotatexy，用来旋转。

```R
dat1=data.frame(x=c(0, 5, 5, 0), y=c(-0.5, -0.5, 0.5, 0.5))
dat2=rotatexy(dat1, angle=seq(0, pi, length.out=8), xmiddle=0, ymiddle=0, todf=TRUE)	
ggplot()+
	coord_fixed()+
	geom_polygon(show.legend=FALSE, data=dat2, aes(x=x, y=y, group=g, fill=factor(g)), alpha=0.2)	
```

### stretchxy，用来拉伸，就不放图了。

# 以上介绍了plothelper包的主要功能。祝大家玩儿得happy。

# END


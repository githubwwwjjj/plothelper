
<img width="430" height="320" src="https://github.com/githubwwwjjj/plothelper/blob/master/w5.PNG">

<img width="417" height="285" src="https://github.com/githubwwwjjj/plothelper/blob/master/bar_spiderman2.PNG">

# Welcome to plothelper

# 帮助你happy地画渐变条形图或其他沙雕图表的R包plothelper

# 2019-11-17更新0.1.5版，增加了多个函数，包括：
### - geom_multi_raster（一次画多个raster）
### - geom_shading_bar（画渐变条形图的图层）
### - get_click_color（通过鼠标点击取色）
### - image_col_numeric（根据灰度着色）
### - image_keep_color（保留个别颜色并把其他部分变为黑白）
### - image_modify_hsv和image_modify_rgb（调整H、S、V、R、G、B通道，特别是用内置的S曲线和C曲线进行调整）
# 下文有对这些函数的介绍。

# 2019-08-02更新0.1.4版，解决了annotation_transparent_text和annotation_shading_polygon可能生成过大图片的问题。

# 2019-06-24更新0.1.3版，新加annotation_transparent_text函数用来加透明文字，annotation_shading_polygon用来画不规则渐变多边形，这就弥补了annotation_raster只能画渐变矩形的不足。具体用法见下边的例子。

# 这个包可用来绘制渐变条形图、形状不随坐标系变化的（椭）圆形和矩形，或用来批量生成坐标点。
# 编写者：Jiang Wu (吴江，首都师范大学)，E-MAIL：textidea %%% sina.com （请把%%%换成@）。
# 本文只是展示一下这个包的功能，具体的参数设置和使用方法还请看详细的英文手册：http://mirrors.ustc.edu.cn/CRAN/web/packages/plothelper/plothelper.pdf

# 一、使用方法
# 要保证R的版本>=3.5.0；同时要保证ggplot2是最新版本（如果不知道是什么版本请重新安装一回）。

安装：

```R
install.packages("plothelper")
library(plothelper)
library(magick)
library(tibble)
```

# 二、函数介绍

plothelper里的函数分成四类，第一类用于画图，第二类用于生成图形的坐标，第三类用于线性变换，第四类用于处理图片（也就是那些image_*的函数）。

## 第一类，用于画图的函数

### geom_shading_bar是一个像geom_point一样的图层，可以代替gg_shading_bar。两者都是用来画渐变条形图的，但前者既然是图层，那显然更加灵活，推荐使用。而gg_shading_bar生成的图也可以和ggplot的其他图层叠加。但注意不要再往上加ggplot()了，也不要使用coord_fixed()。另外，gg_shading_bar跟geom_bar的区别在于前者只接受计算好的数值向量。

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

# 写法一
p=gg_shading_bar(v, raster=r, smooth=40, flip=TRUE)

# 写法二
tib=tibble(x=paste("x", 1: 6, sep=""), v, r)
p=ggplot(tib)+coord_flip()+
	geom_shading_bar(aes(x=x, y=v, raster=r), flip=TRUE)

# 不管用以上哪种写法，都可以往上加东西
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
```

<img width="500" height="400" src="https://github.com/githubwwwjjj/plothelper/blob/master/gg_shading_bar%201.PNG">

注意渐变方式有两种：

第一种是所有条形都使用一个渐变范围。

```R
x=paste("x", 1: 6, sep="")
v=c(-2, -3, 4, 8, 10, 15)
ggplot()+coord_flip()+
	geom_shading_bar(aes(x,v,raster=list(c("blue","red"))), width=0.8, flip=TRUE)
```

<img width="500" height="300" src="https://github.com/githubwwwjjj/plothelper/blob/master/gg_shading_bar%202.PNG">

第二种是每个条形根据它所代表的数值的大小使用部分颜色。请注意，下图与上图使用的都是从蓝色到红色的渐变，但条形的渐变方式是不同的。

```R
ggplot()+coord_flip()+
	geom_shading_bar(aes(x,v,raster=list(c("blue","red"))), width=0.8, flip=TRUE, equal_scale=TRUE)
```

<img width="500" height="300" src="https://github.com/githubwwwjjj/plothelper/blob/master/gg_shading_bar%203.PNG">

### geom_multi_raster可以一次把多个raster画上，也就是说，可以代替annotation_raster

```R
r1=image_read("###图片kula.png") # 从https://github.com/githubwwwjjj/plothelper/blob/master/kula.png下载图片
r2=matrix(c("red", "green", "blue", "purple"), nrow=2)
r3=matrix(rainbow(7))
r=list(r1, r2, r3)
tib=tibble::tibble(xmin=c(1, 3, 5), xmax=c(2, 4, 8), ymin=c(0, 2, 3), ymax=c(3, 4, 6), r)
ggplot(tib)+
	geom_multi_raster(aes(xmin=xmin, xmax=xmax, ymin=ymin, ymax=ymax, raster=r))
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/geom_multi_raster.PNG)

### geom_rect_cm用来画（因为以厘米为单位所以）形状不随坐标系和aspect ratio改变的矩形。

下图中，用geom_polygon画的蓝色矩形本来是正方形，但由于aspect ratio的问题看起来却是长方形，而且其形状会随窗口改变而改变；然而，用geom_rect_cm画的矩形的形状则不会发生变化。

```R
ggplot()+xlim(-0.5, 10.5)+ylim(0, 3)+
	geom_rect_cm(aes(x=1: 10, y=rep(2, 10)), fill="red", width=rep(1, 10), height=rep(1: 2, each=5))+
	geom_polygon(aes(x=c(0, 1, 1, 0), y=c(0, 0, 1, 1)), fill="blue")
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/geom_rect_cm.PNG)

### geom_circle_cm用来画不随坐标系aspect ratio变化的圆形。适合用来对图表的某个区域作标注。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot(dat)+coord_fixed()+xlim(0, 11)+ylim(1, 9)+
  geom_circle_cm(aes(x=x, y=y, rcm=R))
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/geom_circle_cm%201.PNG)

即使使用极坐标，形状也不会改变。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot() + coord_polar()+xlim(0, 11)+ylim(1, 9)+theme_void()+theme(plot.background=element_rect(fill="black"))+
  geom_circle_cm(data=dat, show.legend=FALSE, aes(x=x, y=y, rcm=R, alpha=x), fill="red")+
  geom_line(aes(x=c(0, 11), y=c(8, 8)), color="grey")
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/geom_circle_cm%202.PNG)

### geom_ellipse_cm用来画形状不随坐标系和aspect ratio改变的椭圆形。要注意的是：需要用参数rcm来调整大小，用ab来调整圆被压扁的程度，但这两个数都无法严格指定椭圆的长半径和短半径（也就是说，无法像a=2, b=1这样指定半径）。

```R
dat=data.frame(x=1: 10, y=rep(5, 10), R=rep(c(0.5, 1), 5))
ggplot(dat) + coord_fixed()+xlim(0, 11)+ylim(1, 9)+
  geom_ellipse_cm(aes(x=x, y=y, rcm=R), ab=2)  
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/geom_ellipse.cm.PNG)

### textgif用来生成小广告风格的文字gif。我自己用这个做表情包。

```R
mytext=c("佩服三连", "厉害厉害", "可以可以", "666")
color1=c("khaki", "orange", "orangered", "orangered1")
color2=rep("black", 4)
g=textgif(mytext, text_color=color1, bg_color=color2, fps=2, width=259, height=129) 
# width和height根据需要随便调
# mgaick::image_write(g, "f:/GGG.gif", format="gif") # 假设要以"f:/GGG.gif"为文件名保存
```

<img width="400" height="200" src="https://github.com/githubwwwjjj/plothelper/blob/master/textgif.gif">

### annotation_transparent_text，透明文字，主要作用就是画那种底下的颜色能够透过来的文字，当然，放置字的那个label也可以是图片之类的东西，具体请看这个函数的文档中关于参数bg的说明。

```R
m=colorRampPalette(rainbow(7))(20)
m=matrix(m, nrow=1)
ggplot()+coord_fixed()+
	xlim(0, 7)+ylim(-2, 4)+theme_void()+
	annotation_raster(
		raster=m, 
		xmin=0, ymin=-3, 
		xmax=7, ymax=5, 
		interpolate=TRUE
	)+
	annotation_transparent_text(
		label="R\nDATA\nVISUALIZATION", 
		xmin=0, xmax=7, 
		ymin=-1, ymax=3, 
		family="sans", fontface=2, alpha=0.6, 
		place="left", expand=c(0.08, 0.02)
	)
```

<img width="400" height="410" src="https://github.com/githubwwwjjj/plothelper/blob/master/trans%20text%201.PNG">

### annotation_shading_polygon，用来画不规则渐变图形，渐变的颜色可以是一个普通的raster，也可以是图片的一部分（其实图片也是作为raster被读入的）。

```R
poly=ellipsexy(-1, 0, a=1, b=1)
m=matrix(rainbow(10), nrow=1)
ggplot()+
	coord_fixed(expand=FALSE)+
	scale_y_continuous(limits=c(-1.5, 1.5))+
	scale_x_continuous(limits=c(-3, 6), breaks=-3: 6, labels=-3: 6)+
	annotation_shading_polygon(poly, raster=m)+
	annotation_shading_polygon(poly, xmin=1, xmax=5, ymin=-1, ymax=1, raster=m)
```

<img width="400" height="160" src="https://github.com/githubwwwjjj/plothelper/blob/master/git%20example%20shade%20poly.png">


### 用图片形状截取

```R
img=image_read("###图片kula.png") # 从https://github.com/githubwwwjjj/plothelper/blob/master/kula.png下载图片
img=image_resize(img, "100x300!")
blues=matrix(c("purple", "deepskyblue1", "midnightblue", "midnightblue", "black", "yellow"))
ggplot()+xlim(-1, 1)+ylim(0, 2.5)+coord_fixed()+
	annotation_raster(img, xmin=-1, xmax=0, ymin=0, ymax=2.5)+
	annotation_shading_polygon(shape=img, xmin=0, xmax=1, ymin=0, ymax=2.5, raster=blues)
```

<img width="200" height="400" src="https://github.com/githubwwwjjj/plothelper/blob/master/git%20shade%20kula.png">

### 生成带图案的条形图（或其他什么图）

#### 本说明开头的那个带蜘蛛侠图片的条形图怎么做的呢——只需三步！

```R
# 第一步：读图片
img=magick::image_read("https://raw.githubusercontent.com/githubwwwjjj/plothelper/master/spiderman.PNG")

# 第二步：画条形图
p=ggplot()+geom_bar(aes(x=1: 5, y=1: 5), stat="identity")+coord_flip()

# 第三步：合在一起
p=ggplot()+annotation_shading_polygon(shape=p, raster=img)
p+theme_void()+theme(plot.background=element_rect(fill="grey10"))
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

![image](https://github.com/githubwwwjjj/plothelper/blob/master/ellipsexy.PNG)

### rectxy用来生成矩形的坐标（用a和b控制宽和高，用angle控制旋转角度）。

```R
dat1=rectxy(x=1: 5, y=1: 5, a=2, b=1, angle=seq(0, pi, length.out=5), xytype="middle", todf=TRUE)
ggplot()+coord_fixed()+
	geom_polygon(show.legend=FALSE, data=dat1, aes(x, y, group=g, fill=factor(g)), alpha=0.5)
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/rectxy.PNG)

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

![image](https://github.com/githubwwwjjj/plothelper/blob/master/ANY.PNG)

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

![image](https://github.com/githubwwwjjj/plothelper/blob/master/ABCxy.PNG)

### rotatexy，用来旋转。

```R
dat1=data.frame(x=c(0, 5, 5, 0), y=c(-0.5, -0.5, 0.5, 0.5))
dat2=rotatexy(dat1, angle=seq(0, pi, length.out=8), xmiddle=0, ymiddle=0, todf=TRUE)	
ggplot()+
	coord_fixed()+
	geom_polygon(show.legend=FALSE, data=dat2, aes(x=x, y=y, group=g, fill=factor(g)), alpha=0.2)	
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/rotatexy.PNG)

### stretchxy，用来拉伸，就不放图了。

## 第四类函数用来做图片处理。目前能做的处理有什么，从下边的代码和示意图中可以看到。

```R
img=image_read("###图片spiderman.png") # 从https://github.com/githubwwwjjj/plothelper/blob/master/spiderman.PNG下载图片

# 保存一种或几种颜色并把其他部分变成黑白。由color指定颜色，如果color="click"，则可在图片上点选。fuzz是要保留的颜色离指定的颜色有多近。比如，当color="red"并且fuzz=0时，只有红色会被保留下来；但若设fuzz=20，则接近红色的颜色也会被保留下来。fuzz的值域是0至100。
image_keep_color(img, color="red", fuzz=20)

# 根据灰度重新着争（实际是调用了scales::col_numeric），n是指定要保留几种颜色，最大是256。
image_col_numeric(img, palette=c("orangered", "blue", "green"), n=100)

# 调整hsv的值。本例是先把s的值rescale到一个值域中，再利用S曲线调整v的值。其他用法见英文说明。
image_modify_hsv(img, rescale_s=c(0.5, 0.8), fun_v=list("s", c1=-2, c2=2))

# 调整rgb的值。本例是用C曲线调整r的值。其他用法见英文说明书。
image_modify_rgb(img, fun_r=list("c", value=0.9))
```

![image](https://github.com/githubwwwjjj/plothelper/blob/master/image_function.png)


# 以上介绍了plothelper包的主要功能。祝大家玩儿得happy。

# END

本文地址：https://github.com/githubwwwjjj/plothelper/blob/master/README.md

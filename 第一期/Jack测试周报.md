title: 迷宫与地牢的生成（roguelike）
date: 2016-02-02 13:39:17
tags: [cocos2d-x,dungeon,roguelike]
---

## 写在前面
近日在微博上看到大大们分享了roguelike游戏的迷宫地牢生成，地址在文章最后，经过了学习后作出了一个小demo，写一篇博文来记录学习的的过程，先看成果：
![final](http://7xl9we.com1.z0.glb.clouddn.com/dungeo8.png)

demo中灰色的是走廊，网格代表着房间，红色是连接各个部分的门或者什么东西。怎么说呢，是不是有点nethack的赶脚。
这个程序大致做了这样的事情
```cpp
func dungeontest()
{
	初始化数据;
	创建房间;
	创建走廊;
	连接各部分;
	除去死路;
}
```
demo使用了cocos2dx3.3游戏引擎，语言为C++。主要的难点是要作出roguelike的随机性
## 生成房间
在生成房间之前先要考虑一下整个地牢与迷宫在数据结构中的体现。
将整个区域划分成为一个个的小格子，使用二维结构保存起来。需要注意的是横纵方向个数要保证为奇数。
接下来随机生成每一个房间。
```cpp
func createrooms()
{
	while(生成次数N>0)
	{
		随机房间左下角坐标与尺寸；
		如果不与其他存在的房间重叠:
			创建一个房间；
		N--；
	}
}
```
这个值得注意的是每个房间的长和宽必须是奇数，至于左下角的坐标要看定数据结构的起始为0或1仔细思考后再确定是奇数或是偶数。（demo中虽然从0开始但是最最外面一圈我没有使用，也就是说有效的块从1开始计数）
生成后效果：
![room](http://7xl9we.com1.z0.glb.clouddn.com/dungeo31.png)

## 生成走廊
生成走廊的过程是将创建房间完成后的剩余的块填充成迷宫。先看效果：
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeon1.png)
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeon2.png)
生成迷宫的过程可以抽象为树的生成，生成的过程为连接每一个“节点”。这些“节点”为下图中的灰格子（我实在不知道该怎么用语言描述，只能上图了）
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeo4.png)
每一条迷宫生成的过程：
将起点压入栈stack中，然后：
a.判断起点node上下左右四个方向的下一个几点是否存在。如果不存在把信息纪录为list。
b.如果a中的list不为空，则将node向list中的某一个方向移动两格并将移动后的坐标压入栈stack中。重复a。
c.如果a中的list为空，则node＝stack.pop()。
d.stack为空结束。
其中b中的方向不仅仅是在list中随机一个，还会在一定程度上取决于上次移动的方向。
如图为生成一条迷宫。
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeo5.png)

## 连接各个区域
走廊（迷宫）和房间都搞定了，我们要将它们连接起来，确保每一个区域都连通起来。这里的“区域”是指到现在生成的所有封闭的元素，可能是一段迷宫，可能是一个房间，每个区域互不连通。在之前生成每个房间和每一条迷宫的同时要将其包括的块附上不同的值用来代表他们的区域，同一个区域的块值相等。
基本的思路是找出所有能连接两个（这里最多是两个，不可能会更多）不同区域的点（判断上下左右4个块，不为空且处于的区域），然后将这个点和其能连通的区域保存起来。如下图的所有红色的块：
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeo6.png)

接着我们得到了这样的问题：有N个区域，有M个点，对于每个点p可以连接两个区域，找出一条连通所有区域的策略。
解决的思路是：
a.随机找一个点p，连通并合并两个区域。
b.删除p意外所有能连接这两个区域的点。
c.继续a，直到所有区域都连通。
ps：可以在b中删除的时候有极小的几率生成门。更具体的思路可以搜索“spanning tree”。
![corridor](http://7xl9we.com1.z0.glb.clouddn.com/dungeo7.png)

## 除去死胡同 
对于每一个走廊的块，其上下左右中有3个为空，把这个块干掉就可以了。
![final](http://7xl9we.com1.z0.glb.clouddn.com/dungeo8.png)

## 写在后面

主要参考：http://journal.stuffwithstuff.com/2014/12/21/rooms-and-mazes/
基本的思路都是学习了这位大大的。
整个的学习过程还是有一点烧脑的（主要是我太弱了）
demo源码：https://github.com/JackLN/DungeonTest
源码中没有上传cocos引擎，使用cocos新建一个cpp工程然后替换Classes文件夹即可。
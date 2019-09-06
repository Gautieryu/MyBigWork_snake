# 贪吃蛇自动寻路
###### 17241035 任天雨 


### 概述
这是一个使用C语言编写的贪吃蛇小游戏。当然，我们可以由玩家控制贪吃蛇进行游戏，但是程序的主要内容是对贪吃蛇自动寻路功能的实现。由于我有限的时间和更有限的水平，最终程序实现了在一半情况下可以通关（贪吃蛇的身体铺满整张地图）。

在9*9的地图中，设初始300分，每增长一格加100分，通关分数为8000分（最后一个食物未计分）。我统计了100次运行结果（见test_result文件）：

>6700
8000
8000
8000
8000
7900
7900
8000
6100
8000
8000
8000
8000
8000
7700
7100
6300
7900
7900
7500
8000
7900
8000
6400
5900
8000
8000
7900
7500
8000
7700
7900
7500
7900
7900
8000
8000
7700
7700
8000
8000
7700
8000
8000
8000
7900
8000
7500
5700
8000
7500
6500
8000
8000
7900
7700
7900
8000
7100
7700
7700
7700
8000
5900
5300
7700
6900
7700
8000
8000
8000
8000
6900
7500
8000
7100
7900
8000
7700
7900
8000
7900
6500
7900
8000
8000
8000
8000
7900
8000
8000
8000
7700
8000
8000
7700
8000
7500
8000
7900



平均分为7669分。这个程序还是较为成功的。 
（实际游戏计分规则有所不同，初始计分为0.）
### 游戏介绍
游戏地图为一个标准的正方形，其大小可以通过更改参数SIZE的值来改变。贪吃蛇在地图第三行出生，通过上下左右移动去吃生成在地图随机位置的食物。撞到墙体或本身会导致游戏失败。随着蛇长度的增加，蛇移动速度也会越来越快。

因为人工控制没什么亮点，就没设置模式选择。默认是自动寻路，可以在main函数中通过选择不同的dir函数可以实现小蛇控制方式的变化。

### 代码介绍

代码主要由以下几个函数组成：

>初始化  
void init(int * length, point * foodAt, int * dir, point body[], char map[][SIZE]);

>由玩家控制蛇前进方向  
int getDir(int dir); 

>最初版本的自动寻路  
int getAIDir(int dir, int length, point body[], point foodAt);  

>前进格是否不为墙和身体  
int moveable(point moveTo, int length, point body[]);

>运动函数  
int move(point foodAt, int dir, int length, point body[]);

>画图函数  
void draw(int length, point foodAt, point body[], char map[][SIZE]);

>随机生成食物  
void food(point * foodAt, point body[], int length, char map[][SIZE]); 

>两点间是否存在路径  
int exist_path(point start,point desti,int length,point body[])

>两点间最短路径  
int shortest_path(point start,point desti,point shortest_path[MAX_LENGTH],int length,point body[],char map[SIZE][SIZE])

>得到最佳路径  
int smart_snake_path(point start,point desti,point path[MAX_LENGTH],int length,point body[],char map[SIZE][SIZE],point food_position)

>按照得到的路径前进  
int get_dir_of_a_smart_snake(point start,point desti,point path[MAX_LENGTH],int length,point body[],char map[SIZE][SIZE])

当程序运行时，自动进行初始化，之后进入循环体。在循环体中调用函数得到蛇前进的正确方向、进行移动、反馈结果并进行判断。毫无疑问，本程序最重要的部分便是得到最佳路径的smart_snake_path函数。

### smart_snake_path函数介绍

对于贪吃蛇游戏，我们能够判断出：如果蛇能够到达自己的尾巴，那么蛇一定不会死亡——只要蛇沿着自己的尾巴前进，蛇头永远可以有蛇尾留下的安全空间，而不会碰到墙壁和身体。这是这个代码的基本思想。  

在smart_snake_path函数中，考虑了三种情况：  
  
**1.蛇可以到达食物，并且在吃到食物后可以到达新的蛇尾**  
条件判断与路径寻找：调用exist_path判断蛇是否可以到达食物。如果可以，令一条虚拟小蛇沿调用shortest_path函数得到的最短路径前往食物。虚拟小蛇吃到食物后长度加一，再判断此时是否可以到达蛇尾。  
最终路径：返回调用shortest_path函数得到的最短路径。

**2.在非第一种情况下（蛇吃到食物后无法到达蛇尾或蛇吃不到食物），如果蛇能够到达蛇尾**   
条件判断与：调用exist_path判断是否可以到达蛇尾，如果可以，调用shortest_path找到蛇头至蛇尾的最短路径。考虑到在特殊情况下，沿最短路径会使蛇陷入循环，因此要求蛇沿较长路径前往。


分别使用exist_path判断此时蛇头上下左右四格是否可以在蛇移动至这四格后前往新的蛇尾。如果可以，且非最短路径，则向此格移动。如果只有一条路径，则只能按照最短路径行进。  
最终路径：返回下一步将要移动前往的格子。  


在理想状况下，蛇始终按照上述两种策略行进，则蛇永远可以到达蛇尾，蛇不会死亡。但仍考虑第三种情况作为冗余：  
**3.（蛇吃到食物后无法到达蛇尾或蛇吃不到食物）且（蛇无法到达蛇尾）**  
既然沿最短路径（如果有）前往食物不安全，就前往远离食物的方向。


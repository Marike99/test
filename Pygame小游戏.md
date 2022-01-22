# 项目实战--飞机大战

### 游戏背景

#### 目标

●  背景交互滚动的思路确定

●  显示游戏背景

#### 01. 背景交互滚动的思路确定

运行 **备课代码** ，观察背景图像的显示效果：

●  游戏启动后，**背景图像** 会 **连续不断地 向下方** 移动

●  在 **视觉上**   产生英雄的飞机不断向上方飞行的 **错觉** -- 在很多跑酷类游戏中的套路

​	○   **游戏的背景** 不断变化

​	○  **游戏的主角** 位置保持不变

##### 1.1 实现思路分析

![1](images/1.png)

**解决办法**

1.创建两张背景图像精灵

​	●  第一张 **完全和屏幕重合**

​	●  第二张 **在屏幕的正上方**

2.两张图像 **一起向下方滚动**

​	●  self.rect.y += self.speed

3.当 **任意背景精灵 **的rect.y >= 屏幕的高度，说明已经 **移动到屏幕的下方**

4.将 **移动到屏幕下方的这张图像** 设置到 屏幕的正上方

​	●  rect.y = -rect.height



##### 1.2设计背景类

![2](images/2.png)



●  **初始化方法**

​	○  直接指定 **背景图片**

​        ○  is_alt 判断是否是另一张图片

​		■   Flase表示 **第一张图像** ，需要与屏幕重合

​		■  True表示 **另一张图像** ，在屏幕的正上方

**●  update()方法**

​	○  判断 **是否移动出屏幕** ，如果是，将图像设置到 **屏幕的正上方** ，从而实现 **交互滚动**

​	**继承** 如果父类提供的方法，不能满足子类的需求：

​	●  派生一个子类

 	●  在子类中针对特有的需求，重写父类方法，并且进行扩展



#### 02.显示游戏背景



##### 2.1背景精灵的基本实现

●  在plane_sprites.py 新建 Background 继承自 GameSprite

~~~
class Background(GameSprite):
    """
    游戏背景精灵
    """

    def update(self):
        # 1. 调用父类的方法实现
        super().update()

        # 2. 判断是否移出屏幕,如果移出屏幕,将图像设置到屏幕的上方
        if self.rect.y >= SCREEN_RECT.height:
            self.rect.y = -self.rect.height
~~~

##### 2.2在plane_main.py 中显示背景精灵

1.在__create_sprites 方法中创建精灵和精灵组

2.在__update_sprites 方法中，让 **精灵组** 调用update()和 draw()方法

__create_sprites 方法

~~~
    def __create_sprites(self):
        # 1. 创建背景精灵和精灵组
        bg1 = Background('./images/background.png')
        bg2 = Background('./images/background.png')
        #指定背景图像的初始位置  默认正上方
        bg2.rect.y = -bg2.rect.height
        #将创建的精灵传递到背景精灵组内部
        self.back_group = pygame.sprite.Group(bg1,bg2)
~~~

__update_sprites方法

~~~
    def __update_sprites(self):
        # update() 让组中的所有精灵更新位置
        self.back_group.update()
        # draw() 在screen上绘制所有的精灵
        self.back_group.draw(self.screen)
~~~

##### 2.3利用初始化方法，简化背景精灵的创建

思考 -- 上一小节完成的代码存在什么问题? 能否简化?

●  在主程序中，创建的**两个背景精灵，传入了相同的图像文件路径**

●  创建 **第二个背景精灵** 时，**在主程序中**，设置背景精灵的图像位置

思考 -- 精灵 **初始位置** 的设置，**应该 由主程序负责?** 还是 **由精灵自己负责?**

**答案 -- 由精灵自己负责**

●  根据面向对象设计原则，应该将对象的职责，封装到类的代码内部

●  尽量简化程序调用一方的代码调用

![3](images/3.png)

●  **初始化方法**
​	○  直接指定 **背景图片**

​	○  is_alt 判断是否是另一张图像

​		■   Flase表示 **第一张图像** ，需要与屏幕重合

​		■  True表示 **另一张图像** ，在屏幕的正上方

在plane_sprites.py 中实现 Background 的 **初始化方法**

~~~
class Background(GameSprite):
    """
    游戏背景精灵
    """

    def __init__(self,is_alt = False):
        # 1. 调用父类方法实现精灵的创建(image/rect/speed)
        super().__init__("./images/background.png")
        # 2. 判断是否是交替图像,如果是,需要设置初始位置
        if is_alt:  #如果为True
            # 指定背景图像的初始位置  默认正上方
            self.rect.y = -self.rect.height

    def update(self):
        # 1. 调用父类的方法实现
        super().update()

        # 2. 判断是否移出屏幕,如果移出屏幕,将图像设置到屏幕的上方
        if self.rect.y >= SCREEN_RECT.height:
            self.rect.y = -self.rect.height

~~~

在plane_main.py 主程序文件中调用

~~~
import pygame
#导入所有的工具
from plane_sprites import *

class PlaneGame(object):
    """
    飞机大战主游戏
    """
    def __init__(self):
        print("游戏初始化")

        # 1.创建游戏的窗口
        # self.screen = pygame.display.set_mode((480, 852))
        self.screen = pygame.display.set_mode(SCREEN_RECT.size)
        # 2.创建游戏的时钟
        self.clock = pygame.time.Clock()
        #3. 调用私有方法 完成精灵和精灵组的创建
        self.__create_sprites()

    def __create_sprites(self):
        # 1. 创建背景精灵和精灵组
        # bg1 = Background('./images/background.png')
        # bg2 = Background('./images/background.png')

        bg1 = Background()
        # True表示交替背景
        bg2 = Background(True)
        #将创建的精灵传递到背景精灵组内部
        self.back_group = pygame.sprite.Group(bg1,bg2)

    def start_game(self):
        print("游戏开始...")

        while True:
            # 1. 设置刷新帧率
            # self.clock.tick(60)
            self.clock.tick(FRAME_PRE_SEC)
            # 2. 事件监听
            self.__event_handler()
            # 3. 碰撞检测
            self.__check_collide()
            # 4. 更新/绘制精灵组
            self.__update_sprites()
            # 5. 更新显示
            pygame.display.update()

    def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()

        pass

    def __check_collide(self):
        pass

    def __update_sprites(self):
        # update() 让组中的所有精灵更新位置
        self.back_group.update()
        # draw() 在screen上绘制所有的精灵
        self.back_group.draw(self.screen)


    @staticmethod
    def __game_over():
        print("游戏结束")
        #卸载所有模块
        pygame.quit()
        #终止当前正在执行的程序
        exit()

if __name__ == '__main__':
    # 创建游戏对象
    game = PlaneGame()

    #启动游戏
    game.start_game()
~~~



### 敌机出场



#### 目标

●  使用**定时器**添加敌机

●  设计 Enemy 类

#### 01.使用定时器添加敌机

运行 **备课代码**，**观察** 敌机的 **出现规律**：

1.游戏启动后，**每隔1秒**会出现**一架敌机**

2.每架敌机 **向屏幕下方飞行**，飞行 **速度各不相同**

3.每架敌机出现的**水平位置**也各不相同

4.当敌机 **从屏幕下方飞出**，不会再飞到屏幕中



##### **1.1定时器**

●  在pygame 中可以使用 pygame.time.set_timer()来添加 **定时器**

●  所谓 **定时器**，**每隔一段时间**，去**执行一些动作**

~~~
set_timer(eventid,milliseconds) -> None
~~~

●  set_timer 可以创建一个 **事件**

●  可以在 **游戏循环**的 **事件监听**方法中捕获到该事件

●  第一个参数 **事件代号**，需要基于常量 pygame.USEREVENT 来指定

​	○  USEREVENT 是一个整数，再增加的事件可以使用 USEREVENT + 1指定，以此类推...

●  第二个参数是 **事件触发** 间隔的 **毫秒值**

##### 定时器事件的监听

●  通过 pygame.event.get() 可以获取当前时刻所有的 **事件列表**

●  **遍历列表** 并且判断 event.type 是否等于 eventid,如果相等，表示 **定时器事件** 发生



##### 1.2创建并监听创建敌机的定时器事件

pygame **定时器**  使用套路非常固定：

1.定义 **定时器常量** -- eventid

2.在 **初始化方法** 中，调用 set_timer方法设置 **定时器事件**

3.在 **游戏循环** 中，**监听定时器事件**



**1）定义事件**

●  在 place_sprites.py 的顶部定义事件常量

~~~
#创建敌机的定时器常量
CREATE_ENEMY_EVENT = pygame.USEREVENT 
~~~

●  在plane_main.py 主程序初始化方法中设置定时器事件

~~~
    def __init__(self):
        print("游戏初始化")

        # 1.创建游戏的窗口
        # self.screen = pygame.display.set_mode((480, 852))
        self.screen = pygame.display.set_mode(SCREEN_RECT.size)
        # 2.创建游戏的时钟
        self.clock = pygame.time.Clock()
        #3. 调用私有方法 完成精灵和精灵组的创建
        self.__create_sprites()
        # 4. 设置定时器事件  - 创建敌机 1s
        pygame.time.set_timer(CREATE_ENEMY_EVENT,1000)
        
        
      # 再找到处理事件监听的私有方法,添加elif判断语句  
      def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()
            elif event.type == CREATE_ENEMY_EVENT:
                print("敌机出场...")      

~~~



#### 02.设计Enemy类

1.游戏启动后，**每隔1秒**会出现**一架敌机**

2.每架敌机 **向屏幕下方飞行**，飞行 **速度各不相同**

3.每架敌机出现的**水平位置**也各不相同

4.当敌机 **从屏幕下方飞出**，不会再飞到屏幕中

![5](images/5.png)

●  **初始化方法**

​	○  指定 **敌机图片**

​	○  **随机** 敌机的 **初始位置** 和 **初始速度**

●  **重写update()方法**

​	○  判断 **是否飞出屏幕**，如果是，从 **精灵组** 删除



##### 2.1敌机类的准备

在plane_sprites.py 中实现 Enemy的 **初始化方法**

~~~
class Enemy(GameSprite):
    """
    敌机精灵
    """
    def __init__(self):
        # 1. 调用父类方法,创建敌机精灵,同时指定敌机图片
        super().__init__("./images/enemy1.png")
        # 2. 指定敌机的初始随机速度
        
        # 3. 指定敌机的初始随机位置


    def update(self):
        # 1. 调用父类方法, 保持垂直方向飞行
        super().update()
        # 2. 判断是否飞出屏幕,如果是,需要从精灵组删除敌机
        if self.rect.y >= SCREEN_RECT.height:
            print("飞出屏幕,需要从精灵组删除...")
~~~



##### 2.2创建敌机

**演练步骤**

1.在__create_sprites方法添加 **敌机精灵组**

​	●  敌机是 **定时被创建的**，因此在初始化方法中，不需要创建敌机

2.在__event_handler创建敌机，并且 **添加到精灵组**

​	●  调用 **精灵组** 的add方法，可以向 **精灵组添加精灵**

3.在__update_sprites方法，让 **敌机精灵组**  调用update和draw方法

![6](images/6.png)



修改plane_main.py  的__create_sprites方法

~~~
# 创建敌机的精灵组
self.enemy_group = pygame.sprite.Group()
~~~

修改plane_main.py  的__event_handler方法

~~~
    def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()
            elif event.type == CREATE_ENEMY_EVENT:
                print("敌机出场...")

                # 创建敌机精灵
                enemy = Enemy()
                #将敌机精灵添加到敌机精灵组
                self.enemy_group.add(enemy)
~~~

修改plane_main.py  的__update_sprites方法

~~~
# update() 更新所有敌机的位置
self.enemy_group.update()
# draw() 在screen上绘制所有精灵
self.enemy_group.draw(self.screen)
~~~



##### 2.3敌机随机位置和速度



##### **1) 导入模块**

●  在导入模块时，**建议** 按照以下顺序导入

~~~
1.官方标准模块导入
2.第三方模块导入
3.应用程序模块导入
~~~

●  修改plane_sprites.py 增加 random 的导入

~~~
import random
~~~

##### **2) 随机位置** ![7](images/7.png)

使用 pygame.Rect 提供的 bottom属性，在指定敌机初始位置时，会比较方便

●  bottom = y + height

●  y = bottom - height 



##### 3) 代码实现

●  修改 **初始化方法**，随机敌机出现 **速度** 和 **位置**

修改plane_sprites.py Enemy类中的初始化方法

~~~\
    def __init__(self):
        # 1. 调用父类方法,创建敌机精灵,同时指定敌机图片
        super().__init__("./images/enemy1.png")
        # 2. 指定敌机的初始随机速度 1 ~ 3
        self.speed = random.randint(1,3)

        # 3. 指定敌机的初始随机位置
        self.rect.bottom = 0  #飞机的初始位置最上方

        # 屏幕宽度 - 飞机宽度
        max_x = SCREEN_RECT.width - self.rect.width
        self.rect.x = random.randint(0,max_x)
~~~



##### 2.4移出屏幕销毁

●  敌机移出屏幕之后，如果 **没有撞到英雄**，敌机的历史使命已经终结

●  需要从 **敌机组** 删除，否则会造成 **内存浪费**

检测敌机被销毁

●  __del__内置方法会在对象被销毁前自动调用，在开发中，可以用于 **判断对象是否被销毁**

~~~
def __del__(self):
	print("敌机挂了 %s" % self.rect)
~~~

**代码实现**

![8](images/8.png)

●  判断敌机是否飞出屏幕，如果是，调用kill()方法从所有组中删除

~~~
    def update(self):
        # 1. 调用父类方法, 保持垂直方向飞行
        super().update()
        # 2. 判断是否飞出屏幕,如果是,需要从精灵组删除敌机
        if self.rect.y >= SCREEN_RECT.height:
            # print("飞出屏幕,需要从精灵组删除...")
            #kill方法可以将精灵从所有组中移出,精灵就会被自动销毁
            self.kill()
~~~



### 英雄登场

#### 目标  

●  设计 **英雄** 和 **子弹** 类

●  使用pygame.key.get_pressed() 移动英雄

●  发射子弹



#### 01.设计英雄和子弹类

##### 英雄需求

1.游戏启动后，**英雄** 出现在屏幕的 **水平中间** 位置，距离 **屏幕底部 120 像素**

2.**英雄** 每隔 0.5 秒发射一次子弹，每次 **连发三枚子弹**

3.**英雄** 默认不会移动，需要通过 **左/右** 方向键，控制 **英雄** 在水平方向移动

![10](images/10.png)



**子弹需求**

1.**子弹** 从 **英雄** 的正上方发射，**沿直线** 向 **上方** 飞行

2.**飞出屏幕后**，需要从 **精灵组** 中删除

![11](images/11.png)

**Hero -- 英雄**

● **初始化方法** 

​	○  指定 **英雄图片** 

​	○  初始速度 = 0，-- 英雄默认静止不动

​	○  定义 bullets **子弹精灵组**  保存子弹精灵

●  **重写 update() 方法**

​	○  英雄需要 **水平移动**

​	○  并且需要保证不能 **移出屏幕**

●  增加 bullets 属性，记录所有 **子弹精灵**

●  增加 fire 方法，用于发射子弹



##### Bullet --  子弹

●  **初始化方法**

​	○  指定 **子弹图片**

​	○  **初始速度 = -2** -- 子弹需要向上方飞行

●  **重写update()方法**

​	○  判断 **是否飞出屏幕**，如果是，从 **精灵组** 删除



#### 02创建英雄

##### 2.1准备英雄类

●  在 plane_sprites.py 新建 Hero 类

●  重写 **初始化方法**，直接指定 **图片名称**， 并且将初始速度设置为0

●  设置 **英雄的初始位置**

![12](images/12.png)

在plane_sprites.py文件中代码如下：

~~~
class Hero(GameSprite):
    """
    英雄精灵
    """
    def __init__(self):
        # 1. 调用父类方法,设置image&speed  英雄默认不移动设置0,需要通过按键移动
        super().__init__("./images/me1.png", 0)
        # 2. 设置英雄的初始位置
        self.rect.centerx = SCREEN_RECT.centerx
        # 注意：向上移动做减法
        self.rect.bottom = SCREEN_RECT.bottom - 120
~~~



##### 2.2绘制英雄

1.在__create_sprites，添加 **英雄精灵** 和 **英雄精灵组**

​	●  后续要针对 **英雄** 做 **碰撞检测** 以及 **发射子弹**

​	●  所以 **英雄**  需要 **单独定义成属性**

2.在__update_sprites，**让英雄精灵组** 调用 update 和 draw方法

**代码实现**

●  修改plane_main.py 中  __create_sprites方法如下：

~~~
    def __create_sprites(self):
        # 1. 创建背景精灵和精灵组
        # bg1 = Background('./images/background.png')
        # bg2 = Background('./images/background.png')

        bg1 = Background()
        # True表示交替背景
        bg2 = Background(True)
        #将创建的精灵传递到背景精灵组内部
        self.back_group = pygame.sprite.Group(bg1,bg2)

        # 创建敌机的精灵组
        self.enemy_group = pygame.sprite.Group()

        # 创建英雄的精灵和精灵组
        self.hero = Hero()
        self.hero_group = pygame.sprite.Group(self.hero)
~~~

**接下来更新精灵组**

●  修改plane_main.py 中  __update_sprites方法如下：

~~~
    def __update_sprites(self):
        # update() 让组中的所有精灵更新位置
        self.back_group.update()
        # draw() 在screen上绘制所有的精灵
        self.back_group.draw(self.screen)

        # update() 更新所有敌机的位置
        self.enemy_group.update()
        # draw() 在screen上绘制所有精灵
        self.enemy_group.draw(self.screen)

        # update() 更新所有英雄的位置
        self.hero_group.update()
        # draw() 在screen上绘制所有精灵
        self.hero_group.draw(self.screen)
~~~

运行程序，查看英雄的飞机是否出现在屏幕的中间位置



#### **03.移动英雄的位置**

在 pygame 中针对 **键盘按键的捕获**， 有 **两种** 方式

●  第一种方式判断 event.type == pygame.KEYDOWN

●  第二种方式

​	1.首先使用 pygame.key.get_pressed() 返回 **所有按键元素**

​	2.通过 **键盘常量**， 判断元祖中 **某一个键是否被按下** -- 如果被按下，对应数值为1

**提问** 这两种方式有什么区别那？

●  第一种方式

~~~
elif event.type == pygame.KEYDOWN and event.key == pygame.K_RIGHT:
	print("向右移动...")
~~~



●  第二种方式

~~~
# 返回所有按键的元祖，如果某个键被按下，对应的值是1
keys_pressed = pygame.key.get_pressed()
# 判断是否按下了方向键
if keys_pressed[pygame.K_RIGHT]:
	print("向右移动...")
~~~



●  修改plane_main.py 中  __event_handler方法如下：

~~~
    def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()
            elif event.type == CREATE_ENEMY_EVENT:
                # print("敌机出场...")

                # 创建敌机精灵
                enemy = Enemy()
                #将敌机精灵添加到敌机精灵组
                self.enemy_group.add(enemy)
                
            # 增加捕获按键判断语句
            # elif  event.type == pygame.KEYDOWN and event.key == pygame.K_RIGHT:
            #     print("向右移动...")
            # 使用键盘提供的方法,获取键盘按键
        # 返回所有按键的元祖，如果某个键被按下，对应的值是1
        keys_pressed = pygame.key.get_pressed()
        # 判断元祖中对应的按键索引值是否为 1
        if keys_pressed[pygame.K_RIGHT]:
            print("持续向右移动...")
~~~

总结：在开发时很明显使用键盘模块方式，因为这种方式英雄的飞机移动更加灵活



##### 3.1移动英雄位置

**演练步骤**

1.在Hero类中重写 update() 方法

​	●  用 **速度** speed 和 **英雄**  rect.x 进行叠加

​	●  不需要调用父类方法 -- 父类方法只是单纯实现了垂直运动

2.在__event_handler 方法中根据 **左右方向键** 设置英雄的 **速度**

​	●  向右 -> speed = 2

​	●  向左 -> speed = -2

​	●  其他-> speed = 0

**代码演练**

●  在 Hero 类，重写update()方法，**根据速度水平移动** 英雄的飞机

在plane_sprites.py Hero类中

~~~
    def update(self):
        # 英雄在水平方向移动
        self.rect.x += self.speed
~~~

在plane_main.py __event_handler方法中

~~~
    def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()
            elif event.type == CREATE_ENEMY_EVENT:
                # print("敌机出场...")

                # 创建敌机精灵
                enemy = Enemy()
                #将敌机精灵添加到敌机精灵组
                self.enemy_group.add(enemy)
            # 增加捕获按键判断语句
            # elif  event.type == pygame.KEYDOWN and event.key == pygame.K_RIGHT:
            #     print("向右移动...")
            # 使用键盘提供的方法,获取键盘按键
        # 返回所有按键的元祖，如果某个键被按下，对应的值是1
        keys_pressed = pygame.key.get_pressed()
        # 判断元祖中对应的按键索引值是否为 1
        if keys_pressed[pygame.K_RIGHT]:
            # print("持续向右移动...")
            
            #如果按下了右键,速度增加
            self.hero.speed = 2
        elif keys_pressed[pygame.K_LEFT]:
            #如果按下了左键,速度减少
            self.hero.speed = -2
        else: #其他键
            self.hero.speed = 0
~~~

运行代码，测试左右按键是否能够移动英雄飞机



##### 3.2控制英雄的运动边界

●  在 Hero 类的 update() 方法判断 **英雄**  是否超出 **屏幕边界**

![14](images/14.png)

在plane_sprites.py Hero类中

~~~
    def update(self):
        # 英雄在水平方向移动
        self.rect.x += self.speed

        # 控制英雄不能离开屏幕
        if self.rect.x < 0:
            self.rect.x = 0
        #如果精灵的右侧大于屏幕的右侧,将值设置为相等,右侧就无法飞出了
        elif self.rect.right > SCREEN_RECT.right:
            self.rect.right = SCREEN_RECT.right
~~~



#### 04.发射子弹

**需求回顾 -- 英雄需求**

1.游戏启动后，**英雄** 出现在屏幕的 **水平中间** 位置，距离 **屏幕底部 120 像素**

2.**英雄** 每隔 0.5 秒发射一次子弹，每次 **连发三枚子弹**

3.**英雄** 默认不会移动，需要通过 **左/右** 方向键，控制 **英雄** 在水平方向移动



##### 4.1添加发射子弹事件

pygame 的 **定时器** 使用套路非常固定：

1.定义 **定时器常量** -- eventid

2.在 **初始化方法** 中，调用 set_timer()方法 **设置定时器事件**

3.在 **游戏循环** 中 ，**监听定时器事件**

**代码实现**

●  在plane_sprites.py Hero类中 定义 fire 方法

~~~
def fire(self):
	print("发射子弹...")
~~~

在导入模块下方增加一个英雄发射子弹常量

~~~
# 英雄发射子弹事件
HERO_FIRE_EVENT = pygame.USEREVENT + 1
~~~

![15](images/15.png)



●  在plane_main.py __init__方法中

~~~
# 设置英雄发射子弹定时器，每隔0.5秒英雄发射子弹事件被触发
pygame.time.set_timer(HERO_FIRE_EVENT, 500)
~~~

![16](images/16.png)



在plane_main.py 文件__event_handler方法中

~~~
    def __event_handler(self):
        # 获取当前发生的所有事件列表
        for event in pygame.event.get():
            # 判断是否退出游戏
            if event.type == pygame.QUIT:
                PlaneGame.__game_over()
            elif event.type == CREATE_ENEMY_EVENT:
                # print("敌机出场...")
                # 创建敌机精灵
                enemy = Enemy()
                #将敌机精灵添加到敌机精灵组
                self.enemy_group.add(enemy)
             
            # 捕获英雄发射子弹事件 
            elif event.type == HERO_FIRE_EVENT:
                self.hero.fire()
~~~



##### 4.2定义子弹类

**需求回顾 -- 子弹需求**

1.**子弹** 从 **英雄** 的正上方发射 **沿直线** 向 **上方** 飞行

2.**飞出屏幕后**，需要从 **精灵组** 中删除 



##### Bullet --  子弹

●  **初始化方法**

​	○  指定 **子弹图片**

​	○  **初始速度 = -2** -- 子弹需要向上方飞行

●  **重写update()方法**

​	○  判断 **是否飞出屏幕**，如果是，从 **精灵组** 删除



**定义子弹类**

●  在plane_sprites.py 新建 Bullet 类 继承自 GameSprite

●  重写 **初始化方法** ，直接指定 **图片名称**，并且设置 **初始速度**

●  重写 update()方法，判断子弹 **飞出屏幕后从精灵组删除** 



在plane_sprites.py 定义Bullet子弹类

~~~
class Bullet(GameSprite):
    """
    子弹精灵
    """
    def __init__(self):
        # 调用父类方法,设置子弹图片和初始速度
        super().__init__("./images/bullet1.png", -2)

    def update(self):
        # 调用父类方法,让子弹沿垂直方向飞行
        super().update()

        # 判断子弹是否飞出屏幕
        if self.rect.bottom < 0:
            # 从精灵组中删除
            self.kill()

    def __del__(self):
        print("子弹被销毁...")
~~~

![17](images/17.png)



##### 4.3发射子弹

**演练步骤**

1.在Hero类 **初始化方法** 中创建 **子弹精灵组** 属性

2.修改plane_main.py 的__update_sprites 方法，让 **子弹精灵组** 调用 update 和 draw 方法

3.实现fire() 方法

​	●  创建子弹精灵

​	●  设置初始位置 -- 在 **英雄的正上方**

​	●  将 **子弹** 添加到精灵组 

**代码实现**

● 在plane_sprites.py  Hero类 __init__ **初始化方法**

~~~
    def __init__(self):
        # 1. 调用父类方法,设置image&speed  英雄默认不移动设置0,需要通过按键移动
        super().__init__("./images/me1.png", 0)
        # 2. 设置英雄的初始位置
        self.rect.centerx = SCREEN_RECT.centerx
        # 注意：向上移动做减法
        self.rect.bottom = SCREEN_RECT.bottom - 120
        
        # 3. 创建子弹的精灵组
        self.bullets = pygame.sprite.Group()
~~~

在plane_main.py 的__update_sprites 方法，让 **子弹精灵组** 调用 update 和 draw 方法

~~~
    def __update_sprites(self):
        # update() 让组中的所有精灵更新位置
        self.back_group.update()
        # draw() 在screen上绘制所有的精灵
        self.back_group.draw(self.screen)

        # update() 更新所有敌机的位置
        self.enemy_group.update()
        # draw() 在screen上绘制所有精灵
        self.enemy_group.draw(self.screen)

        # update() 更新所有英雄的位置
        self.hero_group.update()
        # draw() 在screen上绘制所有精灵
        self.hero_group.draw(self.screen)

	   # 更新子弹精灵组
        self.hero.bullets.update()
        self.hero.bullets.draw(self.screen)
~~~

在plane_sprites.py  Hero类 修改fire()方法

~~~
    def fire(self):
        # print("发射子弹...")
        # 1. 创建子弹精灵
        bullet = Bullet()
        # 2. 设置精灵的位置
        bullet.rect.bottom = self.rect.y - 20  # 子弹在英雄的上方一点点
        bullet.rect.centerx = self.rect.centerx # 子弹在英雄的正上方
        # 3. 将精灵添加到精灵组
        self.bullets.add(bullet)
~~~



**一次发射三枚子弹**

![18](images/18.png)

●  修改fire()方法，一次发射三枚子弹

~~~
    def fire(self):
        print("发射子弹...")

        for i in (0,1,2):
            # 1. 创建子弹精灵
            bullet = Bullet()
            # 2. 设置精灵的位置
            bullet.rect.bottom = self.rect.y - i * 20  # 子弹在英雄的上方一点点
            bullet.rect.centerx = self.rect.centerx # 子弹在英雄的正上方
            # 3. 将精灵添加到精灵组
            self.bullets.add(bullet)
~~~



### 碰撞检测



#### 目标

●  了解碰撞检测方法

●  碰撞实现



#### 01.了解碰撞检测方法

●  pygame 提供了 **两个非常方便** 的方法可以实现碰撞检测

##### pygame.sprite.groupcollide()

●  **两个精灵组** 中 **所有的精灵** 的碰撞检测

~~~
groupcollide(group1,group2,dokill,dokill2,collide = None) -> Sprite_dict
~~~

●  如果将dokill设置为True,则 **发生碰撞的精灵将被自动移除**

●  collide 参数是用于 **计算碰撞的回调函数**

​	○  如果没有指定，则每个精灵必须有一个 rect 属性

在plane_main.py 的 __check_collide 检测碰撞方法

```
    def __check_collide(self):
        # 1. 子弹摧毁敌机
        pygame.sprite.groupcollide(self.hero.bullets,self.enemy_group,True,True)
   
```



##### pygame.sprite.spritecollide()

●  判断 **某个精灵**  和 **指定精灵组**  中的精灵的碰撞

~~~
spritecollide(sprite,group,dokill,collide = None)  -> Sprite_list
~~~

●  如果将dokill设置为True,则 **指定精灵组中 发生碰撞的精灵将被自动移除**

●  collide 参数是用于 **计算碰撞的回调函数**

​	○  如果没有指定，则每个精灵必须有一个 rect 属性

●  返回 **精灵组** 中跟 **精灵** 发生碰撞的 **精灵列表**

在plane_main.py 的 __check_collide 检测碰撞方法

~~~
    def __check_collide(self):
        # 1. 子弹摧毁敌机
        pygame.sprite.groupcollide(self.hero.bullets,self.enemy_group,True,True)

        # 2. 敌机撞毁英雄
        enemies = pygame.sprite.spritecollide(self.hero,self.enemy_group,True)

        # 3. 判断列表是否有内容
        if len(enemies) > 0:
            # 让英雄牺牲
            self.hero.kill()
            # 游戏结束
            PlaneGame.__game_over()
~~~

























​	





















​	










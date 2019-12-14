+++
title = "游戏对象模型设计与实现（四）"
author = ["Eviler"]
lastmod = 2019-12-14T12:56:59+08:00
tags = ["game", "oo", "es", "design"]
categories = ["计算机"]
draft = false
creator = "Emacs 26.3 (Org mode 9.3 + ox-hugo)"
weight = 1004
+++

最近几个月一直在写一些与游戏无关的程序放到 AppStore 上, 主要是用了一些 FRP 的东西.
关于 FRP, 以后再写一些文章介绍. 期间抽空在 iOS 把 Artemis 实现了一遍, 开始做一个横版 2.5d 的动作游戏. 等实现完了发现有人把另外一个 Objective-C 的实现放到了 GitHub
上. 不过 GitHub 上的版本貌似是基于 Artemis 0.96 的, 其中后来 Artemis 的一些优化比如
Aspect 等并没有同步进去. 我是直接基于 Artemis 的最新版本做的, 也不算重复发明轮子了.

<!--more-->

本章主要来介绍一下 Entity-System 的一些概念.

-   Entity: 实体, 是组件的一个集合. 当然在实现上, Entity 只是一个引用的计数或者 ID 或者 UUID, 其内部

并没有任何数据结构来包含实体中的 Component, 如果对数据库有一些了解的话, 这个 Entity 相当于某个表的 primary key, 用以索引 Component.

-   Component: 组件, 组件内部只包含组件的数据, 而并不包含任何行为 (在 Component-Based 的架构中不

是这样, 至少 Unity3D 中的 Component 是有 update 等行为的.). 因为 Component 通常的实现方式是用 OOP
的 class 来实现, 所以 Component 中的数据的 setter 和 getter 不算是行为.

-   System/Subsystem: 系统, 处理组件的行为. 既然组件只包含数据, 那么组件的行为当然要表现出来, 表现

组件行为的部分就是系统. 游戏逻辑实现都在系统中实现.

由于 Entity-System 比较灵活, 所以现在也没有所谓的 `设计模式` 来约束开发者如何设计 Component 和 System,
所以在使用 Entity-System 设计 Component 的时候会比较纠结. 这一点和 OOP 的 N 多\`设计模式\`不同, 至少我在使用的时候对于 `什么东西做成 Component` 之类的问题纠结不已.

下面就举一个 OOP 和 Entity-System 相对应的例子来加深我们对于 Entity-System 的理解.

在 OOP 中, 我们知道一个对象实例是数据和行为绑定的, 这个对于惯用 OOP 来思考的我们来说是非常显然的,
因为没有行为的数据是没有意义的, 而没有数据的行为则是废行为. 看以下的 C++ 伪代码:

```c++
class Player : public GameObject
{
    public:
        move (int deltaTime) {
            x = x + velocity * deltaTime;
            y = y + velocity * deltaTime;
        }
    private:
        int x;
        int y;
        int velocity;
};
```

以上的代码从底层的角度来考虑, `Player` 对象其实包含两部分:

-   对象的数据: `x`, `y`, `velocity`
-   对象的行为: `move`

可能大家经常会看到这样的话: \* `OO` 是一种思想假, 并不是 C++ 的专利, 用 C 也可以实现.\* 那我们就用用 C 来实现, 可能更接近于本质的理解:

```c
typedef struct {
    int x;
    int y;
    int velocity;
} Player;

void Player_Move (Player* self, int deltaTime)
{
    self->x = self->x + self->velocity * deltaTime;
    self->y = self->y + self->velocity * deltaTime;
}
```

我们把以上代码中的 `Player` 结构体 看成两个组件的集合, 把 `Player_Move` 函数看成是两个组件的组合在一起的 Move 系统.
代码演化一下:

```c
typedef struct {
    struct {
        int x;
        int y;
    } position;
    struct {
        int velocity;
    } velocity;
} Player;

void System_Move (int deltaTime)
{
    for (Player* p in All_Entities()) {
        if (Player_containsStruct(p, postion) && Player_containsStruct(p, velocity)) {
            p->postion.x = p->position.x + p->velocity.velocity * deltaTime;
            p->position.y = p->position.y + p->velocity.velocity * deltaTime;
        }
    }
}
```

上面是伪代码, 其中 `Player_containsStruct` 是不存在的, 但是不妨碍我们可以用其他手段来实现这个功能.
实际上, `postion` 和 `velocity` 就是两个组件, 而 `System_Move` 则就是系统, 一个实现了 Move 功能的系统,
当然, 这个系统只有在 `Player` 这个 Entity 同时具有 `position` 和 `velocity` 这两个组件的时候才有意义.
这就是 Entity-System 的思想.

对比 OOP 思想来看, OOP 意图把数据封装起来, 只暴露给外部一些行为接口, 其目的是为了达到代码重用. 而
`Entity-System` 又回归了我们在接受 `OOP` 之前的做法, 把数据和行为分离开了.

可以这么理解, 无论是 `OOP` 还是 `Entity-System` 其本质不过是一种 `分类` 的方式, `OOP` 试图用 `Is-A` 的关系来静态组织我们需要处理的对象, 而 `Entity-System` 则是用 `Has-A` 的动态关系来组织我们所需要处理的对象.
诚然, 在 `OOP` 中有 `Has-A` 的关系, 但那是在代码中写死的静态关系, 而 `Entity-System` 的 `Has-A` 关系要求我们实现一种动态的  `Has-A` 关系管理. 这也是 `Entity-System` 的最主要的特点. 实现动态的  `Has-A` 关系有很多方法, 这里就不再赘述.

这种动态的 `Has-A` 关系赋予我们无限扩展的能力而不必去 **修改** 代码, 也不是要求我们在父类中添加新的行为, 我们需要做的只是去给 Entity 添加新的 Component, 或者去增加一个 System 处理. 而且这个 System 处理和原来的
System 是分离的, 没有任何依赖关系. 那以后的项目我们甚至不需要修改任何源代码就可以使用. 回顾前面 `狮鹫` 的例子, 假如我们是用 Entity 方式实现的话, 我们给狮鹫添加一个 Run 的 Component, 则 我们系统原有的 RunSystem
和 RunComponent 可以赋予狮鹫陆地行走的功能, 而再添加一个 FlyComponent, 则我们系统原有的 FlySystem 则赋予狮鹫飞行的功能. ? 为什么不修改源代码呢? 上面的 `Player` 例子里面的两个 component: `position` 和 `velocity`
不就是代码中写死的么? 答案是:  `Entity-System` 的动态 `Has-A` 关系实现允许我们这样来做:

```c
Player_AddComponent (EntityId player, Component* position);
Player_AddComoonent (EntityId player, Component* velocity);
```

以上是伪代码只是方便讲解, 请勿当真.

# Hands-On Design Patterns with C++



## 1. Introduction to Inheritance and Polymorphism

继承和多态

Combining algorithms and data to be operated into single objects

An object is a concrete instantiation of a class

It is one of the signs of good design when the data is well encapsulated in the classes, and the methods have limited interaction with external data. A well-designed class hast mostly, or only, private data members, and the only public methods are those needed to express the public interface of the class

好的设计是，数据被封装在类内，而方法与外界的数据只有有限的交互。一个好的类设计基本上只有私有数据成员，并且，仅有的那些公共方法，也是该类需要有的公共接口，想像成**对外的合同**，承诺对客户类的支持 - 操作、维护不变量以及遵守特定的约束。

struct和class的唯一区别在于：struct默认所有为公共，class默认所有为私有

class的静态方法和数据，和非成员的静态方法和数据一样，都不会作用于成员上。但是，它们拥有对类内成员的访问权限。

类是一个组织它所操作的数据和算法的组合，但是，C++最强大的OO特征，是继承。类继承的目的是表达类之间的关系，以及从简单的类型编制更加复杂的类型（从现有的类定义新类）

---

### 公共继承、私有继承

继承类的一个对象，同时也是父类的一个对象。

公共继承`class B : public A {}`时，继承类的公共接口。

私有继承时，继承类的所有方法成为私有，所有的公共方法必须重新创建。继承类就好像把基类作为了一个数据成员。那么，继承类获得了什么？获得的是父类的实现细节，即数据成员和方法可以被用于继承类自己需要实现的算法，当然，也可以通过再暴露父类的公共方法拥有父类的公共方法：

```c++
class Container : private std::vector<int> {
  public:
  
  // re-expose the public interface of base class
  using std::vector<int>::size;
}
```

### 为什么要用私有继承

原因一：继承类对象的内存大小

编程中一个很常见的操作是，基类只提供方法，没有数据成员。由于这种类没有自己的数据，也就不占据任何内存。但是在C++中，它们必须给一个非零的大小（至少一个字节的空对象），因为任意两个不同的对象/变量，都有唯一的一个地址。这样的话，从来都不被使用的空对象也占用了内存空间，也就浪费了内存。然而，继承对象、基对象和首个被创建的数据成员，可以在同一个地址。这就是C++中有一个基类优化：可以不给空的基对象内存占用，即便使用`sizeof()`会返回一个字节。

如果我们创建了许多继承类对象，基类的优化非常重要。

原因二：虚拟函数 - 多态

---

关于如何定义父类和类，举一个例子：

- 长方形类：拥有2个变长变量
- 正方形类：只需要1个变长变量
- C++中，正方形类不能继承于长方形类，因为长方形类的接口（合同）与正方形类的接口不一样（2个 v.s. 1个）
- C++中，正确的做法是定义一个共同的父类。
- 正如企鹅不能继承鸟类一样，因为鸟类有一个“飞”的接口，而企鹅不能飞。所以需要定义一个鸟的父类，其中不去“承诺”任何其子类不能做到的事情，然后定义“能飞的鸟”和“不能飞的鸟”两个类，继承于“鸟”类。

所以：

```c++
class Base {...};
class Derived : public Base {...};
```

---

### 多态和虚拟函数

定义一个“会飞的鸟”类，有一个`fly()`函数。但是，鹰和秃鹫的飞是不一样的，需要不同的实现。所以就有了虚函数，虚函数被定义在基类中。继承类不光继承了**函数的声明**，也继承了**函数的实现**。如果函数的实现是一样的，就没必要干啥了。如果不一样，就改变函数的实现。例如：

```c++
// 定义会飞的鸟
class FlyingBird : public Bird {
  public:
  virtual void fly(doube speed, double direction){
    ...
  }
};

// 定义秃鹫类，继承于会飞的鸟类
class Vulture : public FlyingBird {
  public:
  virtual void fly(double speed, double direction) {
    // 秃鹫的飞行
  }
}
```

如果调用了虚拟函数，C++在编译时无法确定对象真正的类型，需要在运行时确定。例如：

```c++
void hunt(FlyingBird& b) {
  b.fly(...); // <- 无法确定是鹰还是秃鹫
}

Eagle e;
hunt(e); // <- 可以确定是鹰

Vulture v;
hunt(v); // <- 可以确定是秃鹫
```



> 从基对象调用同一个函数，但其结果取决于实际类对象，叫做运行时多态。支持这种技术的类是多态的。一个多态的类必须有至少一个虚拟函数

如果一个基类有一个纯虚函数，那么继承类必须提供具体实现。无法创建一个拥有纯虚函数的基类对象。

如果需要覆盖基类的虚函数，C++11提供了一个无bug的关键字`override`，继承类不再拥有虚函数。

```c++
class Eagle : public FlyingBird {
  public:
  void fly(int speed, double direction) override;
}
```



问题：

什么情况下用共有继承，什么情况下用私有继承
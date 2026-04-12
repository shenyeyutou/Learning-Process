##继承-派生类

#1.定义派生类时一定要见到基类的定义。
e.g

```cpp

class A;  //声明
class B: public A  //Error
{  
	int z;
 public:
    void h() { g(); }  //Error，编译程序不知道基类中
				       //是否有函数g。
};
......
B b; //Error，编译无法确定b所需内存空间的大小。

#2.就算是继承了并且分配了空间，如果是基类的private，系统禁止访问！！
-->>protected访问控制-->>缓解了封装与继承的矛盾

#3.继承方式（key points）:由基类的访问控制与继承方式共同决定
(默认的继承方式为：private)

e.g
```cpp

class A {
public:
    void f();        // 公有
protected:
    void g();        // 受保护
private:
    void h();        // 私有
};

class B1 : public A {
public:
    void test() {
        f();  // ✅ 继承为 public，仍然可访问
        g();  // ✅ 继承为 protected，派生类内部可访问
        // h(); ❌ 基类 private，派生类不可访问
    }
};

class B2 : protected A {
public:
    void test() {
        f();  // ✅ 继承为 protected，派生类内部可访问
        g();  // ✅ 继承为 protected，派生类内部可访问
        // h(); ❌ 不可访问
    }
};

class B3 : private A {
public:
    void test() {
        f();  // ✅ 继承为 private，仅 B3 内部可访问
        g();  // ✅ 继承为 private，仅 B3 内部可访问
        // h(); ❌ 不可访问
    }
};


//e.g2:

class A {
public:
    void f();        // 公有
protected:
    void g();        // 受保护
private:
    void h();        // 私有
};

class B : protected A {
    // f 变为 protected
    // g 保持 protected
    // h 不可直接访问
public:
    void q() {
        f();  // ✅ OK，f 在 B 中是 protected，类内部可访问
        g();  // ✅ OK，g 在 B 中是 protected，类内部可访问
        h();  // ❌ Error，h 是 A 的 private，派生类不可访问
    }
};

class C : public B {
public:
    void r() {
        f();  // ✅ OK，继承自 B 的 protected，C 内部可访问
        g();  // ✅ OK，同理
        h();  // ❌ Error，A 的 private，仍不可访问
        q();  // ✅ OK，q 是 B 的 public，在 C 内部可访问
    }
};

void func() {
    B b;
    b.f();  // ❌ Error，f 在 B 中是 protected，对外不可见
    b.g();  // ❌ Error，g 在 B 中是 protected，对外不可见
    b.h();  // ❌ Error，A 的 private，不可见
    b.q();  // ✅ OK，q 是 B 的 public，对外可见
}

/*
这个案例没啥好说的，我只想说一点就是：

这里 func() 不能直接调用 f() 或 g()，但可以调用 q()。

而 q() 是 public，它内部调用了 f() 和 g()，所以外部间接实现了对基类函数的访问。

⚡ 设计思想
这正是 封装 的体现：

外部用户不需要知道 f() 和 g() 的细节，只要调用 q() 就能完成操作。

派生类通过 public 方法对外暴露必要的接口，内部再调用继承来的 protected 方法。

这样既保证了安全性，又提供了灵活的功能。
*/

#4.继承方式的调整

```cpp

class A {
public:
    void f1();
    void f2();
    void f3();
protected:
    void g1();
    void g2();
    void g3();
};

class B : private A {
public:
    using A::f1;  // 把 f1 调整为 public
    using A::g1;  // 把 g1 调整为 public
protected:
    using A::f2;  // 把 f2 调整为 protected
    using A::g2;  // 把 g2 调整为 protected
};


⚡ 解释
B : private A → 默认情况下，A 的所有 public/protected 成员在 B 中都会变成 private。

通过 using A::成员名; 可以 重新声明这些成员在 B 中的访问权限。

using A::f1; → 把原本在 B 中变成 private 的 f1 调整为 public。

using A::g1; → 把原本在 B 中变成 private 的 g1 调整为 public。

using A::f2; → 把原本在 B 中变成 private 的 f2 调整为 protected。

这样，外部用户就可以通过 B 的对象调用 f1() 和 g1()，而不能直接调用 f2() 和 g2()。

#5.子类型（派生类看作是基类的子类型）-->派生类对象可以赋值或作为函数参数传给基类变量

e.g(some legal operation)

```cpp

A a;
B b;
b.f(); //OK，基类的操作可以实施到派生类对象
a = b;  //OK，派生类对象可以赋值给基类对象，          
          //属于派生类但不属于基类的数据成员将被忽略

A *p = &b;  //OK，基类指针变量可以指向派生类对象
......
void func1(A *p); 
void func2(A &x);
void func3(A x);
func1(&b); func2(b); func3(b); //OK


there are also some illegal operations:

```cpp

A a;
B b;
a.g(); //Error，a没有g这个成员函数。
b = a;  //Error，a中没有某些数据
B *q = &a;  //Error，
                 //操作“q->g();”将修改不属于a的数据！
......
void func1(B *p); 
void func2(B &x);
void func3(B x);
func1(&a); func2(a); func3(a); //Error


#6.派生类对象的初始化和消亡处理

你这张幻灯片正好讲到 **派生类对象的构造与析构顺序**，这和你刚复习的“引用”一样，都是 C++ 面向对象里很关键的细节。我们来梳理一下：

---

🏗️ 构造过程（Initialization）
当你创建一个派生类对象时，实际上有两部分需要初始化：

1. **基类部分**  
   - 由基类的构造函数负责。  
   - 默认情况下调用基类的默认构造函数。  
   - 如果要调用基类的非默认构造函数，必须在派生类构造函数的 **成员初始化表** 中显式写出。

2. **派生类部分**  
   - 由派生类的构造函数负责。  
   - 在基类构造完成之后才执行。

👉 顺序：**先基类 → 再派生类**

---

🗑️ 析构过程（Destruction）
当派生类对象生命周期结束时：

1. **先执行派生类的析构函数**，清理派生类自己新增的资源。
2. **再自动调用基类的析构函数**，清理继承来的部分。

👉 顺序：**先派生类 → 再基类**

---

 ⚡ 示例代码
```cpp
#include <iostream>
using namespace std;

class A {
public:
    A() { cout << "Base constructor\n"; }
    ~A() { cout << "Base destructor\n"; }
};

class B : public A {
public:
    B() { cout << "Derived constructor\n"; }
    ~B() { cout << "Derived destructor\n"; }
};

int main() {
    B b;
    return 0;
}
```

**输出顺序：**
```
Base constructor
Derived constructor
Derived destructor
Base destructor
```

---

🎯 总结

- **构造**：基类先于派生类。  
- **析构**：派生类先于基类。  
- 这样保证了对象的完整性：先把“底座”搭好，再加上“扩展”；销毁时先拆扩展，再拆底座。

---
exception:

没写构造函数 → 编译器生成默认构造函数 → 调用基类构造函数

没写析构函数 → 编译器生成默认析构函数 → 调用基类析构函数

这样保证了继承体系中的对象始终能被正确初始化和销毁。

notice:

如果一个类D既有基类B、又有成员对象类M，则

在创建D类对象时，构造函数的执行次序为：B->M->D

当D类的对象消亡时，析构函数的执行次序为：D->M->B

#7.派生类的拷贝构造函数

B(const B& b)：A(b) //调用A类的拷贝构造函数



#8.派生类对象的赋值操作

隐式赋值：编译器帮你调用基类赋值 + 派生类赋值。

自定义赋值：你必须手动调用基类赋值，否则基类部分会被忽略。

```cpp

class A {
public:
    int x;
    A& operator=(const A& other) {
        x = other.x;
        return *this;
    }
};

class B : public A {
public:
    int y;
    B& operator=(const B& other) {
        // 必须显式调用基类的赋值操作
        A::operator=(other);
        y = other.y;
        return *this;
    }
};

//the examples in PPT

class A { ...... };
class B: public A
{       ......
    public:
        B& operator =(const B& b)
        {   if (&b == this) return *this;  //防止自身赋值。
             *(A*)this = b; //调用基类的赋值操作符对基类成员
                                  //进行赋值。也可写成： 
                                  //this->A::operator =(b); 
            ...... //对派生类的成员赋值
            return *this;
        }
}; 
......
B b1,b2;
b1 = b2;


#9.派生类成员中标识符的作用域

如果派生类中定义了与基类同名的成员，则基类的成员名在派生类的作用域内不直接可见
（被隐藏，Hidden）。访问基类同名成员时要用基类名指定。例如:

class B: public A
{       int z;
  public:
        void f();
        void h()
        {   f();  //B类中的f
            A::f();  //A类中的f
        }
};

class A {
public:
    int x;
};

class B : public A {
public:
    int x; // 隐藏了 A::x
};

B b;
b.x = 10;        // 使用的是 B::x
b.A::x = 20;     // 强制访问 A::x

即使派生类中定义了与基类同名但参数不同的成员函数，
基类的同名函数在派生类的作用域中也是不直接可见的，仍然需要用基类名指定来使用之：

class B: public A
{       int z;
    public:
        void f(int); 
        void h() 
        {   f(1);  //OK
            f();  //Error
            A::f();  //OK
        }
};
注意：B类中的f与A类中的f不属于函数名重载，因为它们属于不同的作用域。

也可以在派生类中使用using声明把基类中某个的函数名对派生类开放：
class B: public A
{       int z;
    public:
        using A::f;
        void f(int); 
        void h() 
        {   f(1);  //OK
            f();  //OK，等价于A::f();
        }
};

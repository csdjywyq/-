# template 编程和迭代器粗解

## 模板编程

模板的基本语法如下：

```c++
template <typename/class T>
```

template 告诉编译器，接下来是一个模板 ，typename 和 class 都是关键字，在这里二者可以互用没有区别。在`< >`中 `T`叫做模板形参，一旦模板被实例化，`T` 也会变成具体的类型。

## 模板函数

代码实例：

```c++
 template <typename T>
 T  add(const T lva ,const T rva) 
 {

     T a ;

     a = lva + rva ;

    return a;    
 }
```

这是一个模板函数的简单实例，所有模板函数在开始都需要 `template` 语句，以告诉编译器这是一个模板和参数等必要信息，当然里面的 `T` 可以取任意你喜欢的名字 ，模板参数个数也是任意更换的。 还要提醒的一点是：`template <typename T1 ,typename T2 = int>`函数模板是支持默认参数的，T1 、T2顺序在默认情况下是可以任意的，不用严格按照从右到左的顺序。

然后就是使用了，我们可以写出`add(1,2)` 这样的函数,也可以写出`add(2.5,4.6)`这样的函数，向 `add` 函数提供参数时，编译器会自动分析参数的类型，然后将所有用到 T 定义的换成相对性的类型，以上的两个函数在编译期间会生成

```c++
 int add(const int lva ,const int rva) 
{

     int a ;

     a = lva + rva ;

    return a;    
 }
 double add(const double lva ,const double rva) 
 {

     double a ;

      a = lva + rva ;

    return a;    
 }
```

这样的两个具体函数。如果我们使用`add(1,2.0)`是会报错的，编译器无法找到`add(int,double)`。

## 类模板和成员模板

### 类模版

c++11 不仅支持对函数的模板化，也支持对类的模板，下面来看基本的语法是怎样的：

```c++
 template <class T>
 class Myclass
 {
     T a;
     public:
         T add(const T lva ,const T rva);
 };

 template <class T>
 T Myclass<T>::add(const T lva, const T rva)
 {
     a = lva + rva;
     return a;
 }
```

这是一个简单并且典型的类模板，在程序中给出模板并不能使用它，还必须实例化，比如：

`Myclass<int> A；` //用 int 实例化一个类A

`Myclass<double> B；` //用 double 实例化一个类B

当程序编译到这里时就会按照我们给出的类型，声明两组类和两组类函数。注意，在这里我们一定要显式给出类型 T 。类模板不像是函数模板 ，函数模板会根据参数推断类型。 当然类模板也支持默认参数，但是类模板必须严格从右往左默认化。

#### 成员模板

模板的使用范围是广泛的，不仅可以用作函数模板，类模板，还可以用作 class ，struct ，template class 的成员。而要实现 STL 这是我们必须掌握和使用的特性。我们先看一个简单的例子,用上面的类改编而来：

```c++
 template <class T>
 class Myclass
 {
     public:
        T a;
        template <typename type_1 , typename type_2>
         type_1 add(const type_1 lva ,const type_2 rva);
 };

 template <class T>
     template <typename type_1,typename type_2>
 type_1 Myclass<T>::add(const type_1 lva, const type_2 rva)
 {
     a = lva + rva;
     return a;
 }
```

在类的声明中使用了一个嵌套的模板声明。且通过作用域运算符 **::** 指出 add 是类的成员，需要注意的一点，有些编译器不支持模板成员，而有些编译器不支持在类外定义。我们默认大家的编译器都支持。模板如此强大，甚至允许我们在模板类中再建立模板类：

```c++
 template <class T>
 class Myclass
 {
     public:
        T a;
        template <typename type_1 , typename type_2>
         type_1 add(const type_1 lva ,const type_2 rva);

         template <class type_3>
         class Myclass_2;         // 声明放在这里，具体定义放在类外进行。
         Myclass_2<T> C;          // 定义一个Myclass_2 类 A。使用 T 进行实例化
 };

 template <class T>
     template <typename type_1,typename type_2>
 type_1 Myclass<T>::add(const type_1 lva, const type_2 rva)
 {
     a = lva + rva;
     return a;
 }

 template <class T>
     template <class type_3>
     class Myclass<T>::Myclass_2
     {
         public:
             type_3 value;
             type_3 sub(const type_3 a , const type_3 b) {vlaue = a - b;} 
     };
```

## 模板类中的静态成员

在类中定义的静态成员是存储在静态区中，被所有类对象共享，并不属于某一个类所有，同样的在模板类中的静态成员也不会被复制多份，而是被同类实例化的类对象共享，比如所有 int 和所有 double 的类对象，享有相互独立的静态变量。也可以说是编译器生成了 int 和 double 两个版本的类定义。

## typename 和 class

`typename`和`class`是模板中经常使用的两个关键词 ，在模板定义的时候没有什么区别。以前用的是 class，后来 c++ 委员会加入了 typename。因为历史原因，两个是可以通用的。对有些程序员来说，在定义类模板的时候，常常使用 class 作为关键字，增加代码可读性。其它则用 typename，上面的代码大都遵循这样的标准，但是并无强制规定。但是如果二者没有差别，为什么还要加入typename呢？**c++标准委员会不会增加无用的特性**，让我们来看一个例子：

```c++
class Myclass{
    public:
        Myclass();
        typedef int test;  //定义类型别名
}
template <class T>
class Myclass2{
    public:
        Myclass2();
        T::test *a  // 声明一个指向T::test类型的指针。
        //   typename T::test * a
}
```

以上的代码没有全部写完，大家觉得编译器能够过吗？答案是不能,因为在 c++ 中，允许我们在类中定义一个类型别名，且使用的时候和类名访问类成员的方法一样。这样编译器在编译的时候就会产生二义性，它根本不知道这是一个类型还是别名，所以我们加上 typename 显式说明出来。当然如果这里没有二义性，比如`Myclass ::test * a` ,加上 typename 是会报错的。此外，在 class 的 STL 底层还有一个特性，用于保留模板参数，但是在 c++17 中已经舍弃，所以我们没有讲。

## 补充[C++模板之typename和class关键字的区别](https://www.cnblogs.com/cthon/p/9201649.html)

我们都知道，在STL中基本上都使用了模板类的声明，即template。在模板类的声明中，我们有两种方式：

```c++
template < class T>
template < typename T>
```

在这里，`class`和`typename`是相同的。也就是说，在声明一个template type parameter(模板类型参数)的时候，`class`和`typename`意味着完全相同的东西。

但是，在C++中，有的时候必须要使用`typename`.下面我们列举下面一个例子。

关键字`typename`被用来作为型别之前的标识符号。

```
template <class T>
class MyClass{
    typename T::SubType * ptr;
    ...
};
```

在这里，`typename`指出SubType是class T中定义的一个类别，因此ptr是一个指向T::SubType型别的指针。如果没有关键字`typename`，SubType会被当成一个static成员，于是

```
T::SubType * ptr
```

会被解释为型别T内的数值SubType与ptr的乘积。

SubType成为一个型别的条件是,任何一个用来取代T的型别，其内部必须有一个内部型别(inner type)SubType的定义。例如，将型别Q当作template的参数。 

必要条件是型别Q有如下的内部型别定义：

```
class Q{
    typedef int SubType;
    ...
};
```

因此,MyClass的ptr成员应该变成一个指向int型别的指针，子型别SubType也可以成为抽象 
数据型别（例如,class）：

```
class Q{
    class SubType;
    ...
};
```

注意，如果要把一个template中的某个标识符号指定为一种类别，就算是意图显而易见，关键字typename也是不能省略的，因此**C++的一般规则是，除了使用typename修饰之外，template内的任何标识符号都被视为一个值而不是一个类别(对象)。**

### 总结

1. `template<typename T>`与`template<class T>`一般情况下这两个通用，但有一个特例，就是当 T 是一个类，而这个类又有子类(假设名为 innerClass) 时，应该用 `template<typename>`:
2. `typename T::innerClass myInnerObject;`这里的 `typename` 告诉编译器，`T::innerClass` 是一个类，程序要声明一个`T::innerClass` 类的对象，而不是声明 T 的静态成员，而`typename`如果换成`class` 则语法错误。


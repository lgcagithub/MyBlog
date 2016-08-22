---
title: C#中的delegate和event
date: 2016-08-21 16:04:37
tags:
- C#
- Delegate
- Event
---

---------------------------------------

## delegate是C#版的函数指针

　　在`C/C++`中可以利用函数指针对函数进行引用，从而让函数可以作为参数来传递，或者作为函数结果返回，并且通过函数指针可以调用被引用的函数。而在`C#`中，这种类似的活儿就让`delegate`承包了。

<!-- more -->

``` CSharp delegate的声明与使用
class Program
{
    /*delegate的声明，需要指定返回类型和参数列表，
        这里分别是void和string str。*/
    delegate void TestDelegate(string str);

    static void printSomething(string str)
    {
        Console.WriteLine(str);
    }

    static void callSomeMethods(TestDelegate someMethods)
    {
        someMethods("I am a delegate.");
    }

    static TestDelegate getSomeMethods()
    {
        return printSomething;
    }

    static void Main(string[] args)
    {
        TestDelegate someMethods = getSomeMethods();
        callSomeMethods(someMethods);
    }
}
```

输出：

```
I am a delegate.
```

　　可以看到，要声明一个delegate需要**delegate关键字、返回类型、delegate名称、参数列表**这四个部分，其中参数列表可以为空。就拿这里的`TestDelegate`来说，这相当于在C/C++写下`typedef void (*TestDelegate)(string str)`。很明显，delegate的声明比函数指针的声明来得要简洁，而且目的性也更强：`typedef`关键字在C/C++中表示声明一种任意类型的别名，不一定是函数指针，而`delegate`在C#中就只是用来声明对一种类型函数的引用。

------------------------------------

## delegate的性质

　　delegate与函数指针看似发挥着相同的作用，但其实delegate比函数指针更会玩，因为delegate是面向对象的和类型安全的。如何体现？

#### 面向对象

　　用过`C/C++`函数指针的娃十有八九会吐槽函数指针的不便性，特别是指向成员函数时，不仅声明语法恶心，而且使用起来也不够优雅，调用的时候还要知道对象实例地址才可以正确调用，比如：

``` C++
class C
{
public:
    int fun(int i)
    { 
        cout<<"You have entered a number : "<<i<<endl; 
    }
};

int main()
{
    C c;
    int (C::* pfn1)(int) = &C::fun;
    (c.*pfn1)(8);
    return 0;
}
```

　　C/C++中的函数指针在调用时这么拖泥带水，归根结底，因为它只是保存了函数的地址，没有保存对象实例的地址。而C#的delegate就意识到了这点，在引用某个方法时，若该方法通过某个对象实例来引用，则delegate会一并将对象实例的引用进行保存，而在引用静态方法时，行为跟函数指针一样，从之前的例子可以看到。

``` CSharp
class ClassWithSomeMethods
{
    private string m_name;

    public ClassWithSomeMethods(string name)
    {
        m_name = name;
    }

    public void methodInClass()
    {
        Console.WriteLine("I am " + m_name + ".");
    }
}

class Program
{
    delegate void TestMethodOfInstance();

    static void Main(string[] args)
    {
        ClassWithSomeMethods instanceNamedLGC = new ClassWithSomeMethods("LGC");
        ClassWithSomeMethods instanceNamedCGL = new ClassWithSomeMethods("CGL");
        TestMethodOfInstance method = instanceNamedLGC.methodInClass;
        method();
        method = instanceNamedCGL.methodInClass;
        method();
    }
}
```

输出：

```
I am LGC.
I am CGL.
```

#### 类型安全

　　在C/C++中函数指针经过强制转换就可以指向非相同类型的函数了，这就是一种类型不安全的体现之一，这容易造成程序的行为未定义。而在C#中，强制转换对delegate来说是没用的，编译会报错。此外，即使另外声明一个返回类型和参数列表都相同但名字不同的delegate，两者之间也是不能相互转换的，由此可知，delegate的唯一性是靠**返回类型、参数列表和delegate名字**来确定的。

``` CSharp
class Program
{
    delegate void DelegateCannotBeCasted(string str);
    delegate void DelegateCannotBeCasted2(string str);

    static void printSomething(string str)
    {
        Console.WriteLine(str);
    }

    static void saySomething()
    {
        Console.WriteLine("Something.");
    }

    static void Main(string[] args)
    {
        DelegateCannotBeCasted delegateCannotBeCasted = printSomething; //OK
        delegateCannotBeCasted = saySomething;
        delegateCannotBeCasted = (DelegateCannotBeCasted)saySomething;
        DelegateCannotBeCasted2 delegateCannotBeCasted2 = delegateCannotBeCasted;
        delegateCannotBeCasted2 = (DelegateCannotBeCasted2)delegateCannotBeCasted;
    }
}
```

![IntelliSense提示](http://obw0x5bwh.bkt.clouddn.com/Delegate-and-Event-in-CSharp/image/000.png)
![编译报错](http://obw0x5bwh.bkt.clouddn.com/Delegate-and-Event-in-CSharp/image/001.png)

#### 还没完，delegate没那么简单

　　除了面向对象、类型安全，delegate就跟函数指针一样了吗？不~delegate没那么简单。说delegate是对函数的引用其实只说了一半，delegate实际上能同时引用一堆的函数呢！下面就用一个简单的“发布-订阅”模式来说明。

``` CSharp
delegate void NewspaperPublishNotify(string whatNewspaper);

class NewspaperSaler
{
    public NewspaperPublishNotify newspaperPublishNotify;
    public void Publish(string whatNewspaper) {
        newspaperPublishNotify(whatNewspaper);  //通知所有订阅者
    }
}

class Buyer
{
    private string m_name;
    public Buyer(string name) { m_name = name; }
    public void Buy(string what) {
        Console.WriteLine(m_name + " buy " + what + ".");
    }
}

class Program
{
    static void Main(string[] args)
    {
        NewspaperSaler saler = new NewspaperSaler();
        Buyer buyerA = new Buyer("BuyerA");
        Buyer buyerB = new Buyer("BuyerB");
        Buyer buyerC = new Buyer("BuyerC");
        //买家ABC订阅报纸
        saler.newspaperPublishNotify += buyerA.Buy;
        saler.newspaperPublishNotify += buyerB.Buy;
        saler.newspaperPublishNotify += buyerC.Buy;
        //报纸发布
        saler.Publish("南方日报");
        //A取消订阅
        saler.newspaperPublishNotify -= buyerA.Buy;
        //报纸继续发布
        saler.Publish("广州日报");
        saler.Publish("人民日报");
    }
}
```

输出：

```
BuyerA buy 南方日报.
BuyerB buy 南方日报.
BuyerC buy 南方日报.
BuyerB buy 广州日报.
BuyerC buy 广州日报.
BuyerB buy 人民日报.
BuyerC buy 人民日报.
```

　　可以看到，delegate可以保存一堆函数引用，还能增加移除函数引用，就像个容器一样。不过，delegate没有对函数引用检查重复，所以即使你多次增加同样对象的同样方法程序也不会报错，而是按部就班地执行多次同样的方法，同样地，移除的时候也要移除多次才能干净。这些我就不在代码中体现了。

-----------------------------------------

## delegate破坏了封装性？

　　仔细的你会发现在上面的代码中，`NewpaperSaler`这个类中的delegate被声明为`public`，这破坏了面向对象中讲究的封装性。这很不安全，这意味着对象成员可以随便被外界更改，比如上面代码中如果`saler.newspaperPublishNotify -= buyerA.Buy;`不小心写成了`saler.newspaperPublishNotify = buyerA.Buy;`，结果就变成这样了：

```
BuyerA buy 南方日报.
BuyerB buy 南方日报.
BuyerC buy 南方日报.
BuyerA buy 广州日报.
BuyerA buy 人民日报.
```

　　A取消订阅变成了只有A订阅，BC都被挤掉了。当然，聪明的你立刻会想到把delegate由`public`改为`private`，另外再提供两个方法分别负责该delegate的增加移除函数引用。但是这样一来，delegate的使用就变得不够优雅了。其实你想到的粗重活在C#里已经提供了解决手段，那就是`event`。

## event前来搭救

　　只需对`NewspaperSaler`类做如下修改：

``` CSharp
//public NewspaperPublishNotify newspaperPublishNotify;
public event NewspaperPublishNotify newspaperPublishNotify;
```

　　那么，对`newspaperPublishNotify`的赋值操作就会导致编译报错了，提示“`event`只能出现在`+=`或者`-=`的左边”。这就是有了delegate之后还需要event的原因。
　　如果打开`ILSpy`来查看程序的中间语言，就会发现其实event是封装过的delegate，C#编译器对event的处理是，声明一个private同名的delegate，另外再提供了`add`和`remove`方法来分别对该delegate进行增加、移除函数引用操作，那么就是分别对应了`+=`和`-=`操作了。

![event的本质](http://obw0x5bwh.bkt.clouddn.com/Delegate-and-Event-in-CSharp/image/002.png)

　　用ILSpy还能看到很多C#的`dll`里面到底是什么，请自行快乐享用。

--------------------------------------------

## 总结

　　C#中的delegate是一种能存储多个函数引用的类型，该类型可以作为参数来传递，或者作为函数结果返回。通过delegate可以调用被引用的多个函数。此外，delegate还是面向对象和类型安全的。而event是对delegate的封装，限制了外界直接对delegate的重新赋值操作，以达到不破坏封装性又能方便实现事件订阅的目的。

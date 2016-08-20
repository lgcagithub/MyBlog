---
title: 小探C++中std::vector的容量增长
date: 2016-08-20 23:25:09
tags:
- C++
- STL
---

--------------------------------------
　　久闻C++中`std::vector`在插入新元素时，若遇到已分配容量不足的情况，会自动拓展容量大小，而这个拓展容量的过程为：
- 开辟另外一块更大的内存空间，该空间大小通常为原空间大小的两倍；
- 将原内存空间中的数据拷贝到新开辟的内存空间中；
- 析构原内存空间的数据，释放原内存空间，并调整各种指针指向新内存空间。

　　这是很多Cpper闭着眼都能念出来的东西，面试官也很喜欢问这样的问题，特别是`std::vector`的容量增长规律，而我之前也是闭眼张口就说出“新容量一般是原来的两倍”这样的答案。直到今天自己试了下，才发现自己之前回答时都没有漏掉“一般”这两个字而感到庆幸。

<!-- more -->

## 测试代码
``` C++ 测试std::vector容量增长
#include <iostream>
#include <vector>

template<typename T>
void printVectorInfo(const std::vector<T>& vec)
{
	std::cout << "vector::size() = " << vec.size() << "\t";
	std::cout << "vector::capacity() = " << vec.capacity() << std::endl;
}

int main()
{
	const int growCount = 10;   //指定测试的增长次数
	std::vector<int> intArr;
	size_t size = 0;
	size_t capacity = 0;

	printVectorInfo(intArr);

	for (size_t i = 0; i < growCount; ++i)
	{
		intArr.push_back(i);
		printVectorInfo(intArr);

		size = intArr.size();
		capacity = intArr.capacity();
        //插入足够的数据让下轮外层循环插入数据时可以引发容量增长
		for (size_t j = 1; size + j <= capacity; ++j)
		{
			intArr.push_back(size + j);
		}
	}

    return 0;
}
```

## 测试结果

#### Windows 7 64位 + Visual Studio 2015 Community
```
vector::size() = 0      vector::capacity() = 0
vector::size() = 1      vector::capacity() = 1
vector::size() = 2      vector::capacity() = 2
vector::size() = 3      vector::capacity() = 3
vector::size() = 4      vector::capacity() = 4
vector::size() = 5      vector::capacity() = 6
vector::size() = 7      vector::capacity() = 9
vector::size() = 10     vector::capacity() = 13
vector::size() = 14     vector::capacity() = 19
vector::size() = 20     vector::capacity() = 28
vector::size() = 29     vector::capacity() = 42
```

#### Ubuntu 16.04 LTS + g++ 5.3.1
```
vector::size() = 0      vector::capacity() = 0
vector::size() = 1      vector::capacity() = 1
vector::size() = 2      vector::capacity() = 2
vector::size() = 3      vector::capacity() = 4
vector::size() = 5      vector::capacity() = 8
vector::size() = 9      vector::capacity() = 16
vector::size() = 17     vector::capacity() = 32
vector::size() = 33     vector::capacity() = 64
vector::size() = 65     vector::capacity() = 128
vector::size() = 129    vector::capacity() = 256
vector::size() = 257    vector::capacity() = 512
```

## 结果分析
　　我就知道，结果肯定会有惊喜。在`Ubuntu`和`g++`的组合得到的测试结果倒是表现得中规中矩，老老实实地遵循了翻倍增长的规律。而`Windows`与`Visual Studio`这个组合的测试结果也符合微软一贯不按套路出牌的作风，不符合翻倍增长这个规律没有让我觉得奇怪，反而是它如何增长的倒是出乎我的意料。
　　乍一看，`Windows`下的头5行结果表现出来的是每插入一个数据就增长一个容量的这种吝啬行为，而后面接着插入数据却又表现出跳跃式的增长，4到6差2，6到9差3，9到13差4，这很容易让人以为下个容量会是18，然而立刻被打脸，结果是19，后面的更无规律了。
　　这种被耍的感觉很不好受，逼着我硬着头皮追踪代码。虽然`STL`的代码是难看了点，但是在`Visual Studio`的帮助下花点心思还是能找到自己想要的代码的。利用`F12`，沿着代码路径`push_back()`->`_Reserve()`->`_Grow_to()`追踪下去，发现`_Grow_to()`就是std::vector的增长规律，代码如下：

``` C++ std::vector容量增长规律关键代码
size_type _Grow_to(size_type _Count) const
{	// grow by 50% or at least to _Count
    size_type _Capacity = capacity();

    _Capacity = max_size() - _Capacity / 2 < _Capacity
        ? 0 : _Capacity + _Capacity / 2;	// try to grow by 50%
    if (_Capacity < _Count)
        _Capacity = _Count;
    return (_Capacity);
}
```

　　该函数会在std::vector剩余容量不足以容纳新增元素时被调用来计算最终的容量增长结果，传入的参数`_Count`表示刚好可以装下原有元素和本次新增元素的所需的位置个数。可以看到最后返回的是`_Capacity`，该变量的计算逻辑是：
- 首先获取原有容量；
- 然后判断`max_size()`的返回值是否小于`1.5`倍的原有容量，是的话`_Capacity`取0，否则`_Capacity`等于自身的`1.5`倍；
- 而最后`_Capacity`还要和`_Count`做判断，取两者中最大者。

　　其中，`max_size()`返回的是一个理论值，实际操作是：**用机器能表示的最大无符号数除以元素类型所占字节大小**。所以整段代码告诉我们，**Windows下的`std::vector`一般会按原有容量的1.5倍来增长，到了后期内存吃紧的时候或者一次性插入的数据个数多于原有数据的时候会改变策略为增长空间为刚好能容纳原有数据和新增数据**。
现在终于明白Windows下的std::vector是怎么增长的了，看似无规律地测试结果按照上面的代码在自己大脑里运行一下会发现是完全正确的。

　　既然看了Windows的STL代码，那么自然也很想看看Ubuntu的。本人不熟悉Linux开发，不清楚大家平时在Linux是怎么追踪代码的，所以下载了一个`VSCode`，配置一下凑合用。本以为，Windows下的STL代码是已经够难看的了，谁知道Ubuntu下的更加难看，不知怎么形容，哥不懂下划线的世界_(:зゝ∠)_ 。花了比在Windows上更多的时间，找到了代码路径`push_back()`->`_M_insert_aux()`->`_M_check_len()`，发现`_M_check_len()`就是Ubuntu下的std::vector容量增长规律，代码如下：

``` C++
size_type
_M_check_len(size_type __n, const char* __s) const
{
    if (max_size() - size() < __n)
        __throw_length_error(__N(__s));

    const size_type __len = size() + std::max(size(), __n);
    return (__len < size() || __len > max_size()) ? max_size() : __len;
}
```

　　该函数的调用时机与Windows的差不多，而`__n`是表示本次要新增的元素个数，而`__s`是用来作为异常信息的。可以看到一开始函数会检查理论最大容量个数减去当前已使用的个数是否小于新增容量`__n`，若真的小于的话，这时STL就没辙了，只能抛出异常了。接下来就是计算增长后的最终容量了，可以看到有个`max()`的操作，而对于`push_*`这样的操作，`__n`为1，一般会小于`size()`，所以这时候表现出来的就是翻倍增长，假如是一次插入很多个的时候，`__n`就有可能比`size()`大了，这时候也打破翻倍增长的规律，改为增长为刚好容纳原有数据和新增数据的容量了。

## 总结
　　总的来说，Windows下vector的容量增长得比较保守，所以空间占用相对来说就会比较少，但是这间接增加了大规模移动数据所需的次数。而Ubuntu下的刚好相反，一次增加多点容量，间接减少了大规模移动数据的次数。然而，如果你问我哪种方案比较好的话，我就无法下定论啦，毕竟时间和空间永远都是相互制约的，只有具体问题具体分析，根据实际情况来衡量了。
　　哎~以后还是别闭着眼睛随便说“空间翻倍增长”这种笑话了。╮(╯▽╰)╭
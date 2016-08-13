---
title: test new post
date: 2016-08-14 00:53:56
tags:
- test
- hexo markdown
---

普通正文文字

# 一级标题
## 二级标题
### 三级标题
#### 更小标题依此类推

[超链接](http://www.baidu.com "鼠标停留显示文字")

##### 插入代码(简单写法)
``` C "C HelloWorld" http://www.baidu.com HelloWorld.c
#include <stdio.h>

int main(int argc, char** argv)
{
	printf("Hello World!");
	return 0;
}
```

##### 插入代码(插件写法)
{% codeblock "C++ HelloWorld" lang:C++ http://www.baidu.com HelloWorld.cpp %}
#include <iostream>

int main(int argc, char** argv)
{
	std::cout << "Hello World!" << std::endl;
	return 0;
}
{% endcodeblock %}

文字间插入`code`文字间插入

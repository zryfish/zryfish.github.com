---
layout: post
title: "STL next_permutation"
description: "STL next_permutation函数"
category: 
tags: []
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天研究下`STL`中的`next_permutation`函数，发现代码写的真的是`beatuiful`。`next_permutation`通常是被重复调用以产生输入集合的全部排列，而且输入集合必须是按字典排好序的，意味着输入是类似`[1,2,3,4,5]`这样非降序形式。  
描述下程序是如何运行的：

1.  函数的输入是指向集合范围的两个`Iterator`，如果集合只有一个或者没有元素，直接返回`false`。否则，声明一个指向集合尾部的`Iterator i`。  
2.  进入循环，每次循环将`i`向集合头部移一位，直到:
  - `i`指向的元素比先前指向的元素更小  
  - `i`指向了集合的第一个元素  
3.  相当于将集合分割成了两个部分，后半部分是集合中最长的降序序列。如果后半部分是整个集合，返回`false`，没有下一个排列了。将集合反转下就得到整个按字典序排列的集合。
4.  如果后半部分不是整个集合，那么将`i`和`j`指向的元素交换下，并将后半部分反转下，就得到下一个排列了。

{% highlight c++ linenos %}
template<typename Iter>
bool next_permutation(Iter first, Iter last)
{
    if (first == last)
        return false;
    Iter i = first;
    ++i;
    if (i == last)
        return false;
    i = last;
    --i;
        
    for(;;)
    {
        Iter ii = i;
        --i;
        if (*i < *ii)
        {
            Iter j = last;
            while (!(*i < *--j))
            {}
            std::iter_swap(i, j);
            std::reverse(ii, last);
            return true;
        }
        if (i == first)
        {
            std::reverse(first, last);
            return false;
        }
    }
}
{% endhighlight %}

`INPUT 8342666411`  
`8342 666411`  
`834*2* 666*4*11`  
`834*4* 666*2*11`  
`834*4* 11*2*666`  
`8344112666`  

####参考
[1] [http://wordaligned.org/articles/next-permutation](http://wordaligned.org/articles/next-permutation)

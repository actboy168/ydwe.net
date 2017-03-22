---
layout: post
title: "坑爹的php"
date: 2014-04-07 09:41:54 +0800
comments: true
author: actboy168
categories: Other
---

周末研究了一下从c++调用php，看起来是一件很简单的事却折腾了我两天，总之是被坑惨了。虽然php是用c写的，不过看起来开发组只管自己用得溜就算了，完全没有考虑一般使用者的感受。

<!-- more -->

以下是我在msvc2010下编译php5.4.27的记录。

##坑之ZTS

php默认启用ZTS(php线程安全)，但ZTS宏却不是默认开启.如果你编译php时没有关掉ZTS，那么使用php api时会得到以下错误
```
error LNK2001: 无法解析的外部符号 __imp__compiler_globals
error LNK2001: 无法解析的外部符号 __imp__executor_globals
```
你需要自己定义ZTS
```
#define ZTS
```

##坑之PHP_WIN32、ZEND_WIN32

完全想不出不通过_WIN32宏来生成PHP_WIN32、ZEND_WIN32宏的理由，总之你是需要自己来定义了。还有为什么会有PHP_WIN32和ZEND_WIN32两个宏呢。
```
#define ZEND_WIN32
#define PHP_WIN32
```
另外，虽然php不会理会_WIN32宏，但却会理会_WIN64宏。


##坑之ZEND_WIN32_FORCE_INLINE

如果说之前的坑只是懒，那这个坑只能说是逗了。让我们来看看ZEND_WIN32_FORCE_INLINE都干了些什么。
```
#undef inline
#ifdef ZEND_WIN32_FORCE_INLINE
# define inline __forceinline
#else
# define inline
#endif
```

注意，默认是没有定义ZEND_WIN32_FORCE_INLINE的，所以当你引用了php的头文件，你代码中inline就会全部失效。c++标准库的头文件中也使用了inline，所以就算你没用inline，一样会让你的代码无法编译。错误通常为
```
error C2491: “std::flush”: 不允许 dllimport 函数 的定义
error C2491: “std::ws”: 不允许 dllimport 函数 的定义
```

所以ZEND_WIN32_FORCE_INLINE宏必须被定义，或者删掉这段略逗的代码。

##坑之_USE_32BIT_TIME_T

如果你没定义_WIN64宏，那么php会帮你定义
```
#ifndef _WIN64
# define _USE_32BIT_TIME_T 1
#endif
```
但msvc默认是不定义这个宏的，所以当你在未引用php头文件时，使用c++的头文件，使用的是64位time_t，引用php头文件后就变32位time_t了。错误通常为
```
stat.inl(42): error C2466: 不能分配常量大小为 0 的数组
```
或者
```
time.inl(36): error C2664: “_ctime32”: 不能将参数 1 从“const time_t *”转换为“const __time32_t *”
```
解决方法，删掉这段代码。

##坑之ZEND_DEBUG和UNICODE

_zend_executor_globals是php中一个很重要的结构体(也就是EG(xxx)),这个结构体中包含了一个[OSVERSIONINFOEX](http://msdn.microsoft.com/en-us/library/windows/desktop/ms724833%28v=vs.85%29.aspx)成员,而OSVERSIONINFOEX的长度会根据UNICODE宏的定义与否有所不同，php默认的编译选项是没有定义UNICODE宏的，如果你的工程定义了UNICODE宏，那么你就会获得很多迷のbug。ZEND_DEBUG也会有同样的问题，如果你编译时有--enable-debug，就必须定义ZEND_DEBUG，反之就不能定义。

php完全可以增加编译时的检查，以保证不会有类似的abi错误，但它没有这么做。

##坑之导出的全局变量

你可能无法想象，php不但很随意地使用了全局变量，并且还很随意地把它直接导出了。这种莫名其妙的设定遍地都是，不过我今天要说的是编译的坑。

表面上看，你在c++里直接使用php的头文件并没有什么问题，因为php在每个导出函数前都加了extern "C"，以保证c++编译器能找到php导出的函数。但这个并没有包括它导出的全局变量，所以当你在c++中使用它导出的全局变量时，就会得到一个链接错误。所以你必须给每个你要引用的php头文件加上extern "C"，就像这样
```
extern "C" {
#include <zend.h>
}
```
显然它原来自己加的extern "C"就没用了，倒不如你就像lua那样宣称我就是要用c，你要用c++就自己加extern "C"得了，做事做一半就不管了，算是个什么意思。

##坑之debug模式

当你编译为debug模式的php，php会为你做很多额外的检查，这本来是一件好事，但它最通常的做法就是直接崩掉，没有任何有用的提示；关掉debug模式的话，是ok的(至少表面看起来ok)。对于这种暴力的提示错误方式，我真不知道该如何招架，或许就php开发组的人玩得溜了。

---

<br>
<br>
最后吐槽下php的代码到处都充斥着丑陋的TSRM宏，无论是否开启ZTS，tsrm_ls都应该作为一个外部参数传入，php不应该也没必要在内部保留自己的状态，无论是否开启ZTS。

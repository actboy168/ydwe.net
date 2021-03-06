---
layout: post
title: "Jass字节码"
date: 2018-09-01 19:06:00 +0800
comments: true
author: actboy168
categories: YDWE
---

大多数的脚本语言中，都存在字节码。脚本语言先被编译器编译为另一种更基础的语言在运行时使用，这种语言就被成为字节码。Jass也是这样。

<!-- more -->

> 为什么需要字节码？

一般而言，字节码更利于计算机理解和执行，但不利于人类理解，而其对应的脚本语言则正好相反。

> 了解Jass字节码有什么好处？

魔兽有一个特殊的限制条件，单次虚拟机执行不能超过30万个字节码，一旦超过就会强制终止执行。如果你用YDWE来测试的话，YDWE会告诉你“超过了字节码限制”，就是这个意思。当然这个并不需要你深入地了解字节码。

在I2C/C2I还可以用的年代，code转为的整数，实际上就是字节码的序号，所以你可以用I2C(C2I(function XXX) + 5)这种方法来从函数中间开始执行。这就需要深入地了解字节码了。

无论如何，Jass字节码都是深入了解Jass的必备知识。

由于Jass字节码并不需要给人类阅读，所以它并不像Jass那样存在可读文件的格式，只有二进制格式。为了便于我们理解Jass字节码，我做了一个Jass的反汇编器，它可以将Jass字节码（二进制格式）转为可读文件的格式。它看起来是这样的

```
func BJDebugMsg
arg 1, string: msg
local integer: i
mov rC0, 0
mov i, rC0
label 00000001
mov rC1, integer: i
push rC1
call Player
push r00
mov rC2, 0
i2r rC2
push rC2
mov rC3, 0
i2r rC3
push rC3
mov rC4, 60
i2r rC4
push rC4
mov rC5, string: msg
push rC5
call DisplayTimedTextToPlayer
mov rC6, integer: i
push rC6
mov rC7, 1
pop rC8
add rC8, rC8, rC7
mov i, rC8
mov rC9, integer: i
push rC9
mov rCA, integer: bj_MAX_PLAYERS
pop rCB
eq rCB, rCB, rCA
jt rCB, 00000002
jmp 00000001
label 00000002
mov r00, nothing
return
endfunc
```

这个是BJ函数BJDebugMsg对应的字节码。和Jass不同，Jass里一行可以做很多操作，而Jass字节码一行只能做一个操作。这个不同的操作，我把它称之为操作码，在Jass字节码中，一共有41个操作码，它涵盖了Jass里的所有可进行的操作。不过在介绍操作码之前，还得先理解几个概念。

> 寄存器

寄存器大概可以理解为一个公用的临时变量。Jass字节码里一共有256个寄存器，分别以r00，r01，...，rFF命名。
寄存器的读和写比变量快很多，所以大多数的操作码都是和寄存器打交道，而不是变量。

> 调用栈

调用栈和寄存器类似，不过它更像是一个数组。调用栈的作用是在函数调用方和被调用方直接传递函数的参数。

> 全局变量和局部变量

在Jass字节码的层面上，全局变量和局部变量只在定义时有区别，在读写时没有区别。在读写时，Jass字节码会用变量名先从局部变量里查找变量，有则使用它，没有则取全局变量里找。


> 操作码

41个操作码我会逐一介绍一遍，以便你能对Jass字节码有全面的了解。

> 操作码：一元运算符

一共有3个一元运算符，分别是i2r、not、neg。一元运算符就是输入只有一个参数的运算，例如Jass里的not和-，分别对应Jass字节码的not和neg。i2r没有对应的Jass运算，在Jass里整数到实数是支持隐式转换的，也就是说一个函数的参数要求是实数，你传一个整数也是可以的，而在Jass字节码里，Jass编译器就会帮你添加一行i2r的操作码，帮你把整数转为实数。

一元运算符的参数是一个寄存器，它会将这个寄存器里的值进行一次运算，然后再把运算的结果放回这个寄存器。例如
```
neg r01 ; r01里的值是整数1
        ; 执行一次后，r01里的值是整数-1。
```

> 操作码：二元运算符

一共有13个二元运算符，分别是and、or、eq、neq、le、ge、lt、gt、add、sub、mul、div、mod。二元运算符的参数是三个寄存器，它会将第二第三个寄存器里的值作为输入进行一次运算，然后再把运算的结果放回第一个寄存器。例如

```
add r01, r02, r03 ; r02里的值是整数1,r03里的值是整数2
                  ; 执行一次后，r01里的值是整数3。
```

二元运算符和Jass的对应关系

|and|or|eq|neq|le|ge|lt|gt|add|sub|mul|div|mod|
|--|--|--|--|--|--|--|--|--|--|--|--|--|
|and|or|==|!=|<=|>=|<|>|+|-|*|/|%|

注意:取余运算(%)在1.29才被加到Jass里，但Jass字节码的取余运算(mod)是一直存在的。


> 操作码：移动运算符

移动运算符一共有7个，我为了简单起见，都统一使用mov，但每个mov的格式是不同的。

mov的含义是将一个值从某个地方原封不动地搬运到另一处地方。

1> 寄存器之间的移动

```
mov r01, r02 ; r01的值为整数0，r02的值为整数1
      ; 执行后，r01的值为整数1，r02的值为整数1
```

2> 寄存器到变量

```
mov udg_XXX, r01
```
注意：这里不区分全局变量和局部变量，它们的字节码是一样的。下同。

3> 变量到寄存器

```
mov r01, integer: udg_XXX 
```
从变量移动值到寄存器需要执行值的类型，jass字节码不信任变量上记录的类型信息，它总是以自己记录的类型信心为准，这就是为什么变量到寄存器需要类型，而寄存器到变量则不需要。下同。

4> 寄存器到数组变量

```
mov udg_XXX[r02], r01 ; r02的值为数组的索引
```

5> 数组变量到寄存器

```
mov  r01, integer: udg_XXX[r02] ; r02的值为数组的索引
```

6> 函数到寄存器

```
mov  r01, function BJDebugMsg
```
也就是将这个函数的code存到寄存器里

7> 字面值到寄存器

字面值的意思就是你在Jass里直接写出来的值，例如1、1.0、true、false、"字符串"等

```
mov  r01, 1
mov  r01, 1.0f
mov  r01, true
mov  r01, false
mov  r01, nothing
mov  r01, "字符串"
```
值得一提的是nothing，nothing也一个值的类型，在Jass里不存在nothing这种值类型，但Jass字节码里有。Jass唯一出现nothing的地方数函数的参数和返回值定义，实际上Jass字节码也是用于此。

> 操作码：新建变量

新建变量操作码有3个，分别是local（局部变量）、global（全局变量）、const（常量全局变量）。常量全局变量只用于编译时，在运行时和全局变量没有任何区别，理论上并不需要在字节码里区分常量全局变量和全局变量，但Jass字节码里为什么有，原因不明。

```
global integer: i
local integer: i
local integer array: j
```

另一个有趣的地方在于，Jass变量数组的属性也混在值类型里的，也就是说整数和整数数组是两个不同的类型。所以你能看到在定义j时，你把它的类型定义为整数数组，但在使用时(例如mov)，用的类型却是整数。也许我在文本格式的字节码里去掉这种歧义更好，不过至少目前我是原汁原味地保留。

> 操作码：跳转

和跳转相关的操作码有4个，分别是label、jmp、jt、jf。
label是一个占位符，没有实际的作用。
jmp，执行这个操作码就会跳转到指定的label。
jt，如果寄存器里的值为true，就会跳转到指定的label。
jf，如果寄存器里的值为false，就会跳转到指定的label。
```
label 00000001
jmp 00000001   ; 这是一个死循环
```

> 操作码：类型定义

一共有两个操作码，分别是type和extends。含义可以参考common.j最前面的几行。不过在Jass字节码里，它们实际上没有任何实际作用。

> 操作码：函数调用

和函数调用相关的操作码一共有9个，分别为func、endfunc、popn、push、pop、arg、call、return。其中call有两个，一个是call Jass函数，一个是call native函数，native函数就是不存在于Jass里的函数(也就是CJ函数)，但我们不太需要关注其中的区别。

func、endfunc只是标记一个函数的开始和结尾，但没有任何实际的用处。

push将寄存器里的值移动到栈顶，并使栈顶的计数+1。
pop将栈顶里的值移动到寄存器，并使栈顶的计数-1。
popn将栈顶计数-n。

例如

```
mov r01, 10  ; 栈顶的计数: 0, 栈：
push r01     ; 栈顶的计数: 1, 栈：10
mov r01, 11
push r01     ; 栈顶的计数: 2, 栈：10，11
mov r01, 12
push r01     ; 栈顶的计数: 3, 栈：10，11，12
pop r02      ; 栈顶的计数: 2, 栈：10，11    r02: 12
popn 2       ; 栈顶的计数: 0, 栈：
```

arg将新建一个局部变量，并将栈顶计数-n的值移动到这个局部变量里

```
mov r01, 10
push r01
mov r01, 11
push r01
arg 2, integer: i  ; 栈顶的计数: 2, 栈：10，11  i: 10
```

call 一次函数调用。
return 从函数调用返回。


说了这么多理论，来看一个实际的例子吧

```
func TEST
arg 2, integer: i
arg 1, integer: j
mov r00, integer: i
return
endfunc

mov r01, 10
push r01
mov r01, 11
push r01
call TEST
popn 2
```

函数参数传递的方式，就是利用栈。在调用函数前，将参数依次push到栈里，调用完毕后，用popn清理栈里的参数。对于被调用的函数，利用arg操作码读取栈里的值，并创建局部变量。

函数的返回，Jass字节码里函数返回值都放在r00寄存器里，所以只需要在调用函数后，从r00寄存器就可以取到返回值。

细心的童鞋可能已经发现了，我们根本不需要pop，实际上魔兽生成的Jass字节码里函数调用也确实没用上pop。大家可以看看前面BJDebugMsg的例子，里面就有用到pop的地方，由于操作码已经介绍完毕，所以大家可以试着阅读BJDebugMsg的例子，并理解pop到底在干什么，再回来看下面的内容。


> 魔兽生成的字节码

Jass字节码可以说也是一门语言，上面介绍的算是Jass字节码的一些语法。但就像每个人写的Jass可能都不同一样，Jass字节码不同的人也可以写出不同的东西来。所以我们现在来研究下魔兽它写的Jass字节码是什么样的。

首先Jass字节码有256个寄存器，这个数量可谓相当的多。为什么魔兽需要这么多寄存器呢，这就涉及到寄存器分配算法了。当我手写Jass字节码时，我经常会不停复用某些寄存器，例如
```
mov r01, 10
push r01
mov r01, 11
push r01
```
因为我知道什么时候r01的上一个值我不需要了，这时我就可以复用r01。同样地，一个合理的寄存器分配算法也是如此，这样就可以用很少的寄存器来做到你想做的事情。然而魔兽并非如此，它并不管寄存器里的值神时候失效，而是在所有的寄存器里循环使用。除去用于返回值的r00，一共255个寄存器，当用完255个寄存器后，上一轮的寄存器还在用的可能性几乎为零。所以每次魔兽需要新的寄存器时，它都会用当前寄存器的下一个。

不但如此，魔兽生成的字节码甚至不会在两个操作之间共享寄存器。考虑这行代码，会生成怎么样的字节码

```
set i = 1 + 2
```

通过上面的学习，我相信很多人都可以轻易的写出这样的字节码
```
mov r01, 1
mov r02, 2
add r03, r01, r02
mov i r03
```

但魔兽生成的字节码是这样的

```
mov r01, 1
push r01
mov r02, 2
pop r03
add r03, r03, r02
mov i r03
```

为什么会这样呢，我们可以分析分析我们手写的字节码有何难点在里面。对比add这一行，我们的第一个操作数是用r01，而魔兽是用r03。原因就在于此，魔兽它无法计算出第一个操作数的寄存器是什么。由于第二个操作数可能是一连串复杂的运算，并非寄存器-2就是第一个操作数的寄存器。所以魔兽把操作数push到栈里，在用二元运算符前，再把它pop出来。你会发现魔兽生成的字节码里，所有的二元运算符前都会有一个pop的操作，原因就在于此。

> 我们可以做些什么？

我们什么都不可以做。我们可以了解它，但我们无法对它做任何改变。大多数的脚本语言都会开放字节码，也就是运行时支持直接执行字节码，你不满意官方实现生成的字节码质量，你可以自己写一个编译器来生成自己的字节码，甚至你也可以定义另一个语言进而编译成相同的字节码。但这并不包括Jass。很早之前我曾经说过，希望blz可以开放字节码，让魔兽支持运行包含Jass字节码而非Jass的地图，这样魔兽的生态才能迎来新的变化。拾YDWE的牙慧，加那些blzapi，在我看来毫无意义。

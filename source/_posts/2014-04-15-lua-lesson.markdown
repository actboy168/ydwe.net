---
layout: post
title: "简单Lua教程"
date: 2014-04-15 20:38:16 +0800
comments: true
author: 最萌小汐
categories: YDWE
---

本文旨在让有jass基础的用户快速上手Lua,因此需要一定的jass基础.当然如果你有其他语言的基础那更好

<!-- more -->

##1.准备工作

最新版本的[YDWE](http://www.ydwe.net/download.html)

建议在配置中关闭预处理器,因为它和Lua的一个符号有冲突,当然你也可以选择回避使用这个符号.

##2.入口

使用```call Cheat("exec-lua: filename")```来运行地图中名称为"filename"的lua脚本

你可以选择在外部编写该脚本后导入地图,也可以通过YDWE在编辑器内编写并自动导入地图

##3.在YDWE内写Lua脚本

在YDWE的任何自定义代码区域输入以下内容

```lua
<?import("filename")[[

	script
		
]]?>
```

这段代码等价于你在地图内导入了一个名字为"filename",内容为"script"的脚本

你可能迫不及待想要尝试写Lua了,不过在这里我要建议你在运行自己的Lua脚本之前,先**单独**运行一次以下代码(通过```call Cheat("exec-lua: MU_console")```)

```lua
<?import("MU_console.lua")[[

	require("jass.runtime").console = true
	
]]?>
```

这段代码的效果是打开一个控制台窗口,这样如果你的其他Lua代码出现了错误,控制台窗口中将会显示出错误原因和错误位置,非常便利.此外通过Lua的print函数,你也可以将各种内容显示在这里

##4.hello world

没错,第一步是hello world

```lua
	print("hello world")
```

控制台窗口中将会显示"hello world"

##5.Lua的基本介绍

lua是一个简单,高效,强大的脚本语言,其语法和jass有很多相似之处,因此有jass基础的话可以很快掌握lua

lua和jass最大的不同在于lua不需要定义变量类型,例:

```lua
	local a = 1
	if a == 1 then
		a = "等于1"
	end
```

在这个例子中,a的类型分别为数字(number)与字符串(string)

lua的注释符为 ```--```

你也可以进行多行注释

```lua
	--[[
		我已经被注释掉了
	--]]
```

####在lua中有以下几个类型:

* nil   空值,任何变量在赋值前都是nil,给变量赋值nil相当于摧毁该变量,类似于jass中的null

* boolean   布尔值,与jass一样包含true与false

* number   数字,lua中的数字没有整数或实数之分,因此在lua中 5 / 2 == 2.5

* string   字符串,与jass中的字符串基本相同

* table   表,lua中最强大的类型,他可以简单的当做数组或哈希表使用,更是lua构成复杂的高级功能的基础,这将在之后重点学习

* function   函数,与jass中的函数不同,lua中的函数也被视为一个值,你可以随时将它赋值给一个变量,或在其他函数中作为参数或返回值传递

* userdata   自定义数据,你可以将其理解为jass中的handle,lua无法直接对其进行修改

####lua有以下几个保留字:

```lua
	and break do else elseif
	end false for function if
	in local nil not or
	repeat return then true until
	while
```

与jass很像不是吗?

##6.值之间的操作

####运算
```lua
	1 + 2 == 3
	3 * 4 == 12
	2 - 6 == -4
	7 / 2 == 3.5
	4 ^ 2 == 16
	5 % 1.5 == 0.5
	1 .. 1 == "11"
	"6" + "7" == 13
```
lua在必要的时候会自动转换类型,因此需要特别留意字符串是通过 .. 来连接的

此外lua还自带了 幂(^) 与 取模(%)

####逻辑
```lua
	if x == true then --1
		a = 1
	elseif x then --2
		a = 2
	elseif not x then --3
		a = 3
	elseif x and y then --4
		a = 4
	elseif x or y then --5
		a = 5
	end
```
lua的if结构与jass相似,只要注意endif要改成end

第2个条件中,只有当x的值为nil或false才不成立,其他情况包括0或""都是成立的


##7.变量

lua对变量进行操作时不需要 set 关键字

lua使用全局变量无须事先声明,任何时候```a = 10```都是有效的.在a被赋值前,如果去获取a的值将返回"nil"

局部变量的声明方式类似于jass:

```lua
	local a = 10
```

局部变量可以在任何位置声明,影响范围由声明的位置而定,例如:
```lua
	local a = 10
	if a then
		local a = 20
		print(a)
	end
	print(a)
```
将依次显示20与10

记住,lua不需要写变量类型哦

你可以一次声明多个变量

```lua
	x, y = 10, 20 --x = 10, y = 20
	x, y , z = 10, 20 --x = 10, y = 20, z = nil
	x, y = 10, 20, 30 --x = 10, y = 20, 30被丢弃
	x, y = y, x --x = 20, y = 10
```

##8.字符串

lua的字符串有3种符号,例:
```lua
	text1 = "I'm hungry"
	text2 = '"That loli seems delicacies!", I said'
	text3 = [[
"呜喵"\n"呜喵"\n"呜喵"
	]]
```

其中用[[]]表示的字符串将忠实的记录你输入的文字,忽略其中的任何转义符.如果你```print(text3)```,控制台将显示
```
"呜喵"\n"呜喵"\n"呜喵"
```

需要注意的是,[[]]形式的字符串可能会与你的脚本产生一些冲突,例如
```lua
	text = [[
		a = t[v[5]]
	]]
```
你可以改写成如下的形式
```lua
	text = [===[
		a = t[v[5]]
	]===]
```
这样lua只会把拥有相同等号数量的括号认为是一组

同样的,你可以将YDWE内写lua的代码改成如下代码:
```lua
<?import(filename)[=====[

	script
	
]=====]?>
```
以回避一些冲突

##9.简单循环

lua有3种简单循环方式

####① for循环

最常用的循环方式

直接看例子

```lua
	for i = 1, 5 do
		print(i)
	end
```

将依次打印 1, 2, 3, 4, 5

你也可以指定循环的步长

```lua
	for i = 1, 5, 2 do
		print(i)
	end
```

将依次打印 1, 3, 5

需要注意的是,你无法通过修改i的值来控制循环

####② while循环

当条件成立时进行循环

```lua
	local i = 1
	while i <= 5 do
		print(i)
		i = i + 1
	end
```

####③ repeat循环

当条件成立时退出循环

```lua
	local i = 1
	repeat
		print(i)
		i = i + 1
	until i > 5
```

以上三种循环你都可以使用break来提前退出,常见的用法有

```lua
	local i = 1
	while true do
		print(i)
		i = i + 1
		if i > 5 then
			break
		end
	end
```

需要注意的是,break必须放在一段代码的最后面

```lua
	while true do
		break --错误,后面还有代码
		local i
	end

	while true do
		do break end --正确.do XXX end是一段完整的代码,break确实放在了这段代码的最后面
		local i
	end
```

##10.简单的数据结构

lua只有一种数据结构:table(表)

```lua
	t = {} --创建一张表并赋值给变量t
	
	t[1] = 10
	t[2] = 20
```

任何非nil的值都可以作为table的索引

```lua
	t = {}

	t["x"] = 10
	t[true] = 20
	t[1.23] = 30
	t[t] = t
	t[GetTriggerUnit()] = "呜喵"
```

在lua中,可以将```t["x"]```简写成```t.x```

```lua
	t["x"] == t.x
	t["1"] ~= t[1]
```

你可以在创建table的时候直接定义其数组内容,注意lua中的数组索引是从1开始的(jass是0),多个元素之间需要用逗号隔开

```lua
	t = {1, 2, 3, 4, 5} --相当于t[1] = 1, t[2] = 2, t[3] = 3, t[4] = 4, t[5] = 5
	t = {x = 10, y = 20} --相当于t.x = 10, t.y = 20

	t = {
		1,        -- t[1] = 1
		x = 10,   -- t.x = 10
		y = 20,   -- t.y = 20
		2         -- t[2] = 2
	}
```

通过嵌套table,你可以很简单的构造出多维数组

```lua
	t = {}
	t[1] = {}
	t[1][2] = "呜喵"
```

随着你对lua的深入,你可能会越来越多的使用这样的结构:

```lua
	hero = {
		name = "虐农先锋",
		hp = {500, 1000},
		mp = {200, 300},
		["坐标"] = {50, 100},
		["技能"] = {
			["风暴之锤"] ={
				id = |A000|,
				mp = 75,
				cd = {10, 8, 6}
			},
			["雷霆一击"] ={
				id = |A001|,
				mp = {90, 100, 120},
				cd = {15, 14, 13}
			},
		},
		["移动速度"] = 270
	}
```

##11.函数的声明

lua的函数声明有2种形式:

```lua
	function func(a, b, c)
	end
```

```lua
	func = function(a, b, c)
	end
```

这2种声明方式是完全等价的,都是声明了一个名字为"func",参数为"a, b, c"的函数

显然这里的参数不需要填写类型,函数也不需要声明返回值的类型,顺便将endfunction改为end即可


从第二种声明方式中我们可以看出,lua中的函数是可以赋值给变量的:我声明了一个新的函数,然后将其赋值给变量"func"

一个简单的函数重载:

```lua
	R2I = math.floor --math.floor是lua自带的取整函数
	R2I(1.5) --会去调用lua的math.floor而不是jass的R2I
```

既然是变量,那么他也就可以局部化:

```lua
	function func1()
		local func1 = function()
			return 1
		end

		local func2 = function()
			return 2
		end

		return 0
	end

	print(func1()) -- 0
	print(func1()) -- 0
	print(func2()) -- func2不存在
```

lua不限定函数的返回值,你可以返回或不返回;返回一个或返回多个;返回布尔或返回整数:

```lua
	function func(a)
		if a = 1 then
			return 1
		elseif a == 2 then
			return 1, "wumiao"
		end
	end

	print(func(1)) -- 1
	print(func(2)) -- 1	wumiao
	print(func(3)) -- nil
```

需要注意的是,与break一样,return必须放在一段代码的最后面

##12.函数的调用

lua调用函数显然是不需要写call这个关键字的,而且函数作为一个变量,只要他被赋值了你就可以去调用他,因此也没有声明顺序的问题

不过你依然需要关注一下局部函数的有效范围:

```lua
	local func = function(a)
		a = a or 1
		if a < 10 then
			func(a + 1) -- 错误,函数"func"没有被声明
		end
	end
```

正确的写法应该是

```lua
	local func
	func = function(a)
		a = a or 1
		if a < 10 then
			func(a + 1)
		end
	end
```

lua不关心你调用函数时的参数是否与函数声明时的参数一致,多余的参数被抛弃,不足的参数为nil,类似于多变量赋值

```lua
	func = function(a, b, c)
		print(a)
		print(b)
		print(c)
	end

	func(1) -- 1, nil, nil

	func(1, 2, 3, 4) -- 1, 2, 3
```

同样的,一个拥有多返回值的函数也有类似的情况:
```lua
	func = function()
		return 1, 2, 3
	end

	a, b, c = func() -- a = 1, b = 2, c = 3
	a, b = func() -- a = 1, b = 2, 3被抛弃
	a, b, c, d = func() -- a = 1, b = 2, c = 3, d = nil
```

##13.表的简单操作

jass中是直接定义某个变量为数组的,而lua中则是将一个表赋值给一个变量,显然表也可以作为一个普通的对象进行传递

```lua
	t = {[0] = 0, 2, 4, x = 10, y = 20, z = 30, 6, 8, 10}
```

这张表分为了2个部分,数组与哈希表.lua对数组的定义是从索引1开始的连续元素,因此t[0]是哈希表部分,而将t[3]截断后,之后元素不再连续,此时的数组大小便为2.我们可以通过符号#来获取数组部分的长度

```lua
	print(#t) -- 5

	t[3] = nil
	print(#t) -- 2
```

有2种简单的方式来遍历表的数组部分

```lua
	for i = 1, #t do
		print(i .. ":" .. t[i]) --显示 1:2, 2:4, 3:6, 4:8, 5:10
	end

	for i, v in ipairs(t) do
		print(i .. ":" .. v) --显示同上
	end
```

第二种方式也是循环的一种,既然循环,他也可以用过break来提前退出

lua也提供了1种简单的方式来遍历表的所有元素

```lua
	for i, v in pairs(t) do
		print(i .. ":" .. v) --可能显示 1:2, 2:4, 3:6, 4:8, 5:10, x:10, y:20, z:30
	end
```

在例子中我之所说是可能显示,是因为哈希表部分的遍历顺序是随机的,与你存储的顺序无关.不过他依然会按照顺序先遍历出数组的部分

lua提供了2个常用的插入/移除函数:table.insert与table.remove

```lua
	t = {1, 2, 3}

	table.insert(t, 2, 4) --将4插入到t[2]的位置,之前从t[2]开始的元素都被挤得往后挪一个位置,变为{1, 4, 2, 3}

	table.insert(t, 5) --在t的末端添加元素,变为{1, 4, 2, 3, 5}

	table.remove(t, 2) --移除t[3],之前从t[4]开始的元素都向前移动一个位置填补空缺,变为{1, 4, 3, 5}

	table.remove(t) --移除t末端的元素,变为{1, 4, 3}
```


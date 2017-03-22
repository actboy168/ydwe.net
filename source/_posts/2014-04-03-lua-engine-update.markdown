---
layout: post
title: "Lua引擎兼容性修改指导"
date: 2014-04-03 00:09:33 +0800
comments: true
author: actboy168
categories: YDWE
---

刚开始做Lua引擎时，只是抱着玩玩的心态，很多东西没有认真地设计。所以在YDWE1.27，Lua引擎进行了大规模的修改，为了确保你的代码可以顺利过渡到1.28以上的YDWE，请仔细阅读文本。注：1.27支持新旧两种写法，但1.28只会支持新写法。

<!-- more -->

##Lua引擎加载

旧写法
```
call Cheat("run main.lua")
```

新写法
```
call Cheat("exec-lua: 'main.lua'")
call Cheat("exec-lua: \"main.lua\"")
call Cheat("exec-lua: main.lua")
call Cheat("exec-lua:main.lua")
call Cheat("exec-lua: main.lua ")
```
新写法中可以让你的代码看起来更清晰，同时也更加灵活.

##内置库不再默认加载

旧写法

``` lua
jass.DisplayTimedTextToPlayer(cj.GetLocalPlayer(), 0, 0, 60., 'hello')
```

新写法
``` lua
local jass = require 'jass.common'
jass.DisplayTimedTextToPlayer(jass.GetLocalPlayer(), 0, 0, 60., 'hello')
```

在新写法中，Lua引擎不在预定义**任何**全局变量，包括jass/japi/jass_ext/slk。你需要在使用之前用require加载它。这样可以保证Lua引擎不会跟你的代码(的变量名)冲突。同时如果某些库你不用的话，就永远不会被加载，减少资源的消耗。

你可以把这段代码添加到所有代码的最前面，这样就能保证新写法和旧写法完全等价，不过我还是建议你使用局部变量，并且去掉不使用的库。
``` lua
jass = require 'jass.common'
japi = require 'jass.japi'
slk = require 'jass.slk'
jass_ext = {}
jass_ext.hook = require 'jass.hook'
jass_ext.runtime = require 'jass.runtime'
```

##控制台激活改变

旧写法
``` lua
jass_ext.EnableConsole()
```

新写法
``` lua
local runtime = require 'jass.runtime'
runtime.console = true
```

同样地你可以使用以下代码来作兼容
``` lua
function jass_ext.EnableConsole()
	local runtime = require 'jass.runtime'
	runtime.console = true
end
```

##|AHbz|的写法被移除

这不是一个标准的Lua语法，在Lua5.3中增加了|作为运算符，所以|AHbz|这种写法在5.3中会产生问题，我思考再三，还是决定将这种写法移除。不过值得庆幸的是，在5.3中加入了一个新函数string.unpackint，他可以将一个字符串当作二进制数组转为一个整数，这恰恰是可以让一个字符串"AHbz"转为整数的'AHbz'（注意我这里使用了Jass的写法）。就像这样

``` lua
string.unpackint('AHbz', 0, 4)
```

或者这样
``` lua
function ID(id)
	return string.unpackint(id, 0, 4)
end

ID 'AHbz'
```


不过目前Lua5.3还有很多不确定性，比如string.unpackint的第三个参数默认值总是8，所以你不能写成这样
``` lua
string.unpackint('AHbz')
```

或许在正式版中会有所改变，所以|AHbz|的写法**暂时**保留，不过最终应该还是会被我去掉的。

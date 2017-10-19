---
layout: post
title: "在lua中支持replay"
date: 2017-10-18 10:56:00 +0800
comments: true
author: actboy168
categories: Other
---

在游戏中replay是个必不可少的功能，无论是玩家的需求还是对开发和调试的帮助。但是lua的官方实现并不能很好地支持replay的实现。

<!-- more -->

replay的关键在于所有行为都是确定且可重现的。对于随机函数math.random很简单，lua提供了设置随机种子的函数math.randomseed，相同的随机种子总是产生相同的随机序列，我们只需要保证replay使用了相同的随机种子即可。

但是lua还有一个随机行为，考虑以下代码

``` lua
local t = {
    a = 0, b = 1, c = 2, d = 3, e = 4,
    f = 5, g = 6, h = 7, i = 8, j = 9,
}
for k, v in pairs(t) do
    print(k, v)
end
```

你会发现每次的输出都是不一样的。lua遍历表的次序取决于key的hash值，而lua的很多数据类型的hash都是随机的，这包括了string、table、userdata等。而且它并不取决于math.randomseed指定的随机种子，所以不能很简单地解决它。

# 重写pairs函数？

既然pair有随机性，那么我们来写一个没有随机性pairs吧。

``` lua
local function sortpairs(t)
    local sort = {}
    for k, v in pairs(t) do
        sort[#sort+1] = {k, v}
    end
    table.sort(sort, function (a, b)
        return a[1] < b[1]
    end)
    local n = 1
    return function()
        local v = sort[n]
        if not v then
            return
        end
        n = n + 1
        return v[1], v[2]
    end
end
local t = {
    a = 0, b = 1, c = 2, d = 3, e = 4,
    f = 5, g = 6, h = 7, i = 8, j = 9,
}
for k, v in sortpairs(t) do
    print(k, v)
end
```

这是一个根据key排序次序遍历的pairs，但是这个pairs也有几个问题

1. 需要遍历两次表加上一次排序，效率低。
2. 现在的输出是a、b、c、d...，但我更希望是看起来是随机的次序，这样行为看起来会和原版的pairs更像。这就需要在排序比较函数里加上一个hash函数。然而由于lua无法高效地访问字符串，所以如果hash函数不用c实现的话，效率低得可怕。
``` lua
table.sort(sort, function (a, b)
    return hash(a[1]) < hash(b[1])
end)
```
3. hash函数需要保证没有冲突。由于第一次遍历是随机的，所以两个hash值相同的项遍历的次序也是随机的。
4. userdata、table等数据类型的支持。

# 重写lua的hash函数

其实lua的table在插入时，就做了类似我们上面sortpairs所做的事情，对key取hash然后排序插入表，pairs所做的就是按次序遍历了一遍排序的结果。只是lua的这个hash函数带有随机性，所以才让pairs带有随机性。lua的hash函数在ltable.c中

``` c
static Node *mainposition (const Table *t, const TValue *key) {
  switch (ttype(key)) {
    case LUA_TNUMINT:
      return hashint(t, ivalue(key));
    case LUA_TNUMFLT:
      return hashmod(t, l_hashfloat(fltvalue(key)));
    case LUA_TSHRSTR:
      return hashstr(t, tsvalue(key));
    case LUA_TLNGSTR:
      return hashpow2(t, luaS_hashlongstr(tsvalue(key)));
    case LUA_TBOOLEAN:
      return hashboolean(t, bvalue(key));
    case LUA_TLIGHTUSERDATA:
      return hashpointer(t, pvalue(key));
    case LUA_TLCF:
      return hashpointer(t, fvalue(key));
    default:
      lua_assert(!ttisdeadkey(key));
      return hashpointer(t, gcvalue(key));
  }
}
```

这里面有随机性的是LUA_TSHRSTR、LUA_TLNGSTR、LUA_TLIGHTUSERDATA、LUA_TLCF和default。


# 遍历string key

先说字符串，字符串在计算hash时引入了一个随机种子，这个随机种子和math.randomseed不一样，在lstate.c里的lua_newstate初始化。

``` c
LUA_API lua_State *lua_newstate (lua_Alloc f, void *ud) {
  int i;
  lua_State *L;
...
  g->mainthread = L;
  g->seed = makeseed(L);
  g->gcrunning = 0;  /* no GC while building state */
...
```

``` c
static unsigned int makeseed (lua_State *L) {
  char buff[4 * sizeof(size_t)];
  unsigned int h = luai_makeseed();
  int p = 0;
  addbuff(buff, p, L);  /* heap variable */
  addbuff(buff, p, &h);  /* local variable */
  addbuff(buff, p, luaO_nilobject);  /* global variable */
  addbuff(buff, p, &lua_newstate);  /* public function */
  lua_assert(p == sizeof(buff));
  return luaS_hash(buff, p, h);
}
```

为了保证这个随机种子的随机性，lua引入了多种随机变量来保证随机性。所以最好的办法就是重写makeseed，或者把随机种子从lua_newstate的参数传入。因为lua_newstate只需要在lua_State初始化时调用一次，所以随机种子从lua_newstate的参数传入是个不错的方案。

这样我们就可以通过传入的随机种子来改变pairs遍历字符串key时的次序了。

# 遍历userdata key

hash的关键在于相同的值，总是能得到相同的结果。由于lua不知道userdat的d具体含义，无法和string那样根据值来计算hash。不过值得庆幸的是lua里同一个userdata只有一个副本，所以我们只需要在userdata初始化时给它一个全局唯一的ID，就可以作为它的hash值了。

首先要修改userdata的数据结构，加一个hash值，这个在lobject.h中

``` c
/*
** Common Header for all collectable objects (in macro form, to be
** included in other objects)
*/
#define CommonHeader	GCObject *next; lu_byte tt; lu_byte marked
```

然后在初始化时，给一个全局唯一的ID。
``` c
GCObject *luaC_newobj (lua_State *L, int tt, size_t sz) {
  global_State *g = G(L);
  GCObject *o = cast(GCObject *, luaM_newobject(L, novariant(tt), sz));
  o->marked = luaC_white(g);
  o->tt = tt;
  o->next = g->allgc;
  g->allgc = o;
  return o;
}
```

最后需要修改mainposition函数，userdata属于default分支里。需要注意的是，我们并没有修改所有的GCObject，所以计算hash只能计算我们已经修改过的GCObject(所有的userdata)。

# table，thread，function和lightuserdata

其实在修改userdata的时候，我们已经给所有的GCObject对象添加了hash值，所以table和thead我们也可能略作修改就可以支持，只是thread的初始化函数是lua_newthrad而不是luaC_newobj。

而lightuserdata和c function并不是GCObject，修改起来比GCObject麻烦一点，并且需要使用的情况很少，所以我就不改了。lua function虽然也是GCObject，但是在lua层面上，c function和lua function是没有区别的，为了统一也一并不作支持。

# 最后
最终的实现，可以参考我YDWE中的[lua副本](https://github.com/actboy168/YDWE/tree/master/OpenSource/Lua)。

除了replay的支持外，确定行为的代码对于帧同步的多人游戏也尤为的重要。魔兽就是一个帧同步的游戏。简单来说，所有的客户端都会跑同一份代码，定时校验不同客户端之间的状态是否一致，不一致就会导致掉线。如果同一份代码同样的输入却得到不同的输出，就很容易出现状态不一致从而掉线了。


---
layout: post
title: "X-Buff"
date: 2015-09-01 17:01:54 +0800
comments: true
author: actboy168
categories: X-Editor
---

虽然在魔兽编辑器中，buff是一个很弱的概念，我们也很难围绕它干一些事情。但实际上，在现代的RPG类的游戏里，buff是一个很重要的组成部分，甚至不亚于技能。今天就给大家介绍X-Editor中的buff系统。

<!-- more -->

##buff的事件

buff有5个事件，分别是添加事件、删除事件、叠加事件、完成事件和心跳事件。添加事件和移除事件就是在buff被添加和被删除时触发的事件，如果你要做这样一个技能
```
使用后，你的攻击力增加50，持续10秒。
```
你只需要做一个buff，在buff的添加事件里，给buff的所有者增加50点攻击力，在删除事件里，给buff的所有者减少50点攻击力，并设置buff的持续时间为10秒，那么这个buff就做好了。

当单位上已经有一个同名buff，而又再次添加这个buff时就会触发叠加事件(在添加事件之前)。如果叠加失败，则不会触发添加事件。同样是之前的例子，如果你第一次使用之后3秒，再次使用这个技能，那么你希望发生什么事情？
1. 单位增加50点攻击力，持续7秒。
2. 单位增加50点攻击力，持续10秒。
3. 单位增加100点攻击力，持续10秒。
4. 单位增加100点攻击力，持续7秒，之后增加50点攻击力，持续3秒。

这是最常见的四种buff的叠加模式。我们先来考虑更普遍的问题，实际上buff的叠加可以分为两类，共存模式和独占模式。共存模式即是所有同名buff都会共存，独立作用；独占模式即是同名buff在同一时间只会存在一个。情况1~3实际上都是独占模式，情况4是共存模式。情况1，实际上就是再次使用技能什么事情都没有发生，也就是说新buff不能覆盖旧buff；情况2，实际上就是再次使用技能会刷新buff的数据，也就是说新buff会覆盖旧buff。

在新buff添加到单位身上时，会依次触发单位身上同名buff的叠加事件，你需要在事件中返回新旧buff的优先级谁大谁小。（优先度的概念下面还会用到，现在你可以简单地理解为需要返回是新buff可以覆盖旧buff还是新buff不可以覆盖旧buff）所以，情况1和情况2的叠加事件，你只需要简单地返回false和true，就能达到效果。而情况3，实际上是让新buff失败，但同时更新了旧buff的数据。（顺便说一句情况2还有更优的解，让新buff失败，但把新buff的数据复制到旧buff上）而情况4是共存模式，你只需要把buff设置为共存模式，那么所有同名buff就会共存，互不干涉。

buff设定的时间到期后，会依次触发完成事件和删除事件，而被主动删除时，则只会触发删除事件。一般完成事件是用来做一些必须达到指定时间才能触发的效果。心跳事件是指在buff在持续时间内，会定期触发的事件，比如用来做流血、中毒之类的效果。


##buff的叠加模式

在叠加事件中，我已经提到了buff有两种叠加模式，共存模式和独占模式。我们来看一个更复杂的例子，在魔兽中，鞋子的移动速度是不能叠加的，只取最高值。如果我们要用buff来做，要怎么做呢？显然我们的目的是要让同名buff只有效果最高的有效。首先考虑用共存模式还是独占模式，虽然buff只有最高的有效，但是考虑到如果最高的那个buff被移除后要让次高的buff生效，所以我们不能用独占模式，必须要让所有的同名buff都留下来。那么答案就出来了，我们需要在添加事件、删除事件和覆盖事件中，遍历单位身上的同名buff，找到效果最高的移动速度，用它来修改单位的移动速度。接下来还有另外一个问题，因为单位身上有多个同名的buff(但实际上只有一个起作用了)，我们还必须让生效的那个buff显示图标，而其他的不显示；亦或者再做一个纯显示用的buff，实际效果的buff都不显示。

是不是听起来有点复杂？所以对于这种buff，X-Editor里有更简单的做法。之前我有说过，覆盖事件实际上是比较新旧buff的优先级。独占模式下，只有优先级高的buff能留下来。而共存模式下，无论什么优先级都会留下来，但是所有的buff都会按优先级由高到低排序，我们可以把移动速度等价于buff的优先度，那么所有的同名buff就会按照增加的移动速度由高到低排列。此外X-Editor的buff还有一个参数，可以指定最多允许几个buff同时生效，共存模式下默认为无限个，但我们可以把它设置为1，这样就只有优先度最高的那个buff有效了。

``` lua
local mt = ac.buff['鞋-移动速度']
-- 死亡之后保持buff
mt.keep = true
-- 共存模式
mt.cover_type = 1
-- 只有1个同名buff生效
mt.cover_max = 1
-- 添加事件
function mt:on_add()
	self.target:add_move_speed(self.move_speed)
end
-- 删除事件
function mt:on_remove()
	self.target:add_move_speed(-self.move_speed)
end
-- 叠加事件
function mt:on_cover(new)
	return self.move_speed < new.move_speed
end
```

考虑这种情况，有个buff你希望他是独占模式的，但是如果有两个相同的英雄时会出现什么情况呢，有可能两个相同英雄的buff就会相互覆盖。有时候是合理的，有时候却是不合理的。在X-Editor里，所谓的同名buff，实际上指的是buff的名字相同且buff来源也相同，所以两个相同英雄的buff之间不会相互影响。但如果你需要让不同来源但名字相同的buff被当作同名buff处理，你需要为这个buff指定为全局的buff，这样才会变成名字相同即为同名buff。

##模版buff

我们再来讨论流血、中毒等dot类的buff的做法。首先所有的dot都需要能独立作用，所以只能用共存模式，但共存模式有几个缺点，一是同名buff很多时浪费资源，二时大多数情况下我们只希望看到一个buff图标，所以我们可能需要做一个假buff来显示图标。实际上dot类的buff，我们也可以用独占模式做。我们需要把dot每一跳的伤害提前算好存在一个数组里；在叠加事件中，把新buff的每一跳伤害存到对应的数组位置上；在心跳事件中，我们只需要把数组中的对应的伤害直接拿出来用即可。这就是用独占模式来做dot的方法，不过我要说的并不是这个。我们可以看到dot并不是一个buff，而是一类buff，同一类buff有很多代码是相同的，在X-Editor中，会有很多这样的buff模版，你可以很简单地做出一个独占模式的dotbuff。(共存模式的dotbuff，太过于简单不需要模版)

此外在X-Editor中，光环和法球都是属于模版buff。光环就是一个会定时把自身添加给周围单位的buff，这个buff在非所有者身上时，只会持续很短的时间并且失去传染的能力。这是魔兽里圣骑士专注光环的完整实现


``` lua
local mt = ac.aura_buff['专注光环']
-- 魔兽中两个不同的专注光环会相互覆盖，但光环模版默认是不同来源的光环不会相互覆盖，所以要将这个buff改为全局buff。
mt.cover_global = 1
function mt:on_add()
	-- 如果这个buff在所有者身上，则加上一个比较特别的特效
	if self.source == self.target then
		self.source_eff = self.target:add_effect('origin', [[Abilities\Spells\Human\DevotionAura\DevotionAura.mdl]])
	end
	-- 受到影响的单位都会有一个简单的特效
	self.target_eff = self.target:add_effect('origin', [[Abilities\Spells\Other\GeneralAuraTarget\GeneralAuraTarget.mdl]])
	-- 增加护甲
	self.target:add_defence(self.data.defence)
end
function mt:on_remove()
	if self.source_eff then self.source_eff:remove() end
	self.target_eff:remove()
	self.target:add_defence(-self.data.defence)
end

hero:add_buff '专注光环'
{
	-- buff的来源是自己
	source = hero,
	-- buff的数据，会在所有自己的子buff里共享这个数据表
	data = {
		defence = 1.5
	},
	-- 光环的选择器
	selector = ac.selector()
		: in_range(hero, 900) -- 半径为900的圆
		: is_ally(hero)       -- 友方单位
		: of_not_building()   -- 不是建筑
		,
}
```

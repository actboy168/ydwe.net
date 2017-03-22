---
layout: post
title: "Jass编辑器推荐 Everedit"
date: 2014-03-30 21:20:09 +0800
comments: true
author: actboy168
categories: Tool
---

从JassCraft到JassShopPro，再到Notepad++，以及这次的主角Everedit，我换了不少Jass的编辑器。算起来，我一周大概有50个小时以上在写代码，所以一个称手的编辑器对我来说至关重要。先不说对Jass的支持，单从一个通用文本编辑器的角度来看，Everedit也是我用过的最好用的编辑器（比如我现在就正在用Everedit写这篇博文）。

##安装
Everedit3.2起加入了扩展包的功能，极大地降低了扩展的安装难度，所以虽然Jass扩展支持3.2以下的Everedit，但由于安装步骤复杂这里不作介绍。

首先下载Everedit和Jass扩展包，解压Everedit到任意位置，打开Everedit，将Jass扩展包拖到Everedit上，然后点确定即可。

- [Everedit3.2下载地址](http://update.everedit.net/beta.php)
- [Jass扩展包](http://pan.baidu.com/s/1bnnG1ZL)

<!-- more -->

##语法高亮

支持Jass、vJass还有预处理的语法，包括TESH在内，这应该是唯一支持预处理的语法的语法高亮了。如果你是一个YDWE的重度使用者，你一定会对它爱不释手。完美支持YDWE的Jass代码，让我们来看看效果吧

![everedit-1](/images/blog/2014/everedit-1.jpg)

##函数提示

支持所有cj和bj函数的提示，包括参数列表和中文翻译，以前有人为了背那些cj/bj函数，还特意弄个中英文对照的UI。有了它，你还需要翻UI？当词典用都行！

![everedit-2](/images/blog/2014/everedit-2.jpg)
![everedit-3](/images/blog/2014/everedit-3.jpg)

##内置JassHelper支持

你可以在Everedit直接使用JassHelper来检查语法

![everedit-4](/images/blog/2014/everedit-4.jpg)

##题外话

如果你是一个YDWE纯Jass/vJass的使用者，我不推荐你使用YDWE内置的Jass编辑器(也就是TESH)作为你的主力编辑器，而是使用外置编辑器，比如Everedit，当然Notepad++也是个不错的选择。

YDWE支持脱离地图的脚本，你可以简单地把你的Jass脚本和你的地图放在同一个目录，然后在自定义脚本区内添加以下代码。这样你就可以在使用外置的编辑器来编辑你的地图脚本了。

```
#include "脚本A路径.j"
#include "脚本B路径.j"
#include "脚本C路径.j"
```

# ArcadeScenecontrol >
ArcadeZero v4 的新 Scenecontrol API 教程 - 中文（在DeepL翻译的基础上进行了修改）  
如果你需要英文文本作为对照：https://github.com/Kuroikage1732/ArcadeScenecontrol/blob/main/README_Compared.md

[参考文档](https://github.com/Tempestissiman/ArcadeScenecontrol/wiki)

[旧 api 文档](https://github.com/Tempestissiman/ArcadeScenecontrol/tree/a30c13efc25b0a4a6e32a820b3dbb5c363564d71)

[日语版](https://hackmd.io/XW5foJnpQI6JMLQ-igqs2g)

# 引言
本文件是一个教程，旨在指导新人和有旧版Scenecontrol API编写经验的人开始使用新版API。

这并不包括API中的所有功能，但将为你提供足够的知识，让你自己探索其余的功能。请查阅[参考文档](https://github.com/Tempestissiman/ArcadeScenecontrol/wiki)以了解更多信息。

像往常一样，编写脚本需要有Lua语言的知识，因此本指南将假定你对它有一定的熟悉程度。如果你是新手或者想复习一下该语言，请参考[官方教程](https://www.lua.org/pil/contents.html)

###### 贡献者: 你可以帮助我们将文档翻译为其他语言！

# 有什么新东西？
新的API是为了解决旧的API的这些问题而制作的：

1. 缓慢。该API严重依赖用户编写的Lua代码，甚至在运行时也是如此。这不仅会很慢，我们还冒着它永远无法被移植到其他平台（如iOS）的机会。

2. 不灵活。寄存器系统过于复杂，使简单的任务，如多个场景控制器控制同一个对象变得过于复杂（这与它的目的相反！）。

3. 丑陋且不直观。

考虑到这几点，这就是我为什么更推荐新的系统：

1. 完全的确定性：新的API依赖于 *Channels* 的概念（后面会有更多的介绍！）。只需要执行一次lua代码，而且还可以输出到其他不需要lua的格式中去。

2. 开放：你对新系统有完全的控制权，这使你可以更快地编写脚本。

3. 协作：使你可以更容易地使用别人的脚本！

这是一些与核心API无关的新功能：

1. 一个用于场景控制的编辑窗口。你可以在ArcadeZero中编辑你的事件。

2. 扩展功能，如后期处理效果。

3. 内置的 scenecontrol 命令。

如果你已经开始激动了，那就不多废话，直接进入教程吧！

## 对熟悉旧系统的人的警告
你完全可以把你所有的旧脚本移植到新系统中，但请注意，像坐标和旋转这样的东西可能需要改变。我们做了一些变更使编辑它们更加直观和规范。

# 开始工作
你所需要的工具：
- ArcadeZero v4 或更高版本，或任何基于 ArcadeZero v4 的分支版本。
- 一个足够好的IDE或文本编辑器，例如VSCode。
- 你还应该安装VSCode的 "Vscode Arcaea File Format Support" (https://github.com/yojohanshinwataikei/vscode-arcaea-file-format) 以及 vscode-lua 扩展。

# 第一部分：内置命令
让我们从最基本的开始：使用内置的命令。我们首先熟悉一下新的 Scenecontrol 编辑窗口。

### aff语句的语法
你可能已经知道如何在 .aff 谱面文件中用手写出 scenecontrol 语句。如果你不知道，这里是 scenecontrol 的基本语法。

```
#aff file:
scenecontrol(timing, scenecontrolType, argument0, argument1, argument2, ...);

e.g:
scenecontrol(1000, trackdisplay, 1000, 127);
```
- `timing` （一般）表示效果发生的时间。
- `scenecontrolType` 定义了这条语句的类型。
- `argument0, argument1,...` 定义了所有的参数，这些参数基本上决定了这条语句该如何执行。

你也可以在一个 timinggroup 内使用 scenecontrol 语句，这不一定会产生影响（取决于命令的类型）。

### 使用内置指令
ArcadeZero 原生支持这些命令类型。
```
#aff file:
scenecontrol(timing, trackdisplay, duration, changeToAlpha);
scenecontrol(timing, hidegroup, duration, changeToAlpha);
scenecontrol(timing, enwidenlane, duration, changeToState);
scenecontrol(timing, enwidencamera, duration, changeToState);
```

- `trackdisplay` 会改变主轨道的透明度（或者说是*alpha值* ）。
- `hidegroup` 会改变场景控制的同一 timinggroup 内所有 note 的透明度。
- `enwidenlane` 会在主轨道的两侧增加2条轨道（在 Pentiment、Arcana Eden 和 Testify 中被使用）
- `enwidencamera` 会移动视角以支持 enwidenlane。

`timing` 和 `duration` 参数的单位都是毫秒。`changeToAlpha` 是一个从0到255的值。`changeToState` 输入0表示或者输入1表示禁用或启用效果。

> 从技术上讲，这与官方游戏的规范不同，但我故意选择偏离它，以便拥有更好的一致性。

如果你不习惯这些 scenecontrol 语句，那么我强烈建议你先自己编写语句一段时间。这是一个很好的技能！

### 编辑窗口
现在让我们熟悉一下用于编辑 scenecontrol 的新窗口。打开谱面，然后点击右边带有星星图标的按钮。你会看到一个和下图一样的小窗口：

<img src="https://i.imgur.com/oBUABzr.png">

在顶部，你会看到：
- Scenecontrol 类型菜单。你可以在这里选择要使用的类型。
- 切换按钮。这会让每个参数在分离字段和组合字段之间进行切换。你也可以按几下来看看会发生什么。
- 刷新按钮。这将刷新列表并重新加载所有包含的脚本。

中间的是事件列表。现在它可能是空的，因为你的谱面不包含任何与当前选定的 scenecontrol 类型相对应的事件。让我们在任何一行按"+"按钮来添加一个。你可以看到一个新的事件出现了：

<img src="https://i.imgur.com/Im7es5n.png">

在这里，你可以根据你的想法改变参数！

还要注意的是，该窗口只显示当前所处的 timinggroup 中包含的 scenecontrol 事件。你很可能会忘记这一点，并想知道为什么你看到了一个效果，尽管你的列表中什么都没有显示。

基本的就这样了！现在我们已经大致了解了 scenecontrol 的工作原理，我们将在下一节开始深入研究脚本部分。

# 第二部分：编写脚本

### 2.1：项目结构
到目前为止，你一直在使用应用所提供的 scenecontrol 类型。但是定制 Arcaea 谱面的关键就是属于你的场景控制类型。让我们为此努力吧！

开始浏览你的项目文件夹（包括`Arcade`文件夹以及你的音乐和图片文件等）。创建一个名为 `Scenecontrol` 的文件夹（区分大小写），并在其中创建一个名为 `init.lua` 的文件。

`Scenecontrol/init.lua` 是ArcadeZero在加载谱面文件时默认运行的脚本。`Scenecontrol` 文件夹也是你放置所有用于被脚本读取的资源文件（图片等）的地方。

> 对于高级用户：你也可以使用 `require` 将你的lua代码分割成多个脚本文件。文件名并不重要！（除了默认的 `init.lua`）

让我们打开新创建的lua文件并开始编码吧! 然而，现在我们还不会真正定义我们自己的场景控制类型，而是专注于一个更基本的概念：*Controllers* 和 *Channels*。（我们将在下一部分学习如何定义你自己的类型！）

### 2.2：控制器和通道 - 初级教程
> Controllers 和 Channels 的初级教程将使你能够利用场景中已经存在的任何东西，并按照你的喜好进行操作!

Controllers 译为控制器，你可以用自定义lua代码与之交互，以控制场景中的对象。例如，你可以与一个轨道控制器交互以控制轨道，与一个精灵控制器交互以控制一个2D图像精灵对象等等。这里的交互是指修改它的属性，如位置、旋转、比例、颜色等。

到目前为止，它和旧系统是一样的！然而，这次你不会像旧系统那样*直接*地修改属性，而是更间接地进行修改。接下来介绍：**Channels**。

所以什么是 channel 呢？channel 中文译为通道。所有的通道有一个共同的作用：它们会回答这个问题：
###### 现在是歌曲的第n毫秒了，这些数值应该是多少？

每个控制器的属性都会有一个附加的通道，这会决定该属性在任一时间点的变化。

现在你可能还不清楚这意味着什么。让我们看看一个实际的lua代码例子来说明这一点：

我们将从定义一个控制器开始。现在我们只使用一个内部控制器，以后再考虑导入你自己的控制器的问题。我们将通过使用 `Scene` 来抓取一个，这是你抓取大部分控制器的地方（也是你创建大部分控制器的地方）:
```lua
-- init.lua
local controller = Scene.track --抓取轨道控制器
```

将控制器的坐标设置为其他位置：
```lua
-- init.lua
...
controller.translationX = Channel.constant(1)
controller.translationY = Channel.constant(2)
controller.translationZ = Channel.constant(3)
```

点击 scenecontrol 编辑窗口中的刷新按钮。这将重新加载你的脚本，并告诉 Arcade 将轨道移到位于（1, 2, 3）的新位置。你的轨道应该位于一个新的位置，看起来像这样：

<img src="https://i.imgur.com/JK7SyQ5.png">

下面是对刚刚发生的事情的详细解释：
1. 使用 `Channel.constant(value)` 定义一个通道，无论在什么时候，它将永远返回你传入的值。
2. 你定义了3个通道，并将其赋值给一个控制器的三个属性
3. 现在控制器将根据设定的通道更新其数值，我们将其定义为常数1、2和3。

现在你的脚本并没有做什么有趣的事情，但是你可以通过刚才写的东西理解整个系统。如果你理解了如何使用通道，你就理解了新的 scenecontrol。把你的时间花在这里吧！

但如果你确信你完全懂了，那么让我们继续吧，通过使对象随着时间的推移而移动，可以使事情变得更加有趣。对于这一点，`Channel.constant()` 根本不可能做到。让我们来到：**Keyframe channels**。

**Keyframe channels** 中文译为 **关键帧通道**。一个关键帧通道是由一系列的**键**定义的。如果你以前做过动画，那么你应该会对这个概念很熟悉。对于那些没有接触过的，让我们看一个例子：

<img src="https://i.imgur.com/t7w0ezs.png">

这个通道有3个键。
- 第一个键位于时刻0，其值为0。
- 第一个键位于时刻1000，其值为1。
- 第一个键位于时刻2000，其值为0。
然后你“连接这些点”，这将定义在任何时刻的输出值。

实际上，使用关键帧通道是很简单的。让我们用lua代码对上面描述的通道进行编码：
```lua
--init.lua
local channel = Channel.keyframe()
channel.addKey(0, 0)
channel.addKey(1000, 1)
channel.addKey(2000, 0)
controller.translationX = channel
```

> 这只是一种写法，也是最能说明原理的一种方式。你也可能会想这样写来缩短它：
```lua
controller.translationX = Channel.keyframe().addKey(0, 0).addKey(1000, 1).addKey(2000, 0)
```

> 你可以重复使用通道! 同一通道可以被赋值给多个属性，改变一个通道将改变所有从它读取的属性。在我看来，这是一个很好的做法，因为它意味着你只需要输入一次就可以改变多个属性。
```lua
local channel = Channel.keyframe().addKey(0, 0).addKey(1000, 1).addKey(2000, 0)
controller.translationX = channel
controller.translationY = channel
controller.translationZ = channel
-- 即使是完全不相关的属性也没关系
controller.colorR = channel
```

刷新看看效果。轨道在0ms到1000ms应该以线性方式移动到另一个位置，然后在1000ms到2000ms回到原位。

但如果你不希望它是线性移动的，你将会使用缓动! 只需像这样传递一个额外的字符串：
```lua
local channel = Channel.keyframe()
channel.addKey(0, 0, 'so')
channel.addKey(1000, 1, 'so')
channel.addKey(2000, 0, 'so')
```

> 警告：这里的'so'实际上是类似于 si arc 的曲线（一开始快，最后慢）。官方的规范是错误的。这只是其中的一个小问题，你必须要习惯。

你也可以在创建通道时改变默认的缓动方式。下面的代码与上面的等价：
```lua
local channel = Channel.keyframe().setDefaultEasing('so')
channel.addKey(0, 0)
channel.addKey(1000, 1)
channel.addKey(2000, 0)
```

下面是所有支持的缓动的列表。每种缓动类型都可以用多种方式编写。

| 名称 | 别名 | 形态 |
| - | - | - |
| linear | l | <img src="https://i.imgur.com/RW6npU1.png" width=120em> |
| inconstant | inconst, cnsti | <img src="https://i.imgur.com/OtPvAGb.png" width=120em> |
| outconstant | outconst, cnsto | <img src="https://i.imgur.com/UYKymRq.png" width=120em> |
| inoutconstant | inoutconst, cnstb | <img src="https://i.imgur.com/D916zO3.png" width=120em> |
| insine | si | <img src="https://i.imgur.com/6FWfL7v.png" width=120em> |
| outsine | so | <img src="https://i.imgur.com/VDB347V.png" width=120em> |
| inoutsine | b | <img src="https://i.imgur.com/uprRIh9.png" width=120em> |
| inquadratic | inquad, 2i | <img src="https://i.imgur.com/qafhmH3.png" width=120em> |
| outquadratic | outquad, 2o | <img src="https://i.imgur.com/lPvQZiq.png" width=120em> |
| inoutquadratic | inoutquad, 2b | <img src="https://i.imgur.com/KrMCfGe.png" width=120em> |
| incubic | incube, 3i | <img src="https://i.imgur.com/pBtlUPe.png" width=120em> |
| outcubic | outcube, 3o | <img src="https://i.imgur.com/pBtlUPe.png" width=120em> |
| inoutcubic | inoutcube, 3b | <img src="https://i.imgur.com/2vsaAfH.png" width=120em> |
| inquartic | inquart, 4i | <img src="https://i.imgur.com/1gkdwj3.png" width=120em> |
| outquartic | outquart, 4o | <img src="https://i.imgur.com/1gkdwj3.png" width=120em> |
| inoutquartic | inoutquart, 4b | <img src="https://i.imgur.com/DpLpSKE.png" width=120em> |
| inquintic | inquint, 5i | <img src="https://i.imgur.com/lO6CS1R.png" width=120em> |
| outquintic | outquint, 5o | <img src="https://i.imgur.com/cjNq1sq.png" width=120em> |
| inoutquintic | inoutquint, 5b | <img src="https://i.imgur.com/0ElZdrQ.png" width=120em> |
| inexponential | inexpo, exi | <img src="https://i.imgur.com/CxM7iHY.png" width=120em> |
| outexponential | outexpo, exo | <img src="https://i.imgur.com/zSi35WU.png" width=120em> |
| inoutexponential | inoutexpo, exb | <img src="https://i.imgur.com/bYFbAOK.png" width=120em> |
| incircle | incirc, ci | <img src="https://i.imgur.com/XlLATi4.png" width=120em> |
| outcircle | outcirc, co | <img src="https://i.imgur.com/V7jXmBe.png" width=120em> |
| inoutcircle | inoutcirc, cb | <img src="https://i.imgur.com/Nq5JkvD.png" width=120em> |
| inback | bki | <img src="https://i.imgur.com/vL2NZps.png" width=120em> |
| outback | bko | <img src="https://i.imgur.com/YKBd78p.png" width=120em> |
| inoutback | bkb | <img src="https://i.imgur.com/JnSeIih.png" width=120em> |
| inelastic | eli | <img src="https://i.imgur.com/BsUF01a.png" width=120em> |
| outelastic | elo | <img src="https://i.imgur.com/IhlWlew.png" width=120em> |
| inoutelastic | elb | <img src="https://i.imgur.com/oULApFZ.png" width=120em> |
| inbounce | bni | <img src="https://i.imgur.com/fCHgebv.png" width=120em> |
| outbounce | bno | <img src="https://i.imgur.com/58pzeQD.png" width=120em> |
| inoutbounce | bnb | <img src="https://i.imgur.com/dhdjvX0.png" width=120em> |


> 为了好玩! 你也可以把一个通道的外推打开。外推法基本上是将曲线延续到你所添加的关键帧之外。我不知道为什么要用这个，但它就在这里
```lua
local channel = Channel.keyframe()
			.setDefaultEasing('so')
			.setIntroExtrapolation(true)
			.setOuttroExtrapolation(true)
```

现在你已经掌握了控制器和通道的基本知识，现在可以改变任何属性的行为了！但API提供了更多有用的功能可以为你简化这一过程。我们将在控制器和通道的高级教程回到这里。

现在，让我们真正着手做我们开始时打算做的事情：*定义一个 scenecontrol 类型*。

### 2.3：添加一个 scenecontrol 类型

你可以完全忽略整个 scenecontrol 类型的概念，只在 `init.lua` 中键入所有内容。但为了重用性，这里仍然建议定义类型。

让我们记住，为了实现这个目的，我们必须做什么：
- 指定我们的 scenecontrol 类型的名称（在这里可以自由选择你自己的名称！）
- 指定参数，我们将使用多少个参数，以及它们的名称。
- 定义 scenecontrol 类型的行为。

其中 2/3 的内容涵盖在一个单一的函数中。其余的就是我们在上一章中刚刚做的。让我们看看吧!

我们将制作一个简单的 scenecontrol 类型，它将读取3个数字，并将轨道从其当前位置移动到由这3个数字指定的新位置。
```lua
local track = Scene.track
track.translationX = Channel.keyframe()
track.translationY = Channel.keyframe()
track.translationZ = Channel.keyframe()

addScenecontrol("mytypename", 3, function(cmd)
	local timing = cmd.timing
	local x = cmd.args[1] -- 与其他语言不同，lua从1开始计数。
	local y = cmd.args[2]
	local z = cmd.args[3]
	track.translationX.addKey(timing, track.translationX.valueAt(timing))
	track.translationX.addKey(timing + 500, x)
	track.translationY.addKey(timing, track.translationY.valueAt(timing))
	track.translationY.addKey(timing + 500, y)
	track.translationZ.addKey(timing, track.translationZ.valueAt(timing))
	track.translationZ.addKey(timing + 500, z)
end)
```

那么究竟发生了什么？
1. 我们首先定义了关键帧通道来添加我们的按键。
2. 我们定义了一个名为 "mytypename" 的 scenecontrol 类型。它有3个参数。
	这意味着我们的aff命令将看起来像这样。`scenecontrol(timing, mytypename, arg1, arg2, arg3)`。
3. 我们通过一个函数定义了我们的scenecontrol类型的行为。基本上这个函数是为谱面中的每一个  scenecontrol 事件运行的。我们的函数将读取该命令，并采取相应的行动，也就是给我们的通道添加一个关键帧。

>如果你熟悉 Arcade宏，你应该很熟悉这个模式! 唯一不同的是，我们要接受一个参数来读取 scenecontrol 语句的数据。

还要注意 `channel.valueAt(timing)` 的使用。这是一个在处理关键帧时很方便的功能。

这个例子也很好地说明了 scenecontrol 脚本的基本结构
- 设置控制器。
- 指派通道。
- 定义 scenecontrol 类型，将其填入键值。

现在我们成功地定义了我们的类型，打开Arcade，然后打开scenecontrol编辑窗口。你应该看到 "mytypename" 现在出现在下拉菜单中，并且添加事件应该可以像预期的那样工作。

顺便说一下，你可能会在场景控制编辑窗口的第一行上面看到一些小文字。它们表示每一列中的参数名称，是的，你可以改变它！
```lua
addScenecontrol("mytypename", {"xpos", "ypos", "zpos"}, function(cmd)
	...
end)
```

`{"xpos", "ypos", "zpos"}` 会创建一个字符串的列表，被解析为参数的名称。刷新脚本并再次查看窗口！

> 参数为零的事件也可以工作。下面这段代码对应于以下aff命令类型：`scenecontrol(timing,mytypename);`
```lua
addScenecontrol("mytypename", 0, function(cmd) ... end)
-- or --
addScenecontrol("mytypename", {}, function (cmd) ... end)
```

现在你应该已经能够重复使用大量的效果了。但是，让我们更进一步，去了解 `Scene` 的内部和外部！

### 2.4：Scene

`Scene` 中文译为场景。`Scene` 对象有两个功能：抓取内置控制器，以及创建新的控制器。我们已经做了前者，但到目前为止只接触了其中的一种类型。在本指南中详细描述每种类型的控制器是不现实的，所以请参考文档以获得更多信息。

下面是所有内置控制器的列表

| 对象 | 类型 | 简介 |
| - | - | - |
| Scene.gameplayCamera | CameraController | 主摄像头 |
| Scene.combo | TextController | Combo 文本 |
| Scene.score | TextController | 分数文本 |
| Scene.jacket | ImageController | 封面图像 |
| Scene.title | TextController | 这首歌的标题文本 |
| Scene.composer | TextController | 这首歌的作曲家文本 |
| Scene.difficultyText | TextController | 显示难度的文本组件 |
| Scene.difficultyBackground | ImageController | 显示难度的图像组件 |
| Scene.hud | CanvasController | 包含暂停按钮和信息面板的画布 |
| Scene.pauseButton | ImageController | 暂停按钮 |
| Scene.infoPanel | ImageController | 信息面板背景的图像 |
| Scene.background | ImageController | 背景图像 |
| Scene.videoBackground | SpriteController | 视频背景组件 |
| Scene.track | TrackController | 主轨道 |
| Scene.track.divideLine01 | SpriteController | 轨道0和1的分割线 |
| Scene.track.divideLine12 | SpriteController | 轨道1和2的分割线 |
| Scene.track.divideLine23 | SpriteController | 轨道2和3的分割线 |
| Scene.track.divideLine34 | SpriteController | 轨道3和4的分割线 |
| Scene.track.divideLine45 | SpriteController | 轨道4和5的分割线 |
| Scene.track.divideLines | Table (of SpriteController) | 所有分割线组成的列表 |
| Scene.track.criticalLine0 | SpriteController | 轨道0的判定线 |
| Scene.track.criticalLine1 | SpriteController | 轨道1的判定线 |
| Scene.track.criticalLine2 | SpriteController | 轨道2的判定线 |
| Scene.track.criticalLine3 | SpriteController | 轨道3的判定线 |
| Scene.track.criticalLine4 | SpriteController | 轨道4的判定线 |
| Scene.track.criticalLine5 | SpriteController | 轨道5的判定线 |
| Scene.track.criticalLines | Table (of SpriteController) | 所有判定线组成的列表 |
| Scene.track.extraL | SpriteController | 左侧的额外轨道（轨道0） |
| Scene.track.extraR | SpriteController | 右侧的额外轨道（轨道5） |
| Scene.track.edgeExtraL | SpriteController | 左侧额外轨道的边缘 |
| Scene.track.edgeExtraR | SpriteController | 右侧额外轨道的边缘 |
| Scene.singleLineL | SpriteController | 左侧的 `Memory Archive line` 控制器 |
| Scene.singleLineR | SpriteController | 右侧的 `Memory Archive line` 控制器 |
| Scene.skyInputLine | SpriteController | skyinput线的控制器 |
| Scene.skyInputLabel | SpriteController | skyinput标签的控制器 |
| Scene.darken | SpriteController | 暗化的精灵控制器（被 trackdisplay 类型使用）。 |

> 你可能想知道 ImageController 和 SpriteController 的区别是什么。简单地说：
> - SpriteController 即为**精灵控制器**，ImageController 即为**图像控制器**。SpriteController 可以设置自己的图层和排序顺序，而ImageController不能。
> - 你可以用 pivots 和 anchors 更容易地控制  ImageController 的位置。ImageController 的排序顺序和图层只能通过将其链接到另一个画布上来改变。

然而，与其再次使用内置控制器，不如让我们尝试在这里创建我们自己的控制器，并通过这种方式探索不同类型的控制器。同样，你将需要 `Scene` 来实现这个目的

| 方法 | 返回类型 | 简介 |
| - | - | - |
| Scene.createSprite(string imgPath, string material = "default", bool newMaterialInstance = false) | SpriteController | 通过路径上的图像创建一个精灵，并指定 material 类型 |
| Scene.createCanvas(bool worldSpace = false) | CanvasController | 创建一个画布，可以用场景坐标或屏幕坐标来显示 |
| Scene.createImage(string imgPath, string material = "default", bool newMaterialInstance = false) | ImageController | 从路径中创建一个具有指定 material 类型的图像 |
| Scene.createText(string font = "default", number fontSize = 40, number lineSpacing = 1, string alignment = "middlecenter", string material = "default") | ImageController | 通过文本创建一个具有指定 material 类型的图像 |
| Scene.createMesh(string objPath, string texturePath) | MeshController | 从一个.obj文件和一个纹理图像创建一个三维物体 |
| Scene.getNoteGroup(number group) | NoteGroupController | 获取与一个 timinggroup 相关的控制器 |

当你看到参数中的 `=` 时（例如`material = "default"`），这意味着该参数有一个默认值，一般情况下你不需要指定它。

你可以使用 `material` 参数来改变融合模式。下面是 `material` 所有参数的列表
- default
- add
- colorburn
- darken
- difference
- exclusion
- fastadd
- fastdarken
- fastlighten
- fastmultiply
- fastscreen
- hardlight
- lighten
- linearburn
- lineardodge
- linearlight
- multiply
- overlay
- screen
- softlight
- subtract
- vividlight
`newMaterialInstance` 在大多数情况下应该是 `false`。如果你需要改变控制器的纹理偏移或缩放，请将其设置为 `true`。

考虑到这一点，让我们试着在我们的场景中添加一个自定义精灵。默认情况下，精灵被放置在背景图层，也就是轨道所属图层的下面。所以我们需要更改它，才能看到任何东西。一个精灵的图层可以随着时间的推移而改变，所以我们需要再次使用一个通道，但这次是一个 `StringChannel`。

```lua
local sprite = Scene.createSprite("test.png")
sprite.layer = StringChannel.constant("Foreground") -- 更改所属图层
sprite.order = Channel.constant(1) -- 更改图层的顺序
```

在这之后我们应该能看到场景中的图像：
<img src="https://i.imgur.com/tAEbq6Q.png">

可以随意将 "test.img" 改为任何图像文件的名称。

让我们用一张图片试试，为了稳妥起见，这次我们也要改变它的 material。将上面的代码改为。
```lua
local image = Scene.createImage("test.png", "fastadd")
image.rectW = Channel.constant(300) -- 设置适当的宽度和高度
image.rectH = Channel.constant(200)

local canvas = Scene.createCanvas(false)
image.setParent(canvas) -- 设置图像的父画布
canvas.layer = StringChannel.constant("Foreground") -- 更改图层
canvas.sort = Channel.constant(1) -- 更改图层的排序
```

请注意，这次必须指定图像的宽度和高度。这就是精灵和图像的一个主要区别。另外，希望添加融合样式没有让你的图像太难看，在这里我把它改成conflict。但你也可以随时尝试其他融合样式。

<img src="https://i.imgur.com/vstHQ7X.png">

最后，让我们尝试创建一些文本：
```lua
local text = Scene.createText("Forte") -- 该字体必须被安装在用户的操作系统上
text.text = StringChannel.create().addKey(0, "Hello").addKey(1000, "World")

local canvas = Scene.createCanvas(false)
text.setParent(canvas)
canvas.layer = StringChannel.constant("Foreground") -- 更改图层
canvas.sort = Channel.constant(1) -- 更改图层的顺序
```

正如你所看到的，字符串通道也可以被关键帧化。这里我们让它在0ms时显示 "Hello"，然后在1000ms时改变自己为 "World"。

> 你也可以在字符串通道上使用缓动！试着把它改为 `addKey(0, "Hello", "so")`，看看会发生什么。

这里是我们得到的效果：
<img src="https://i.imgur.com/vPq6Zm8.png">

这涵盖了创建控制器时需要注意的重要事项。如果你想看看每种类型的控制器的可用属性，请参考文档。

### 2.5：控制器和通道 - 进阶教程

> 控制器与通道的进阶教程将教你如何在使用通道时节省大量的时间。你将学习不同类型的通道，以及如何将通道组合在一起产生各种效果。

让我们首先考虑如何处理一个非常简单的效果：以正弦波的形式无限期地来回移动一个对象。我们的通道看起来将会像这样：

(img)

手动添加每个关键帧来实现这种效果是完全可以做到的。事实上，为了了解这需要我们怎么实现，让我们看一下这样做的代码：
```lua
-- 你不会需要用到这个。这是一个反面教材
Channel c = Channel.keyframe()
for timing = 0, Context.songLength, 2000 do
	c.addKey(timing, 0, "so")
	c.addKey(timing + 500, 1, "b")
	c.addKey(timing + 1500, -1, "si")
end
```
这个通道将以正弦波的形式在-1和1之间不断摇晃。这是有效的，但我们可以做得更好。

事实上，API提供了一堆除关键帧通道之外的*其他*类型的通道来帮助你。上面的所有代码都可以缩短为：
```lua
Channel c = Channel.sine(2000, -1, 1, 0)
-- 一个正弦波，范围从-1到1，时间偏移为0
-- 该波每2000ms重复一次
```

你还有一系列的其他通道可供选择! 下面是它们的列表。

| 方法 | 类型 | 简介 |
| - | - | - |
| Channel.keyframe() | KeyChannel | 一个关键帧通道 |
| Channel.constant(value) | ValueChannel | 一个具有不变的数值的通道 |
| Channel.random(min, max, seed = 0) | ValueChannel | 一个每次都能返回随机值的通道 |
| Channel.noise(frequency, min, max, offset = 0, octave = 1) | ValueChannel | 一个佩林噪声通道 |
| Channel.sine(period, min, max, offset = 0) | ValueChannel | 一个正弦波通道 |
| Channel.saw(string easing, period, min, max, offset = 0) | ValueChannel | 一个在max到min之间移动的具有特定缓动类型的锯齿波 |
| Channel.fft(freqBandMin, freqBandMax, min, max, smoothness = 0.1, scalar = 1) | ValueChannel | 返回当前播放的音频在指定频率范围内的平均响度 |
| Channel.max(channelA, channelB) | ValueChannel | 取两个通道的最大值 |
| Channel.min(channelA, channelB) | ValueChannel | 取两个通道的最小值 |
| Channel.clamp(valueChannel, minChannel, maxChannel) | ValueChannel | 将 valueChannel 限定在 minChannel 和 maxChannel 的值之间 |

> 如果你好奇的话，FFT是快速傅里叶变换的意思。
> 顺便说一下，默认情况下FFT通道以256个频段运行，但你可以通过 `Channel.setGlobalFFTResolution(resolution)` 来更改。分辨率必须是2的倍数(64, 128, 256, 512, ...)

没有比一行代码就能解决的更好的了，是吗？事实上，它确实如此。你可以把不同的通道组合在一起，获得无限的可能：

```lua
-- 在-2和2之间振动
Channel vibrate = Channel.noise(100, -2, 2) 
-- 每隔1000ms：把它的值从1变成0，然后跳回去
Channel dampen = Channel.saw("so", 1000, 1, 0) 
-- 它的振动程度随时间变化
Channel combined = vibrate * dampen
```
通过将 `vibrate` 通道与 `dampen`相乘，我们限制了通道随时间变化的振动程度。当 `dampen` 返回1时，它的振动最大，当 `dampen` 返回0时，它不振动。由于`dampen`的形状是一个锯齿波，我们得到一个有趣的脉冲效果。

当然，你也可以用这种方式把关键帧通道和其他通道结合起来。这也为一种非常有效的技术提供了可能性，实际上内部使用了这种技术来实现内置的 scenecontrol 类型。让我们来看看其中一个，`enwidenlane`。

这个 scenecontrol 有这些作用：
- 将两个额外的轨道轨道的不透明度从0改成255。
- 将两个额外的轨道边缘的不透明度从0改为255。
- 将默认的两条轨道边缘的不透明度从255改为0。
- 将两条额外的分割线（属于0和5道）从0改为255。
- 将车道0-1和4-5之间的分界线从0改为255。
- 将两条额外轨道的位置从-100改为0。

你可以手动设置所有的关键帧，只是这样做很烦人，而且要花很多笔墨。取而代之的是内部发生的类似这样的事情：

```lua
-- 注意！内部 C# 代码翻译成 lua。这并不能正常运行
local track = Scene.track
  
-- 主通道，默认为0
local enwidenLaneFactor = Channel.keyframe().setDefaultEasing("l").addKey(0, 0);
  
-- 这些对象在默认情况下是禁用的。启用它们：
track.extraL.active = Channel.constant(1)
track.extraR.active = Channel.constant(1)
track.criticalLine0.active = Channel.constant(1)
track.criticalLine5.active = Channel.constant(1)
track.divideLine01.active = Channel.constant(1)
track.divideLine45.active = Channel.constant(1)
track.edgeExtraL.active = Channel.constant(1)
track.edgeExtraR.active = Channel.constant(1)
  
-- 指定通道，这实际上只是主通道的简单转换
track.edgeExtraL.colorA = track.edgeExtraL.colorA * enwidenLaneFactor
track.edgeExtraR.colorA = track.edgeExtraR.colorA * enwidenLaneFactor
track.criticalLine0.colorA = track.criticalLine0.colorA * enwidenLaneFactor
track.divideLine01.colorA = track.divideLine01.colorA * enwidenLaneFactor
track.divideLine45.colorA = track.divideLine45.colorA * enwidenLaneFactor
track.criticalLine5.colorA = track.criticalLine5.colorA * enwidenLaneFactor
track.extraR.colorA = track.extraR.colorA * enwidenLaneFactor
track.extraL.colorA = track.extraL.colorA * enwidenLaneFactor
  
local posY = -100 * (1 - enwidenLaneFactor)
track.extraL.translationY = track.extraR.translationY + posY
track.extraR.translationY = track.extraR.translationY + posY
  
local alpha = (1 - enwidenLaneFactor)
track.edgeLAlpha = track.edgeLAlpha * alpha
track.edgeRAlpha = track.edgeRAlpha * alpha
  
-- 我们需要对每个scenecontrol命令做的就是：
addScenecontrol("enwidenlane", {"duration", "toggle"}, function(cmd)
    local timing = cmd.timing
    local duration = cmd.args[1]
    local toggle = cmd.args[2]
    enwidenLaneFactor.addKey(timing, enwidenLaneFactor.valueAt(timing))
    enwidenLaneFactor.addKey(timing + duration, toggle)
end)
```

如果你仍然不清楚这段代码在做什么，这里有一个简单的解释。
- 第二个参数，"toggle"，是一个0或1的值。我们把这个值直接写入`enwidenLaneFactor`通道中。
- 我们需要改变的每一个属性只是主通道的一些变体，这些变体是直接在通道本身而不是单独的键上计算的。

你可能还注意到，我们正在用 `posY` 和 `alpha` 通道在一个通道和一个数字值之间进行运算。在内部，数字实际上被转换为常数通道，所以这只是一个方便的简称。

请注意这个脚本实际上是不起作用的，因为在内部，额外的轨道已经被键入，将它们的alpha设置为0，所以在此基础上的乘法不会有任何作用。所以你应该这样做：
```lua
track.extraEdgeL.colorA = track.extraEdgeL.colorA + 255 * enwidenLaneFactor
-- 判定线，分割线，额外轨道与此类似

local posY = 100 * enwidenLaneFactor
track.extraL.translatonY = Channel.min(track.extraEdgeL.translationY + posY, Channel.constant(0))
-- extraR 也一样
```

这里我们正在把我们的通道和内置通道加在一起。对于颜色来说，255以上的值是没有区别的，所以我们不必担心这个问题，但是对于额外轨道的位置，我们不希望它超过0，以防有人同时使用两种场景控制类型。我们可以使用`Channel.min`来确保这一点。

当然，如果你不关心内置类型，那么：
```lua
track.extraEdgeL.colorA = 255 * enwidenLaneFactor
-- 判定线，分割线，额外轨道与此类似

track.extraL.translationY = -100 * (1 - enwidenLaneFactor)
-- extraR 也一样
```
也是可以的

所以这个脚本的实际 lua 版本应该是：
```lua
local track = Scene.track
  
-- 主通道，默认为0
local enwidenLaneFactor = Channel.keyframe().setDefaultEasing("l").addKey(0, 0);
  
-- 不需要启用任何东西。内置类型已经为我们启用了它们

track.edgeExtraL.colorA = track.edgeExtraL.colorA + 255 * enwidenLaneFactor
track.edgeExtraR.colorA = track.edgeExtraR.colorA + 255 * enwidenLaneFactor
track.criticalLine0.colorA = track.criticalLine0.colorA + 255 * enwidenLaneFactor
track.divideLine01.colorA = track.divideLine01.colorA + 255 * enwidenLaneFactor
track.divideLine45.colorA = track.divideLine45.colorA + 255 * enwidenLaneFactor
track.criticalLine5.colorA = track.criticalLine5.colorA + 255 * enwidenLaneFactor
track.extraR.colorA = track.extraR.colorA + 255 * enwidenLaneFactor
track.extraL.colorA = track.extraL.colorA + 255 * enwidenLaneFactor

local posY = 100 * enwidenLaneFactor
track.extraL.translationY = Channel.min(track.extraR.translationY + posY, Channel.constant(0))
track.extraR.translationY = Channel.min(track.extraR.translationY + posY, Channel.constant(0))
  
local alpha = (1 - enwidenLaneFactor)
track.edgeLAlpha = track.edgeLAlpha * alpha
track.edgeRAlpha = track.edgeRAlpha * alpha
  
-- 使用不同的名称避免冲突
addScenecontrol("enwidenlanelua", {"duration", "toggle"}, function(cmd)
    local timing = cmd.timing
    local duration = cmd.args[1]
    local toggle = cmd.args[2]
    enwidenLaneFactor.addKey(timing, enwidenLaneFactor.valueAt(timing))
    enwidenLaneFactor.addKey(timing + duration, toggle)
end)
```

本节的内容到此结束。有了这些知识，你就可以用几行代码实现复杂的运动了！

#### 关于调试的额外贴士：
你可以通过输出日志来查看通道是由什么组成的。比如说：
```lua
local channel = Channel.saw("si", 1000, 1, 0) * Channel.noise(100, -1, 1)
channel += Channel.keyframe()
channel *= 5
log(channel)
```

在打开错误日志时，你应该看到以下结果：
```
((unnamed@saw(si,1000,0,1,0)) * (unnamed@noise(100,-1,1,0,1)) + unnamed@key(0)) * (5)
```
这将有助于弄清为什么效果没有按照你的想象运作。

### 2.6：在 note groups 中运作

Note group 的工作有点奇怪。到目前为止，我们一直使用的定义控制器，然后在 scenecontrol 定义工作流程中键入这些控制器的工作方式是行不通的，原因很简单：我们不知道我们想要哪些控制器。因为我们不知道我们的事件将被提前放在哪个timing group中。

唯一的正确方法是为每个 scenecontrol 命令查询控制器。

```lua
addScenecontrol("myType", {}, function(cmd)
	local noteGroup = Scene.getNoteGroup(cmd.timingGroup)
end)
```

这完全没问题，但我们不能真的为每个 scenecontrol 命令创建一个新的通道。这将覆盖每个 scenecontrol 命令的旧通道：
```lua
addScenecontrol("myType", 1, function(cmd)
	local noteGroup = Scene.getNoteGroup(cmd.timingGroup)
	noteGroup.translationX = Channel.keyframe().addKey(0, cmd.args[1])
end)
-- 这样做是不行的，note group的属性只会被赋予最后一个值。
```

相反，我们需要一种方法来找出我们是否已经为一个 note group 创建了一个通道，如果没有，就创建一个新的。事实上，你可以用lua代码做到这一点。只是对每一个通道和每一个道具都这样做的话，就显得非常笨拙和乏味了。

介绍一下：命名通道。还记得你在输出你的通道时，里面有一堆“unnamed”吗？那是每个频道的默认名称。你也可以将通道改为你想要的名称。

```lua
local channel = Channel.named("myChannelName").keyframe()
```

如果一个通道是由多个通道组合而成的，你可以通过使用`.find(name)`将每个通道分离出来进行编辑。让我们看看怎么做：

```lua
local myChannel = Channel.named("IWantThisBackLater").keyframe()
-- 让我们添加几个键，这样我们就能确定这是同一条通道了
myChannel.addKey(0, 0).addKey(1000, 1).addKey(2000, 2)
log(myChannel.keyCount) -- 应该在错误日志中输出3

local otherChannel = Channel.named("OthersCreatedThisAndIDontCareAboutIt").keyframe()
local combinedChannel = myChannel * otherChannel

local myChannelFound = combinedChannel.find("IWantThisBackLater")
log(myChannelFound.keyCount) -- 也应该在错误日志中输出3
```

combinedChannel 可以从它的组件中搜索一个有你想要的名字的通道。这个名字也是该通道的局部名字，所以你甚至不必为每个属性命名不同的通道。

这使我们能够解决我们先前在note group中遇到的问题。让我们试着在note group的属性中找到我们的通道，如果它不存在，那么我们将创建一个新的通道并将其赋值给它。

```lua
addScenecontrol("myType", 1, function(cmd)
	local noteGroup = Scene.getNoteGroup(cmd.timingGroup)

	-- 尝试从属性中找到通道
	local channel = noteGroup.translationX.find("myType")
	if channel == nil then -- 如果没有找到
		-- 创建一个新的通道
		channel = Channel.named("myType").keyframe()
		-- 赋值到属性
		noteGroup.translationX = channel
	end
		
	channel.addKey(0, cmd.args[1])
end)
```

总是给你的通道命名也是一个很好的方法，因为这有助于调试。特别是当你打算与其他人分享你的脚本时。

### 2.7：后处理

后期处理是在你的整个屏幕上可见的效果。这包括：模糊屏幕、扭曲屏幕、添加光晕效果、使屏幕产生噪点、添加发光效果等。

说实话，我只是……复制了unity支持的所有东西，并以通道的形式把它们包装起来。所以我自己都不知道80%的参数是做什么的。Unity在这方面的文档也很糟糕，所以你在这里主要靠自己。

说到这里，如果你已经掌握了通道，你能够使用90%的后期处理。唯一的区别是，你必须从 `PostProcessing` 对象中抓取控制器，而且你还必须单独启用每个参数，以使其生效（因为启用它们会对性能产生巨大影响）。

每个后处理控制器也可以有不随时间变化的属性。我不会演示它们，因为它们看起来都不怎么有用，但如果你好奇的话，可以随时查看文档，了解更多这方面的细节。

这里有一个可能是最有用的后期处理效果的演示，即颜色分级：
```lua
local colorGrading = PostProcessing.colorGrading
colorGrading.enableEffects({"temperature", "colorfilter", "saturation"}) -- 启用3个属性
colorGrading.temperature = Channel.constant(10) -- 提高色温
colorGrading.ColorS = Channel.constant(-0.2) -- 降低饱和度
```

这是效果：
<img src="https://i.imgur.com/3raINS4.png">

### 2.8：一些额外的提示和注意事项

这些都是你应该知道的小细节，但还不足以用一整个章节来介绍。

##### 注意事项
1. 摄像机在aff camera命令中的和在scenecontrol API中是非常不同的!
2. 所有的文件路径都是相对于 *`Scenecontrol`文件夹的*，而不是相对于运行中的脚本的。
> 例如，如果在你的 `init.lua` 脚本中，你引用了 `folder/other_script.lua` 脚本，并且在 `other_script.lua` 中你试图创建一个路径为 `image.jpg` 的精灵，实际的文件路径应该是 `Scenecontrol/image.jpg` 而不是 `Scenecontrol/folder/image.jpg`
3. 当选择精灵、图像和文本的 materials 时，要注意任何不以 `fast` 开头的 material 都会大大影响性能，特别是在低端硬件上。

#### 提示
1. 对于熟悉宏的人来说。可以使用 `Event` 和 `Context` 对象。我不知道为什么会需要 `Event`，但为什么不需要呢？
2. 替代 `log`，`notify` 也是一个选项，它将在弹窗通知上显示一个信息。
3. 你可以随时使用 `log(channel.valueAt(timing))` 来记录某物在任何时间点的值。这对于弄清场景中物体的坐标也很有用。
4. 你可以用`Context.availableFonts`获得所有有效安装的字体列表。不过，这只能用于调试，不要在实际的脚本中包含这个。
5. 总是使用`local`，除非你确实打算让一个变量成为全局变量。

# 接下来应该做什么
祝贺你看完这个关于新 scenecontrol API 的教程！

你现在应该有足够的知识来有效地使用参考文件，该文件将详细列出关于API的一切，供你在编写自己的脚本时参考。[在这里查看](https://github.com/Tempestissiman/ArcadeScenecontrol/wiki)

祝你编写脚本愉快！

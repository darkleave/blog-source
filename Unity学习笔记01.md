---
title: Unity 学习笔记01 - unity编辑器
date: 2019-07-14 15:09:42
tags: [unity]
---

## unity编辑器
Unity编辑器拥有非常直观，明了的界面布局，熟悉Unity界面是学习Unity的基础

<!--more-->

## 界面布局
Unity 主编辑器由若干个选项卡窗口组成，这些窗口统称为视图。每个视图都有其特定的作用：

* 场景视图（Scene View):用于设置场景以及放置游戏对象,是构造游戏场景的地方。
* 游戏视图（Game View):由场景中相机所渲染的游戏画面，是游戏发布后玩家所能看到的内容。
* 层级视图（Hierarchy):用于显示当前场景中所有游戏对象的层级关系。
* 项目视图(Project):整个工程中所有可用的资源，例如模型，脚本等。
* 检视视图（Inspector):用于显示当前所选择游戏对象的相关属性与信息。


**在实际工作中经常需要在各种不同的视图中切换，以下是常用的视图切换快捷键，熟练使用快捷键可以大大提高工作效率**

* Ctrl+1:切换到Scene视图
* Ctrl+2:切换到Game视图。
* Ctrl+3:切换到Inspector视图
* Ctrl+4：切换到Hierarchy视图
* Ctrl+5:切换到Project视图
* Ctrl+6:切换到Animation视图
* Ctrl+7:切换到Profiler视图.
* Ctrl+8：切换到Audio Mixer视图
* Ctrl+9:切换到Asset Store视图
* Ctrl+0：切换到Version Control 视图
* Ctrl+Shift+C: 切换到Console视图

## 工具栏
Unity工具栏位于菜单栏的下方，主要有5个控制区域组成，它提供了常用功能的快捷访问方式，工具栏主要包括Transform Tools(变换工具),Transform Gizmo Tools(变换辅助工具),Play(播放控制),Layers(分层下拉列表）和Layout(布局下拉列表）。

### Transform Tools (变换工具)

Transform Tools(变换工具): 主要针对Scene视图，用于实现所选择游戏对象的位移，旋转以及缩放等操作控制，变换工具从左到右依次是Hand（手形)工具，Translate(移动)工具，Rotate（旋转)工具，Scale(缩放）工具和Rect(矩形)工具。

### Transform Gizmo Tools (变换辅助工具)
Pivot/Center | Global/Local:
Center 显示游戏对象的轴心参考点。Center为以所有选中物体所组成的轴心作为游戏对象的轴心参考点（常用于多物体的整体移动）；Pivot为以最后一个选中的游戏对象的轴心为参考点。
Global显示物体的坐标，Global为所选中游戏对象使用世界坐标；Local为该游戏对象使用自身坐标。

### Play（播放控制）
Play（播放控制）：应用于Game视图，当单击播放按钮时，Game视图会被激活，实时显示游戏运行的画面效果，用户可在编辑和游戏状态之间随意切换，使游戏的调试和运行变得便捷，高效。

### Layers(分层下拉列表）
Layers(分层下拉列表）：用来控制游戏对象在Scene视图中的显示，在下拉列表中显示状态为“睁眼”的物体将被显示在Scene视图中

### Layout(布局下拉列表)
Layout (布局下拉列表):用来切换视图的布局，用户也可以存储自定义的界面布局.

## 菜单栏

Unity 5.0默认情况下有7个菜单项，分别是File,Edit,Assets,GameObject,Component,Window和Help.

### File(文件)菜单


File(文件)菜单主要包含工程与场景的创建，保存以及输出的功能，

### Edit(编辑菜单)
Edit(编辑)菜单主要用来实现场景内部相应编辑设置.

### Assets(资源)菜单
Assets(资源)菜单提供了针对游戏资源管理的相关工具,通过Assets菜单的相关命令,用户不仅可以在场景内部创建相应的游戏对象，还可以导入和导出所需要的资源包.

### GameObject(游戏对象)菜单
GameObject(游戏对象)菜单主要创建游戏对象，如灯光，粒子，模型，UI等，了解GameObject菜单可以更好地实现场景内部的管理与设计.

### Component(组件)菜单
Component(组件)可以实现GameObject的特定属性,本质上每个组件是一个类的实例。在Component菜单中，Unity为用户提供了多种常用的组件资源。

### Windows(窗口)菜单
Window(窗口) 菜单可以控制编辑器的界面布局,还能打开各种视图以及访问Unity的Asset Store在线资源商店.

### Help(帮助)菜单
Help(帮助)菜单汇聚了Unity的相关资源链接,例如Unity手册，脚本参考,论坛等，同时也可以对软件的授权许可进行相应的管理.

## 常用工作视图

### Project(项目)视图

Project(项目)视图是Unity整个项目工程的资源汇总,保存了游戏场景中用到的脚本，材质，字体，贴图，外部导入的网络模型等资源文件.

### Scene(场景)视图

Scene(场景)视图是Unity最常用的视图之一，该视图用来构造游戏场景，用户可以在这个视图中对游戏对象进行操作.

**Scene视图常用的操作方法**

* 旋转操作：按Alt+鼠标左键,可以在场景中沿所注视的位置旋转视角.
* 移动操作: 按住鼠标的滚轮键,或者按键盘上的【Q】键，可移动场景视图下的观看位置。。
* 缩放操作：使用滚轮键，按Alt+鼠标右键可以放大和缩小视图的视角
* 居中显示所选择的物体：按【F】键可以将选择的游戏对象剧中显示
* Flythrough(飞行浏览）模式：鼠标右键+W/A/S/D键可以切换Flythrough模式下加按【Shift】键会使移动加速。

### Game (游戏)视图

Game(游戏)视图是显示游戏最终运行效果的预览窗口.通过单机工具栏的“播放”按钮即可在Game窗口进行游戏的实时预览,方便游戏的调试和开发.

### Inspector(检视)视图

Inspector(检视)视图用于显示游戏场景中当前所选择游戏对象的详细信息和属性设置,包括对象的名称，标签，位置坐标，旋转角度，缩放，组件等信息.

* Transform: 用户可以通过Transform组件对游戏对象的Position(位置),Rotation(旋转)和Scale(缩放)这三个属性进行修改.Transform组件是个基础组件，每个游戏对象都有这个组件.
* Mesh Filter:网格过滤器用于从对象中获取网格信息(Mesh)并将其传递到用于渲染至屏幕的网格渲染器当中.
* Mesh Collider: Mesh 碰撞体，为了防止物体被穿透,需要给对象添加碰撞体.
* Mesh Renderer:网格渲染器是从网格过滤器获得几何形状，并且根据游戏对象的Transform组件所定义的位置进行渲染.
* Materials:设置游戏对象的颜色，贴图等信息。

### Hierarchy(层级) 视图
Hierarchy(层级) 视图用于显示当前场景中每个游戏对象.
在Hierarchy视图中提供了parenting（父子化)关系,通过为游戏对象建立Parenting关系，可以使多个对象的移动和编辑变得更为方便和准确.

任何游戏对象都可以有多个子对象，但只能有一个父对象,用户对父对象进行的操作，都会影响到其下所有的子对象，即子对象继承了父对象的数据。子对象还可以对自身进行独立的编辑操作。

虽然通过Scene视图可以非常直观地对场景资源进行编辑和管理，但是在Scene视图中游戏对象容易重叠和遮挡,这时候就需要在Hierarchy视图中进行操作，文件显示方式，更易于用户对游戏对象的管理.

### Console(控制台)视图

Console(控制台) 是Unity的调试工具, 用户可以编写脚本在Console视图输出调试信息.

快捷键：Window->Console命令或按【Ctrl+Shift+C】

### Animation（动画)视图

Animation(动画)视图用于在Unity中创建和编辑游戏对象的动画剪辑(Animation Clips),用户可以依次选择菜单栏中的Window->Animation命令或按【Ctrl+6】组合键弹出Animation视图.

### Animator(动画控制器)视图

Animator(动画控制器)视图可以预览和设置角色行为，用户可以依次选择菜单栏中的Window->Animator 命令打开Animator视图.

### Sprite Editor(Sprite编辑器)
Sprite Editor (Sprite编辑器)是用于建立Sprite的工具,使用它可以提取复杂图片中的元素,并风别建立Sprite精灵.

### Sprite Packer(Sprite打包工具)
Sprite Packer(Sprite打包工具) 是用于制作Sprite图集的工具,可以将各个Sprite制作成图集,这样可以把图片的空间利用率提高,减少资源的浪费.

### Lightmaps(光照贴图烘培) 视图

Unity内置了光照贴图烘培工具Beast.使用Beast 可以根据场景的网格物体,材质贴图和灯光属性的设置来烘培场景，从而得到完美的光照贴图.

### Occlusion(遮挡剔除) 视图

Occlusion(遮挡剔除) 视图技术是指当一个物体被其他物体遮挡住,而不在摄像机的可视范围内时不对其进行渲染.

### Navigation(导航寻路)视图
导航寻路是游戏中常用的技术,通过点击场景上的一个位置,游戏角色就会自动寻路过去,行走过程中角色会自动绕过障碍物,最终到达终点.

### Version Control(版本控制)视图
使用版本控制可以轻松回到某一个时间点的版本.
默认情况下Unity的版本控制是关闭的，可依次选择菜单栏中的Edit->Project Setting-> Editor 命令，然后在Inspector视图中将Version Control 的Mode设置为Asset Server,接着按[Ctrl + 0]组合键即可打开Version Control视图.

### Asset Store（资源商店)
Unity的在线资源商店Asset Store拥有丰富的资源素材库,全球各地的开发者都在这里分享自己的工作成果,涵盖了材质，模型，动画，插件到完整的项目实例，可以在Unity编辑器里下载并直接导入项目工程.




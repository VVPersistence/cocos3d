# cocos3d(7)
## 添加场景内容
### 加载POD文件
为所有，但最简单的内容，你会想在3D编辑器，如创建模型搅拌机，玛雅，或Cheetah3D，你提供了巨大的动力和灵活性来设计和动画制作精良的3D模型。这些编辑器允许您将模型导出格式Cocos3D可以导入。

Cocos3D支持的PowerVR POD 3D模型文件格式，这是适合于在一个运行时环境有效装载致密，二进制格式。

要导出POD您的3D编辑器文件，请按照下列指示。一旦生成，你应该添加您POD的资源文件到您的Xcode项目。

注：该POD文件格式可以包括，以覆盖该模型中的纹理的引用POD文件，但POD文件格式不嵌入内侧这些纹理POD文件本身。一定要在你还可以添加在资源Xcode项目，由您的POD模型所需的纹理文件。
要加载的内容POD在文件Cocos3D，实例化一个CC3PODResourceNode从实例POD文件，如下所示：

```
CC3ResourceNode* rezNode = [CC3PODResourceNode nodeFromFile: @"hello-world.pod"];
```
通常情况下，你会从您的自定义的方法之一调用此3D场景。

上面的代码行创建了一个资源节点，并从指定的内容填充该POD文件。一个资源节点是一种特殊类型的结构的节点，其内容从一个资源文件的装载而得。资源节点在代表Cocos3D的CC3ResourceNode类。CC3PODResourceNode是一种资源节点的子类，它能够加载内容POD的文件。

加载后，内容POD文件显示为的后代节点CC3PODResourceNode实例。

作为一个具体的例子，上面的代码线被从取CC3HelloWorld从模板应用Cocos3D分布。由此产生的CC3PODResourceNode实例具有以下结构：

```
CC3PODResourceNode 'hello-world.pod':5
  CC3PODMeshNode 'Hello':6 (POD index: 0)
```
这个非常简单的一个资源节点的示例包含显示为资源节点实例的孩子只有一个网状节点实例。

一个更复杂的例子，从使用的机器人手臂模型采取CC3DemoMashUp从演示程序Cocos3D分布如下：

```
CC3PODResourceNode 'RobotPODRez':5
  CC3PODMeshNode 'Cylinder01':6 (POD index: 0)
    CC3PODMeshNode 'BottomArm':7 (POD index: 1)
      CC3PODMeshNode 'TopArm':8 (POD index: 2)
  CC3PODMeshNode 'GeoSphere01':9 (POD index: 3)
  IntroducingPODLight 'FDirect01':10 light index: 0 (POD index: 0)
  CC3PODCamera 'Camera01':11 (POD index: 0)
  CC3PODNode 'Camera01Target':12 (POD index: 6)
  CC3PODNode 'FDirect01Target':13 (POD index: 7)
```
本实施例中加载一个单一POD，结果在一个文件中CC3PODResourceNode，其中包含一个完整的实例结构节点组件包括一个机器人臂，一个相机节点，一个光点。和一对夫妇看不见的节点作为目标的摄像头和光线。

一个CC3PODResourceNode一个完整的动画人物（如来自流道和龙CC3DemoMashUp演示应用程序）可能包含数十个子节点。

不论的POD文件的内容的复杂性，加载的结果是一个单一的CC3PODResourceNode包含从所有节点实例POD文件作为后代节点。这种资源节点然后可以在场景中的任何地方添加，如场景本身，或在现场的一些其他的子节点的子：

```
[self addChild: rezNode];
```

通过这样做，该资源节点及其所有子节点被添加到场景中。

在上述第二示例中，添加整个资源节点到场景也将相机和光添加到场景为好。在相机的情况下，如果这是被添加到场景中的第一相机，摄像机将自动在设置现场的属性实例。activeCamera

既然是一个节点，你可以保留的一个参考CC3PODResourceNode实例来操纵整个模型，定位，旋转，缩放，颜色或褪色完整的模型为一个单位。

如果你不希望一个资源节点的全部内容添加到您的场景（也许你只想增加从资源的一个或两个特定节点），则可以使用getNodeNamed:的资源节点实例的方法来检索各子节点（或节点集）。例如，您可以只提取机器人手臂器节点组件，并添加如下的情景：

```
[self addChild: [rezNode getNodeNamed: @"Cylinder01"]];
```
这增加了，而不是仅仅在气缸节点，而是整个机器人手臂到现场，因为这些节点是气缸节点的结构后代。

如果你不希望保留的资源节点本身的引用，你可以用方便的方法addContentFromPODFile:或addContentFromPODFile:withName:任一节点（包括现场节点）上加载POD文件，并自动添加新的资源节点实例作为节点的子。
#### 资源缓存
基础的资源节点实例是实例CC3Resource类，它实际上处理文件内容的加载。

资源被缓存，并多次加载相同的POD单线程循环中的文件实际上不会导致POD文件可加载一遍又一遍。一旦POD文件被加载一次，后续调用加载它会从缓存中检索。

为方便起见，资源高速缓存自动地在当前线程循环结束清除。资源节点和模型所包含将继续存在，如果你已经将它们添加到现场，或通过其它方式保留了他们在其他地方，但同样加载它们会导致他们从文件中再次加载。

如果要加载相同的POD重复文件，在多个线程的回路（也就是说，作为载入游戏的每个级别的一部分），你可以安排将您的资源永久保存，如果你的类侧设置CC3Resource.isPreloading属性YES之前，加载该POD文件如上所述，以表明你是预加载资源，并希望它被永久缓存，直到你从缓存中手动删除它。随后可以设置isPreloading属性NO，以允许载入未来资源和在当前线程循环结束从缓存清除。
### 添加参数内容
除了从外部资源加载的内容，您可以通过编程创建简单的网格的内容。基本形状像飞机，圆盘，线，盒，球体，圆柱体，并且可以通过代码参创建锥体。

#### 预先定义的参数丝网​​形状

在CC3MeshNode与CC3Mesh类定义了一些方便的方法用于创建参数形状。这些方法可以在被发现ParametricShapes在类扩展类别CC3ParametricMeshNodes.h和CC3ParametricMeshes.h文件。方法创建以下类型的形状存在：

扁平的三角形
扁平矩形
平盘
线框箱
实心方框
为立方体地图固盒
领域
锥体
线带
对于这些，您可以指定参数，如大小和镶嵌。

例如，您可以快速通过实例化创建一个矩形平面CC3MeshNode实例，然后调用populateAsCenteredRectangleWithSize:来填充内部网格法。同样，您可以使用populateAsSphereWithRadius:andTessellation:方法来填充内部网格作为一个球体来代替。

#### 从头开始构建网格
除了建立上述预先定义的形状之一，您也可以通过编程从头开始，通过指定每个顶点自己的内容创建一个网格。这允许您创建的列表中没有涵盖上述的其他参数网格，或者允许您创建完全自定义的拓扑网格。

要建立一个新的网格，通过实例化一个启动CC3Mesh的实例。由于新的网格开始是空的，你首先需要定义的，它包含的内容的类型和数量：

设置vertexContentTypes属性来定义的内容类型，将每个顶点来定义，如顶点位置，法线，纹理坐标等。
设置allocatedVertexCapacity在网格属性顶点的数量。这种分配足够的内存来保存你指定的顶点数量，使每个顶点可以包含在你指定的内容类型的vertexContentTypes属性。
如果要使用索引顶点更高效的渲染，设置allocatedVertexIndexCapacity在网状财产顶点索引的数量。
一旦你分配你可以建立一个网格编程方式使用同一个家庭的setVertex...:at:使用方法，访问顶点的内容进行修改，一个网格顶点或顶点索引。

你已经建立了你的顶点网格后，实例化一个CC3MeshNode实例及其网格属性设置为你刚刚建立的网格实例。您可以在此网实例分配到任意数量的网格节点的实例。

由于建立一个网格顶点，由顶点从头开始可能会有点势不可挡，可用于指导上面列出的预先定义的参数网格的实现。这些方法的实现可以在中找到CC3ParametricMeshes.m的文件。

### 添加内容动态
您现场过程中添加3D场景的初始内容初始化时，首先呈现在现场。对于某些应用，用相对静态的场景内容，这可能是足够了。对于其他应用程序，您可能需要动态添加内容的用户交互，计时器，游戏或其他互动要求的结果。

载入和创建内容需要时间和处理能力。对于少量的动态内容，您可以创建并添加内容上飞，利用在主线程中连续帧之间的空闲处理能力。

然而，一旦场景是活动的，可见的，它可能成为破坏性的动作加载和主线程上添加显著内容的流动。您可能会发现这样做会导致短，但明显的，跳跃在场景中的作用为主线需要时间来加载和添加内容。

如果这种情况发生在你的应用程序，也可以采取消除或同时加入新的内容到场景中尽量减少视觉干扰的若干方法。

#### 隐藏的内容，直到需要
第一种方法是加载和创建场景初始化过程中所有主要的内容，但对用户隐藏的视图的某些内容，使用下列方法之一：

所有需要的内容添加到您的场景，但设置visible属性为NO你不想让用户看到任何节点组件的顶部节点。

在您的应用程序创建一个屏幕外的节点缓存，来保存你创建你需要将它们添加到场景之前的节点。然后拉出来缓存并将其添加为需要的。根据您的需求的复杂性，你可能会写一本专门的缓存，或只使用一个标准的收集，如字典。

#### 复制的内容在需要时
如果您需要添加大量相同类型的节点集（例如，一个僵尸大军），可以屏幕外的节点集（一个僵尸）的模板创建，然后创建副本需要的时候这个模板的。该CC3DemoMashUp演示应用程序使用此技术来降大部队在轻触按钮现场入侵的机器人手臂。
#### 添加内容在后台
为了保持主线程明确的更新和渲染场景，并加快现场初始化，您可以卸载加入大量的内容，以辅助后台线程。

如果您正在加载引用OpenGL的行为的内容，如创建纹理，顶点缓冲，或着色器，必须在共享OpenGL的内容与你的主线程辅助线程执行此活动。为了支持在后台线程加载内容，Cocos3D提供了CC3Backgrounder类。这个类自动处理管理线程之间的OpenGL的组件。CC3Backgrounder是一个单，并且可以使用访问其实例sharedBackgrounder类端属性。

要在后台运行的活动，你简单地传递，你想在后台运行一个代码块runBlock:或runBlock:after:方法共享背景资料单的。该新闻背景将自动运行在后台线程代码块，而你的主线程继续呈现现有的场景内容。

该代码放置要在后台可以做同样的，你的场景在初始化过程中包括活动运行块内。您可以从负荷模型POD文件，加载纹理文件，编程方式创建内容，创建顶点缓冲区，并添加所有这些内容到你的场景。

例如，您可能会引发下面的代码动画龙模型添加到您的场景作为用户的触摸事件，定时器，更新，或其他现场互动的结果：

```
[CC3Backgrounder.sharedBackgrounder runBlock: ^{ 
    CC3ResourceNode* dgnRez = [CC3PODResourceNode nodeFromFile: @"Dragon.pod"];
    dgnRez.touchEnabled = YES;
    dgnRez.location = cc3v(0, 500, 1300);
    [dgnRez runAction: [[CC3ActionAnimate actionWithDuration: 3.0] repeatForever]];
    [dgnRez runAction: [CCActionFadeIn actionWithDuration: 0.5];
    [self addChild: dgnRez];
}];
```

在本实施例中，由于龙被添加到场景即用户已经可见，我们淡入龙，以增强用户的视觉效果。或者，你可以选择简单地弹出龙开始存在，或从屏幕外位置带来的龙。

在后台加载时可能是有问题的唯一内容是着色器。这是因为着色往往完成最后动态链接阶段，他们呈现首次一个节点之前，这将发生在主渲染线程上，即使该着色器装载在后台线程。这个最后链接阶段有时会导致在视觉动作一个短的暂停。

出于这个原因，您可能希望通过加载场景所需的所有着色器，场景初始化过程中，并永久缓存起来，这样他们将可以使用以后，当更多的场景内容将被载入。

要做到这一点，场景初始化期间，设置CC3ShaderProgram isPreloading类端属性YES，加载着色器程序，然后设置isPreloading属性回NO（如果你想）。例如，您可能包括像场景中的初始化代码如下：

```
CC3ShaderProgram.isPreloading = YES;

[CC3ShaderProgram programFromVertexShaderFile: @"CC3Texturable.vsh"
                        andFragmentShaderFile: @"CC3SingleTexture.fsh"];
[CC3ShaderProgram programFromVertexShaderFile: @"CC3TexturableBones.vsh"
                        andFragmentShaderFile: @"CC3SingleTextureReflect.fsh"];
[CC3ShaderProgram programFromVertexShaderFile: @"CC3PointSprites.vsh"
                        andFragmentShaderFile: @"CC3PointSprites.fsh"];

CC3ShaderProgram.isPreloading = NO;
```

这个例子开启着色器预压，使着色器程序被永久保存，载入了三个着色器程序，然后打开着色器预压回去了，所以未来的着色器程序不会被永久保存。
一旦这一活动已经场面初始化期间被执行，然后可以加载POD，使用了使用这些着色器程序模型CC3Backgrounder的单身，没有因为这些模型添加到场景中造成延误。

您可以在看到这种预加载的更完整的版本，preloadAssets该方法CC3DemoMashUpScene在课堂上CC3DemoMashUp演示程序。

由于背景资料自动管理OpenGL的内容，你只能在后台一次OpenGL的环境已经在主线程上运行的既定活动。对于大多数应用程序，这意味着你应该避免试图从内部发起背景资料任务initializeScene的第一个场景的方法。通常情况下，最早可以启动一个后台任务是在onOpen你的第一个场景的方法。一旦现场启动并运行，然后你可以开始从任何更新或用户互动事件后台任务。
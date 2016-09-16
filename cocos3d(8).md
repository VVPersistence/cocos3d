# cocos3d(8)
# 着色器
### 加载和编译一个着色器程序
着色器程序是由两部分组成：一个顶点着色器是运行在网格的每个顶点，并确定各顶点的位置和风格，以及片段着色器，其上运行的每个片段（像素）被呈现给屏幕上，并且确定每个像素的最终外观。这两种着色器必须共同努力，并链接整合到一个单一着色器程序。

在Cocos3D中，每个着色器程序被表示用的一个实例CC3ShaderProgram的类。反射的着色器程序的结构，的每个实例CC3ShaderProgram包含的两个实例CC3Shader类，一个表示顶点着色器，以及其他的片段着色器。

您可以加载，配置和链接CC3ShaderProgram使用类端的实例programFromVertexShaderFile:andFragmentShaderFile:方法，该方法为顶点和片段着色器，加载文件名 ​​和编译每两个着色器文件和链接它们来创建着色器程序。

着色器程序会被缓存。第一次调用此方法，它会自动添加着色器程序的着色器程序缓存。用相同的着色器的文件名参数此方法的后续调用将从缓存中检索现有的着色器程序的实例，而不是装载，编译和链接相同的着色器程序的另一个版本。因此，您可以只要你想在一个着色器程序连接到一个网状节点，而不必担心招致装载的开销，编译和存储相同着色器程序的多个副本使用此方法。

下面的例子说明了应用程序如何使用这种方法来着色器程序添加到一个网格节点：

```
CC3MeshNode * aMeshNode = ...;
aMeshNode.shaderProgram = [CC3ShaderProgram programFromVertexShaderFile： @“ MyVertexShader.vsh ”
                                                   andFragmentShaderFile： @“ MyFragmentShader.fsh ” ];
```
在Cocos3D，着色器装载包括导入使用标准的嵌套文件的支持#import指令。与其他代码管理，使用 #import指示让您灵活地组织你的shader代码到基于文件的模块。

下面是使用的例子#import指令模块化着色器代码，请从所CC3SingleTexture.fsh提 ​​供的片段着色器Cocos3D分布：

```
#import "CC3LibDefaultPrecision.fsh"
#import "CC3LibDualSidedFragmentColor.fsh"
#import "CC3LibLightProbeIllumination.fsh";
#import "CC3LibSingleTexture2D.fsh"
#import "CC3LibSetGLFragColor.fsh"

void main() {
    initFragmentColor();
    illuminateWithLightProbes();
    applyTexture2D();
    setGLFragColor();
}
```
在这个例子中，从调用的函数main()被导入着色器库文件（在也提供了提供功能Cocos3D分布），并且可以在各种其他着色的容易地重复使用。

#### 着色器缓存
从文件中加载着色器和着色器程序会被缓存，并多次加载相同的着色器文件实际上并不会导致加载着色器文件，一遍又一遍。一旦着色器文件被加载一次，后续调用加载它会从缓存中检索。

为方便起见，着色和着色器程序的高速缓存自动在未在现场被使用，或以其他方式在其他地方保留的任何着色的各线环的端部清除。

如果你想重复加载相同的着色器，而无需以确保它在某个地方保留下来，你可以安排，如果您的类侧设置纹理永久保存，CC3ShaderProgram.isPreloading财产YES上述加载着色器文件之前，指示你是预加载着色器和希望他们被永久缓存，直到你从缓存中手动删除它们。随后，您可以设置isPreloading属性NO，以允许从缓存中加载，清除未来着色器时的情景不再一部分。



### 着色器匹配

虽然你可以通过编程指定着色器程序到每个节点，如上面的例子中，当你的3D世界上有很多动态创建的节点，这种技术可以是相当繁琐和容易出错。相反，你可以依赖自动着色器匹配自动每个节点有其需要的最合适的着色器程序相匹配。

的CC3ShaderProgram类保留的实施方案的一个实例CC3ShaderMatcher中的协议programMatcher类侧属性。这一计划匹配单的目的是自动合适的默认着色器程序匹配到网状节点，如果你不明确指定该网格节点的程序。这种机制可以让你网格节点添加到场景中，而无需担心确切定义哪些程序着色器需要适当地加以渲染。

第一次的网格节点呈现，如果不具有着色器程序中定义，它被传递给在类侧节目匹配它programMatcher属性的网格节点到一个适当的着色器程序相匹配。然而，由于新的着色器程序的加载和编译可能需要一段时间，你也可以使用相同的机制建立在创建时的网格节点的着色器程序，使用selectShaderPrograms方法单一网格节点或节点集上，甚至整个场景。

缺省CC3ShaderMatcher的实现是CC3ShaderMatcherBase，它定义，载荷和高速缓存的一组标准着色程序的应用基于一定的匹配准则，如网格节点是否具有一个或多个纹理，或者是否正在使用凹凸映射啮合节点等你可以自由地更换自己的着色器匹配执行此默认实现和使用这种机制作为分配着色器程序网格节点的主要手段。



### 着色器变量：顶点属性和制服
编译顶点和片段着色器的代码执行在GPU上。像任何程序，所述着色器代码需要访问外部内容为了知道如何定位的顶点和呈现视觉内容。该外部内容通过传递到着色器程序顶点属性和均匀的变量。
#### 顶点属性
正如其名称所暗示的，顶点属性是描述一个顶点的属性的着色器程序的变量。顶点属性的例子是一个顶点的位置，颜色，或纹理坐标。每个顶点属性为阵列的着色器程序，每个顶点一个元件通过，但在着色器程序中，由于顶点着色器运行为每个顶点独立地各顶点属性显示为单个元件。

顶点属性只能传递到着色器程序语义。这是自动处理的，由CC3VertexArray包含在每个子类的实例CC3Mesh。
#### 制服
一个统一的变量是由应用程序来帮助着色器程序确定位置和每个顶点和片段的视觉风格传递给着色器程序的一般环境或语境参数。制服的例子可能顶点变革矩阵，光的位置和特征，材料颜色和纹理标识符。该应用程序也可以自由定义和传递任何其他参数可能会感兴趣的着色器程序。

制服可以由该应用手动传递到着色器程序，或更常见的是，可以在语义上通过，如在下面的部分中描述的着色器语义。

#### 手动设置统一的变量
最直接的方法来设置统一的变量的值在着色器程序是从应用程序内编程设置它们，任何特定的网格节点的更新过程中。网格节点包含的一个实例CC3ShaderContext，它管理的着色器程序的上下文时，它呈现该节点。要设置用于该节点的着色器程序一个统一的变量的值，可以使用从程序上下文请求变量uniformOverrideNamed:的方法，然后该变量的值设置为你想要的渲染节点时使用的值材料。你可以做到这一点，如下所示：

```
CC3ShaderContext * shCtx = meshNode.shaderContext;
CC3GLSLUniform *均匀= [shCtx uniformOverrideNamed： @“ u_cc3MaterialDiffuseColor ” ];
uniform.color4F = kCCC4FMagenta ;
```
在此代码，u_cc3MaterialDiffuseColor是一个统一的变量的名称，如着色器源代码的声明。在着色器程序中定义的每个均匀变量由实例表示CC3GLSLUniform。这个类有许多不同的setter方法设置统一的价值，取决于如何统一变量已着色器源代码中声明。该变量的值可以被设置为一个整数，浮点矢量，颜色或矩阵型，无论是作为一个单一的值，或作为在阵列中的元素。见Cocos3D关于更多信息API文档CC3GLSLUniform和可用于设置统一的值的方法。

虽然制服可以直接设置，依靠这样的负担会用胶水代码的大量应用程序的所有制服。相反，对于设置顶点属性和均匀的变量值所建议的机制是定义着色器语义，其提供声明的着色器变量和上下文内容之间自动映射。然后可以使用上面只描述的方案均匀设定技术来覆盖语义衍生，如果需要的话，或以增强和填充，不能由一个语义自动导出均匀变量值的值。着色器语义将在下一节中介绍。

### 着色器语义
着色器的语义是一致的声明语义在着色器程序中声明的顶点属性和制服之间的映射，以及3D场景的上下文环境的内容。这意味着，在本质上，宣称一个特定制服，如u_cc3MaterialDiffuseColor上述确定的均匀的变量，语义意味着覆盖当前正在呈现的节点的材料的漫反射颜色。通过建立的顶点属性和均匀的变量和它们的语义含义之间的映射，我们允许框架自动提取并应用适当的内容（在本例中，当前的漫射材料的颜色）作为着色器程序呈现的每个节点。这将自动发生，并且应用程序需要的任何节点的更新或渲染的时候不采取任何行动。

广泛的默认和标准的内容语义在定义CC3Semantic枚举。此枚举意图是开放式的，您可以自由定义进一步的语义内容可能使用在特定的顶点和片段着色器。请参阅的文件和声明CC3Semantic枚举这些标准的节目内容语义的更多信息，以及如何添加自己的自定义内容的语义。

每CC3ShaderProgram持有到其特殊的委托对象semanticDelegate属性。这代表的是一个执行CC3ShaderSemanticsDelegate协议。在程序链接，着色器程序会自动调用configureVariable:这个委托方法给每个变量映射到一个程序的语义。然后，运行时节点渲染，每当着色器程序需要填充一个变量的值时，它使用该语义关系自动填充顶点属性或均匀变量的值。

顶点属性的变量，语义将被自动解析到的一个CC3VertexArrays由保持CC3Mesh。对于统一变量着色器程序调用populateUniform:withVisitor:的委托方法来获取环境适量本品均匀值。此方法的实施方案可以检索从网格节点本身的均匀值（如在上述的实例中u_cc3MaterialDiffuseColor以上给出均匀），或者它可以进一步考虑场景或运行时环境的统一内容（例如，以检索照明或照相机的环境的内容）。

你可以自由地创建你希望该协议的任何实现，但Cocos3D提供CC3ShaderSemanticsByVarName一流为标准的默认实现。如果要定义自己的语义映射，一个非常便捷的方式为您扩展标准变量语义，或者定义新的语义，是一个方法添加到CC3ShaderSemanticsByVarName类，在Objective-C的类别，并有该方法调用mapVarName:toSemantic:方法为每个自定义的统一的变量名，每个映射到一个标准或自定义，CC3Semantic。然后，在应用程序初始化时，您可以通过访问默认调用自定义扩展方法CC3ShaderSemanticsByVarName的默认程 ​​序匹配召开的实例：

```
 CC3ShaderSemanticsByVarName * semDel =（CC3ShaderSemanticsByVarName *）CC3ShaderProgram.shaderMatcher.semanticDelegate;
    [semDel populateWithMyCustomVariableNameMappings ]

```
当默认的语义映射自动添加populateWithDefaultVariableNameMappings方法是在这个类调用，您可以使用此方法的实现，作为加入自己的语义映射的模板。



### 加载着色器程序，并使用影响文件语义
如果您正在使用现有的顶点和片段着色器，或已在其他环境中运行开发着色器，你也可以是具有着色器源代码的立场并不意味着可以很容易地映射到一个标准的一个标准的变量命名约定语义集。在这种情况下，Cocos3D允许您定义着色器程序的变量名，并使用着色器语义之间的映射关系影响的文件，它在运行时，当您加载着色器程序加载。

正如其名称所暗示的，一个效果文件包含对于一组的效果，其中每个效果收集顶点和片段着色器一起的定义，顶点属性的声明和由着色预期均匀的变量，这些变量的语义，和任选该着色器所期望的纹理应用于材料。

Cocos3D使用来自该PFX影响文件格式的PowerVR SDK由Imagination Technologies公司，在iOS设备中使用的GPU的供应商。该PowerVR的SDK是一样的工具包是POD模型文件出口商的来源。您可以下载这个SDK和工具在这里。在PFX文件影响的结构和内容在文件中描述PFX语言格式规范的范围内的PowerVR SDK下载。

PFX文件提供的机会，以声明每个顶点属性和均匀，以限定一个语义的每个Cocos3D然后可以自动地映射到的语义之一CC3Semantic枚举。

您可以应用效果网格节点如下：

```
[meshNode applyEffectNamed： @“ EtchedEffect ”  inPFXResourceFile： @“ MaskEffects.pfx ” ];
```
其中，EtchedEffect是包含在PFX效果文件名 ​​为效果MaskEffects.pfx。

如果您的网格节点从POD文件加载，你也可以识别POD文件中的每一种材料的定义要加载的效果和作用的文件。这提供了一个完全自动化的分辨率。你只需从你的应用程序加载POD文件，然后将文件POD会自动识别并加载需要提供的POD文件中所述的材料效果和语义的PFX文件。您可以使用PVRShaman刀具从PowerVR的 SDK工具编辑您的POD文件设置为使用在POD文件中的每个材料的影响。

在幕后，PFX文件正在使用的加载CC3PFXResource类。像其他资源，加载CC3PFXResource自动放置在一个高速缓存中，因此它可以由许多不同的节点的引用。当您试图访问使用方法的效果如你不需要管理这个加载和缓存，因为它会自动出现applyEffectNamed:inPFXResourceFile:上述方法。

一个专门的语义委托CC3PFXShaderSemantics（的子类CC3ShaderSemanticsByVarName）提供包含在PFX文件文本语义和所列举的语义之间的映射CC3Semantic枚举。

Cocos3D提供了一个默认的文本PFX语义映射CC3PVRShamanShaderSemantics类，它在默认情况下，当您使用方法加载PFX文件，或访问效果，如应用applyEffectNamed:inPFXResourceFile:中CC3MeshNode。这个缺省映射是由内部使用的相同的映射PVRShaman从工具的PowerVR工具箱。该PVRShaman工具可用于查看POD文件和开发和测试着色器和PFX文件。通过使用这种PVRShaman内语义映射Cocos3D，POD模型和着色器可以开发并使用测试PVRShaman，然后中运行Cocos3D没有变化。

名单PVRShaman语义定义可参见附录A中的PVRShaman用户手册，其中包括在PVRShaman配送为一体的一部分PowerVR的SDK下载。

如果你想在PFX文件来定义，而不是使用默认自己的语义，PVRShaman语义，你也可以继承CC3PFXShaderSemantics来提供这种映射。

为了提高灵活性，您还可以映射到PFX文件本身语义统一变量混合标准统一的变量名，如上述，从PFX文件中加载shader代码，并排侧。但是，如果你正在尝试开发可运行在两种着色器Cocos3D和PVRShaman，要知道，这些内部标准变量将无法使用PVRShaman（除非你PFX文件中映射它们）。

在该方法中演示应用程序，包括在Cocos3D分布，提供装载PFX文件（实施例），通过上述方法。
CC3DemoMashUpScene addPostProcessingCC3DemoMashUpPostProc.pfxapplyEffectNamed:inPFXResourceFile:

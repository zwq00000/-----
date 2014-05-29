测试驱动的开发模式（TDD）
========================

先零零碎碎的说一下。
原先对什么什么驱动的模式有些模糊，认为那些都属于概念炒作。其实炒作这些概念的人真的是如此认为的。
什么驱动决定着项目推进过程中的主导因素，也决定了代码的风格。
风格决定着开发人员的交流习惯，缺少这种交流的习惯，开发人员是会天天打架的。
单元测试是对自己负责，更是对团队负责的基本态度。
测试驱动（TDD）不一定要形式化为 先写测试在写实现，那是一种非常理想的状态，也可以实现一个类再去写测试，但是构思的过程中时刻考虑着单元测试的需求。
#从代码风格说起#
从VB（不是VB.Net，那个时代还有一个奇迹就是 Power Builder）盛行开始，窗体就是代码的全部。初学者常常习惯把所有代码都放到窗体事件中，能够写几个类的就算是高手了。当时几乎所有的教程都是这样叫人写程序的，从Hello World 开始就如此。
那么这种风格也是一种风格，即界面驱动的开发方式，这样做非常符合人的直觉。
既然是界面驱动，那么测试手段就是把界面上所有的功能都用鼠标点一遍，出了错误再单步调试。
好了，问题出来了，在稍微复杂一些的应用中，测试环节是最低效的环节，还会引发开发部门和测试部门的矛盾。
其实开发人员也没有什么底气，不敢说代码一点问题也没有，只是强调要如何如何使用才是正确的。
这样的代码风格当然无从说起，非常凌乱，由于业务封装需要很强大的开发，所以通常会交给专门的团队开发，封装好了再给项目用，当时最盛行的是各种控件，而且控件越来越复杂。

还好，软件行业还是在一直进步的，而且一直自己要求进步。
MVC 来了（我第一次认识MVC是从Java的界面设计开始的，大约是1997年），所有程序都可以如是进行拆解，结构精巧而且清晰。
设计模式中不认为MVC是一种设计模式，认为MVC是一种代码组织方式，不限于OO设计，可以适用于多种层次。
MVC带来的好处是，可以独立开发控制功能和数据模型，甚至没有界面的时候就可以独立运行和测试Model和Control 的组合功能。
虽然MVC对领域模型的解构直接带来的好处就是把业务模型从界面逻辑中拆分处理，使每个部分可以单独存在，但是并没有针对单元测试进行的优化。

开发模式前进的动力来自组织内的分工，精细化的分工要求知识和业务相对完整和独立，无限细分不一定带来生产力的提高，反而增加了沟通成本和形成复杂冗余的设计。

#解构的目标#
从最初界面和业务逻辑都由一个人完成的微型软件到几十人的团队的开发，最大的区别是任务分解，一个人开发可以不需要分解任务（其实也需要的啦，只是都在一个人脑子里）。分解成需要相对独立的任务，分解的粒度为一个人时效率最高。（*这是前提假设，如果假设不成立可以明确指出问题所在*）

组件之间的解耦是OO设计的重点，所谓高内聚低耦合，是OO设计的基本原则。但是在现实中这个设计目标其实很难验证，组件之间的关联关系天生就是这么复杂，相互依赖。
如果一个组件的单元测试很难写，那么就意味着这个组件依赖的组件环境比较复杂，以至于无法很容易的搭建这个基础。也就是和一个复杂的组件耦合度过高。
好比构造一个建筑，这个建筑的每个房间设计都不依赖于特定的空间结构，那么这个房间可以事先搭好，然后现场组装。
一个组件的测试难度取决于相对基础类型（如 byte、int、float和string）的层级。
一个实体类（POCO类型）相对于基础类型层级为1，一个实体类的类型化集合相对层级为2。
```C#
public class SampleEntry{
    //名称
    public string Name{get;set;}
    
    //值
    public int Value{get;set;}
}

[TestFixture]
public class SampleEntryTest{
    
    [Test]
    public void TestConStructor(){
        var entry = new SampleEntry(){Name = "Test1",Value = 1};
        Assert.IsNotNull(entry);
        Assert.AssertEqual(entry.Name,"Test1");
        Assert.AssertEqual(entry.Value,1);
    }
}
```
比如 SampleEntry 实体初始化的方法很简单，一行代码就可以完成，单元测试成本很低。

**从这个意义上说，TDD本质上是一种任务分解的方法论和最佳实践。**
你可能会说，这个例子太简单了，没有借鉴意义，非常对，因为...
问题还远远没有结束。

```C#
/// <summary>
///  遍历程序集集合，根据查找条件增加类型节点
/// </summary>
/// <param name="assemblys"></param>
/// <returns></returns>
public void LoadAssemblies(IEnumerable<Assembly> assemblys){
    if (assemblys == null){
        throw new ArgumentNullException("assemblys");
    }
    BeginUpdate();
    foreach (Assembly assembly in assemblys){
        //检查Assembly 是否已经加载
        string assemblyName = assembly.GetName().Name;
        if (_assembliesRootNode.HasChildren(assemblyName, false)){
            //判断程序集名称是否出现在程序集树节点中
            _assembliesRootNode.Nodes.Find(assemblyName, false).First().ExpandAll();
        } else{
            var assemblyNode = new AssemblyTreeNode(assembly);
            assemblyNode.FullChildNodes(TypeFilter);
            //程序集节点没有子节点,即该程序集中没有符合条件的类型
            if (assemblyNode.HasChildren()){
                _assembliesRootNode.Nodes.Add(assemblyNode);
            }
        }
    }
    EndUpdate();
    _assembliesRootNode.Expand();
}
```

这是一个实际例子，*LoadAssemblies*负责解析程序集集合，把符合过滤条件的类型（Class）生成TreeNode并增加到TreeView视图中，这个方法是可重入的，即可以多次调用该方法，以便手动增加程序集。
AssemblyTreeNode 是程序集节点，它下面是命名空间节点和类型节点。
这个功能虽然运行良好，但是完全没法进行单元测试，因为它根本没有输出。
测试方法必须初始化整个窗体，然后打开窗体进行手工测试。
这个方法把几项任务结合到了一起，遍历程序集->查找所需的类型->增加程序集节点（程序集节点负责增加命名空间节点和类型节点）->展开树节点。
从功能上符合语义要求，即命令式编程的要求，但是不符合单元测试要求。
这段代码的功能无法离开组件环境独立存在，也无法得到更多的关注，因为它只有一个唯一的调用者。
什么是代码中的坏味道，就是当无法用准确的名称命名一个方法时就说明这个方法中承载了过多的功能。
我也在犹豫这点功能到底叫什么比较合适，是 *AddFromAssemblies*、*LoadAssembliesToAddTreeNodes*，还是*AddAssemblyNodes* 或者*ParseAssemblies*。

特别是这两句
```C#
        var assemblyNode = new AssemblyTreeNode(assembly);
        assemblyNode.FullChildNodes(TypeFilter);
```
构造了一个 *AssemblyTreeNode*的对象却不一定增加到树节点中，就这样轻易的抛弃了。

如果不关注整个类和他所处的环境，这些细节探讨是没有意义的，如果一个类可以正常运转就算做OK的话。
AssembliesTreeView类的是这样的，为了简化只显示方法定义。
```C#
// 支持类型过滤功能的程序集树视图
public class AssembliesTreeView : TreeView{
    public AssembliesTreeView(){}
    
    public AssembliesTreeView(Func<Type, bool> typefilter){
            TypeFilter = typefilter;
    }
    // 类型过滤器
    public Func<Type, bool> TypeFilter { get; set; }

    // 初始化 树视图
    public void InitTreeView(){
        InitTreeRoot(_baseType);
    }
    
    // 增加程序集节点
    public void AddAssemblyNodes(IEnumerable<Assembly> assemblys){
        ...
    }
    
    /// <summary>
    ///     程序集 子树节点
    /// </summary>
    internal class AssemblyTreeNode : TreeNode{
     ...
    }
}
```
AssembliesTreeView 集中了几个要素，1、类型过滤器 2、树视图控件 3、子树节点（程序集下面是命名空间）的分解。
如果单讲一个功能性TreeView控件，是没有问题的，但是如果扩大一下应用场景，这部分的代码就失去了意义，比如用在BS前端或者没有界面的后台处理或者用于 WPF等。
在MVC架构中**V**和**C**都是无法从具体的界面框架中分离出来，导致**M**也遭受染污。在 WinForm 体系中，TreeNode 实际上是一个非常轻量级的组件，甚至没有继承 Control,但是也不适宜用于其他场景（难以想象ASP.Net MVC 的后端竟然引用 System.Windows.Form 程序集）。
那么如何把算法和模型抽象出来呢？
在正统的Java设计领域通常把 POJO 对象分为三类:
>- VO (Value Object) 值对象
*通常用于业务层之间的数据传递，和PO一样也是仅仅包含数据而已。但应是抽象出的业务对象,可以和表对应,也可以不,这根据业务的需要.个人觉得同DTO(数据传输对象),在web上传递。*
- BO (Business Object) 业务对象
*从业务模型的角度看,见UML元件领域模型中的领域对象。封装业务逻辑的java对象,通过调用DAO方法,结合PO,VO进行业务操作。*   
- PO (Persistant Object) 持久对象
*在o/r映射的时候出现的概念，如果没有o/r映射，没有这个概念存在了。通常对应数据模型(数据库),本身还有部分业务逻辑的处理。可以看成是与数据库中的表相映射的java对象。最简单的PO就是对应数据库中某个表中的一条记录，多个记录可以用PO的集合。PO中应该不包含任何对数据库的操作。*

我们要做的就是实现 **BO**，即业务对象。业务对象中包含业务逻辑，可以独立存在，通过 **VO** 适配不同的视图。这种适配包括不同体系架构的适应性，使之独立于界面实现。

*实现**BO**有什么好处吗，好像很麻烦耶。*

好处就是，你可以踢开数据库存储，踢开界面实现，单独对**BO**编写单元测试。
我经常想到下面这句话
>**我想到了一个绝妙的实现方法，只是没想好放到哪里最合适。**

答案是，很多不知道该放到哪里的代码，其实应该放到**BO**中。

**BO**包含了业务逻辑和必要的数据，自身形成一个自完备的逻辑体系，不依赖任何具体实现，是绝佳的业务抽象层。
同时构造和回收的成本很低。







> Written with [StackEdit](https://stackedit.io/).
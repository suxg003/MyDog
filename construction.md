## MyDog 结构概述

### 模块及功能介绍

##### mydog-core <br/>
  
* 包含生成代码的核心类，以及相关定义类

##### mydog-web <br/>
* 使用者的入口web项目，用户用过mydog-web界面选择和定制自己要生成的代码，最后一件生成。

##### mydog-shell <br/>
* 一个全功能的命令行操作入口，不喜欢web的用户可以通过该入口进行代码生成。

##### mydog-plugin-xxx

* ``xxx`` 代表插件的名字，一个插件就是一个功能的定义，在插件中可以定义本功能相关的信息。
* 初版里的插件有: 
  1. **project**    --> 生成项目的工程相关文件
  2. **datasource** --> 生成项目的数据源相关文件
  3. **entity**     --> 生成实体，实体相关的后台文件
  4. **entityui**   --> 生成实体对应的前段文件

### 特点

##### MyDog的``插件化``是最大的特点。

 1. 通过mydog-web，让用户自己选择自己需要的功能（插件）
 2. 插件是按照功能进行隔离的，一个功能只在插件内封装，而不会向外扩散。这样对后期的开发和维护都提供了便利。
 3. 在代码生成的过程，mydog-core的执行器不关注具体功能，插件化使得添加或取出功能对生成的过程无感知。即生成的过程与生成的内容解耦。

##### 插件接口定义



### 代码生成过程

关键类在子项目 ``mydog-core`` 的 ``org.huangpu.mydog.core.flow.FlowController``这个类中。


##### 生成代码过程示意图

![代码生成流程图](https://raw.githubusercontent.com/PowerShenli/MyDog/master/mydog-doc/src/main/resources/mydog_gen_flow.png)

##### 生成代码的步骤如下：
1. 启动系统后初始化插件。这一步系统初始化所有已知的插件定义，并加载插件相关的配置到内存。
2. 用户通过mydog-web选择所需要的插件，按照需要进行配置（填写元数据）。这里一是选择要什么功能，不要什么功能。二是选择过填写要的插件的具体定义，这里可以做的很强大，比如数据源插件可以支持业界大部分数据源类型，并当用户选择其中一种后，继续让用户填写此种数据源需要填写的元数据，或给出默认配置等等。三是选择最终生成的格式。<br/> 用户选择好后，点击“一键生成”，mydog-web会将生成好的元数据发送给FlowController。
3. FlowController 首先检测元数据对应插件的依赖关系，比如datasource依赖Project，那么选择datasource则必须选中project，校验不通过则提示用户。
4. FlowController 解析``元数据``列表. 这里的元数据可以理解为一系列用户选择的插件生成的功能定义。可以参考 demo.json
5. 合并元数据属性。这是为了应对新添加的插件对已有插件进行扩展的情况. 比如app.properties 是由 mydog-plugin-project 负责生成的，而mydog-plugin-datasource 插件也需要向 app.properties 中添加属性，则在这一步骤中将需要添加的属性合并到 project插件对应的属性中，最终一并生成。 （这里引入了两个问题：插件的依赖问题 与 属性的可见性问题。后面会说明）
6. 迭代元数据列表，找到每个元数据对应的功能插件，通过插件定义找到生成器、输出项，然后调用生成器执行代码生成动作，这一步输出的代码以字符串存储在内存中。
7. 将生成好的代码按照持久化配置方式进行持久化（目前是简单的写文件，未来可能扩展为war包、jar包、docker 镜像、甚至直接部署至云平台并启动运行  等等）



### 一个插件都包含哪些内容
* 依赖哪些插件（属性）
* 对哪些插件产生影响
* 最终会输出些什么（file？sql？）
* 用什么生成器生成，怎么生成。
* 用户都有哪些选择？（对应mydog-web改如何展现）

**这也正是插件接口的定义** ：

```java
public interface MyDogPlugin {

    /**
     * 初始化
     */
    void init();

    /**
     * 获取元数据类型
     * @return 元数据类型
     */
    String getMetadataType();

    /**
     * 获取本插件依赖的其它插件的属性
     * @return 依赖的属性
     */
    JSONObject getDependencyProps();

    /**
     * 获取元数据属性解析器
     * @return 元数据的属性解析器
     */
    MetaDataPropResolver getPropResolver();

    /**
     * 获取该插件输出定义
     * @return 输出定义
     */
    OutputDef getOutputDef();

    /**
     * 获取该插件的元数据相关的装饰器
     * 该方法是为了让插件可以扩展默认生成器的行为
     * @return 返回对应装饰器
     */
    GeneratorDecorator getGeneratorDecorator(Generator generator);

    /**
     * 获取用户可选择的内容
     * @return
     */
    JSONObject getViewProps();

}
```

**这些接口**
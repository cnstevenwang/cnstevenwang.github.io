# 结论

## 实例模式
Spring给到两种实例模式：
1. 全局单例模式，对应注解为 @EnableStateMachine
2. 工厂模式，对应注解为 @EnableStateMachineFactory

## 执行模型

先来一波名词解释：

+ State：状态，等于图模型的节点
+ Transition：状态间的连线
+ Event：事件，状态间变更的触发方式之一，另外一种方式是Timer
+ Action：动作
+ Guard：条件

StateMachine的执行顺序取决于 1. 节点类型 2. 点线间的顺序 3. 条件是否通过

具体结论如下：`这些结论是基于External类型Transition`

### 关于节点
需要设定起始节点，可以设定起始动作。  
状态机开始后，会先执行起始动作，然后进入节点。  

节点上分为下述动作

+ EntryAction：进入状态时执行；此时状态已经变更该进入的状态。
+ StateAction：在状态上的动作，执行完EntryAction后执行。
+ ExitAction：离开状态时执行，在下一个状态的EntryAction前执行；此时状态仍为将要离开的状态。

常用的节点类型

+ 一般节点
+ Choice：选择节点；该节点会关联几个“选项节点”，进入该节点后，找到第一个符合Guard条件的选项节点，并进入

### 关于连线
某个节点的下一个节点，由连线决定。  
走哪条连线的选择逻辑，和Event、Guard相关，具体选择逻辑如下：

1. 如果A节点之后的连线(直接跟在A后面的连线) Event相同（或者都没有配置Event），则按配置顺序选择第一个Guard为True的连线
2. 如果A节点之后的连线(直接跟在A后面的连线) 部分有Event，部分没有Event，即便触发的事件与配置的Event相同，也是优先跑没有Event的；这和配置的TransitionConflictPolicy相关
3. 如果A节点之后连续的两个连线 Event相同，只执行第一个连线逻辑；所以在选择连线时，既判断Event，也判断起始状态
4. 如果被选中的连线没有Event且Guard为True，则直接执行该连线
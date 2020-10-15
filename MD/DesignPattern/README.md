# 设计模式

## 概述

设计模式（Design Pattern）是前辈们对代码开发经验的总结，是解决特定问题的一系列套路。

它不是语法规定，而是一套用来提高代码可复用性、可维护性、可读性、稳健性以及安全性的解决方案。

## 软件设计模式的产生背景

“设计模式”这个术语最初并不是出现在软件设计中，而是被用于建筑领域的设计中。

1977 年，美国著名建筑大师、加利福尼亚大学伯克利分校环境结构中心主任克里斯托夫·亚历山大（Christopher Alexander）在他的著作《建筑模式语言：城镇、建筑、构造（A Pattern Language: Towns Building Construction）中描述了一些常见的建筑设计问题，并提出了 253 种关于对城镇、邻里、住宅、花园和房间等进行设计的基本模式。

1979 年他的另一部经典著作《建筑的永恒之道》（The Timeless Way of Building）进一步强化了设计模式的思想，为后来的建筑设计指明了方向。

1987 年，肯特·贝克（Kent Beck）和沃德·坎宁安（Ward Cunningham）首先将克里斯托夫·亚历山大的模式思想应用在 Smalltalk 中的图形用户接口的生成中，但没有引起软件界的关注。

直到 1990 年，软件工程界才开始研讨设计模式的话题，后来召开了多次关于设计模式的研讨会。

1995 年，艾瑞克·伽马（ErichGamma）、理査德·海尔姆（Richard Helm）、拉尔夫·约翰森（Ralph Johnson）、约翰·威利斯迪斯（John Vlissides）等 4 位作者合作出版了《设计模式：可复用面向对象软件的基础》（Design Patterns: Elements of Reusable Object-Oriented Software）一书，在本教程中收录了 23 个设计模式，这是设计模式领域里程碑的事件，导致了软件设计模式的突破。这 4 位作者在软件开发领域里也以他们的“四人组”（Gang of Four，GoF）匿名著称。

直到今天，狭义的设计模式还是本教程中所介绍的 23 种经典设计模式。

## 设计模式的分类

### 根据目的分类

根据模式是用来完成什么工作来划分，这种方式可分为创建型模式、结构型模式和行为型模式3种。

- 创建型模式：用于描述“怎样创建对象”，它的主要特点是“将对象的创建与使用分离”。
- 结构型模式：用于描述如何将类或对象按某种布局组成更大的结构。
- 行为型模式：用于描述类或对象之间怎样相互协作共同完成单个对象都无法单独完成的任务，以及怎样分配职责。

### 根据作用范围分类

根据模式是主要用于类上还是主要用于对象上来分，这种方式可分为类模式和对象模式两种。

- 类模式：用于处理类与子类之间的关系，这些关系通过继承来建立，是静态的，在编译时刻便确定下来了。
- 对象模式：用于处理对象之间的关系，这些关系可以通过组合或聚合来实现，在运行时刻是可以变化的，更具动态性。

### GoF 的 23 种设计模式的分类表

| 范围\目的 | 创建型模式                   | 结构型模式                                        | 行为型模式                                                       |
| --------- | ---------------------------- | ------------------------------------------------- | ---------------------------------------------------------------- |
| 类模式    | 工厂方法                     | (类）适配器                                       | 模板方法、解释器                                                 |
| 对象模式  | 单例、原型、抽象工厂、建造者 | 代理、(对象）适配器、桥接、装饰、外观、享元、组合 | 策略、命令、职责链、状态、观察者、中介者、迭代器、访问者、备忘录 |

## 设计模式的设计原则

### 开闭原则（Open Close Principle）

**定义：对扩展开放，对修改关闭。**

开闭原则（Open Closed Principle，OCP）由勃兰特·梅耶（Bertrand Meyer）提出，他在 1988 年的著作《面向对象软件构造》（Object Oriented Software Construction）中提出：软件实体应当对扩展开放，对修改关闭（Software entities should be open for extension，but closed for modification），这就是开闭原则的经典定义。

这里的软件实体包括以下几个部分：

- 项目中划分出的模块
- 类与接口
- 方法

开闭原则的含义是：当应用的需求改变时，在不修改软件实体的源代码或者二进制代码的前提下，可以扩展模块的功能，使其满足新的需求。

### 里氏代换原则（Liskov Substitution Principle）

**定义：继承必须确保超类所拥有的性质在子类中仍然成立。**

里氏替换原则（Liskov Substitution Principle，LSP）由麻省理工学院计算机科学实验室的里斯科夫（Liskov）女士在 1987 年的“面向对象技术的高峰会议”（OOPSLA）上发表的一篇文章《数据抽象和层次》（Data Abstraction and Hierarchy）里提出来的，她提出：继承必须确保超类所拥有的性质在子类中仍然成立（Inheritance should ensure that any property proved about supertype objects also holds for subtype objects）。

里氏替换原则主要阐述了有关继承的一些原则，也就是什么时候应该使用继承，什么时候不应该使用继承，以及其中蕴含的原理。里氏替换原是继承复用的基础，它反映了基类与子类之间的关系，是对开闭原则的补充，是对实现抽象化的具体步骤的规范。

### 依赖倒转原则（Dependence Inversion Principle）

**定义：高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。**

依赖倒置原则（Dependence Inversion Principle，DIP）是 Object Mentor 公司总裁罗伯特·马丁（Robert C.Martin）于 1996 年在 C++ Report 上发表的文章。

依赖倒置原则的原始定义为：高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象（High level modules shouldnot depend upon low level modules.Both should depend upon abstractions.Abstractions should not depend upon details. Details should depend upon abstractions）。其核心思想是：要面向接口编程，不要面向实现编程。

依赖倒置原则是实现开闭原则的重要途径之一，它降低了客户与实现模块之间的耦合。

### 单一职责原则（Single Responsibility Principle）

**定义：一个类应该有且仅有一个引起它变化的原因，否则类应该被拆分。**

单一职责原则（Single Responsibility Principle，SRP）又称单一功能原则，由罗伯特·C.马丁（Robert C. Martin）于《敏捷软件开发：原则、模式和实践》一书中提出的。这里的职责是指类变化的原因，单一职责原则规定一个类应该有且仅有一个引起它变化的原因，否则类应该被拆分（There should never be more than one reason for a class to change）。

该原则提出对象不应该承担太多职责，如果一个对象承担了太多的职责，至少存在以下两个缺点：

- 一个职责的变化可能会削弱或者抑制这个类实现其他职责的能力；
- 当客户端需要该对象的某一个职责时，不得不将其他不需要的职责全都包含进来，从而造成冗余代码或代码的浪费。

### 接口隔离原则（Interface Segregation Principle）

**定义：要为各个类建立它们需要的专用接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。**

接口隔离原则（Interface Segregation Principle，ISP）要求程序员尽量将臃肿庞大的接口拆分成更小的和更具体的接口，让接口中只包含客户感兴趣的方法。

2002 年罗伯特·C.马丁给“接口隔离原则”的定义是：客户端不应该被迫依赖于它不使用的方法（Clients should not be forced to depend on methods they do not use）。该原则还有另外一个定义：一个类对另一个类的依赖应该建立在最小的接口上（The dependency of one class to another one should depend on the smallest possible interface）。

以上两个定义的含义是：要为各个类建立它们需要的专用接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。

接口隔离原则和单一职责都是为了提高类的内聚性、降低它们之间的耦合性，体现了封装的思想，但两者是不同的：

- 单一职责原则注重的是职责，而接口隔离原则注重的是对接口依赖的隔离。
- 单一职责原则主要是约束类，它针对的是程序中的实现和细节；接口隔离原则主要约束接口，主要针对抽象和程序整体框架的构建。

### 迪米特法则，又称最少知识原则（Demeter Principle）

**定义：如果两个软件实体无须直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。**

迪米特法则（Law of Demeter，LoD）又叫作最少知识原则（Least Knowledge Principle，LKP)，产生于 1987 年美国东北大学（Northeastern University）的一个名为迪米特（Demeter）的研究项目，由伊恩·荷兰（Ian Holland）提出，被 UML 创始者之一的布奇（Booch）普及，后来又因为在经典著作《程序员修炼之道》（The Pragmatic Programmer）提及而广为人知。

迪米特法则的定义是：只与你的直接朋友交谈，不跟“陌生人”说话（Talk only to your immediate friends and not to strangers）。其含义是：如果两个软件实体无须直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。其目的是降低类之间的耦合度，提高模块的相对独立性。

迪米特法则中的“朋友”是指：当前对象本身、当前对象的成员对象、当前对象所创建的对象、当前对象的方法参数等，这些对象同当前对象存在关联、聚合或组合关系，可以直接访问这些对象的方法。

### 合成复用原则（Composite Reuse Principle）

**定义：要求在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现。**

合成复用原则（Composite Reuse Principle，CRP）又叫组合/聚合复用原则（Composition/Aggregate Reuse Principle，CARP）。它要求在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现。

如果要使用继承关系，则必须严格遵循里氏替换原则。合成复用原则同里氏替换原则相辅相成的，两者都是开闭原则的具体实现规范。

## 设计模式的类型

创建型模式

- [ ] 简单工厂模式（Simple Factory Pattern）
- [ ] 工厂方法模式（Factory Method Pattern）
- [ ] 抽象工厂模式（Abstract Factory Pattern）
- [ ] 建造者模式（Builder Pattern）
- [ ] 单例模式（Singleton Pattern）
- [ ] 原型模式（Prototype Pattern）

行为型模式

- [ ] 命令模式（Command Pattern）
- [ ] 中介者模式（Mediator Pattern）
- [ ] 观察者模式（Observer Pattern）
- [ ] 状态模式（State Pattern）
- [ ] 策略模式（Strategy Pattern）
- [ ] 职责链模式（Chain of Responsibility Pattern）
- [ ] 解释器模式（Interpreter Pattern）
- [ ] 迭代器模式（Iterator Pattern）
- [ ] 备忘录模式（Memento Pattern）
- [ ] 模板方法模式（Template Method Pattern）
- [ ] 访问者模式（Visitor Pattern）
- [ ] 空对象模式（Null Object Pattern）

结构型模式

- [ ] 适配器模式（Adapter Pattern）
- [ ] 桥接模式（Bridge Pattern）
- [ ] 装饰模式（Decorator Pattern）
- [ ] 外观模式（Facade Pattern）
- [ ] 享元模式（Flyweight Pattern）
- [ ] 代理模式（Proxy Pattern）
- [ ] 组合模式（Composite Pattern）

J2EE 模式

- [ ] MVC 模式（MVC Pattern）
- [ ] 业务代表模式（Business Delegate Pattern）
- [ ] 组合实体模式（Composite Entity Pattern）
- [ ] 数据访问对象模式（Data Access Object Pattern）
- [ ] 前端控制器模式（Front Controller Pattern）
- [ ] 拦截过滤器模式（Intercepting Filter Pattern）
- [ ] 服务定位器模式（Service Locator Pattern）
- [ ] 传输对象模式（Transfer Object Pattern）

## 资料

- [CS-Notes - 设计模式](https://cyc2018.github.io/CS-Notes/#/notes/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F)
- [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/index.html)
- [Java设计模式：23种设计模式全面解析（超级详细）](http://c.biancheng.net/design_pattern/)
- [设计模式 | 菜鸟教程](https://www.runoob.com/design-pattern/design-pattern-intro.html)

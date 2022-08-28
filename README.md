## 什么是领域驱动

领域驱动，俗称DDD，即：domain-driven-design

首先，不能单纯的直接说DDD就是另外一种分层方式，这是错误的想法。

更不是什么dto转do转po，会让大家误以为不就是三层结构再加上各种实体的Mapper转换吗。

领域驱动是为了解决降低复杂的软件设计工程的一种思想。注意，我说的是思想，并不是代码的架构设计。

领域驱动，有两大部份：战略设计 和 战术设计。而核心思想是战略设计，即告诉我们如何正确有效的使用领域驱动的思想。

而战术设计通常是指我们的项目中在具体代码上是如何利用这个思想来转换成编程语言代码的一种设计分层实现。

在使用战术设计之前，首先得保证你已经对项目中有了科学的领域划分、聚合划分。在这之后才是使用战术设计达到代码的实现。

要想学透领域驱动，得先掌握好战略设计部份，这个推荐大家看：《实现领域驱动设计》这本书。此书我拜读了两遍，受益匪浅。

前几年微服务非常火热，当时也一直困扰我的一个难题是：如何划分微服务，这个服务如果切分。其实如果掌握了战略设计，相信大家心中就会有答案了。

在传统计的三层架构中，数据访问层是在最底层，由上至下，依次被上层依赖。而领域驱动恰恰相反，基础设施层是作为最外层来依赖各层的。

在三层架构中，我们通常会先设计数据库表，然后再根据表来一一对应各种逻辑服务。这将导致做项目时，最优先考虑的就是数据库的设计。

这将导致一个严重的问题是，忽略了软件本身内部的复杂性。并且通常我们用三层架构时，会使用贫血模型，并允许上层任意修改实体的属性。当时间久了之后，我们将会失忆忘记，这些属性在什么背景下会被修改，因为什么修改。在后续的优化时，更不敢随意修改代码。

再比如，原来我们使用RockerMQ，后公司要求变更为Rabbit时，我们不得不对原本没有问题的代码，最终被改的到处是BUG。并且难以做单元测试。

因此，对于复杂的项目来说，使用领域驱动是非常有必要的。就算你不用，我也建议大家需要花时间去学习它。

回到领域驱动的战术部份来说，领域驱动分为四大层：接口层、应用层、领域层、基础设施层。具体各层如何依赖的以及职责，详见下文

另外，使用领域驱动的项目中，我在.NET和GO都有开源项目供大家学习。

也欢迎大家学习GO，加入到我们GO团队中一起维护GO框架。[farseer-go](https://github.com/farseer-go)

## 接口层

该层会依赖应用层，这里要注意的是，不能在此层调用领域层。所有对领域层的调用，需要通过应用层来发起。

通常我们的Web、Api、控制台、Winform就属于接口层。本身没有任何逻辑，仅是作为对外的输入、输出入口。

## 应用层

该层只有一个依赖，那就是：领域层。

应用层，会有应用服务。用于提供给到接口层的调用。应用层主要是解决：

1、数据库事务控制

2、1个或多个限界上下文下的聚合（或领域服务）的调用。

3、领域事件发布。

应用层不会有太多的逻辑，主要是为了控制调用的顺序，起到一个组织协调的作用。类似于设计模式的：外观层。

应用层还会定义一个实体类DTO。DTO允许被接口层直接使用。在调用聚合的时候，通常也会把DTO Mapper 成DO

开头说过，应用层只会依赖领域层。那我们需要数据库持久化方案怎么处理呢，我们可以利用IOC的依赖反转，通过IOC获取仓储接口的实例来达到持久化目的。

## 领域层

业务逻辑的主要实现，不依赖任何层、也不依赖技术组件（如：数据库、缓存等）

一般，一个限界上下就是一个领域，一个领域下会有：聚合（多个）、每个聚合对应一个仓储接口定义、领域事件（可以没有）、领域服务（可以没有），每个聚合内会有实体（可以没有）、值对象（可以没有）

### 限界上下文

一般来说，一个限界上下文只包含一个领域，限界上下文的划分，主要是以<u>通用语言</u>的维度来划分。

简单的理解为，限界上下文，即Bounded Context，是为我们的业务模块划分的一个或多个边界，边界与边界相互之间不能依赖。

限界上下文是最大的单元（一个整体），在其中，会有聚合、领域服务。

### 聚合

Domain Object 是一个带有唯一标识的实体类，且有自己的行为（方法函数，充血模型）。通常这个聚合命名在.NET中以DO结尾，在go中，我会直接命名：domainObject

聚合的属性字段不允许外部改变（赋值），只能由聚合内的方法来修改。（需要充分理解这个意义，好处非常多，慢慢体会）

聚合与聚合之间，不能相互依赖，且在一个事务中，只能修改一个聚合。（如果要修改多个，可以使用领域事件或消息机制）

聚合会包含仓储接口、实体、值对象、领域事件

聚合主要就是提供给外部使用的对象。且主要的逻辑都放在聚合内部。

聚合可以由：应用层、领域服务、基础设施层调用。

### 实体

Entity Object 实体也是带有唯一标识的。一般我将这个实体的命名以EO结尾。与聚合的一样，也有自己的行为，只允许自己的行为修改状态。

实体是聚合的一个属性，可以是集合或是单个对象

通常在聚合中会有非常多的属性，这时我们可以将其中一些关联在一起的属性改成实体。（但注意，实体必须是包含唯一标识的）

这里要注意的是，实体的属性，不能由聚合来修改，只能是实体本身。聚合可以通过调用实体提供的行为来实现修改

### 值对象

Value Object 没有唯一标识。一般我将这个值对象的命名以VO结尾。

但值对象是不允许修改内部状态。如果要修改，只能整个替换。

同样，我们可以将聚合中一些关联在一起的属性改成值对象。但这些属性没有唯一标识。

### 仓储接口

仓储接口，主要就是针对数据库或缓存的持久化实现的接口定义（而具体实现是在基础设施层）

在应用层调用了聚合，并改变了行为（执行业务逻辑）后，我们始终需要将这个聚合持久化到比如数据库或缓存的，

但是聚合本身，不能直接或间接调用持久化的实现。前面也描述了，领域层是不依赖仓储的。

仓储接口一般是由：应用层、领域服务调用。聚合本身不能调用仓储接口。

常见的比如，从数据库取一条记录，然后执行聚合的行为时，这个获取记录的动作是在应用层直接调仓储接口。

### 领域事件

领域事件主要是为了解决不同聚合之间的耦合。

如果两个聚合需要发生交互。可以由A聚合完成自身的处理时，发起一个事件通知。再由B聚合来订阅此事件，然后完成对应B聚合的行为。

因为聚合还有一个硬性要求：一次事务，只能服务于一个聚合。所以领域事件便可以通过此种方式来达到解耦。

领域服务也有类似的解决方案，但是领域事件是可以跨限界上下文的。

### 领域服务

在同一个限界上下文中，多个聚合需要依赖时，则可以使用领域服务来解决多聚合调用问题。

但领域服务只能用在这两个聚合是在同一个限界上下文。（而领域事件可以跨限界上下文。）

这里要注意，尽可能少用领域服务。如果使用过多，则会导致贫血模型的问题，并且会像之前三层中的逻辑服务层一样泛滥，导致业务逻辑扁平化。

## 基础设施层

基础设施层会依赖：接口层、应用层、领域层。没错，基础设施层才是最外的一层。这与传统的三层架构有非常大的区别。

主要是实现技术功能，如数据库、缓存、ES、MQ，与业务逻辑无关，也包括一些领域事件的订阅、MQ的消费、JOB等。

基础设施层作为最外层的好处是，业务逻辑（领域层）不变的情况下，更换技术功能组件时，只需修改该层即可。

基础设施层会实现聚合中的仓储接口，然后通过IOC容器技术，实现注册。


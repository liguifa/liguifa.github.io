---
title: 「SQLServer」灵活的DBContext生命周期管理
date: 2023-07-27 13:19:23
tags: [SQLServer,C#]
---
我们在开发过程中总是回遇到的许多架构上的问题，在这些架构问题中，最常见的是 DbContext 生命周期的管理以及跨域调用时遇到的 request 上下文限制的问题。  
  
   
首先说说 DBContext的Scope 定义问题。根据微软的建议， DBContext是一个轻量级的，类似于 Unit of work模式的实体上下文，其生命周期对应一次业务操作，比如一次 REST API 调用，一次线程中的批处理业务等等，但总的原则是其生命周期应该尽可能短，另外 DBContext的生命周期也隐含地对应一次数据库事务。  
   
DBContext生命周期定义有许多的方式，如在一个 DAO函数中创建 DBContext，并在离开函数前销毁的模式，或者在HTTP request开头创建，在 HTTP request结尾销毁的模式，这些模式各有利弊，在一个 DAO函数中创建 DBContext这中模式较好地避免了死锁的发生，但是频繁且碎片化的生命周期导致事务完全失效且对性能有一定的影响，HTTP request的方式灵活性较差，一旦脱离 web上下文，就难以找到更合适的粒度，并且完全依赖运行环境上下文的模式为业务的在不同环境下的复用带来了较大的限制。 这个DBContext的模式有一个共同的问题，就是 DBContext生命周期的定义是不可变的，不能根据业务的需要进行覆盖和重新定义 ，这也是导致我们目前开发中遇到各种问题和障碍的主要原因。  
   
DBContext以及其对应的生命周期归根到底是一种资源（类似于 Session），既然是资源，就应该被灵活使用， 在不同的运行环境中拥有不同的行为，而在我们开发 Service和业务逻辑时，并不能完全确定资源的使用方式，我们可以定义 DBContext在当前业务下的生命周期，但是当局部的业务逻辑被放到更大或者完全不同的业务层面中时，作为资源（ Session）是需要被重新定义以适应新的业务上下文的。所以，我们需要一种可以在任何粒度下使用，并可以按需覆盖的 DBContext生命周期管理架构，以便我们在任何业务上下文中都能灵活控制事务和生命周期。  
   
针对这个需求，目前已经有不少成熟的思路和解决方案，比较好的是DbContextScope模式和其对应的开源代码，可以参考其在 GitHub上的工程：[https://github.com/mehdime/DbContextScope](https://github.com/mehdime/DbContextScope)  
   
DbContextScope使用.Net 的CallContext机制（[http://www.cnblogs.com/vwxyzh/archive/2009/02/21/1395416.html](http://www.cnblogs.com/vwxyzh/archive/2009/02/21/1395416.html) ）来实现同一调用上下文中资源的共享，其典型的调用方式如下面的截图所示：  
![](282A72E2-72B7-4CA1-891C-4F403D54C9E9.png)  
使用中在 Service层中每个可能被单独调用的方法中显示声明 DBContextScope，当同一AppDomain 中如果有嵌套调用，最外部的 scope会保证所有内部的scope都使用同一个 DBContext，保证了事务和生命周期能被灵活的管理，大家还可以根据不同的业务来选择最合适的 scope，但是即使大家没有精力考虑生命周期的合理定义，也只需要简单的在所有函数中都加上 DBContextScope，就能保证适应绝大多数的调用场景。使用这种模式后，大家也不再需要依赖 HTTP Request或者将Service 声明为多实例，为开发提供更多的灵活性。  
   
另外针对跨域调用的限制， CallContext也是解决该问题的合理方案， Plugin Framework随后会提供一套统一的框架共大家在跨域调用时将必要的资源上下文通过 CallContext跨域传递给被调用方，包括在 Web Controller中对跨同一AppDomain 的service多次调用的 DBContext的生命周期进行统一管理，以及提供方法让大家将 HTTP Context中的必要信息传递给跨域的 Service层。

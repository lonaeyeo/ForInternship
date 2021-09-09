##### *mybatis 中 #{}和 ${}的区别是什么*

\#{}是预编译处理，使用#{}可以有效的防止SQL注入，提高系统安全性。



##### *mybatis 是否支持延迟加载？延迟加载的原理是什么*

Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。

在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。

它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。



##### *mybatis 的一级缓存和二级缓存*

一级缓存: 基于 PerpetualCache 的 **HashMap** 本地缓存，其**存储作用域为 Session**，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，**默认打开**一级缓存。

二级缓存与一级缓存其机制相同，默认也是采用 **PerpetualCache**，**HashMap** 存储，不同在于其**存储作用域**为 **Mapper(Namespace)**，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置 ；

对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。
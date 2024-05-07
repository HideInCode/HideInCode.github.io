# Mybatis

> 1. 半自动ORM框架，对JDBC进行了封装。
> 2. 使用xml/注解进行配置和映射。
> 3. 灵活编写SQL，不用与代码耦合。
> 4. 兼容各种数据库。
> 5. 提供了映射标签。
> 6. 但是SQL编写量变大，多表时麻烦；SQL语句以来数据库，不好随便换库。
> 7. 不要用RowBounds进行分页，这是个内存分页。

## 缓存

1. 一级缓存：基于 PerpetualCache 的 HashMap 本地缓存，其存储作⽤域为 Session，当 Session flush 或 close 之 后，该 Session 中的所有 Cache 就将清空，默认打开⼀级缓存；
2. 二级缓存：与⼀级缓存其机制相同，默认也是采⽤ PerpetualCache，HashMap 存储，不同在于其存储作⽤域为 Mapper(Namespace)，并且可⾃定义存储源，如 Ehcache。默认不打开⼆级缓存，要开启⼆级缓存，使⽤⼆级缓 存属性类需要实现 Serializable 序列化接⼝(可⽤来保存对象的状态)，可在它的映射⽂件中配。

3. 对于缓存数据更新机制，当某⼀个作⽤域(⼀级缓存 Session / ⼆级缓存 Namespaces)的进⾏了 C/U/D 操作后，默 认该作⽤域下所有 select 中的缓存将被 clear。
### Sql解析执行-缓存的实现


    Mybatis主要有两种缓存：一级缓存和二级缓存。
    一级缓存的生命周期与SqlSession的生命周期一样。一级缓存是在BaseExecutor中实现。
    二级缓存的生命周期跟SqlSessionFactory一样，通常在整个应用中有效。二级缓存是通过CachingExecutor来实现的。



在BaseExecutor中会有一个localCache对象，就是来保存缓存数据的
再来看BaseExecutor中的query方法是怎么实现一级缓存的


```java

public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {  
    BoundSql boundSql = ms.getBoundSql(parameter);  
    //利用sql和执行的参数生成一个key，如果同一sql不同的执行参数的话，将会生成不同的key  
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);  
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);  
 }  
  
  @SuppressWarnings("unchecked")  
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {  
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());  
    if (closed) throw new ExecutorException("Executor was closed.");  
    if (queryStack == 0 && ms.isFlushCacheRequired()) {  
      clearLocalCache();  
    }  
    List<E> list;  
    try {  
      queryStack++;  
      //从缓存中取出数据  
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;  
      if (list != null) {  
        //如果缓存中有数据，处理过程的缓存  
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);  
      } else {  
       //如果缓存中没有数据，将sql执行生成结果，并加入localCache中。  
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);  
      }  
    } finally {  
      queryStack--;  
    }  
    if (queryStack == 0) {  
      for (DeferredLoad deferredLoad : deferredLoads) {  
        deferredLoad.load();  
      }  
      deferredLoads.clear(); // issue #601  
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {  
        //如果配置为STATEMENT时，将清除所有缓存。说明STATEMENT类型的查询只有queryFromDatabase方法中有效。  
        clearLocalCache(); // issue #482  
      }  
    }  
    return list;  
  }  
```

### 以上代码可以看出一级缓存中的基本策略。
- [x] 一级缓存只在同一个SqlSession中共享数据
 在同一个SqlSession对象执行相同的sql并参数也要相同，缓存才有效。
如果在SqlSession中执行update/insert/detete语句的话，SqlSession中的executor对象会将一级缓存清空。



### 二级缓存
二级缓存对所有的SqlSession对象都有效。需要注意如下几点：
二级缓存是跟一个命名空间绑定的。
在一个SqlSession中可以执行多个不同命名空间中的sql,也是就说一个SqlSession需要对多个Cache进行操作。
调用SqlSession.commit()之后，缓存才会被加入到相应的Cache。
下面来看CachingExecutor是怎么实现的。

详细请看：http://blog.csdn.net/ashan_li/article/details/50393489




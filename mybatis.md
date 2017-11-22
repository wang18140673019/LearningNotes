### Configuration的属性

 * configuration的属性主要分为两大部分：<br>
  * 从mybatis-config.xml中读取的配置<br>
  * 从mapper配置文件或Mapper注解读取的配置
  



###Configuration加载过程

* 针对mybatis-config.xml配置文件和Mapper配置文件，Mybatis也是由两个相对应的类来解析的。
  XMLConfigBuilder解析mybatis-config.xml的配置到Configuration中
  XMLMapperBuilder解析Mapper配置文件的配置到Configuration中



### 小结

* ResultMap对象是结果集中的一行记录和一个java对象的对应关系。
* ResultMapping对象是结果集中的列与java对象的属性之间的对应关系。
* ResultMapp由id,type等基本的属性组成外，还包含多个ResultMapping对象。这类似于一个java对象由多个属性组成一个道理。
* ResultMapping最主要的属性column(结果集字段名),property(java对象的属性)，ResultMapping可以指向一个内查询或内映射。
* XMLMapperBuilder调用如下方法来解析并生成ResultMap对象





### MappedStatement说明

* 一个MappedStatement对象对应Mapper配置文件中的一个select/update/insert/delete节点，主要描述的是一条SQL语句



### Mybatis中的缓存类型

* Mybatis支持两种缓存
 * 一级缓存，也叫本地缓存。这个缓存是在sqlSession中的实现的,sqlSession关闭之后这个缓存也将不存在，默认是开启的，当然了也可以在Mybatis-config配置文件中关闭。对于这个缓存策略后面会析到。
 * 二级缓存。这个缓存是在命名空间有效，可以被多个sqlSession共享。<br>
开启这个缓存是在mapper.xml中配置的


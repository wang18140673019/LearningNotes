###  DynamicContext动态上下文
* 重要属性


//参数上下文，ContextMap为一个Map（传入的bean的值） 
    private final ContextMap bindings;<br>
    sql,sqlNode中的apply()方法调用了appendSql(text)方法，最终会将sql保存在这个属性中<br>
    private final StringBuilder sqlBuilder = new StringBuilder();  <br>

```java
<![CDATA[
    		select user_id,user_name,user_type,cust_id
				from tf_f_user a 
				where a.user_id=#{userId} 
		]]>
		<if test="userName!=null"> 
			and 
			user_name=${userName} 
		</if>

```


### 这个DynamicSqlSoure的结构如下(以上面的SQL为例)

![](http://img.blog.csdn.net/20151221105942363)

### 结合例子说明一下sql在sqlNode中是怎么分布的
* StaticTextSqlNode2:保存了"from tf_f_user a"
* TextSqlNode3:保存了"where a.user_id=#{userId}",同时标识为动态的,因为他有占位符
* StaticTextSqlNode4:保存了"and"
* TextSqlNode5:保存了"user_name=#{userName}"
* IfSqlNode:保存了其test属性值，StaticTextSqlNode4和TextSqlNode5是否加入的context中也是由其控制的
接下来看看每一种SqlNode是怎么解析sql并生成parameterMapping的


### StaticTextSqlNode.apply()方法


```java

 public boolean apply(DynamicContext context) {
    context.appendSql(text);
    return true;
  }

```

只是简单的把对应的test追加到context中。
所以StaticTextSqlNode1和StaticTextSqlNode2的apply方法执行后,DynamicContext中的sql内容为:


    select  user_id,user_name,user_type,cust_id from tf_f_user a  
    
    
   
   
   
   ### TextSqlNode.apply()方法
    public boolean apply(DynamicContext context) {  
    //GenericTokenParser为一个占用符解析器  
    //BindingTokenParsery为一个TohenHandler:解析具体的占位符  
    GenericTokenParser parser = createParser(new BindingTokenParser(context));  
    context.appendSql(parser.parse(text));  
    return true;  
  } 
    
    
```java
private GenericTokenParser createParser(TokenHandler handler) {  
   //解析${tab_name}这种占位符，注意不是这种#{propertyName}  
   return new GenericTokenParser("${", "}", handler);  
 }  
 
 
 public String parse(String text) {  
    StringBuilder builder = new StringBuilder();  
    if (text != null && text.length() > 0) {  
      char[] src = text.toCharArray();  
      int offset = 0;  
      int start = text.indexOf(openToken, offset);  
      while (start > -1) {  
        if (start > 0 && src[start - 1] == '\\') {  
          // the variable is escaped. remove the backslash.  
          builder.append(src, offset, start - 1).append(openToken);  
          offset = start + openToken.length();  
        } else {  
          int end = text.indexOf(closeToken, start);  
          if (end == -1) {  
            builder.append(src, offset, src.length - offset);  
            offset = src.length;  
          } else {  
            builder.append(src, offset, start - offset);  
            offset = start + openToken.length();  
            String content = new String(src, offset, end - offset);  
            //关键是这句，调用了handler.handleToken()方法  
            builder.append(handler.handleToken(content));  
            offset = end + closeToken.length();  
          }  
        }  
        start = text.indexOf(openToken, offset);  
      }  
      if (offset < src.length) {  
        builder.append(src, offset, src.length - offset);  
      }  
    }  
    return builder.toString();  
  }  
```

       前面分析到SqlNode.apply()后，Sql还是个半成品。只处理了"${}"这种占位符，"#{}"这种占位符还没有处理，而且Sql执行时的参数也没有生成。
    再来看DynamicSqlSource.getBoundSql()方法
    

```java
   public BoundSql getBoundSql(Object parameterObject) {
    DynamicContext context = new DynamicContext(configuration, parameterObject);
    //这里只处理了"${}"占位符
    rootSqlNode.apply(context);
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();
    //这里就是处理"#{}"占位符的地方
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());
    //这个BoundSql就是数据库可执行的Sql,同时还包含了运行时的参数。
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    for (Map.Entry<String, Object> entry : context.getBindings().entrySet()) {
      boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());
    }
    return boundSql;
  }
```
    

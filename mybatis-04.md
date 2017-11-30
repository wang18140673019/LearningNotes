###插件(plugins)拦截器
mybatis拦截器使用JDK动态代理来实现
>一个拦截器就相当于把要拦截的方法复写一遍,下一个拦截器嵌套着上个拦截器，是由jdk动态代理实现的

### 测试代码

```java
package aurora.test;

public interface AshanService  
{  
    public void service(String name);  
}  
```



```java
package aurora.test;

public class AshanServiceImpl implements AshanService  
{  
  
    @Override  
    public void service(String name)  
    {  
        System.out.println("Hello "+name);  
    }  
  
} 
```


```java
package aurora.test;

import java.util.Properties;

import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.InterceptorChain;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Plugin;
import org.apache.ibatis.plugin.Signature;

//这个注解会被Plugin读取  
@Intercepts  
(  
      {  
          @Signature//定义方法签名  
          (  
                  type=AshanService.class,  
                  method="service",  
                  args={String.class}  
          )  
      }  
)  
public class AshanInterceptor implements Interceptor  
{  
    
  private String name="";  
    
  public AshanInterceptor(String name)  
  {  
      super();  
      this.name = name;  
  }  

  //只有Intercepts在定义的方法才会被拦截  
  @Override  
  public Object intercept(Invocation invocation) throws Throwable  
  {  
      System.out.println("AshanInterceptor["+name+"]...before...");  
      Object obj=invocation.proceed();  
      System.out.println("AshanInterceptor["+name+"]...after...");  
      return obj;  
  }  

  @Override  
  public Object plugin(Object target)  
  {  
      if(target!=null && target instanceof AshanService)  
      {  
          //Plugin是mybatis提供的  
          return Plugin.wrap(target, this);  
      }  
      return target;  
  }  

  @Override  
  public void setProperties(Properties properties)  
  {  
        
  }  
    
  public static void main(String[] args)  
  {  
      AshanService service=new AshanServiceImpl();  
      AshanInterceptor aiNoName=new AshanInterceptor("no_name");  
      //直接用Interceptor.plugin生成一个代理  
      AshanService proxy1=(AshanService)aiNoName.plugin(service);  
      proxy1.service("ashan");  
        
        
      System.out.println("######################");  
        
      InterceptorChain chain=new InterceptorChain();  
      chain.addInterceptor(new AshanInterceptor("1"));  
      chain.addInterceptor(new AshanInterceptor("2"));  
      chain.addInterceptor(new AshanInterceptor("3"));  
      //用InterceptorChain生成一个代理  
      AshanService proxy2=(AshanService)chain.pluginAll(service);  
      proxy2.service("chain");  
        
        
  }  

}  
```
> 输出结果为
AshanInterceptor[no_name]...before...  
Hello ashan  
AshanInterceptor[no_name]...after...  
######################  
AshanInterceptor[3]...before...  
AshanInterceptor[2]...before...  
AshanInterceptor[1]...before...  
Hello chain  
AshanInterceptor[1]...after...  
AshanInterceptor[2]...after...  
AshanInterceptor[3]...after...  

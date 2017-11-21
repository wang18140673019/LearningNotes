# Collections.unmodifiableList的用法

### 返回值：在方法调用返回指定列表的不可修改视图。
```java

import java.util.List;

public class Student
{
        private String userName ;
       
        private List<String> courses ;
       

        public Student (String userName , List<String> courses)
       {
               this.userName = userName;
               this.courses = courses;
       }

        public String getUserName()
       {
               return userName ;
       }

        public void setUserName(String userName)
       {
               this.userName = userName;
       }

        public List<String> getCourses()
       {
               return courses ;
       }

        public void setCourses(List<String> courses)
       {
               this.courses = courses;
       }
       
       

}

```


重构后，按照上面介绍的原则，这样`重构`：


```java

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Student
{
        private String userName ;

        private List<String> courses ;

        public Student (String userName , List<String> courses)
       {
               this.userName = userName;
               this.courses = courses;
       }

        public String getUserName()
       {
               return userName ;
       }

        public void setUserName(String userName)
       {
               this.userName = userName;
       }

        public void addCourse(String course)
       {
               courses.add(course);
       }

        public boolean removeCourse(String course)
       {
               return courses .remove(courses );

       }

        public List<String> getCourses()
       {
               return Collections.unmodifiableList( courses);
       }

        public static void main(String[] args)
       {
              List<String> list = new ArrayList<String>();
              list.add( "数学");
              list.add( "语文");
              Student s = new Student("lily" , list);

              List<String> anotherList = s.getCourses();

               /**
               * throws java.lang.UnsupportedOperationException should replace with
               * s.addCourse(String course)
               */
              anotherList.add( "英语");

               // 不会走到这一步，因为上边抛出了异常
              System. out.println("lily's course.length = " + s.getCourses().size());
       }

}

```

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=#0099ff size=7 face="黑体">color=#0099ff size=72 face="黑体"</font>
<font color=#00ffff size=72>color=#00ffff</font>
<font color=red size=72>color=red</font>



##总结
<font face="微软雅黑">我是微软雅黑</font>

<font color="red" face="微软雅黑"></font>使用这种方法重构的意义：就好比我们网上购物一样，你可以往购物车添加自己想买的东西，但是商户不能在不通知顾客（我们）的情况下，就任意的添加商品，并修改商品的价格等，入口只能是一个，也就是在顾客手中。比喻可能不是很恰当，反正意思大概就是这样。</font>




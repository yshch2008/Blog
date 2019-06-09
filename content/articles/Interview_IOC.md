### 5.Spring也是阿里面试基础题，比如Spring依赖注入的实现，AOP实现，也稍微看一下
##  IOC
### IOC的别名：依赖注入(DI)
我们可以把IOC容器的工作模式看做是工厂模式的升华，可以把IOC容器看作是一个工厂，这个工厂里要生产的对象都在配置文件中给出定义，然后利用编程语言的的反射编程，根据配置文件中给出的类名生成相应的对象。

所谓依赖注入，就是由IOC容器在运行期间，动态地将某种依赖关系注入到对象之中。
对象A依赖于对象B,当对象 A需要用到对象B的时候，IOC容器就会立即创建一个对象B送给对象A。
提高了模块的可复用性。符合接口标准的实现，都可以插接到支持此标准的模块中。
IOC生成对象的方式转为外置方式，也就是把对象生成放在配置文件里进行定义，这样，当我们更换一个实现子类将会变得很简单，只要修改配置文件就可以了，完全具有热插拨的特性。

[简书](https://www.jianshu.com/p/9e3d40651e80)
### 知识点
1. 默认单例
2. 通过BeanFactory getBean获取实例
3. ApplicationContext构造容器
例子：如何获取Bean
``` java
User.java
---
@Getter  
@Setter  
public class User {  
  
    private Long id;  
  
    private String name;  
  
    private String desc;  
  
}

 ApplicationConfig.java 
 ---
 @Configuration//代表这是一个Java配置文件，Spring会据此生成IOC容器去装配Bean
public class ApplicationConfig {  
      
    @Bean(name = "user")//将该方法返回的POJO装配到IOC容器中，name代表这个Bean的名称
    public User initUser() {  
        User user = new User();  
        user.setId(1L);  
        user.setName("name_1");  
        user.setDesc("desc_1");  
        return user;  
    }
    
}

调用
---
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(
                ApplicationConfig.class//指定
        );//构造一个自己的IOC容器
        // 根据类型获取 Bean
        User user = ctx.getBean(User.class);
        log.info(user.getId() + "");
    }
```

```java
public interface Person {//定义接口
    
    // 使用动物服务
    void service();
    
    // 设置动物
    void setAnimal(Animal animal);    
}

public interface Animal {//定义接口

    // 使用
    void use();    
}

@Component
public class BussinessPerson implements Person {
//@Qualifier("dog")也可以通过 这个注解指定注入
    @Autowired //根据属性找到对应的Bean进行注入，此处会找到Dog实例，注入到animal的实例
    private Animal animal = null;//如果是多个，则会找到对应名称的实例注入，没有会报错
    //也可以在实例上打 @Primary 标签，表示默认注入
    @Override
    public void service() {
        this.animal.use();
    }

    @Override
    public void setAnimal(Animal animal) {
        this.animal = animal;
    }

}

@Slf4j
@Component
public class Dog implements Animal {

    @Override
    public void use() {
        // getSimpleName() 获取类简称
        log.info("狗【" + Dog.class.getSimpleName() + "】是看门用的。");
    }

}

    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(
                ApplicationConfig.class
        );
        Person person = ctx.getBean(BussinessPerson.class);
        person.service();
    }
```
打印`[main] INFO com.tpsix.spring.pojo.Dog - 狗【Dog】是看门用的。`

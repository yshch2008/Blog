## `AOP 技术实现就是代理类`
`通过代理模式，把委托对象和要附加的操作编织到一起，实现面向切面的编程。`

1. 静态代理：根据AOP框架提供的命令进行编译，在编译阶段生成AOP代理类，也称为编译时增强
2. 动态代理：在运行时借助于JDK或者CGlib生成临时代理类，然后加载到JVM 。JDK通过接口，CGlib借助继承类
3. JDK：java.lang.reflect.Proxy  +java.lang.reflect.InvocationHandler 通过实现InvocationHandler接口的中间类，把对中间类的方法调用转换成对应的委托类的方法调用，本质是两组静态调用（代理类调用中间类，中间类调用委托类）
4. CGlib :
  1. 查找目标类上的所有非final的public类型的方法(final的不能被重写)
  2. 将这些方法的定义转成字节码
  3. 将组成的字节码转换成相应的代理的Class对象然后通过反射获得代理类的实例对象
  4. 实现 MethodInterceptor 接口,用来处理对代理类上所有方法的请求

##静态和动态对比：
静态代理：代理对象和实际对象都继承了同一个接口，在代理对象中指向的是实际对象的实例，这样对外暴露的是代理对象而真正调用的是 Real Object.

##JDK方式和CGlib对比：
|JDK|CGlib|
|---|---|
|基于反射，实现 InvocationHandler 接口，重写 invoke 方法，利用反射生成代理类 Proxyxx.class 代理类字节码，并生成对象，花费的时间少，能代理final方法|基于ASM框架生成业务类的子类，通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，来完成动态代理的实现。|
|JDK自身支持|不用写接口，仍可以继承父类（JDK必须继承Proxy类）|

https://www.jianshu.com/p/1682ed0d0c16
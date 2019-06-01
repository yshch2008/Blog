---
date: "2019-05-30+08:00"
publishdate: "2019-05-30+08:00"
lastmod: "2019-05-30+08:00"
draft: false
title: "Java类加载器"
tags: ["修行", "Java", "底层原理"]
series: ["技术"]
categories: ["学习"]
toc: true
---


####一般来说，Java 应用的开发人员不需要直接同类加载器进行交互。Java 虚拟机默认的行为就已经足够满足大多数情况的需求了。不过如果遇到了需要与类加载器进行交互的情况，而对类加载器的机制又不是很了解的话，就很容易花大量的时间去调试 ClassNotFoundException和 NoClassDefFoundError等异常。

>顾名思义，类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。
一般来说，Java 虚拟机使用 Java 类的方式如下：

1. Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。
2. 类加载器负责读取 Java 字节代码，并转换成java.lang.Class类的一个实例。
每个这样的实例用来表示一个 Java 类。
通过此实例的 newInstance()方法就可以创建出该类的一个对象。实际的情况可能更加复杂，比如 Java 字节代码可能是通过工具动态生成的，也可能是通过网络下载的。

> ###基本上所有的类加载器都是 java.lang.ClassLoader类的一个实例。
java.lang.ClassLoader类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个 Java 类，即 java.lang.Class类的一个实例。
除此之外，ClassLoader还负责加载 Java 应用所需的资源，如图像文件和配置文件等。不过本文只讨论其加载类的功能。为了完成加载类的这个职责，ClassLoader提供了一系列的方法，比较重要的方法如 表 1所示:

|name:类的二进制名称|||
|- |- |- |
|getParent()|返回该类加载器的父类加载器。|
|loadClass(String name)|加载名称为 name的类，返回的结果是 java.lang.Class类的实例。|
|findClass(String name)|查找名称为 name的类，返回的结果是 java.lang.Class类的实例。|
|findLoadedClass(String name)|查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例。|
|defineClass(String name, byte[] b, int off, int len)|把字节数组 b中的内容转换成 Java 类，返回的结果是 java.lang.Class类的实例。这个方法被声明为 final的。|
|resolveClass(Class<?> c)|链接指定的 Java 类。|

> ###类加载器的树状组织结构
- 引导类加载器（bootstrap class loader）：它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自 java.lang.ClassLoader。
- 扩展类加载器（extensions class loader）：它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。
- 系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。
- 除了系统提供的类加载器以外，开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。

---
  
> ####除了引导类加载器之外，所有的类加载器都有一个父类加载器。
- 每个 Java 类都维护着一个指向定义它的类加载器的引用，通过 getClassLoader()方法就可以获取到此引用。
- 有些 JDK 的实现对于父类加载器是引导类加载器的情况，getParent()方法返回 null
- 如何判定两个 Java 类是相同的? - - Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即便是同样的字节代码，被不同的类加载器加载之后所得到的类，也是不同的。

> ###代理模式是为了保证 Java 核心库的类型安全。所有 Java 应用都至少需要引用java.lang.Object类，也就是说在运行的时候，java.lang.Object这个类需要被加载到 Java 虚拟机中。如果这个加载过程由 Java 应用自己的类加载器来完成的话，很可能就存在多个版本的 java.lang.Object类，而且这些类之间是不兼容的。通过代理模式，对于 Java 核心库的类的加载工作由引导类加载器来统一完成，保证了 Java 应用所使用的都是同一个版本的 Java 核心库的类，是互相兼容的。
> ###不同的类加载器为相同名称的类创建了额外的名称空间。相同名称的类可以并存在 Java 虚拟机中，只需要用不同的类加载器来加载它们即可。不同类加载器加载的类之间是不兼容的，这就相当于在 Java 虚拟机内部创建了一个个相互隔离的 Java 类空间。

---

```java
public class Sample {
    private Sample instance;

    public void setSample(Object instance) { 
            this.instance = (Sample) instance; 
    } 

    public void testClassIdentity() {
        String classDataRootPath = "C:\workspace\Classloader\classData";
        FileSystemClassLoader fscl1 = new FileSystemClassLoader(classDataRootPath);
        FileSystemClassLoader fscl2 = new FileSystemClassLoader(classDataRootPath);
        String className = "com.example.Sample";
        try {
             Class<?> class1 = fscl1.loadClass(className);
            Object obj1 = class1.newInstance();
            Class<?> class2 = fscl2.loadClass(className);
            Object obj2 = class2.newInstance();
            Method setSampleMethod = class1.getMethod("setSample", java.lang.Object.class);
            setSampleMethod.invoke(obj1, obj2);
        } catch (Exception e) {
             e.printStackTrace();
        }
    }
}


```
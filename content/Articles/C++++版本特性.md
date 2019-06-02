---
date: "2019-02-27+08:00"
publishdate: "2019-02-27+08:00"
lastmod: "2019-05-28+08:00"
draft: false
title: "C++++版本特性"
tags: ["修行", "c++++", "基础"]
series: ["技术"]
categories: ["学习"]
toc: true
---

| C# 2.0| | |
|--- |--- |---|
|| 泛型（Generics)| 编译时就可以保证类型安全|
|| | 不用做类型装换，获得一定的性能提升|
|| | |
|| 泛型约束| 约束|
|| 类型参数需是值类型| where T: struct|
|| 类型参数需是引用类型| where T : class|
|| 类型参数要有一个public的无参构造函数| where T : new()|
|| 类型参数要派生自某个基类| where T : <base class name>|
|| 类型参数要实现了某个接口| where T : <interface name>|
|| 这里T和U都是类型参数，T必须是或者派生自U| where T : U|
|| | |
|| Default关键字| default<T>|
|| | 对于值类型返回0|
|| | 对于引用类型返回null|
|| | 对于结构类型返回成员值全部为0的结构实例|
|| | |
|| 可空类型（Nulable Type）| |
|| | system.nullable<T> 简写为T?|
|| | 使用HasValue判断|
|| | 使用空值会抛出异常|
|| | |
|| 匿名方法| |
|| |  System.Threading.Thread t1 = new System.Threading.Thread (delegate() { System.Console.Write("Hello, "); } ); |
|| | ThreadStart ts2 = Method1;|
|| | |
|| 委托的协变和逆变| |
|| | 协变针对返回值|
|| | 逆变针对参数|
|| | 针对父类的委托可以改为指向它的子类，只要参数和返回值都是父子关系|
|| | |
|| 部分类（Partial）| |
|| | 在申明一个雷，结构或者接口的时候，用partial关键字，可以让源代码分布在不同的文件中|
|| | |
|| 静态类（static class）| |
|| | 静态类就是一个只能有静态成员的类，使用static关键字对类进行标识。|
|| | 静态类不能被实例化|
|| | 可以看成一个只有静态成员并且构造函数为私有的普通类，但是处于栈上|
|| | 编译器会保证静态类中不会添加任何非静态的成员和引用|
|| | |
|| global：：| |
|| | 代表了该类的根命名空间，可以用于区分不同命名空间下同名的类|
|| | |
|| extern alias| |
|| | extern alias aliasName; //这行需要在using这些语句的前|
|| |  using System.Text; using aliasName.XXX;|
|| | 用于消除不同程序集中类名重复的冲突，这样可以引用一个程序集的不同版本|
|| | |
|| Accessor访问控制| |
|| | public virtual int TestProperty { protected set { } get { return 0; } }|
|| | 友源程序集：让其他程序访问自己的internal成员，private不行|
|| | [assembly:InternalsVisibleTo("cs_friend_assemblies_2")]|
|| | //作用于整个程序集|
|| | |
|| fixed关键字| |
|| | 创建固定长度的成员为基础类型的数组，方便于处理托非托管代码|
|| | public fixed char XXX [128]|
|| | |
|| volatile关键字| |
|| | 表示相关的字可以被多个线程同时访问，编译器不会对相应的值做针对单线程下的优化|
|| | 保证相关的值在任何时候访问都是即时的|
|| | |
|| #pragma warning| |
|| | 用来取消或者添加编译时的警告信息|
|| | #pragma warning disable 414, 3021|
|| | CS414 和XS3021的警告信息都不会再显示了|
|| | |
|C# 3.0| | |
|| 类型推断| |
|| | var申明|
|| | |
|| 扩展方法| |
|| | 必须是定义在静态类中的非泛型、非嵌套的静态类|
|| | "public static class JeffClass{|
|    public static int StrToInt32(this string s){|
|        return Int32.Parse(s);|
|    }|
||
|    public static T[] SomeMethd<T>(this T[] source, int pram1, int pram2)    {|
|        /**/|
|    }|
|}"|
|| | |
|| | 给string添加了一个扩展方法，给所有类型的数组添加了一个需要两个整形参数的方法|
|| | 扩展方法只在当前的命名空间类有效，如果被其他命名空间import引入，则也可以在其他命名空间生效|
|| | 扩展方法的优先级低于常规方法|
|| | |
|| lambda表达式| |
|| | 类型推断，表达式树，匿名方法|
|| | |
|| 匿名类型| |
|| | new｛name="",age=15｝|
|| | |
|| 自动属性| |
|| | 类型中的字段，只需要写public int age {get;set;}|
|| | 不需要再写private指定字段用于存放|
|| | |
|| 查询表达式| |
|| | 扩展方法的运用|
|| | list.Groupby(d=>d.classNo).Select(d=>new {ClassNo=d.Key,Count=d.count()})|
|| | |
|| 表达式树| |
|| | 匿名方法Func<int,int> AddOne=x=>x+1;|
|| | 表达式树Expression<Func<int,int>> AddOne=x=>x+1;|
|| | |
|C# 4.0| | |
|| 协变和逆变的泛型支持| |
|| | 仅针对引用类型|
|| | |
|| 动态绑定| "class BaseClass{|
|    public void print()    {|
|        Console.WriteLine();|
|    }|
|}Object o = new BaseClass();|
|dynamic a = o;|
|a.print();"|
|| | |
|| | a可以推断出自己的类型，可以调用a.print()，但是编译器不会检查方法是否合法，写错则会报错|
|| | |
|| 可选参数| |
|| | 方法的参数制定默认值，即成为可选的参数|
|| | 避免很多重载的情况|
|| | 调用时可以指定参数的名字传值，而不用按照固定顺序，增强扩展性|
|| | |
|C# 5.0| | |
|| 异步编程| |
|| | 加入async和await关键字，支撑基于任务的异步模型（TAP)|
|| | 通过类似同步的方式编写异步代码，简化了异步编程|
|| | |
|| 调用方信息| |
|| |  [CallerMemberName] string memberName = "",|
|| | [CallerFilePath] string sourceFilePath = "",|
|| |  [CallerLineNumber] int sourceLineNumber = 0)|
|| | |
|C# 6.0| | https://www.cnblogs.com/VVStudy/p/6426923.html|
|| 自动属性增强| |
|| | 自动属性初始化 public int Age{get;set;}=0|
|| | 只读属性初始化--和上一条一样，（不写set即为只读）|
|| | |
|| 表达式树方法| |
|| | public Point NewPoint(int dx,int dy)=> new Point(dx,dy);|
|| | 非静态扩展方法 public override string ToString()=>string.Format("{0}--{1}","x","y");|
|| | |
|| 引用静态类| |
|| | using System.Math|
|| | //后面直接写Math下的方法，而不用在写Math.XXX|
|| | |
|| 空值判断| |
|| | List<Customer> list=null;|
|| | Customer firstCustomer=customer?[0];|
|| | //firstCustomer 会被赋值为0，而不再报引用null值错误|
|| | |
|| 字符串嵌入值| |
|| | 以$修饰字符串，大括号引用变量，等同于之前的string.Format|
|| | CallerMemberName = $"{str}";|
|| | 因为 : 总被编译器解释为表达式与字符串格式的分隔符，所以表达式中若有条件运算符则我们需要用括号来强制编译将其解析成当前语境所要，如Console.WriteLine($"平均成绩：{(student.GPA > 80 ? student.GPA : 0):F2}");|
|| | 字符串插值语法可以嵌套，如：var score = $"我的各科成绩：{ $"数学：{student.MathScores}；英语：{student.EnglishScore};"}";|
|| | |
|| nameOf|  表达的意义|
|| | 在需要用变量名的场合不再需要使用""固定，防止变更变量名的时候没有提示|
|| | 少写一个类似ToString的方法|
|| | |
|| 带索引的对象初始化器| |
|| | var EnumFunctions=new Dictionary<int,string>{[0]="Custody",[5]="FundOps"};|
|| | |
|| 异常过滤器| |
|| | try|
|| |   {|
|| |       throw new WebException("Request timed out..", WebExceptionStatus.Timeout);|
|| |   }|
|| |   catch (WebException webEx) when (webEx.Status == WebExceptionStatus.Timeout)|
|| |   {//在使用恰当的情况下可以不丢失异常触发点的对战信息，这对程序的排错至关重要。|
|| |       // Exception handling|
|| |   }|
|| | |
|| await可以用在catch和finally| |
|| | C# 5.0 是不行的|
|| | |
|| 无参数的结构体构造函数| |
|| | public Person():this("NameTest",37){}|
|| | public Person(string name,int age){Name=name;Age=age;}|
|| | |
|C# 7.0| | http://www.cnblogs.com/VVStudy/p/6551300.html|
|| out 支持声明变量| |
|| | 在使用时声明一个新变量|
|| | if (int.TryParse("15", out var result))|
|| | |
|| 元祖（Tuples)| |
|| | ValueTuple,.NET 4.72 CORE 2.0之前的版本需要引入|
|| | Install-Package “System.ValueTuple”|
|| | 支持成员命名：|
|| | var tup=(one:1,two:2)|
|| | 或者(int one,int two) tup =(1,2);|
|| | 左右同时有指定，会提示忽略右边|
|| | ValueTuple是值类型struct，更轻量|
|| | 支持解构：|
|| | var  (one,two)=GetTuple();|
|| | static (int,int) GetTuple(){return (1,2);}|
|| | 解构也支持自定义的类型，但需要增加Deconstruct扩展或者实例|
|| | |
|| 模式匹配(pattern matching)| |
|| | is表达式  （item is int val)  会先把item as 成int类型的结果val 然后判断val是否为null返回bool|
|| | switch 中可内嵌模式匹配，并叠加判断语句|
|| | case string val when val.Contains('s')||int.TryParse(val,out var tempVal):|
|| | |
|| 局部引用和引用返回| |
|| | static ref int GetLocalRef(int[,] arr, Func<int, bool> func)|
|| | int[,] arr = { { 10, 15 }, { 20, 25 } };|
|| | ref var num = ref GetLocalRef(arr, c => c == 20);|
|| | 处理大对象，减少GC，提高性能|
|| | |
|| 局部函数| |
|| | 在方法中增加方法，可以实现无限嵌套|
|| | 编译后是intenal修饰下的静态函数|
|| | 由编译器生成对应的包含上文中所有要用的成员的新类型，实现局部函数对上文中变量和函数的调用|
|| | |
|| 更多的表达式支持| |
|| | 构造函数，析构函数，索引器，以及get，set|
|| | public string Name{get=>_name;set=>_name=value??"DefaltName";}|
|| | |
|| Throw表达式| |
|| | 在任何地方抛出异常！包括lanmbda表达式和判断语句等等|
|| | Null合并运算符string name=GetValue()??throw new ArgumentNullException(nameOf(GetName));|
|| | 三目运算符string name=GetValue()==""?throw new ArgumentException("参数不合法"):GetValue();|
|| | |
|| 扩展异步返回类型| |
|| | 新增ValueTask<T>|
|| | 以前只有Task,Task<T>,void|
|| | |
|| 数字文本分隔符| |
|| | 0b0000_0000|
|| | 0.233_233|
|| | 增强数字可读性|
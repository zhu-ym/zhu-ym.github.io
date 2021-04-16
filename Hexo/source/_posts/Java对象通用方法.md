---
title: Java对象通用方法
typora-root-url: ../
date: 2021-04-16 15:07:59
tags:
 - Java语法
categories:
 - [Java语法,基础]
---

清除Java通用方法的作用、重写及编程规范

<!--more-->

Java所有的对象都有一个超类Object，其有以下通用方法

![image-20210416152247341](/images/image-20210416152247341.png)

后面的方法几个方法是final的，也是本地的，不能被重写

前面所有的非final方法`（equals,hashCode,toString,clone）`都有一些通用约定，当覆盖这些方法时，需要遵循这些约定。

# 一、equals方法

```java
 public boolean equals(Object obj) {
        return (this == obj);
    }
```

显然超类中的equals方法是判断**引用是否相当**，如果我们需要编写自己的相等逻辑，比如是值相等，我们就需要重写`equals`方法

重写equals方法时需要注意下面几个规范

- 自反性：`x.equals(x) == true`
- 对称性：`x.equals(y) == y.equals(x)`

- 传递性：`x.equals(y) == true; y.equals(z) == true; 则x.equals(z) == true`

- 一致性：x，y指向对象未修改，则多次比较结果不变
- `x.equals(null)`必须返回false

一般编码过程中需要注意**对称性和传递性**，这两个规定容易被违反，一般主要出现在子类和父类之间

在这之前，首先得清楚两个方法

- **getClass：**getClass得到该类的实际类型，即**运行时类型**

- **instanceof：**判断该类是否属于**该类或者该类派生类**

我们在重写equals方法中，首先就得判断两个对象是否可以比较，这时可以选择上诉两种方法来判断两个类是否属于同一类型，针对两种方法也有两种重写equals的手段

**1.利用getClass**

显然，使用了`b.getClass() == xxx`,既是严格判断是否一致，可以保证对称性和传递性

这里有一个问题，有人认为这样违反了**里氏替换原则（子类对象必须能够替换掉所有父类对象）**，比如HashSet和TreeSet，你可能希望比较两个任意的集合

**2.利用instanceof**

如果利用instanceof就可以规避这个问题，这时由于instanceof关键字的作用决定的，子类、父类、甚至不同子类之间可以比较，体现了LSP原则

这里又有另外一个问题，如果子类增加了**值组件**，子类想有自己的比较方法，又会带来**传递性或者一致性**的问题

比如子类重写equals没有考虑父类违反传递性、考虑了父类减少值组件比较又会违反传递性



**决策**

由上诉两个，即**在扩展可实例化的类的同时，不能即增加新的值组件，又同时保留equals约定，除非放弃抽象优势**

- 如果子类有自己的相对性概念，强制用getClass比较
- 如果超类决定相对性概念，可以使用instanceof检测，这样可以在不同子类的对象之间进行相等性比较，这时最好将比较方法定为final



最后有一个重写equals的高效的方法

```java
public boolean equals(Object obj) {
        // 性能优化，比较昂贵时很有用
		if (this == obj)
			return true;
        // 检查正确参数类型，这里也可以用getClass
		if(!(obj instanceof 正确参数类型))
			return false;
		// 类型转换
        正确类型 other = (正确类型) obj;
		// 比较操作
        ...
        // 返回比较结果，一般最可能不一致的域先比较
}
```





# 二、重写equals需覆盖hashCode方法

如标题，当你重写了equals方法就需要覆hashCode方法，主要因为在Set或者Map判断元素是否存在首先是利用`hashCode`方法

hashCode方法方法也有一些通用约定，这是其在基于散列的集合中正常运行的基础：

- 一致性：一个对象equals用到的比较信息没有修改，则多次调用hashCode方法返回值一致
- **如果两个对象根据equals方法比较相等，调用hashCode方法必须产生相同的整数结果**
- 如果两个对象根据equals方法比较不相等，对hashCode结果不要求，但是，**不同对象不同结果有助于提高散列表性能**

如何重写一个合适的hashCode函数

对于不同关键域的散列码int c：

- 基本类型：Type.hashCode(f)，可以减少对象的创建
- 对象引用：为这个域递归调用hashCode方法得到c，null返回0
- 数组：每个元素就是一个域对只对重要元素处理，或者是Array.hashCode方法对所有元素处理

确定好每个域c之后，可以得到hashCode标准流程

```
result = c1;
迭代，对每个域有result = 31*result + c;
返回result
```

其中关键域应该仅仅是equals中的比较信息

# 三、toString方法

始终要覆盖toString方法，因为好的toString方法可以便于使用和调试。

好的toString方法一般应该返回对象中包含的所有值得关注的信息

需要决定返回值是否要指定格式，无论指定与否需要在文档中明确的表明自己的意图

Java的开发手册也有一条提到**POJO 类必须写 toString 方法**。在方法执行抛出异常时，可以直接调用 POJO 的 toString()方法打印其属性值，便于排查问题。

# 四、clone方法

一般要覆盖超类中的clone方法需要实现Cloneable接口，不然就会出现CloneNotSupportException。但是Cloneable中并没有clone方法，只是一个标记，标记这个对象可以克隆。

实现Cloneable接口后，重写clone方法时，就可以利用super.clone()实现对对象的浅拷贝，而这个**浅拷贝实际是Object.clone() 实现的**，Object.clone()有特殊的语义，它就是能把当前对象的整个结构完全浅拷贝一份出来：可以把JVM原生实现的 Object.clone() 的语义想像为拿到 this 引用之后通过反射去找到该对象实例的所有字段，然后逐一字段拷贝，这个过程没有调用构造器

根据上诉原理：**一般需要谨慎的覆盖clone**，主要原因有几点：

- 浅拷贝和深拷贝的的问题：当对象内部域含可变域时，clone会比较复杂
- Cloneable接口没有clone方法，只是一个标记，也无法执行多态clone操作
- cloneable框架和可变对象的final域不兼容

一般只是
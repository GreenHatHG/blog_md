---
title: Java虚拟机笔记-接口初始化规则
date: 2019-04-26 07:45:34
categories: 读书笔记
tags:
- jvm
---

接口初始化规则

<!-- more -->

# 样例1

```java
public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = 5;
}

interface  Mychild5 extends MyParent5{
    int b = 6;
}
```

上面样例输出6，当我们删除`MyParent5.class`后依旧输出6

**所以当一个接口在初始化时，并不要求其父接口都完成了初始化**

----

当我们再把`MyChild5.class`删掉后，发现还是输出6。

接着改变代码，同时将`MyChild5.class`与`MyParent5.class`删掉

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = 5;
}

interface  Mychild5 extends MyParent5{
    int b = new Random().nextInt(2);
}
```

运行后程序抛出`NoClassDefFound`异常

明显b那个值不是一个确定的值，所以是在运行期才会产生出来，而没有对应的class文件，所以这个常量是不能放到常量池中，如果我们`rebuild`一次再重新生成对应的class后且没有删除，发现还是可以输出的

----

（子类和父类的class文件都有）接着改变一下

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = new Random().nextInt(3);
}

interface  Mychild5 extends MyParent5{
    int b = 5;
}
```

此时输出5，改变一下

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = new Random().nextInt(3);
}

interface  Mychild5 extends MyParent5{
    int b = new Random().nextInt(6);
}
```

此时输出3

然后我们把`MyParent5.class`删除，运行后程序抛出`NoClassDefFound`异常

接着把a的值改成5并且删除`MyParent5.class`

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = 5;
}

interface  Mychild5 extends MyParent5{
    int b = new Random().nextInt(6);
}
```

运行后程序抛出`NoClassDefFound`异常

所以

**只有在真正使用到父接口的时候（如引用接口中所定义的常量时）才会初始化**

# 样例2

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

interface MyParent5{
    int a = new Random().nextInt(3);
}

interface  Mychild5 extends MyParent5{
    int b = 5;
}
```

把`MyParent5.class`删除后，发现程序依旧可以输出

然后我们改为

```java
import java.util.Random;

public class Main{
    public static void main(String[] args){
        System.out.println(Mychild5.b);
    }
}

class MyParent5{
    int a = new Random().nextInt(3);
}

class  Mychild5 extends MyParent5{
    int b = 5;
}
```

把`MyParent5.class`删除后，运行后程序抛出`NoClassDefFound`异常

对于一个接口来说，它里面一个属性就是`public static final`的，而对于一个类来说，如果不加final的话，它就不是常量

所以它就不会被纳入常量池中，所以在初始化时会导致父类的初始化。

（不会初始化不意味着不会被加载）
## 1.简介

- 是一种类java语言，是一门面向对象和函数式编程的特性结合在一起的静态编程语言
- 是多范式的编程语言，支持面向对象和函数式
- 编译生成字节码，可以运行在jvm上，可以调用java,实现语言的无缝对接
- 跟Java相似

## 2.环境配置

- 配置java环境

- 配置scala环境

- IDEA配置

  1.插件市场选择scala,然后重启IDEA

  ![image-20200411154933081](scala%E5%85%A5%E9%97%A8/image-20200411154933081.png)

  2.选择下图所示，应用scala

![image-20200411155250030](scala%E5%85%A5%E9%97%A8/image-20200411155250030.png)

## 3.入门Demo

```scala
object Date01 {
      //在编译之后会自动生成 public static void main
  //scala是完成面向对象的语言，只能通过模拟生成静态方法
  def main(args: Array[String]): Unit = {
    print("hello world")
  }
}
```

- 编程当前类会生成一个一个特殊的类=>Date01$,然后用创建对象来调用这个对象的main方法

- 一般情况下，将加$的类对象称之为“伴生对象”

- 伴生对象都可以通过类名来访问

- 伴生对象的语法规则，使用object声明

  ![image-20200411161413287](scala%E5%85%A5%E9%97%A8/image-20200411161413287.png)

## 4.scala语法

- scala中没有public关键字，默认所有的访问权限都是公开的
- scala没有void关键字，采用对象模拟：unit
- scala采用def关键字声明方法
- 方法后面的小括号是方法的参数列表（参数名：类型）
- 方法的声明和方法体通过“=”连接
- scala返回方法类型放置在方法声明的后面，使用冒号连接
- 语法严格区分大小写
- 每个语句不需要 “；”

### 4.1scala执行流程分析

![image-20200411162457661](scala%E5%85%A5%E9%97%A8/image-20200411162457661.png)

### 4.2字符串打印测试

```java
object Date01 {
  //在编译之后会自动生成 public static void main
  //scala是完成面向对象的语言，只能通过模拟生成静态方法
  def main(args: Array[String]): Unit = {
    println("hello world")
    //打印字符串
    print("xiao wu")
    var name = "xiao"
    var name2 = "wu"
    println("name = " + name + "name2" + name2)
    //插值输入
    println(s"name=${name}")
    print(raw"name=${name}")
  }

}
```


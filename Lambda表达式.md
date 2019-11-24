# Java lambda
众所周知Java是一门面对对象的变成语言，所以在Java中执行的lambda表达式实际上是被jvm在内部转换成了一个个函数式接口（FunctionalInterface）的实现来进行处理。

[toc]

## 函数式接口
在Java1.8之前，很多的api中就已经出现了基于回调的编程方式。比如下面的代码
```java
public interface ActionListener {
  void actionPerformed(ActionEvent e);
}

//在大多数的情况下，并不会创建一个ActionListener的实例，而是使用匿名类把操作代码内联
button.addActionListener(new ActionListener() {
  public void actionPerformed(ActionEvent e) {
    ui.dazzle(e.getModifiers());
  }
});
```
匿名类的方式有几个问题
- 语法过于冗余，`高度问题`
- 匿名类中的 this 和变量名容易使人产生误解
- 类型载入和实例创建语义不够灵活
- 无法捕获非 final 的局部变量
- 无法对控制流进行抽象

冗余问题很好理解，上面的5行代码中，实际工作的只有`ui.dazzle(e.getModifiers());`这一行。造成了大量的无用代码。

在Java8引入lambda后，把上面那种只有一个方法的接口，称之为`函数式接口`。针对函数式接口的调用可以写成lambda的形式，比如上面的回调形式可以改为。
`button.addActionListener((ActionEvent e) -> ui.dazzle(e.getModifiers()))`。

## 函数式接口的定义和FunctionalInterface注解
Java在1.8版本加入了`@FunctionalInterface`注解,用来标识一个接口是不是函数式接口。
但我们要明确一个事情，并不是只有添加了`@FunctionalInterface`的接口才是函数式接口。只要满足函数式接口的定义，JVM就会认为这是一个函数式接口，就可以写成lambda表达式的形式。

### 函数式接口的定义
函数接口的定义就是，这个接口中只能有一个符合条件的abstract方法。（排除接口的默认方法、从Object重写的`public`方法(这里注意只能排除Object的public方法)）。下面通过举例来说明。
```Java
//这是最标准的函数式接口
interface Runnable {
    void run();
}
```

```Java
//equals方法是从Object中重写的，需要排除，排除之后这个接口不含有任何abstract的方法，
//所以它不是函数式接口
interface NonFunc {
    boolean equals(Object obj);
}

```

```Java
//这个接口继承了NonFunc，所以可以认为它含有equals和compare两个方法，
//排除掉equals，还有一个compare，所以它符合函数式接口的定义
interface Func extends NonFunc {
    int compare(String o1, String o2);
}
```

```java

//这个接口比较特殊,它重写了Object的clone方法，但是这个方法并不是public的方法，
//所以不能排除，所以这个接口中含有两个abstract的方法，所以这个接口不符合函数式接口的定义
interface Foo {
    int m();
    Object clone();
}

```
参考:[Functional Interface](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.8)

### @FunctionalInterface注解
如果添加了这个注释，编译器就会验证该接口是否满足函数式接口的要求，如果这个接口不满足要求，就会给出错误提示。
所以这个注解起到的作用是强制性校验，和进行标识。


## Java为什么使用函数式接口来实现lambda
实现函数化编程的另一种方案是引入一种新的函数类型，`箭头`类型。但是这种改动会对现有的Java系统引入二外的复杂度，并造成一些代码风格的分歧。所以决定依赖函数式接口来实现lambda。这样既能和现有的Java系统保持统一，而且也可以让现有的一些满足函数式接口的接口，直接使用lambda。


## 常见的函数式接口
Runnable(虽然没有biao)
Supplier
Consumer


## 双冒号操作符
双冒号`::`是在lambda中很常用的一个操作符。
我认为这个操作符和lambda表达式的作用都是，帮助我们快速的将一个操作转化成为Java的FunctionalInterface

举一个我们常见的例子
```java
execute( 
    () -> System.out.println("Worker invoked using Lambda expression") 
);
```
我们能这么写的原因是，上面的lambda表达式其实是返回了一个Runable对象。
而Runnable这个接口就是我们接触最多的 FunctionalInterface。

## 



## FunctionalInterface在流式编程中的应用
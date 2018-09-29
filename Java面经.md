# 1. Java面经

jess

<!-- TOC -->

- [1. Java面经](#1-java面经)
    - [1.1. jdk1.7 和 jdk1.8区别](#11-jdk17-和-jdk18区别)
        - [1.1.1. Lambda表达式](#111-lambda表达式)
        - [1.1.2. 接口的默认方法和静态方法](#112-接口的默认方法和静态方法)
        - [1.1.3. 方法引用](#113-方法引用)
        - [1.1.4. 重复注解](#114-重复注解)
        - [1.1.5. 更好的类型推断](#115-更好的类型推断)
        - [1.1.6. 拓宽注解的应用场景](#116-拓宽注解的应用场景)
        - [1.1.7. Data API](#117-data-api)
        - [1.1.8. Stream API](#118-stream-api)
    - [1.2. springboot](#12-springboot)
        - [1.2.1. 快速开始](#121-快速开始)
    - [1.3. 回溯算法](#13-回溯算法)
    - [1.4. 堆栈](#14-堆栈)
    - [1.5. 排序算法](#15-排序算法)
    - [1.6. 查找算法](#16-查找算法)
    - [1.7. tcp/ip](#17-tcpip)
    - [1.8. 多线程(生产者消费者，购票)](#18-多线程生产者消费者购票)
    - [1.9. java设计模式](#19-java设计模式)
        - [1.9.1. 单例模式](#191-单例模式)
            - [1.9.1.1. 分类](#1911-分类)
            - [1.9.1.2. 懒汉式单例](#1912-懒汉式单例)
            - [1.9.1.3. 双重检查锁定](#1913-双重检查锁定)
            - [1.9.1.4. 静态类级内部类](#1914-静态类级内部类)
            - [1.9.1.5. 饿汉式单例](#1915-饿汉式单例)
            - [1.9.1.6. 单例和枚举](#1916-单例和枚举)
            - [1.9.1.7. 各种实现方法优缺点](#1917-各种实现方法优缺点)
            - [1.9.1.8. 线程安全](#1918-线程安全)
        - [1.9.2. 建造者模式](#192-建造者模式)
        - [1.9.3. 工厂模式](#193-工厂模式)
        - [1.9.4. 适配器模式](#194-适配器模式)
    - [1.10. mysql和mongodb区别](#110-mysql和mongodb区别)
    - [1.11. mysql常用命令、顶层结构：B+树](#111-mysql常用命令顶层结构b树)
    - [1.12. linux 常用shell命令、统计文件行数、查找文件、查找文件内容、linux脚本编程](#112-linux-常用shell命令统计文件行数查找文件查找文件内容linux脚本编程)
    - [1.13. java hashmap和hashtable区别](#113-java-hashmap和hashtable区别)

<!-- /TOC -->

---

## 1.1. jdk1.7 和 jdk1.8区别

### 1.1.1. Lambda表达式

允许把函数作为参数传递给某个方法，或者把代码当作数据来处理。  
最简单的Lambda表达式可由逗号分隔的参数列表、->符号和语句块组成，例如：
```java
Arrays.asList("a","b","d").forEach(e->System.out.println(e));
```
在上面这个代码中的参数e的类型是由编译器推理得出的，你也可以显式指定该参数的类型，例如：
```java
Arrays.asList("a","b","d").forEach((String e)->System.out.println(e));
```
如果Lambda表达式需要更复杂的语句块，则可以使用花括号将该语句块括起来，类似于Java中的函数体，例如：
```java
Arrays.asList("a","b","d").forEach(e->{
    System.out.print(e);
    System.out.print(e);
});
```
Lambda表达式可以引用类成员和局部变量（会将这些变量隐式得转换成final的），例如下列两个代码块的效果完全相同：
```java
String separator=",";
Arrays.asList("a","b","d").forEach( 
    (String e)->System.out.print(e+separator));

final String separator = ",";
Arrays.asList("a","b","d").forEach( 
    (String e)->System.out.print(e+separator));
```
Lambda表达式有返回值，返回值的类型也由编译器推理得出。如果Lambda表达式中的语句块只有一行，则可以不用使用return语句，下列两个代码片段效果相同：
```java
Arrays.asList("a","b","d").sort((e1,e2)->e1.compareTo(e2));

Arrays.asList("a","b","d").sort((e1,e2)->{
    int result=e1.compareTo(e2);
    return result;
});
```
解决方法是使用函数接口。函数接口是只包含一个抽象方法声明的接口。

@FunctionInterface用于显式说明该接口是函数接口
```java
@FunctionalInterface
public interface Functional {
    void method();
}
```
默认方法和静态方法不会破坏函数式接口的定义，因此如下的代码是合法的。
```java
@FunctionalInterface
public interface FunctionalDefaultMethods {
    void method();

    default void defaultMethod() {
    }
}
```
### 1.1.2. 接口的默认方法和静态方法

java8现在可以定义接口的默认方法和静态方法
```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or
    // may not implement (override) them.
    default String notRequired() {
        return "Default implementation";
    }
}

public static DefaultImpl implements Defaultable{
}

public static OverrideImple implements Defaultable{
    @Override
    default String a(){
        return "override Defaultable default method"
    }
}

private interface DefaulableFactory {
    // Interfaces now allow static methods
    static Defaulable create( Supplier< Defaulable > supplier ) {
        return supplier.get();
    }
}

public static void main( String[] args ) {
    Defaulable defaulable = DefaulableFactory.creat(
        DefaultableImpl::new);
    System.out.println( defaulable.notRequired() );
    defaulable = DefaulableFactory.create( OverridableImpl::new );
    System.out.println( defaulable.notRequired() );
}
```
### 1.1.3. 方法引用

方法引用使得开发者可以直接引用现存的方法、Java类的构造方法或者实例对象。方法引用和Lambda表达式配合使用，使得java类的构造方法看起来紧凑而简洁，没有很多复杂的模板代码。

例子中，Car类是不同方法引用的例子，可以帮助读者区分四种类型的方法引用。
```java
public static class Car {
    public static Car create(final Supplier<Car> supplier ) {
        return supplier.get();
    }

    public static void collide(final Car car ) {
        System.out.println("Collided "+car.toString());
    }

    public void follow(final Car another) {
        System.out.println("Following the "+another.toString());
    }

    public void repair() {
        System.out.println("Repaired "+this.toString());
    }
}
```
第一种方法引用的类型是构造器引用，语法是Class::new，或者更一般的形式：`Class<T>::new`。注意：这个构造器没有参数。  
```java
final Car car = Car.create(Car::new);
final List<Car> cars=Arrays.asList(car);
```
第二种方法引用的类型是静态方法引用，语法是`Class::static_method`。注意：这个方法接受一个Car类型的参数。
```java
cars.forEach(Car::collide);
```
第三种方法引用的类型是某个类的成员方法的引用，语法是`Class::method`，注意，这个方法没有定义入参：
```java
cars.forEach(Car::repair);
```
第四种方法引用的类型是某个实例对象的成员方法的引用，语法是`instance::method`。注意：这个方法接受一个Car类型的参数：
```java
final Car police=Car.create(Car::new);
cars.forEach(police::follow);
```

### 1.1.4. 重复注解

### 1.1.5. 更好的类型推断

### 1.1.6. 拓宽注解的应用场景

### 1.1.7. Data API

### 1.1.8. Stream API 

Java 8拓宽了注解的应用场景。现在，注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上。

---

## 1.2. springboot
  
### 1.2.1. 快速开始

---

## 1.3. 回溯算法

---

## 1.4. 堆栈

---

## 1.5. 排序算法

---

## 1.6. 查找算法

---

## 1.7. tcp/ip

---

## 1.8. 多线程(生产者消费者，购票)

---

## 1.9. java设计模式

### 1.9.1. 单例模式

#### 1.9.1.1. 分类

- 懒汉式单例  
- 双重检查锁定  
- 静态类级内部类   
- 饿汉式单例  
- 单例和枚举  

#### 1.9.1.2. 懒汉式单例
  
```java
public class Singleton{

    private Singleton(){}
    private static Singleton instance=null;
    public static synchronized Singleton getInstance(){
        if(instance==null){
            instance=new Singleton();
        }
        return instance;
    }

}
```

#### 1.9.1.3. 双重检查锁定
  
```java
public class Singleton{

    private Singleton(){}
    private volatile static Singleton instance=null;
    public static Singleton getInstance(){
        if(instance==null){
            synchronized(this){
                if(instance==null){
                    instance=new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 1.9.1.4. 静态类级内部类
  
```java
public class Singleton{

    private Singleton(){}
    private static class SingletonHolder{

        private static Singleton instance=new Singleton();
    }
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }
}
```

#### 1.9.1.5. 饿汉式单例
```java
public class Singleton{

    private Singleton(){}
    private static Singleton instance=new Singleton();
    public static Singleton getInstance(){
        return instance;
    }
}
```
#### 1.9.1.6. 单例和枚举
  
```java
public enum Singleton{

    INSTANCE1

    public void singletonOperation(){
        
    }
}
```

#### 1.9.1.7. 各种实现方法优缺点
  
- 懒汉式单例：
    
    缺点：每次获取实例都要同步，效率低

- 双重检查锁定：

    优点：只有在创建实例的时候才会同步  
    缺点：volatile会使jvm屏蔽一些优化，效率不高，不如静态类级内部类方法

- 静态内部类：

    优点：懒汉式单例中效率最高的实现方法，由于静态内部类在jvm加载类时会自动初始化，因此延迟初始化实例没有任何时间成本增加

- 饿汉式单例：

    优点：线程安全  
    缺点：jvm加载后，占有一定空间

- 单例和枚举

    优点：线程安全，简洁，无偿提供了序列化机制，防止多次实例化  

#### 1.9.1.8. 线程安全

如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。 

### 1.9.2. 建造者模式

### 1.9.3. 工厂模式

### 1.9.4. 适配器模式


---

## 1.10. mysql和mongodb区别

---

## 1.11. mysql常用命令、顶层结构：B+树

---

## 1.12. linux 常用shell命令、统计文件行数、查找文件、查找文件内容、linux脚本编程

---

## 1.13. java hashmap和hashtable区别

---
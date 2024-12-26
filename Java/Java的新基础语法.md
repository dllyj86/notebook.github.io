# @interface
这是Java提供的一种语法, 用来定义**注解类型**

# :: 语法
是个语法糖
对 () -> xxx 这种lambda的简写

**静态方法引用**：`ClassName::staticMethod`
**实例方法引用**：`instance::instanceMethod`
**构造方法引用**：`ClassName::new`
**特定类型的任意对象的实例方法引用**：`ClassName::instanceMethod`

比如:
```java
//静态方法
Runnable task = MyClass::myStaticMethod;
// 等价于
Runnable task = () -> MyClass.myStaticMethod();
// 实例方法
Runnable task = example::doWork;
Runnable task = () -> example.doWork();
// 构造方法引用
Supplier<MyClass> supplier = MyClass::new;
Supplier<MyClass> supplier = () -> new MyClass();
// 特定类型的任意对象的实例方法引用
List<String> list = Arrays.asList("a", "b", "c");
list.forEach(System.out::println); // 调用每个字符串对象的 println 方法
```

# Lambda表达式
## 基本语法:
(parameters) -> expression
(parameters) -> { statements; }

左侧可以没有参数, 或者省略参数类型, 或者参数带类型
右侧可以是代码块或者表达式

例子:
```java
// 不带参数
() -> System.out.println("Hello, Lambda!");

// 带一个参数
x -> x * 2;

// 带多个参数
(x, y) -> x + y;

// 带显式类型和代码块
(int x, int y) -> {
    int sum = x + y;
    return sum;
};
```

## 核心用法1
用于 实现 **函数式接口** . 
函数式接口, 就是只有一个抽象函数的接口

例子:
```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b);
}

public class LambdaExample {
    public static void main(String[] args) {
	    // 下面两句相当于实现了MathOperation接口的operate函数
        MathOperation addition = (a, b) -> a + b;
        MathOperation multiplication = (a, b) -> a * b;

        System.out.println("Addition: " + addition.operate(5, 3)); // 8
        System.out.println("Multiplication: " + multiplication.operate(5, 3)); // 15
    }
}
```

Java 8 提供了一组通用的函数式接口，位于 `java.util.function` 包中，常见接口包括：

|接口|参数类型|返回类型|示例用法|
|---|---|---|---|
|`Predicate<T>`|`T`|`boolean`|用于测试条件|
|`Consumer<T>`|`T`|`void`|用于执行操作|
|`Function<T,R>`|`T`|`R`|将输入类型 `T` 转换为输出类型 `R`|
|`Supplier<T>`|无参数|`T`|提供一个结果|
|`BiFunction<T,U,R>`|`T,U`|`R`|接受两个参数并返回结果|

例子:
```java
import java.util.function.*;

public class FunctionalInterfacesExample {
    public static void main(String[] args) {
        // Predicate: 判断是否为偶数
        Predicate<Integer> isEven = x -> x % 2 == 0;
        System.out.println(isEven.test(4)); // true

        // Consumer: 打印字符串
        Consumer<String> print = s -> System.out.println(s);
        print.accept("Hello, Consumer!");

        // Function: 计算平方
        Function<Integer, Integer> square = x -> x * x;
        System.out.println(square.apply(5)); // 25

        // Supplier: 提供当前时间戳
        Supplier<Long> currentTime = System::currentTimeMillis;
        System.out.println(currentTime.get());

        // BiFunction: 求两个数的和
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println(add.apply(3, 7)); // 10
    }
}
```

## 核心用法2
与集合操作结合. Lambda 表达式和 Java 8 的集合框架（`Stream` API）结合，可以轻松实现过滤、映射和归约等操作。
例子:
```java
import java.util.*;
import java.util.stream.*;

public class StreamExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // 过滤名字长度大于3的
        List<String> filteredNames = names.stream()
                                          .filter(name -> name.length() > 3)
                                          .collect(Collectors.toList());
        System.out.println(filteredNames); // [Alice, Charlie, David]

        // 转换为大写
        List<String> upperCaseNames = names.stream()
                                           .map(name -> name.toUpperCase())
                                           .collect(Collectors.toList());
        System.out.println(upperCaseNames); // [ALICE, BOB, CHARLIE, DAVID]

        // 计算名字总长度
        int totalLength = names.stream()
                               .mapToInt(name -> name.length())
                               .sum();
        System.out.println("Total Length: " + totalLength); // 19
    }
}
```

## 注意的点:
如何lambda中会产生异常, 必须在lambda中catch. 




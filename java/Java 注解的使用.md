## Java 注解的使用

> 2019/5/6

看 Retrofit 源码的时候用到 Java 注解，学习一下：

```java
@Documented // 将注解中的元素包含到 Javadoc 中去
@Target(METHOD)  // 表明了注解的使用位置是在方法上
@Retention(RUNTIME) // 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们
public @interface OhGET {
    // 注解的属性，使用 default 声明默认值, 使用方法：@OhGET("属性名" = xxx)
    // 如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接接属性值填写到括号内
    String value() default "";
}
```

更加详细的内容可以看这篇文章：https://blog.csdn.net/briblue/article/details/73824058
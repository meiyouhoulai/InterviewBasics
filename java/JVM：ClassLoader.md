## JVM：ClassLoader

#### 概述

Java 中 class 文件会在下面三种情况下加载到内存

1. 调用类的构造方法
2. 调用类中的 static 变量或者方法
3. 主动加载，比如调用 Class.forName() 方法

JVM 中自带 3 个 ClassLoader：

1. APPClassLoader，负责加载 CLASS_PATH 路径下的 class 文件，我们自己写的类以及第三方 jar 都是由它来加载的
2. ExtClassLoader，会加载 `java.ext.dirs` 配置下的类文件，就是 jre/lib/ext 这个目录
3. BootstrapClassLoader，这个类是 C/C++ 代码写的，是虚拟机的一部分，我们无法在 Java 层直接获取它的引用，会加载 `sun.boot.class.path` 配置下的类文件，都是 jre 目录下的 jar 包或者 class 文件

#### 双亲委托机制

Java 虚拟机使用双亲委托机制将一个类加载到内存中，所谓双亲委派模式，先检测这个类是否已经加载了，如果加载了直接获取，如果没有的话是调用父类加载器去加载，依次递归，当父类加载器找不到要加载的类时，才会执行自身的类加载过程。

```java
protected Class<?> loadClass(String name, boolean resolve)
  throws ClassNotFoundException
{
  // 类加载过程是线程安全的
  synchronized (getClassLoadingLock(name)) {
    // 1. 判断类是否已经加载过, 如果加载过直接返回
    Class<?> c = findLoadedClass(name);
    if (c == null) {
      try {
        // 2. 没有加载过, 调用 parent 去加载, 注意这是一个递归
        if (parent != null) {
          c = parent.loadClass(name, false);
        } else {
          // 3. parent 为 null, 调用 BootstrapClassLoader 去加载, 这里是会调用 native 方法
          c = findBootstrapClassOrNull(name);
        }
      } catch (ClassNotFoundException e) {
        // ClassNotFoundException thrown if class not found
        // from the non-null parent class loader
      }
      // 4. 还没有加载到, 调用 findClass 方法去加载
      if (c == null) {
        c = findClass(name);
      }
    }
    if (resolve) {
      resolveClass(c);
    }
    return c;
  }
}
```

我们说的父类加载器，值得是类加载器中 parent 变量，而不是继承关系中类加载器的父类。

这个 parent 是通过 ClassLoader 的构造方法传进去的，AppClassLoader 的 parent 是 ExtClassLoader，ExtClassLoader 的 parent 是 null。

虚拟机加载 Test 类的从 AppClassLoader 的 loadClass 方法开始流程如下：

1. AppClassLoader 调用 findLoadedClass 判断类是否加载过
2. 调用 ExtClassLoader 的 loadClass 方法
3. 调用 ExtClassLoader 的 findLoadedClass 判断类是否加载过
4. ExtClassLoader 的 parent 为 null，调用 BootstrapClassLoader 去加载，没有找到
5. ExtClassLoader 调用 findClass 去加载，没有找到
6. ExtClassLoader 加载失败，它的 loadClass 方法返回 null
7. AppClassLoader 调用 findClass 方法去加载

双亲委派模型有两个好处，一是可以避免类的重复加载，二是防止 Java 核心 API 被篡改

我们注意到 loadClass 方法是 synchronized 的，在自定义 ClassLoader 的时候如果需要保持双亲委派模型应该重写 findClass 方法，如果想破坏双亲委派模型可以重新 loadClass 方法。

#### 自定义 ClassLoader

主要就是通过重写 findClass 方法来加载类

```java
public class DiskClassLoader extends ClassLoader {

    private String classPath;

    public DiskClassLoader() {
        this.classPath = "file:///Users/ssyijiu/my/DeepJava/src/main/java/classLoader/MyClass.class";
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 使用 NIO 读取 class 文件的二进制字节码
        byte[] classByte = null;
        try {
            Path path = Paths.get(new URI(classPath));
            classByte = Files.readAllBytes(path);
        } catch (URISyntaxException | IOException e) {
            e.printStackTrace();
        }
        // 使用 defineClass 方法加载
        return defineClass(name, classByte, 0, classByte.length);
    }
}
```

测试：

```java
public static void main(String[] args) {
  DiskClassLoader classLoader = new DiskClassLoader();
  try {
    // 注意这里要写全类名
    Class c = classLoader.loadClass("classLoader.MyClass");
    if (c != null) {
      Object obj = c.newInstance();
      Method method = c.getMethod("print", null);
      // 反射方法调用成功, 说明类加载成功
      method.invoke(obj, null);
    }
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```

#### Android 中的 ClassLoader

Android 中的 ClassLoader 有这下面几个：

1. BaseDexClassLoader：Android 中无法直接运行 class 文件，Android 会将所有的 class 文件转换为一个 dex 文件，Android 加载 dex 文件的实现就在 BaseDexClassLoader 中
2. BootClassLoader：继承自 ClassLoader，用来加载 Android Framework 中的 class 文件，例如 Activity 就是这个类加载的。
3. PathClassLoader 和 DexClassLoader：这两个 ClassLoader 都继承 BaseDexClassLoader，它们唯一的区别就是  DexClassLoader 多了一个 optimizedDirectory，optimizedDirectory 用来指定 odex 的存放目录，PathClassLoader 这个参数为 null，在 API 26 之后，这个参数被废弃，他们就完全一样了。

##### odex 是什么？

在 Android 应用打包的时候，会使用 dx 工具将所有的 class 文件打包为一个 dex 文件。

在 Android 5.0 之前在进行 Apk 安装的时候，会验证代码的合法性及优化代码的执行速度，在验证和优化之后，会产生一个 odex 文件（保存在 data/dalvik-cache 目录下），运行 App 的时候，直接加载这个 odex 文件，避免重复的验证和优化。

在 5.0 之后，采用 ART 虚拟机，在安装的时候，会使用 dex2oat 工具将 dex 文件翻译成一个 oat 文件（依然保存在 data/dalvik-cache 目录下，依旧是 .dex 后缀，实际上是 oat 文件），ART 加载 oat 后不需要处理就可以直接运行，省去了从字节码转换为机器码的过程。

##### BaseDexClassLoader 是如何加装 dex 文件的呢？

在 BaseDexClassLoader 的构造方法中，会对 DexPathList 进行初始化，在 DexPathList 中会使用一个 Element 数组保存所有的 dex 文件。

之后在 loadClass 的时候，也会使用双亲委托机制来进行类加载，双亲委托机制最后会调用 BaseDexClassLoader 的 findClass 方法，接着调用 DexPathList 的 findClass 方法。

```java
public Class<?> findClass(String name, List<Throwable> suppressed) {
  // 遍历 Element 数组
  for (Element element : dexElements) {
    // 找到目标类, 直接返回
    Class<?> clazz = element.findClass(name, definingContext, suppressed);
    if (clazz != null) {
      return clazz;
    }
  }

  if (dexElementsSuppressedExceptions != null) {
    suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
  }
  return null;
}
```

这里就是热修复技术的核心原理了，在加载一个类的时候，会遍历 dexElements，如果找到之后，直接返回，也就是说，如果当两个 dex 中有相同的类时，会优先加载前面那个 dex 文件中的类，这样我们将需要修复了打包成 dex 文件插入到 dexElements 前面就可以完成热修复了。
---
title: "动态代理的原理？"
description:
date: "2024-03-27T17:24:03+08:00"
slug: "java-dynamic-proxy"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Java", "动态代理", "反射"]
categories: ["Java", "反射"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制。

> 很多场景都是利用类似机制做到的，比如用来包装 RPC 调用、面向切面的编程（AOP）。
>
> 通过代理可以让调用者与实现者之间解耦。比如进行 RPC 调用，框架内部的寻址、序列化、反序列化等，对于调用者往往是没有太大意义的。

在Java中，动态代理主要基于以下两种技术实现:

- JDK 动态代理
- CGLIB 动态代理，基于 ASM

## JDK 动态代理

JDK 动态代理基于 Java 的[反射机制](/p/java-reflection)，代理对象需要 **实现至少一个接口**，代理类在运行时动态生成要求。

- 代理对象由 `Proxy.newProxyInstance()` 动态生成，其实现了 `HelloService` 接口。
- 在 `InvocationHandler` 中对方法调用进行了拦截，可在调用前后加入额外处理逻辑。

```java
// HelloService.java
public interface HelloService {
    void sayHello(String name);
}

// HelloServiceImpl.java
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello(String name) {
        System.out.println("Hello, " + name);
    }
}

// DynamicProxyDemo.java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class DynamicProxyDemo {
    public static void main(String[] args) {
        // 创建原始对象
        HelloService original = new HelloServiceImpl();

        // 利用 Proxy.newProxyInstance 创建代理对象
        HelloService proxyInstance = (HelloService) Proxy.newProxyInstance(
            original.getClass().getClassLoader(),
            new Class[]{HelloService.class},
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("Before method: " + method.getName());
                    Object result = method.invoke(original, args);
                    System.out.println("After method: " + method.getName());
                    return result;
                }
            }
        );

        // 调用代理对象的方法
        proxyInstance.sayHello("World");
    }
}
```

## CGLIB 动态代理

当目标类没有实现接口时，可以使用 `CGLIB` 动态代理，基于 ASM 字节码操作框架，通过继承目标类生成代理类。 ***需要添加 CGLIB 的依赖***。

因为是子类化，我们可以达到近似使用被调用者本身的效果。在 Spring 编程中，框架通常会处理这种情况。

- `Enhancer` 用于生成目标类（这里是 Person 类）的子类。
- 在 `MethodInterceptor` 的 `intercept` 方法中可以对方法调用进行拦截，添加前置或后置逻辑。

```xml
<!-- pom.xml -->
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

```java
// CGLIBProxyDemo.java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class CGLIBProxyDemo {

    public static class Person {
        public void sayHello(String name) {
            System.out.println("Hello, " + name);
        }
    }

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        // 指定需要代理的目标类
        enhancer.setSuperclass(Person.class);
        // 设置方法拦截器
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                System.out.println("Before method: " + method.getName());
                Object result = proxy.invokeSuper(obj, args);
                System.out.println("After method: " + method.getName());
                return result;
            }
        });

        // 创建代理对象
        Person proxy = (Person) enhancer.create();
        proxy.sayHello("World");
    }
}

```

### CGLIB 注意事项

自 **JDK 9** 引入 **模块化系统（JPMS）** 以来，对 CGLIB 的使用受到了一定限制。

​CGLIB 通过生成目标类的子类来实现代理，这需要在运行时访问目标类的构造方法和方法。

**​然而，JDK 9 的模块系统对[反射](/p/java-reflection)访问进行了严格控制，默认情况下，模块之间的访问是受限的**。因此，CGLIB 在尝试通过[反射](/p/java-reflection)访问 `ClassLoader.defineClass` 时会抛出 `InaccessibleObjectException`。

**解决办法**: 可以在启动应用时添加 JVM 参数，显式打开需要的模块包，以允许 CGLIB 进行[反射](/p/java-reflection)访问。

```bash
--add-opens java.base/java.lang=ALL-UNNAMED
```

该参数的作用是将 java.base 模块中的 java.lang 包对所有未命名模块（即没有显式声明模块的代码）开放，从而允许[反射](/p/java-reflection)访问。

## JDK 动态代理 VS CGLIB 动态代理

JDK 动态代理

- 优点
  - **简洁易用**：利用 Java 内置的[反射](/p/java-reflection)机制，无需引入第三方依赖
  - **接口驱动**：代理类必须实现接口，符合面向接口编程的设计原则
- 缺点
  - **只能代理接口**：无法对没有实现接口的类进行代理
  - **性能开销**：通过[反射](/p/java-reflection)实现，性能较低

CGLIB 动态代理

- 优点
  - **无需接口**：可以代理没有实现接口的类
  - **性能较高**：生成的代理类方法调用性能优于 JDK动态代理
- 缺点
  - **类限制**：无法代理被 `final` 修饰的类和方法，因为 CGLIB 通过继承实现代理
  - **引入依赖**：需要引入 CGLIB 的依赖

> 在实际应用中，框架如 Spring 会根据目标对象的情况，选择使用 JDK 动态代理或 CGLIB 动态代理，以实现 AOP 功能。​

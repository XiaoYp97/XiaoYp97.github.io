---
title: "动态代理？"
description:
date: "2024-03-27T17:24:03+08:00"
slug: "java-dynamic-proxy"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Java", "动态代理", "反射", "代理模式"]
categories: ["Java", "反射", "代理模式"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---
## 代理模式

**代理模式（Proxy Pattern）** 是一种结构型设计模式，通过为其他对象提供一种代理以控制对这个对象的访问。

​它通常用于在不修改原始对象的情况下，增加对对象的控制和扩展功能，例如访问控制、延迟加载、日志记录等。​

### 静态代理

- **定义:**  在编译时，代理类已经存在，代理对象和被代理对象之间是一对一的关系。
- **实现方式:**  手动创建代理类，实现与被代理类相同的接口，在代理类中持有被代理类的实例，通过调用被代理类的方法来实现代理功能。
- **优点:**  代码简单，易于理解。
- **缺点:**  每增加一个被代理类，都需要创建一个对应的代理类，代码重复度高，维护困难。

### 动态代理

- **定义:**  在运行时动态生成代理类，无需在编译时预先定义代理类。
- **实现方式:**
  - **JDK 动态代理:**  利用 Java 的反射机制，在运行时生成实现了指定接口的代理对象。要求被代理类必须实现接口。
  - **CGLIB 动态代理:**  通过继承目标类，生成目标类的子类来实现代理。适用于没有实现接口的类，但无法代理 `final` 类和方法。
- **优点:**  提高了代码的灵活性和复用性，减少了代理类的数量。
- **缺点:**  相对于静态代理，性能开销略大，因为涉及到反射和动态生成类。

> 很多场景都是利用类似机制做到的，比如用来包装 RPC 调用、面向切面的编程（AOP）。
>
> 通过代理可以让调用者与实现者之间解耦。比如进行 RPC 调用，框架内部的寻址、序列化、反序列化等，对于调用者往往是没有太大意义的。

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

**解决办法:**  可以在启动应用时添加 JVM 参数，显式打开需要的模块包，以允许 CGLIB 进行[反射](/p/java-reflection)访问。

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

## 与装饰器模式的区别？

代理模式和装饰器模式在结构上可能相似，但它们在**目的**、**关注点**和**使用方式**上有显著区别。

**1. 目的:**

- **代理模式（Proxy Pattern）:**  主要目的是控制对目标对象的访问。代理对象通常用于在客户端和真实对象之间提供一个中介，控制对真实对象的访问权限或添加额外的操作，例如延迟加载、安全检查或远程调用等。
- **装饰器模式（Decorator Pattern）:**  主要目的是动态地增强对象的功能。通过将原始对象包装在装饰器中，可以在不修改原始类的情况下，添加新的行为或功能。

**2. 关注点:**

- **代理模式:**  关注于对对象访问的控制和管理，强调如何代理和控制对真实对象的操作。
- **装饰器模式:**  关注于对对象功能的扩展和增强，强调如何在运行时添加新的功能或行为。

**3. 使用方式:**

- **代理模式:**  代理类通常在编译时就确定，与真实对象实现相同的接口，并持有对真实对象的引用。在客户端通过代理对象访问真实对象，代理对象可以在调用真实对象的方法前后添加额外的操作。
- **装饰器模式:**  装饰器类也实现与被装饰对象相同的接口，但通常在运行时动态创建，并将被装饰对象作为构造参数传入。装饰器对象通过组合的方式，将增强功能添加到被装饰对象上。

**示例:**

- **代理模式:**

```java
  public interface Subject {
      void request();
  }

  public class RealSubject implements Subject {
      public void request() {
          System.out.println("RealSubject request");
      }
  }

  public class ProxySubject implements Subject {
      private RealSubject realSubject;

      public ProxySubject() {
          this.realSubject = new RealSubject();
      }

      public void request() {
          // 代理操作，例如访问控制
          System.out.println("ProxySubject request");
          realSubject.request();
      }
  }
  ```

- **装饰器模式:**

```java
  public interface Subject {
      void request();
  }

  public class RealSubject implements Subject {
      public void request() {
          System.out.println("RealSubject request");
      }
  }

  public class DecoratorSubject implements Subject {
      private Subject subject;

      public DecoratorSubject(Subject subject) {
          this.subject = subject;
      }

      public void request() {
          // 装饰操作，例如功能增强
          System.out.println("DecoratorSubject request");
          subject.request();
      }
  }
  ```

总结而言，代理模式侧重于控制对对象的访问，而装饰器模式侧重于增强对象的功能。代理对象通常用于控制对真实对象的访问权限，而装饰器对象用于在运行时动态地添加新的功能或行为。

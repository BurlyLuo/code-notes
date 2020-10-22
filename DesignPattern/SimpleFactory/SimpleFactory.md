# 简单工厂模式（Simple Factory Pattern）

## Intent

定义一个创建产品对象的工厂接口，将产品对象的实际创建工作推迟到具体子工厂类当中。

## 优缺点

**优点：**

工厂类包含必要的逻辑判断，可以决定在什么时候创建哪一个产品的实例。客户端可以免除直接创建产品对象的职责，很方便的创建出相应的产品。工厂和产品的职责区分明确。

客户端无需知道所创建具体产品的类名，只需知道参数即可。

也可以引入配置文件，在不修改客户端代码的情况下更换和添加新的具体产品类。

**缺点：**

简单工厂模式的工厂类单一，负责所有产品的创建，职责过重，一旦异常，整个系统将受影响。且工厂类代码会非常臃肿，违背高聚合原则。

使用简单工厂模式会增加系统中类的个数（引入新的工厂类），增加系统的复杂度和理解难度。

系统扩展困难，一旦增加新产品不得不修改工厂逻辑，在产品类型较多时，可能造成逻辑过于复杂。

简单工厂模式使用了 static 工厂方法，造成工厂角色无法形成基于继承的等级结构。

## 应用场景

工厂类负责创建的对象比较少：由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。

客户端只知道传入工厂类的参数，对于如何创建对象不关心：客户端既不需要关心创建细节，甚至连类名都不需要记住，只需要知道类型所对应的参数。

## Class Diagram

简单工厂模式的主要角色如下：

- 简单工厂（SimpleFactory）：是简单工厂模式的核心，负责实现创建所有实例的内部逻辑。工厂类的创建产品类的方法可以被外界直接调用，创建所需的产品对象。
- 抽象产品（Product）：是简单工厂创建的所有对象的父类，负责描述所有实例共有的公共接口。
- 具体产品（ConcreteProduct）：是简单工厂模式的创建目标。

![SimpleFactory](/DesignPattern/SimpleFactory/SimpleFactory.png)

## Implementation

```java
public class Client {
    public static void main(String[] args) {
    }

    //抽象产品
    public interface Product {
        void show();
    }

    //具体产品：ProductA
    static class ConcreteProduct1 implements Product {
        public void show() {
            System.out.println("具体产品1显示...");
        }
    }

    //具体产品：ProductB
    static class ConcreteProduct2 implements Product {
        public void show() {
            System.out.println("具体产品2显示...");
        }
    }

    final class Const {
        static final int PRODUCT_A = 1;
        static final int PRODUCT_B = 2;
    }

    static class SimpleFactory {
        public static Product makeProduct(int kind) {
            switch (kind) {
                case Const.PRODUCT_A:
                    return new ConcreteProduct1();
                case Const.PRODUCT_B:
                    return new ConcreteProduct2();
            }
            return null;
        }
    }
}
```

## 模式应用

1.JDK类库中广泛使用了简单工厂模式，如工具类`java.text.DateFormat`，它用于格式化一个本地日期或者时间。

```java
public final static DateFormat getDateInstance();
public final static DateFormat getDateInstance(int style);
public final static DateFormat getDateInstance(int style,Locale locale);
```

2.Java加密技术

获取不同加密算法的密钥生成器：

```java
KeyGenerator keyGen=KeyGenerator.getInstance("DESede");
```

创建密码器：

```java
Cipher cp=Cipher.getInstance("DESede");
```

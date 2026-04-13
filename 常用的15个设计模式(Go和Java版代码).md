## 一、创建型设计模式

### 1.1 单例模式（Singleton Pattern）

#### 1.1.1 编程中常见的问题

在软件开发过程中，我们经常会遇到这样的场景：需要确保某个类在整个应用程序生命周期内只有一个实例存在。例如，配置管理器负责读取和管理应用程序的配置信息，如果存在多个实例，可能会导致配置不一致的问题。再如，日志记录器需要集中管理所有的日志输出，多个实例会造成日志分散和性能问题。数据库连接池作为稀缺资源，也需要严格控制实例数量以避免资源耗尽。这些场景共同的核心问题在于：我们需要一个全局唯一的访问点来获取某个对象。

在实际开发中，如果不加控制地创建实例，常见的问题包括：配置不同步导致的行为不一致、内存占用过高、系统资源竞争、不可预期的错误等。特别是在多线程环境下，如果没有适当的同步机制，可能会创建出多个实例，违背了最初的设计意图。

#### 1.1.2 有什么解决方案

传统的解决方案是使用全局变量或者静态方法来实现单例。全局变量虽然简单，但会污染命名空间，且无法保证延迟初始化。静态方法方案虽然能够保证实例的唯一性，但无法支持面向接口编程，且难以进行单元测试。现代的解决方案则是通过设计模式本身来解决这个问题，即单例模式。

单例模式通过将类的构造函数私有化，防止外部直接实例化对象。然后在类内部创建一个静态方法或者通过其他机制来控制实例的创建过程，确保全局只有一个实例存在。这个内部实例可以通过延迟初始化的方式创建，即只在第一次需要时才创建，这样可以提高应用程序的启动性能。

#### 1.1.3 设计模式的解决方案

单例模式的核心思想是控制实例的创建过程，它通过以下几个关键机制来实现：首先，私有化构造函数，防止外部通过new关键字创建实例；其次，在类内部维护一个静态的实例变量；然后，提供一个公开的静态方法供外部获取实例；最后，在获取实例的方法中实现线程安全的实例创建逻辑。

单例模式根据实现方式可以分为多种类型，包括饿汉式（类加载时就创建实例）、懒汉式（延迟加载，第一次使用时创建）、双重检查锁定（兼顾性能和线程安全）、静态内部类（利用类加载机制保证线程安全）等。每种实现方式都有其适用场景，开发者需要根据具体需求选择合适的实现。

#### 1.1.4 Go语言实现

```go
package singleton

import (
	"fmt"
	"sync"
)

// Singleton 接口定义
type Singleton interface {
	GetName() string
	DoSomething()
}

// singleton 实例结构体
type singleton struct {
	name string
}

func (s *singleton) GetName() string {
	return s.name
}

func (s *singleton) DoSomething() {
	fmt.Println("Singleton instance is doing something")
}

// ============ 懒汉式实现（线程不安全）============

type UnsafeLazySingleton struct {
	instance *singleton
}

func (u *UnsafeLazySingleton) GetInstance() *singleton {
	if u.instance == nil {
		// 此处存在线程安全问题
		u.instance = &singleton{name: "UnsafeLazySingleton"}
	}
	return u.instance
}

// ============ 懒汉式实现（线程安全）============

type SafeLazySingleton struct {
	instance *singleton
	mu       sync.Mutex
}

func (s *SafeLazySingleton) GetInstance() *singleton {
	s.mu.Lock()
	defer s.mu.Unlock()
	if s.instance == nil {
		s.instance = &singleton{name: "SafeLazySingleton"}
	}
	return s.instance
}

// ============ 双重检查锁定实现 ============

type DoubleCheckSingleton struct {
	instance *singleton
	mu       sync.RWMutex
}

func (d *DoubleCheckSingleton) GetInstance() *singleton {
	// 第一次检查，不需要加锁
	if d.instance == nil {
		d.mu.Lock()
		defer d.mu.Unlock()
		// 第二次检查，需要加锁
		if d.instance == nil {
			d.instance = &singleton{name: "DoubleCheckSingleton"}
		}
	}
	return d.instance
}

// ============ 饿汉式实现 ============

type HungrySingleton struct {
	instance *singleton
}

func init() {
	// 在init函数中初始化，程序启动时即创建实例
	_ = &singleton{name: "HungrySingleton"}
}

func (h *HungrySingleton) GetInstance() *singleton {
	return &singleton{name: "HungrySingleton"}
}

// ============ 静态内部类实现（推荐）============

type StaticInnerSingleton struct {
}

func (s *StaticInnerSingleton) GetInstance() *singleton {
	return staticHolder.instance
}

// holder 会在首次引用时才会被加载，实现延迟加载
type holder struct {
	instance *singleton
}

var staticHolder = holder{
	instance: &singleton{name: "StaticInnerSingleton"},
}

// ============ sync.Once 实现（最简洁）============

type OnceSingleton struct {
	instance *singleton
	once     sync.Once
}

func (o *OnceSingleton) GetInstance() *singleton {
	o.once.Do(func() {
		o.instance = &singleton{name: "OnceSingleton"}
	})
	return o.instance
}

// NewOnceSingleton 构造函数
func NewOnceSingleton() *OnceSingleton {
	return &OnceSingleton{}
}
```

#### 1.1.5 Java语言实现

```java
package com.designpatterns.singleton;

/**
 * 单例模式完整实现
 */
public class Singleton {
    
    private String name;
    
    // 私有构造函数
    private Singleton(String name) {
        this.name = name;
        System.out.println("Singleton instance created: " + name);
    }
    
    public String getName() {
        return name;
    }
    
    public void doSomething() {
        System.out.println("Singleton instance is doing something");
    }
    
    // ============ 饿汉式实现 ============
    // 类加载时就创建实例，线程安全，但可能造成资源浪费
    private static final Singleton HUNGRY_INSTANCE = new Singleton("Hungry");
    
    public static Singleton getHungryInstance() {
        return HUNGRY_INSTANCE;
    }
    
    // ============ 懒汉式实现（线程不安全）============
    // 延迟加载，但多线程环境下不安全
    private static Singleton unsafeLazyInstance;
    
    public static Singleton getUnsafeLazyInstance() {
        if (unsafeLazyInstance == null) {
            unsafeLazyInstance = new Singleton("UnsafeLazy");
        }
        return unsafeLazyInstance;
    }
    
    // ============ 懒汉式实现（线程安全）============
    // 通过synchronized保证线程安全，但性能开销较大
    private static Singleton safeLazyInstance;
    
    public static synchronized Singleton getSafeLazyInstance() {
        if (safeLazyInstance == null) {
            safeLazyInstance = new Singleton("SafeLazy");
        }
        return safeLazyInstance;
    }
    
    // ============ 双重检查锁定实现 ============
    // 既保证线程安全，又减少同步开销
    private static volatile Singleton doubleCheckInstance;
    
    public static Singleton getDoubleCheckInstance() {
        if (doubleCheckInstance == null) {
            synchronized (Singleton.class) {
                if (doubleCheckInstance == null) {
                    doubleCheckInstance = new Singleton("DoubleCheck");
                }
            }
        }
        return doubleCheckInstance;
    }
    
    // ============ 静态内部类实现 ============
    // 利用类加载机制保证线程安全，同时实现延迟加载
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton("StaticInner");
    }
    
    public static Singleton getStaticInnerInstance() {
        return SingletonHolder.INSTANCE;
    }
    
    // ============ 枚举实现（最佳实践）============
    // 线程安全、防止反序列化攻击、简洁高效
    public enum EnumSingleton {
        INSTANCE;
        
        private final Singleton instance;
        
        EnumSingleton() {
            instance = new Singleton("EnumSingleton");
        }
        
        public Singleton getInstance() {
            return instance;
        }
    }
}
```

#### 1.1.6 适用场景

单例模式的应用场景非常广泛。在系统级资源的统一管理方面，数据库连接池是最典型的应用场景。数据库连接是稀缺资源，频繁创建和销毁连接会带来巨大的性能开销，通过单例模式管理一个连接池，可以让所有数据库操作共享同一个连接池实例，既保证了资源的合理利用，又避免了连接泄漏。

配置管理器也是单例模式的经典应用。应用程序通常需要读取各种配置文件，如数据库配置、缓存配置、日志配置等。如果每次使用配置都重新读取文件，不仅性能低下，还可能导致配置不一致。使用单例模式可以确保所有配置信息在内存中只有一份，并且可以在应用程序启动时预加载，提高响应速度。

日志系统是另一个广泛使用单例模式的领域。日志需要集中管理，所有的日志输出应该写入同一个日志文件或者通过同一个日志框架进行处理。使用单例模式可以确保日志系统的一致性，同时方便实现日志级别的统一控制。此外，线程池、缓存管理器、任务调度器等也常常采用单例模式来实现。

#### 1.1.7 优缺点分析

单例模式的主要优点包括：第一，提供了对唯一实例的受控访问，由于单例类封装了它的唯一实例，所以可以严格控制客户怎样以及何时访问它，这比全局变量更加安全；第二，在内存中只有一个实例，减少了内存开支，特别是对于一些需要频繁创建和销毁的对象来说，单例模式可以显著降低内存占用；第三，可以避免对资源的多重占用，比如写文件操作，由于只有一个实例存在，可以避免对同一资源的同时写操作。

然而，单例模式也存在明显的缺点：第一，单例模式扩展困难，由于构造函数是私有的，继承单例类非常困难，而且即使能够继承，也无法改变实例数量；第二，单例模式违反了单一职责原则，它既负责管理自己的实例，又承担了业务逻辑的职责；第三，单例模式在多线程环境下需要特别注意，如果不正确实现可能导致多个实例被创建；第四，单例模式难以进行单元测试，因为单例类的构造函数是私有的，很难mock其行为。

---

### 1.2 工厂方法模式（Factory Method Pattern）

#### 1.2.1 编程中常见的问题

在面向对象编程中，对象的创建是一个核心问题。当系统中存在大量相似但具体类型不同的对象时，如何优雅地管理这些对象的创建就变得尤为重要。假设我们正在开发一个图形编辑软件，需要支持创建多种图形，如圆形、矩形、三角形等。如果在客户端代码中直接使用new关键字创建具体图形类，就会造成代码与具体类的高耦合。

这种耦合带来的问题是多方面的。首先，当需要添加新的图形类型时，必须修改所有使用new关键字的代码，这违反了开闭原则。其次，客户端代码需要了解每个具体类的构造函数签名，这增加了代码的复杂性和维护难度。再次，在需要根据配置文件或用户输入动态创建对象时，switch语句或if-else链会变得臃肿不堪。最后，单元测试变得困难，因为客户端代码与具体类直接绑定，无法轻易替换为mock对象。

#### 1.2.2 有什么解决方案

一种简单的解决方案是使用简单工厂模式，即创建一个工厂类，根据参数决定创建哪种产品。这种方式虽然简单，但违背了开闭原则，因为每当添加新产品时都需要修改工厂类。更优雅的解决方案是工厂方法模式，它将对象的创建延迟到子类中进行，让子类决定实例化哪个类。

工厂方法模式的核心思想是定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法模式使得一个类的实例化延迟到其子类。在工厂方法模式中，抽象工厂类负责定义创建对象的公共接口，而具体工厂类则负责创建具体的产品对象。这种设计使得系统可以在不修改具体工厂类的情况下引入新的产品类型。

#### 1.2.3 设计模式的解决方案

工厂方法模式的基本结构包含四个核心角色：Product（产品接口）定义了产品的公共接口，所有具体产品都要实现这个接口；ConcreteProduct（具体产品）是产品接口的实现类，定义了具体产品的行为；Factory（工厂接口）声明了创建产品的工厂方法，该方法返回一个产品类型的对象；ConcreteFactory（具体工厂）是工厂接口的实现类，负责创建具体产品实例。

当客户端需要产品时，它调用工厂对象的工厂方法，而不是直接使用new操作符创建产品。具体工厂类实现了工厂方法，根据自己的逻辑决定创建哪种具体产品。这种设计使得系统可以在运行时切换具体工厂类，从而改变产品创建逻辑，而无需修改客户端代码。

#### 1.2.4 Go语言实现

```go
package factory

import (
	"fmt"
	"errors"
)

// ============ 产品层定义 ============

// Product 抽象产品接口
type Product interface {
	Use() string
	GetName() string
}

// ConcreteProduct 具体产品实现

type ConcreteProductA struct {
	name string
}

func (c *ConcreteProductA) Use() string {
	return "Using Product A"
}

func (c *ConcreteProductA) GetName() string {
	return c.name
}

type ConcreteProductB struct {
	name string
}

func (c *ConcreteProductB) Use() string {
	return "Using Product B"
}

func (c *ConcreteProductB) GetName() string {
	return c.name
}

type ConcreteProductC struct {
	name string
}

func (c *ConcreteProductC) Use() string {
	return "Using Product C"
}

func (c *ConcreteProductC) GetName() string {
	return c.name
}

// ============ 工厂层定义 ============

// Factory 抽象工厂接口
type Factory interface {
	CreateProduct(productType string) (Product, error)
}

// ConcreteFactory 具体工厂实现
type ConcreteFactory struct{}

func (f *ConcreteFactory) CreateProduct(productType string) (Product, error) {
	switch productType {
	case "A":
		return &ConcreteProductA{name: "ProductA"}, nil
	case "B":
		return &ConcreteProductB{name: "ProductB"}, nil
	case "C":
		return &ConcreteProductC{name: "ProductC"}, nil
	default:
		return nil, errors.New("unknown product type")
	}
}

// ============ 扩展版本：每个产品对应独立工厂 ============

// AbstractFactory 更抽象的工厂接口
type AbstractFactory interface {
	Factory
}

// ProductAFactory 专门生产ProductA的工厂
type ProductAFactory struct{}

func (f *ProductAFactory) CreateProduct(productType string) (Product, error) {
	if productType == "A" {
		return &ConcreteProductA{name: "ProductA"}, nil
	}
	return nil, errors.New("ProductAFactory can only create ProductA")
}

func (f *ProductAFactory) Create() Product {
	p, _ := f.CreateProduct("A")
	return p
}

// ProductBFactory 专门生产ProductB的工厂
type ProductBFactory struct{}

func (f *ProductBFactory) CreateProduct(productType string) (Product, error) {
	if productType == "B" {
		return &ConcreteProductB{name: "ProductB"}, nil
	}
	return nil, errors.New("ProductBFactory can only create ProductB")
}

func (f *ProductBFactory) Create() Product {
	p, _ := f.CreateProduct("B")
	return p
}

// ============ 使用示例 ============

func FactoryMethodDemo() {
	// 使用统一工厂
	factory := &ConcreteFactory{}
	
	productA, _ := factory.CreateProduct("A")
	productB, _ := factory.CreateProduct("B")
	
	fmt.Println(productA.Use())
	fmt.Println(productB.Use())
	
	// 使用专门工厂
	factoryA := &ProductAFactory{}
	productFromA := factoryA.Create()
	fmt.Println(productFromA.Use())
}
```

#### 1.2.5 Java语言实现

```java
package com.designpatterns.factory;

// ============ 产品层定义 ============

/**
 * 抽象产品接口
 */
public interface Product {
    String use();
    String getName();
}

/**
 * 具体产品A
 */
public class ConcreteProductA implements Product {
    private final String name;
    
    public ConcreteProductA() {
        this.name = "ProductA";
    }
    
    @Override
    public String use() {
        return "Using Product A";
    }
    
    @Override
    public String getName() {
        return name;
    }
}

/**
 * 具体产品B
 */
public class ConcreteProductB implements Product {
    private final String name;
    
    public ConcreteProductB() {
        this.name = "ProductB";
    }
    
    @Override
    public String use() {
        return "Using Product B";
    }
    
    @Override
    public String getName() {
        return name;
    }
}

/**
 * 具体产品C
 */
public class ConcreteProductC implements Product {
    private final String name;
    
    public ConcreteProductC() {
        this.name = "ProductC";
    }
    
    @Override
    public String use() {
        return "Using Product C";
    }
    
    @Override
    public String getName() {
        return name;
    }
}

// ============ 工厂层定义 ============

/**
 * 抽象工厂接口
 */
public interface Factory {
    Product createProduct(String type);
}

/**
 * 具体工厂实现
 */
public class ConcreteFactory implements Factory {
    
    @Override
    public Product createProduct(String type) {
        switch (type) {
            case "A":
                return new ConcreteProductA();
            case "B":
                return new ConcreteProductB();
            case "C":
                return new ConcreteProductC();
            default:
                throw new IllegalArgumentException("Unknown product type: " + type);
        }
    }
}

// ============ 扩展版本：产品族工厂 ============

/**
 * 抽象工厂接口（支持产品族）
 */
public interface AbstractFactory {
    ProductA createProductA();
    ProductB createProductB();
}

/**
 * 产品族A的工厂
 */
public class ProductFamilyAFactory implements AbstractFactory {
    
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA();
    }
    
    @Override
    public ProductB createProductB() {
        return new ConcreteProductB();
    }
}

/**
 * 工厂方法模板
 */
public abstract class FactoryMethodTemplate {
    
    // 工厂方法，子类实现
    protected abstract Product createProduct();
    
    // 模板方法，定义算法骨架
    public String produceAndUse() {
        Product product = createProduct();
        return product.use();
    }
}

public class ProductAFactoryMethod extends FactoryMethodTemplate {
    
    @Override
    protected Product createProduct() {
        return new ConcreteProductA();
    }
}
```

#### 1.2.6 适用场景

工厂方法模式的应用场景主要分为几类。第一类是当你不知道需要创建哪种具体对象时，比如根据配置文件或运行时参数决定创建哪种产品。典型的应用是日志框架，常见的日志框架如SLF4J允许用户在运行时选择具体的日志实现（Log4j、Logback等），工厂方法模式可以优雅地实现这种灵活性。

第二类是当你想要为库或框架扩展提供灵活性时。数据库驱动程序是很好的例子，JDBC使用工厂方法模式，允许数据库厂商提供自己的Driver实现，而应用程序无需修改代码就可以切换数据库。插件系统也常用这种模式，允许第三方开发者通过实现工厂接口来扩展系统功能。

第三类是将创建逻辑封装起来以提供更好的可测试性时。通过依赖注入框架配合工厂模式，可以更容易地创建mock对象进行单元测试。复杂对象的构建也适合使用工厂方法，比如文档解析器需要根据文件类型创建不同的解析器对象。

#### 1.2.7 优缺点分析

工厂方法模式的主要优点是提高了系统的灵活性和可扩展性。由于工厂方法模式将产品创建的逻辑封装在子类中，当需要添加新产品时，只需添加一个新的具体工厂类和具体产品类，而无需修改现有代码，这完全符合开闭原则。同时，工厂方法模式通过依赖抽象（产品接口），降低了客户端与具体产品之间的耦合度。

工厂方法模式的另一个优点是提供了更好的封装性。客户端不需要知道具体产品类的类名，只需要知道工厂类提供的工厂方法即可。这种信息隐藏使得代码更加清晰，也便于维护。此外，工厂方法模式支持并行开发，不同的开发者可以同时工作，一个开发具体产品类，另一个开发工厂类。

然而，工厂方法模式也有其缺点。主要缺点是类的数量会增加。对于每一个具体产品类，都需要对应创建一个具体工厂类，这会导致系统中类的数量成对增加，增加了系统的复杂度。对于简单的应用场景，这种设计可能显得过于复杂。另外，工厂方法模式要求每个产品都必须有一个具体工厂类，当产品种类很多时，工厂类的数量也会很多。

---

### 1.3 抽象工厂模式（Abstract Factory Pattern）

#### 1.3.1 编程中常见的问题

在复杂的软件系统中，往往需要创建一系列相关的产品对象。考虑一个UI组件库的例子，它需要支持多种皮肤主题，如现代风格、复古风格等。每种风格都包含一组相关的组件，如按钮、文本框、复选框等。这些组件虽然功能相同，但外观不同。更重要的是，同一时刻只能使用一种风格的组件，否则界面会出现混乱。

如果使用简单工厂来创建这些组件，那么每添加一种新风格都需要修改工厂类。更严重的是，客户端代码可能会不小心混用不同风格的组件，导致界面不统一。当系统需要支持多种数据库时，也会遇到类似的问题。在企业应用系统中，可能需要同时支持MySQL、Oracle、SQL Server等数据库，每种数据库都有一套相关的对象，如连接、命令、结果集等，这些对象必须配合使用才能正常工作。

这种“产品族”的创建问题，单一的工厂方法模式无法优雅地解决，因为工厂方法模式强调的是一个产品等级结构（继承体系）的创建，而我们需要处理的是多个产品等级结构的问题。

#### 1.3.2 有什么解决方案

一种方案是创建多个独立的工厂，每个工厂负责创建一种产品族中的所有产品。这种方案虽然简单，但会导致工厂类过多，而且客户端需要了解所有工厂类的存在。另一种方案是使用抽象工厂模式，它提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

抽象工厂模式的核心思想是将产品族的创建封装在一个抽象工厂接口中，每个具体工厂类负责创建一种产品族的所有产品。这样，客户端代码只需要与抽象工厂和抽象产品接口交互，无需关心具体的产品实现。当需要切换产品族时，只需要替换具体工厂即可，整个系统无需修改。

#### 1.3.3 设计模式的解决方案

抽象工厂模式包含四个核心角色：AbstractFactory（抽象工厂）声明了一组创建抽象产品的方法；ConcreteFactory（具体工厂）实现了创建具体产品对象的操作；AbstractProduct（抽象产品）为每种产品声明了接口；ConcreteProduct（具体产品）定义了要创建的产品的具体实现。

在抽象工厂模式中，每个具体工厂对应一个产品族，负责创建该产品族中的所有产品。例如，在皮肤主题系统中，ModernFactory负责创建所有现代风格的组件，而ClassicFactory负责创建所有经典风格的组件。当客户端需要创建产品时，它调用抽象工厂的方法，而具体创建过程由具体工厂完成。

#### 1.3.4 Go语言实现

```go
package abstractfactory

import (
	"fmt"
	"errors"
)

// ============ 产品接口定义 ============

// Button 按钮产品接口
type Button interface {
	Paint() string
	Click() string
}

// TextBox 文本框产品接口
type TextBox interface {
	Paint() string
	SetText(text string) string
}

// CheckBox 复选框产品接口
type CheckBox interface {
	Paint() string
	Toggle() string
}

// ============ 抽象工厂接口 ============

// GUIFactory 抽象工厂接口
type GUIFactory interface {
	CreateButton() Button
	CreateTextBox() TextBox
	CreateCheckBox() CheckBox
}

// ============ 现代风格产品实现 ============

type ModernButton struct{}

func (b *ModernButton) Paint() string {
	return "Rendering Modern Button with flat design"
}

func (b *ModernButton) Click() string {
	return "Modern Button clicked"
}

type ModernTextBox struct {
	text string
}

func (t *ModernTextBox) Paint() string {
	return "Rendering Modern TextBox with rounded corners"
}

func (t *ModernTextBox) SetText(text string) string {
	t.text = text
	return fmt.Sprintf("Text set to: %s", text)
}

type ModernCheckBox struct {
	checked bool
}

func (c *ModernCheckBox) Paint() string {
	return "Rendering Modern CheckBox with minimal style"
}

func (c *ModernCheckBox) Toggle() string {
	c.checked = !c.checked
	return fmt.Sprintf("Checkbox is now: %v", c.checked)
}

// ModernFactory 现代风格工厂
type ModernFactory struct{}

func (f *ModernFactory) CreateButton() Button {
	return &ModernButton{}
}

func (f *ModernFactory) CreateTextBox() TextBox {
	return &ModernTextBox{}
}

func (f *ModernFactory) CreateCheckBox() CheckBox {
	return &ModernCheckBox{}
}

// ============ 经典风格产品实现 ============

type ClassicButton struct{}

func (b *ClassicButton) Paint() string {
	return "Rendering Classic Button with 3D effect"
}

func (b *ClassicButton) Click() string {
	return "Classic Button clicked"
}

type ClassicTextBox struct {
	text string
}

func (t *ClassicTextBox) Paint() string {
	return "Rendering Classic TextBox with border"
}

func (t *ClassicTextBox) SetText(text string) string {
	t.text = text
	return fmt.Sprintf("Text set to: %s", text)
}

type ClassicCheckBox struct {
	checked bool
}

func (c *ClassicCheckBox) Paint() string {
	return "Rendering Classic CheckBox with square style"
}

func (c *ClassicCheckBox) Toggle() string {
	c.checked = !c.checked
	return fmt.Sprintf("Checkbox is now: %v", c.checked)
}

// ClassicFactory 经典风格工厂
type ClassicFactory struct{}

func (f *ClassicFactory) CreateButton() Button {
	return &ClassicButton{}
}

func (f *ClassicFactory) CreateTextBox() TextBox {
	return &ClassicTextBox{}
}

func (f *ClassicFactory) CreateCheckBox() CheckBox {
	return &ClassicCheckBox{}
}

// ============ 工厂生成器 ============

type FactoryProducer struct{}

func (fp *FactoryProducer) GetFactory(factoryType string) (GUIFactory, error) {
	switch factoryType {
	case "modern":
		return &ModernFactory{}, nil
	case "classic":
		return &ClassicFactory{}, nil
	default:
		return nil, errors.New("unknown factory type")
	}
}

// ============ 客户端代码 ============

func CreateUI(factory GUIFactory) {
	button := factory.CreateButton()
	textBox := factory.CreateTextBox()
	checkBox := factory.CreateCheckBox()
	
	fmt.Println("--- UI Components ---")
	fmt.Println(button.Paint())
	fmt.Println(textBox.Paint())
	fmt.Println(checkBox.Paint())
	
	fmt.Println("\n--- Interactions ---")
	fmt.Println(button.Click())
	fmt.Println(textBox.SetText("Hello"))
	fmt.Println(checkBox.Toggle())
}
```

#### 1.3.5 Java语言实现

```java
package com.designpatterns.abstractfactory;

// ============ 产品接口定义 ============

/**
 * 按钮产品接口
 */
public interface Button {
    void paint();
    void click();
}

/**
 * 文本框产品接口
 */
public interface TextBox {
    void paint();
    void setText(String text);
}

/**
 * 复选框产品接口
 */
public interface CheckBox {
    void paint();
    void toggle();
}

// ============ 现代风格产品实现 ============

public class ModernButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering Modern Button with flat design");
    }
    
    @Override
    public void click() {
        System.out.println("Modern Button clicked");
    }
}

public class ModernTextBox implements TextBox {
    private String text;
    
    @Override
    public void paint() {
        System.out.println("Rendering Modern TextBox with rounded corners");
    }
    
    @Override
    public void setText(String text) {
        this.text = text;
        System.out.println("Text set to: " + text);
    }
}

public class ModernCheckBox implements CheckBox {
    private boolean checked;
    
    @Override
    public void paint() {
        System.out.println("Rendering Modern CheckBox with minimal style");
    }
    
    @Override
    public void toggle() {
        checked = !checked;
        System.out.println("Checkbox is now: " + checked);
    }
}

// ============ 经典风格产品实现 ============

public class ClassicButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering Classic Button with 3D effect");
    }
    
    @Override
    public void click() {
        System.out.println("Classic Button clicked");
    }
}

public class ClassicTextBox implements TextBox {
    private String text;
    
    @Override
    public void paint() {
        System.out.println("Rendering Classic TextBox with border");
    }
    
    @Override
    public void setText(String text) {
        this.text = text;
        System.out.println("Text set to: " + text);
    }
}

public class ClassicCheckBox implements CheckBox {
    private boolean checked;
    
    @Override
    public void paint() {
        System.out.println("Rendering Classic CheckBox with square style");
    }
    
    @Override
    public void toggle() {
        checked = !checked;
        System.out.println("Checkbox is now: " + checked);
    }
}

// ============ 抽象工厂接口 ============

/**
 * 抽象工厂接口
 */
public interface GUIFactory {
    Button createButton();
    TextBox createTextBox();
    CheckBox createCheckBox();
}

/**
 * 现代风格工厂
 */
public class ModernFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new ModernButton();
    }
    
    @Override
    public TextBox createTextBox() {
        return new ModernTextBox();
    }
    
    @Override
    public CheckBox createCheckBox() {
        return new ModernCheckBox();
    }
}

/**
 * 经典风格工厂
 */
public class ClassicFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new ClassicButton();
    }
    
    @Override
    public TextBox createTextBox() {
        return new ClassicTextBox();
    }
    
    @Override
    public CheckBox createCheckBox() {
        return new ClassicCheckBox();
    }
}

// ============ 工厂生成器 ============

public class FactoryProducer {
    public static GUIFactory getFactory(String type) {
        if (type.equalsIgnoreCase("modern")) {
            return new ModernFactory();
        } else if (type.equalsIgnoreCase("classic")) {
            return new ClassicFactory();
        }
        throw new IllegalArgumentException("Unknown factory type: " + type);
    }
}

// ============ 客户端代码 ============

public class Application {
    private GUIFactory factory;
    private Button button;
    private TextBox textBox;
    private CheckBox checkBox;
    
    public Application(GUIFactory factory) {
        this.factory = factory;
    }
    
    public void createUI() {
        button = factory.createButton();
        textBox = factory.createTextBox();
        checkBox = factory.createCheckBox();
    }
    
    public void paint() {
        button.paint();
        textBox.paint();
        checkBox.paint();
    }
}
```

#### 1.3.6 适用场景

抽象工厂模式特别适合需要创建产品族的场景。最典型的应用是跨平台UI工具包。以Java Swing为例，它需要为不同的操作系统提供不同的本地化组件，如Windows风格、Linux风格、Mac风格等。每种风格都包含按钮、菜单、对话框等组件，这些组件必须保持风格的一致性。抽象工厂模式允许开发者在不修改客户端代码的情况下切换整个产品族。

企业级应用的数据库支持也是抽象工厂模式的重要应用场景。在大型企业系统中，可能需要同时支持多种数据库，或者根据客户环境部署不同的数据库。数据库相关对象如Connection、Statement、ResultSet等必须来自同一种数据库，否则会出现兼容性问题。抽象工厂模式确保了这些对象的创建保持一致性。

游戏开发中的主题和关卡设计也常用抽象工厂模式。不同的游戏主题包含不同风格的背景、角色、道具等资源，使用抽象工厂可以方便地切换整个游戏主题。此外，文档转换器需要创建各种文档格式的读写器，这些读写器必须配套使用才能正常工作。

#### 1.3.7 优缺点分析

抽象工厂模式的主要优点是它确保了产品族的一致性。当使用抽象工厂创建产品时，客户端保证获得的是同一产品族的产品，这避免了不兼容组件的混用。抽象工厂模式还提供了良好的隔离性，客户端代码与具体产品类解耦，只需要知道抽象工厂和抽象产品接口即可。此外，添加新的产品族非常容易，只需要添加新的具体工厂类即可，符合开闭原则。

然而，抽象工厂模式的缺点也很明显。最主要的问题是难以支持新种类的产品。当需要在产品族中添加新的产品类型时，必须修改抽象工厂接口，这会导致所有具体工厂都需要修改，违反了开闭原则。另一个缺点是类的数量会大幅增加，每个产品族都需要一个具体工厂类，这可能导致系统变得复杂。

---

### 1.4 建造者模式（Builder Pattern）

#### 1.4.1 编程中常见的问题

在软件开发中，我们经常遇到这样的场景：需要创建一个复杂的对象，这个对象由多个部分组成，每个部分都有多种配置选项。考虑一个计算机配置系统，用户可以自定义CPU、内存、硬盘、显示器等组件，每个组件又有多种型号可选。如果使用传统的构造函数来创建计算机对象，可能会遇到构造函数参数过多、参数顺序难以记忆、难以处理可选参数等问题。

一个常见的解决方法是使用大量的构造函数重载，但这会导致代码膨胀，且客户端代码难以理解。更糟糕的是，这种方法无法清晰表达对象的结构，用户很难知道哪些参数是必需的，哪些是可选的。此外，当对象的构建逻辑复杂时，这些逻辑会被分散在多个构造函数或setter方法中，难以维护和测试。

另一个常见的问题是构建不可变对象时面临的挑战。为了保证线程安全或表达清晰的语义，我们希望某些对象一旦创建就不能修改。但传统的构建方式往往需要提供setter方法，这在一定程度上破坏了不可变性。

#### 1.4.2 有什么解决方案

传统的解决方案包括两种：一种是使用带参数列表的构造函数，另一种是使用重叠构造函数（telescoping constructor pattern），即创建多个构造函数，每个构造函数接受不同数量的参数。这两种方案虽然可行，但都有明显的缺点：参数过多时难以使用，容易出错，且无法表达清晰的语义。

更优雅的解决方案是建造者模式。建造者模式将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。在建造者模式中，我们将对象的构建过程分解为多个步骤，每个步骤负责构建对象的一个部分。然后，通过一个指挥者（Director）来控制构建的顺序和过程，最终生成完整的产品对象。

#### 1.4.3 设计模式的解决方案

建造者模式的核心角色包括：Product（产品）是最终要构建的复杂对象；Builder（抽象建造者）定义了创建Product各个部分的抽象接口；ConcreteBuilder（具体建造者）实现了Builder接口，完成复杂产品的各个部件的具体构建方法；Director（指挥者）负责安排已经建造的模块的顺序，从而构建出完整的产品对象。

建造者模式的关键在于将构建过程封装在Builder中，客户端通过Director控制构建过程，而具体的构建细节由ConcreteBuilder实现。这种设计使得相同的构建过程可以创建不同的产品表示，同时也使得构建逻辑可以复用。

#### 1.4.4 Go语言实现

```go
package builder

import (
	"fmt"
	"strings"
)

// ============ 产品定义 ============

// Computer 产品类
type Computer struct {
	cpu      string
	memory   string
	storage  string
	gpu      string
	os       string
	hostname string
}

// ComputerBuilder 建造者接口
type ComputerBuilder interface {
	SetCPU(cpu string) ComputerBuilder
	SetMemory(memory string) ComputerBuilder
	SetStorage(storage string) ComputerBuilder
	SetGPU(gpu string) ComputerBuilder
	SetOS(os string) ComputerBuilder
	SetHostname(hostname string) ComputerBuilder
	Build() *Computer
}

// ConcreteBuilder 具体建造者
type concreteComputerBuilder struct {
	computer *Computer
}

func NewComputerBuilder() ComputerBuilder {
	return &concreteComputerBuilder{
		computer: &Computer{},
	}
}

func (b *concreteComputerBuilder) SetCPU(cpu string) ComputerBuilder {
	b.computer.cpu = cpu
	return b
}

func (b *concreteComputerBuilder) SetMemory(memory string) ComputerBuilder {
	b.computer.memory = memory
	return b
}

func (b *concreteComputerBuilder) SetStorage(storage string) ComputerBuilder {
	b.computer.storage = storage
	return b
}

func (b *concreteComputerBuilder) SetGPU(gpu string) ComputerBuilder {
	b.computer.gpu = gpu
	return b
}

func (b *concreteComputerBuilder) SetOS(os string) ComputerBuilder {
	b.computer.os = os
	return b
}

func (b *concreteComputerBuilder) SetHostname(hostname string) ComputerBuilder {
	b.computer.hostname = hostname
	return b
}

func (b *concreteComputerBuilder) Build() *Computer {
	return b.computer
}

// Director 指挥者
type Director struct {
	builder ComputerBuilder
}

func NewDirector(builder ComputerBuilder) *Director {
	return &Director{builder: builder}
}

// BuildGamingComputer 构建游戏电脑
func (d *Director) BuildGamingComputer() *Computer {
	return d.builder.
		SetCPU("Intel i9-13900K").
		SetMemory("64GB DDR5").
		SetStorage("2TB NVMe SSD").
		SetGPU("RTX 4090").
		SetOS("Windows 11").
		SetHostname("Gaming-PC").
		Build()
}

// BuildOfficeComputer 构建办公电脑
func (d *Director) BuildOfficeComputer() *Computer {
	return d.builder.
		SetCPU("Intel i5-13400").
		SetMemory("16GB DDR4").
		SetStorage("512GB SSD").
		SetGPU("Integrated Graphics").
		SetOS("Windows 11 Pro").
		SetHostname("Office-PC").
		Build()
}

// BuildServerComputer 构建服务器
func (d *Director) BuildServerComputer() *Computer {
	return d.builder.
		SetCPU("AMD EPYC 9654").
		SetMemory("256GB ECC RAM").
		SetStorage("4TB NVMe RAID").
		SetGPU("None").
		SetOS("Ubuntu Server 22.04 LTS").
		SetHostname("Server-01").
		Build()
}

// Computer 的方法
func (c *Computer) Display() string {
	var parts []string
	parts = append(parts, fmt.Sprintf("CPU: %s", c.cpu))
	parts = append(parts, fmt.Sprintf("Memory: %s", c.memory))
	parts = append(parts, fmt.Sprintf("Storage: %s", c.storage))
	parts = append(parts, fmt.Sprintf("GPU: %s", c.gpu))
	parts = append(parts, fmt.Sprintf("OS: %s", c.os))
	parts = append(parts, fmt.Sprintf("Hostname: %s", c.hostname))
	return strings.Join(parts, "\n")
}

// ============ 链式调用示例 ============

func BuilderDemo() {
	// 使用建造者模式
	builder := NewComputerBuilder()
	director := NewDirector(builder)
	
	fmt.Println("=== Gaming Computer ===")
	gamingPC := director.BuildGamingComputer()
	fmt.Println(gamingPC.Display())
	
	fmt.Println("\n=== Office Computer ===")
	officePC := director.BuildOfficeComputer()
	fmt.Println(officePC.Display())
	
	// 直接使用链式调用
	fmt.Println("\n=== Custom Computer ===")
	customPC := NewComputerBuilder().
		SetCPU("AMD Ryzen 9 7950X").
		SetMemory("32GB DDR5").
		SetStorage("1TB NVMe SSD").
		SetGPU("RX 7900 XTX").
		SetOS("Linux Mint").
		SetHostname("Custom-Build").
		Build()
	fmt.Println(customPC.Display())
}
```

#### 1.4.5 Java语言实现

```java
package com.designpatterns.builder;

// ============ 产品定义 ============

/**
 * 复杂产品类 - 计算机配置
 */
public class Computer {
    private String cpu;
    private String memory;
    private String storage;
    private String gpu;
    private String os;
    private String hostname;
    
    // 私有构造函数，强制使用建造者
    private Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.memory = builder.memory;
        this.storage = builder.storage;
        this.gpu = builder.gpu;
        this.os = builder.os;
        this.hostname = builder.hostname;
    }
    
    // Getter方法
    public String getCpu() { return cpu; }
    public String getMemory() { return memory; }
    public String getStorage() { return storage; }
    public String getGpu() { return gpu; }
    public String getOs() { return os; }
    public String getHostname() { return hostname; }
    
    @Override
    public String toString() {
        return String.format(
            "Computer:\n  CPU: %s\n  Memory: %s\n  Storage: %s\n  GPU: %s\n  OS: %s\n  Hostname: %s",
            cpu, memory, storage, gpu, os, hostname
        );
    }
    
    // ============ 静态内部 Builder 类 ============
    public static class Builder {
        // 必需参数
        private final String cpu;
        private final String memory;
        
        // 可选参数 - 初始化为默认值
        private String storage = "256GB SSD";
        private String gpu = "Integrated Graphics";
        private String os = "None";
        private String hostname = "localhost";
        
        public Builder(String cpu, String memory) {
            this.cpu = cpu;
            this.memory = memory;
        }
        
        public Builder storage(String storage) {
            this.storage = storage;
            return this;
        }
        
        public Builder gpu(String gpu) {
            this.gpu = gpu;
            return this;
        }
        
        public Builder os(String os) {
            this.os = os;
            return this;
        }
        
        public Builder hostname(String hostname) {
            this.hostname = hostname;
            return this;
        }
        
        public Computer build() {
            return new Computer(this);
        }
    }
}

// ============ 抽象建造者接口 ============

public interface ComputerBuilder {
    void setCPU(String cpu);
    void setMemory(String memory);
    void setStorage(String storage);
    void setGPU(String gpu);
    void setOS(String os);
    void setHostname(String hostname);
    Computer build();
}

// ============ 具体建造者实现 ============

public class GamingComputerBuilder implements ComputerBuilder {
    private Computer computer;
    
    public GamingComputerBuilder() {
        this.computer = new Computer.Builder("Intel i9-13900K", "64GB DDR5")
            .storage("2TB NVMe SSD")
            .gpu("RTX 4090")
            .os("Windows 11")
            .hostname("Gaming-PC")
            .build();
    }
    
    @Override
    public void setCPU(String cpu) {
        // 实现细节
    }
    
    // ... 其他方法实现
    
    @Override
    public Computer build() {
        return computer;
    }
}

// ============ 指挥者 ============

public class Director {
    private ComputerBuilder builder;
    
    public Director(ComputerBuilder builder) {
        this.builder = builder;
    }
    
    public Computer buildGamingComputer() {
        return new Computer.Builder("Intel i9-13900K", "64GB DDR5")
            .storage("2TB NVMe SSD")
            .gpu("RTX 4090")
            .os("Windows 11")
            .hostname("Gaming-PC")
            .build();
    }
    
    public Computer buildOfficeComputer() {
        return new Computer.Builder("Intel i5-13400", "16GB DDR4")
            .storage("512GB SSD")
            .gpu("Integrated Graphics")
            .os("Windows 11 Pro")
            .hostname("Office-PC")
            .build();
    }
    
    public Computer buildServerComputer() {
        return new Computer.Builder("AMD EPYC 9654", "256GB ECC RAM")
            .storage("4TB NVMe RAID")
            .gpu("None")
            .os("Ubuntu Server 22.04 LTS")
            .hostname("Server-01")
            .build();
    }
}

// ============ 使用示例 ============

public class BuilderDemo {
    public static void main(String[] args) {
        // 使用链式调用（内部 Builder 模式）
        Computer gamingPC = new Computer.Builder("Intel i9-13900K", "64GB DDR5")
            .storage("2TB NVMe SSD")
            .gpu("RTX 4090")
            .os("Windows 11")
            .hostname("Gaming-PC")
            .build();
        
        System.out.println(gamingPC);
        
        // 使用指挥者
        Director director = new Director(new GamingComputerBuilder());
        Computer server = director.buildServerComputer();
        
        System.out.println(server);
    }
}
```

#### 1.4.6 适用场景

建造者模式的主要应用场景包括创建复杂对象。当一个类的构造函数有多个参数，且参数有必需和可选之分时，建造者模式是理想的选择。例如，在Web开发框架中，构建HTTP请求或响应对象时经常使用建造者模式，因为这些对象有很多可选的头部信息、参数等。

另一个典型应用是构建不可变对象。为了保证对象的线程安全或表达清晰的语义，我们希望某些对象创建后不可修改。使用传统的setter方法会破坏不可变性，而建造者模式在构建阶段可以使用setter方法，最终构建出的对象是不可变的。Java中的StringBuilder就是一个很好的例子，虽然它不是严格的建造者模式，但其设计思想是一致的。

文档生成和报表构建也是建造者模式的常见应用。一份复杂的文档通常包含标题、正文、图表、脚注等多个部分，使用建造者模式可以逐步构建文档的各个部分，最终组合成完整的文档。类似的，报表构建器可以逐步添加标题、数据、图表等元素，生成最终的报表。

#### 1.4.7 优缺点分析

建造者模式的主要优点是代码清晰易读。通过链式调用方法，每个参数的作用一目了然，客户端代码更容易理解和维护。其次，建造者模式支持变种构建，可以创建同一个产品的不同表示，只需实现不同的Builder即可。此外，建造者模式可以做到对象的不可变性，在构建过程中可以修改，但一旦构建完成，对象就是不可变的。

建造者模式的另一个优点是将构建逻辑封装在Builder中，与使用逻辑分离。对于复杂的构建逻辑，这使得代码更加模块化。同时，建造者模式允许构建过程分步进行，某些参数可以在不同的地方设置，最后再组装。

建造者模式的主要缺点是对于简单的对象显得有些多余。如果对象的构建很简单，使用构造函数可能更直接。其次，建造者模式会产生多余的Builder对象，在性能敏感的场景下需要考虑这个问题。另外，与抽象工厂模式类似，当需要添加新的产品组件时，可能需要修改Builder接口。

---

### 1.5 原型模式（Prototype Pattern）

#### 1.5.1 编程中常见的问题

在软件开发中，我们经常需要创建多个相似的对象。考虑一个图形编辑软件，用户可能需要复制一个复杂的图形对象来创建多个相似的图形。如果使用new关键字重新创建并配置每个对象，不仅代码繁琐，而且当对象的配置很复杂时，很容易遗漏某些配置，导致复制出的对象与原对象不一致。

另一个常见问题是某些对象的创建成本很高。比如，一个对象需要从数据库获取大量数据，或者需要复杂的计算才能初始化。如果每次需要类似的对象都重新创建，会造成很大的性能开销。此外，在游戏开发中，同一种敌人可能有成千上万个实例，如果每个实例都独立创建和配置，会消耗大量内存和时间。

还有一个问题是对象创建与对象本身紧密耦合。当客户端代码需要使用new关键字创建对象时，它必须知道具体的产品类名，这违反了依赖倒置原则，使得更换产品实现变得困难。

#### 1.5.2 有什么解决方案

一种解决方案是使用简单工厂模式，但这种方法仍然需要知道具体的工厂类。另一种解决方案是原型模式，它的核心思想是通过复制一个已存在的对象来创建新对象，而不是通过new关键字。

原型模式通过“复制”操作来创建对象，这个复制过程由被复制的对象自己完成，而不是由客户端代码控制。这种方式确保了复制出的对象与原对象完全一致，因为复制逻辑就在原对象内部。同时，通过定义一个抽象的原型接口，所有实现该接口的对象都可以被复制，这解耦了客户端代码与具体的产品类。

#### 1.5.3 设计模式的解决方案

原型模式包含两个核心角色：Prototype（原型）定义了一个用于克隆自身的接口；ConcretePrototype（具体原型）实现克隆自身的操作。原型模式的关键在于Clone方法，这个方法在具体原型类中实现，负责创建并返回当前对象的一个副本。

实现Clone方法有两种主要方式：浅拷贝和深拷贝。浅拷贝只复制对象的引用，不复制引用指向的对象；深拷贝则会递归复制所有引用指向的对象，确保副本与原对象完全独立。对于包含引用类型成员变量的对象，需要根据实际需求选择合适的拷贝方式。

#### 1.5.4 Go语言实现

```go
package prototype

import (
	"fmt"
)

// ============ 原型接口 ============

// Prototype 原型接口
type Prototype interface {
	Clone() Prototype
	GetName() string
	SetName(name string)
}

// ============ 具体原型实现 ============

// ConcretePrototype 具体原型类
type ConcretePrototype struct {
	name    string
	id      int
	tags    []string
	nested  *NestedObject
}

// NewConcretePrototype 构造函数
func NewConcretePrototype(name string, id int) *ConcretePrototype {
	return &ConcretePrototype{
		name:   name,
		id:     id,
		tags:   make([]string, 0),
		nested: &NestedObject{Value: "default"},
	}
}

// Clone 实现原型接口 - 深拷贝
func (c *ConcretePrototype) Clone() Prototype {
	// 创建一个新的副本
	clone := &ConcretePrototype{
		name: c.name,
		id:   c.id,
	}
	
	// 深拷贝切片
	clone.tags = make([]string, len(c.tags))
	copy(clone.tags, c.tags)
	
	// 深拷贝嵌套对象
	if c.nested != nil {
		clone.nested = &NestedObject{
			Value: c.nested.Value,
		}
	}
	
	return clone
}

// ShallowClone 浅拷贝实现
func (c *ConcretePrototype) ShallowClone() *ConcretePrototype {
	// 注意：切片和嵌套对象使用相同的引用
	return &ConcretePrototype{
		name:   c.name,
		id:     c.id,
		tags:   c.tags,      // 共享切片引用
		nested: c.nested,     // 共享对象引用
	}
}

func (c *ConcretePrototype) GetName() string {
	return c.name
}

func (c *ConcretePrototype) SetName(name string) {
	c.name = name
}

func (c *ConcretePrototype) AddTag(tag string) {
	c.tags = append(c.tags, tag)
}

func (c *ConcretePrototype) Display() {
	fmt.Printf("Name: %s, ID: %d, Tags: %v, Nested: %s\n", 
		c.name, c.id, c.tags, c.nested.Value)
}

// NestedObject 嵌套对象
type NestedObject struct {
	Value string
}

// ============ 原型注册表 ============

// PrototypeRegistry 原型注册表
type PrototypeRegistry struct {
	prototypes map[string]Prototype
}

func NewPrototypeRegistry() *PrototypeRegistry {
	return &PrototypeRegistry{
		prototypes: make(map[string]Prototype),
	}
}

func (r *PrototypeRegistry) Register(key string, prototype Prototype) {
	r.prototypes[key] = prototype
}

func (r *PrototypeRegistry) Get(key string) Prototype {
	if p, ok := r.prototypes[key]; ok {
		return p.Clone()
	}
	return nil
}

// ============ 使用示例 ============

func PrototypeDemo() {
	// 创建原型
	prototype := NewConcretePrototype("Original", 1)
	prototype.AddTag("important")
	prototype.AddTag("prototype")
	
	fmt.Println("=== Original Prototype ===")
	prototype.Display()
	
	// 克隆原型
	clone := prototype.Clone()
	clone.SetName("Clone")
	
	// 修改克隆对象，验证是否独立
	if cp, ok := clone.(*ConcretePrototype); ok {
		cp.AddTag("cloned")
		cp.nested.Value = "Modified"
	}
	
	fmt.Println("\n=== After Clone and Modification ===")
	fmt.Println("Original:")
	prototype.Display()
	fmt.Println("Clone:")
	clone.Display()
	
	// 使用原型注册表
	registry := NewPrototypeRegistry()
	registry.Register("default", prototype)
	
	clonedFromRegistry := registry.Get("default")
	clonedFromRegistry.SetName("From Registry")
	
	fmt.Println("\n=== Cloned from Registry ===")
	clonedFromRegistry.Display()
}
```

#### 1.5.5 Java语言实现

```java
package com.designpatterns.prototype;

import java.util.ArrayList;
import java.util.List;

/**
 * 原型接口
 */
public interface Prototype extends Cloneable {
    Prototype clone();
    String getName();
    void setName(String name);
}

/**
 * 具体原型类
 */
public class ConcretePrototype implements Prototype {
    private String name;
    private int id;
    private List<String> tags;
    private NestedObject nested;
    
    public ConcretePrototype(String name, int id) {
        this.name = name;
        this.id = id;
        this.tags = new ArrayList<>();
        this.nested = new NestedObject("default");
    }
    
    // 拷贝构造函数（Java中没有内置，需要手动实现）
    public ConcretePrototype(ConcretePrototype source) {
        this.name = source.name;
        this.id = source.id;
        // 深拷贝List
        this.tags = new ArrayList<>(source.tags);
        // 深拷贝嵌套对象
        this.nested = new NestedObject(source.nested.getValue());
    }
    
    @Override
    protected Object clone() {
        // 实现深拷贝
        ConcretePrototype clone = new ConcretePrototype(this);
        return clone;
    }
    
    // Java的clone方法实现
    @Override
    public ConcretePrototype shallowClone() {
        try {
            return (ConcretePrototype) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Clone not supported", e);
        }
    }
    
    @Override
    public String getName() { return name; }
    
    @Override
    public void setName(String name) { this.name = name; }
    
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public List<String> getTags() { return tags; }
    
    public void addTag(String tag) {
        this.tags.add(tag);
    }
    
    public NestedObject getNested() { return nested; }
    
    public void display() {
        System.out.printf("Name: %s, ID: %d, Tags: %s, Nested: %s%n", 
            name, id, tags, nested.getValue());
    }
}

/**
 * 嵌套对象
 */
class NestedObject implements Cloneable {
    private String value;
    
    public NestedObject(String value) {
        this.value = value;
    }
    
    public String getValue() { return value; }
    public void setValue(String value) { this.value = value; }
    
    @Override
    protected NestedObject clone() {
        return new NestedObject(this.value);
    }
}

/**
 * 原型注册表
 */
public class PrototypeRegistry {
    private Map<String, Prototype> prototypes = new HashMap<>();
    
    public void register(String key, Prototype prototype) {
        prototypes.put(key, prototype);
    }
    
    public Prototype get(String key) {
        Prototype prototype = prototypes.get(key);
        return prototype != null ? prototype.clone() : null;
    }
}

/**
 * 使用示例
 */
public class PrototypeDemo {
    public static void main(String[] args) {
        // 创建原型
        ConcretePrototype prototype = new ConcretePrototype("Original", 1);
        prototype.addTag("important");
        prototype.addTag("prototype");
        
        System.out.println("=== Original Prototype ===");
        prototype.display();
        
        // 克隆原型
        ConcretePrototype clone = (ConcretePrototype) prototype.clone();
        clone.setName("Clone");
        clone.addTag("cloned");
        
        System.out.println("\n=== After Clone ===");
        System.out.println("Original:");
        prototype.display();
        System.out.println("Clone:");
        clone.display();
        
        // 使用注册表
        PrototypeRegistry registry = new PrototypeRegistry();
        registry.register("default", prototype);
        
        Prototype fromRegistry = registry.get("default");
        fromRegistry.setName("From Registry");
        
        System.out.println("\n=== Cloned from Registry ===");
        fromRegistry.display();
    }
}
```

#### 1.5.6 适用场景

原型模式的应用场景主要包括需要创建大量相似对象的情况。在游戏开发中，同一种敌人、子弹、特效等可能有成千上万个实例，使用原型模式可以显著提高性能。通过复制原型对象，可以快速创建新实例，避免重复的初始化过程。

文档编辑软件中的复制粘贴功能也是原型模式的典型应用。当用户复制一个复杂的图形或文本块时，系统实际上是通过复制原型对象来创建新的实例。这种方式确保了复制出的对象与原对象完全一致，同时避免了重复的配置工作。

配置复制是另一个常见的应用场景。在企业应用中，可能需要为不同的环境（开发、测试、生产）创建不同的配置，这些配置大部分是相同的，只有少数参数不同。使用原型模式可以先创建一个基础配置，然后复制并修改特定的参数，快速生成新环境的配置。

#### 1.5.7 优缺点分析

原型模式的主要优点是提高了对象创建的效率。当创建对象的成本较高时，通过复制已有对象可以显著减少开销。其次，原型模式简化了对象的创建过程，客户端不需要知道对象的具体类，只需要知道原型接口即可。此外，原型模式可以动态地添加或删除产品类，只需注册一个原型实例即可。

原型模式的另一个优点是保存了对象的状态快照。当需要创建对象的历史版本或者需要撤销功能时，可以将当前对象作为原型保存，需要时再复制出来。原型模式也支持异步创建对象，这在某些场景下可以提高响应速度。

然而，原型模式也有明显的缺点。对于包含循环引用的对象，克隆过程可能变得复杂，需要仔细处理。此外，克隆操作与其他创建模式相比，可能引入隐蔽性，调试时需要特别注意。另一个问题是，如果原型对象包含大量共享资源，频繁克隆可能导致资源管理困难。

---

## 二、结构型设计模式

### 2.1 适配器模式（Adapter Pattern）

#### 2.1.1 编程中常见的问题

在软件开发中，经常会遇到接口不兼容的问题。考虑这样的场景：系统需要集成一个第三方库，这个库提供了丰富的功能，但其接口与我们系统中定义的接口不一致。如果直接使用这个库，代码会变得混乱，难以维护。更糟糕的是，如果库升级后改变了接口，所有直接调用该库的代码都需要修改。

另一个常见的场景是遗留系统与现代系统的集成。许多企业都有大量的遗留代码，这些代码虽然已经过时，但仍然运行着核心业务逻辑。当需要将现代系统与遗留系统集成时，往往会遇到接口不匹配的问题。直接重写遗留系统代价太大，也不现实。

此外，在团队协作开发中，不同的模块可能由不同的开发者实现，他们定义的接口可能不完全一致。当需要将这些模块组合在一起时，接口适配就成为必要的工作。

#### 2.1.2 有什么解决方案

一种简单的解决方案是修改现有代码，让所有接口保持一致。但这在很多情况下是不可行的，因为我们无法修改第三方库的代码，或者修改遗留系统的风险太大。另一种方案是使用适配器模式，通过创建一个中间层来转换接口。

适配器模式的核心思想是将一个类的接口转换成客户端期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。这种模式允许我们使用现有的类，但通过一个转换层来适配新的接口需求。

#### 2.1.3 设计模式的解决方案

适配器模式有两种主要实现方式：类适配器和对象适配器。类适配器通过继承来实现，它同时继承目标接口和被适配者类，这种方式在Go语言中受限于单继承，在Java中则可以通过接口和继承组合使用。对象适配器通过组合来实现，它包含一个被适配者的引用，更灵活也更常用。

适配器模式包含四个核心角色：Target（目标接口）定义了客户端使用的特定领域的接口；Adaptee（被适配者）是一个已存在的接口，需要被适配；Adapter（适配器）是一个转换器，通过继承或引用Adaptee来实现Target接口；Client（客户端）配合符合Target接口的对象协同工作。

#### 2.1.4 Go语言实现

```go
package adapter

import (
	"fmt"
)

// ============ 目标接口 ============

// Player 播放器接口（客户端期望的接口）
type Player interface {
	Play(filename string) string
	Pause() string
	Stop() string
}

// ============ 被适配者（需要适配的接口）============

// LegacyMediaPlayer 旧式媒体播放器接口
type LegacyMediaPlayer interface {
	OpenFile(path string)
	StartPlayback()
	EndPlayback()
}

// OldMP3Player 旧式MP3播放器
type OldMP3Player struct{}

func (p *OldMP3Player) OpenFile(path string) {
	fmt.Printf("Legacy: Opening file: %s\n", path)
}

func (p *OldMP3Player) StartPlayback() {
	fmt.Println("Legacy: Starting playback")
}

func (p *OldMP3Player) EndPlayback() {
	fmt.Println("Legacy: Ending playback")
}

// AdvancedMediaPlayer 高级媒体播放器接口（另一种被适配者）
type AdvancedMediaPlayer interface {
	LoadMedia(path string) error
	Play() error
	Stop() error
}

type ModernMediaPlayer struct{}

func (m *ModernMediaPlayer) LoadMedia(path string) error {
	fmt.Printf("Modern: Loading media: %s\n", path)
	return nil
}

func (m *ModernMediaPlayer) Play() error {
	fmt.Println("Modern: Playing")
	return nil
}

func (m *ModernMediaPlayer) Stop() error {
	fmt.Println("Modern: Stopping")
	return nil
}

// ============ 适配器实现 ============

// MP3Adapter 适配器：将LegacyMediaPlayer适配为Player接口
type MP3Adapter struct {
	legacyPlayer *OldMP3Player
	currentFile string
}

func NewMP3Adapter() *MP3Adapter {
	return &MP3Adapter{
		legacyPlayer: &OldMP3Player{},
	}
}

func (a *MP3Adapter) Play(filename string) string {
	a.currentFile = filename
	a.legacyPlayer.OpenFile(filename)
	a.legacyPlayer.StartPlayback()
	return fmt.Sprintf("Playing: %s", filename)
}

func (a *MP3Adapter) Pause() string {
	a.legacyPlayer.EndPlayback()
	return "Paused"
}

func (a *MP3Adapter) Stop() string {
	a.legacyPlayer.EndPlayback()
	a.currentFile = ""
	return "Stopped"
}

// AdvancedPlayerAdapter 适配器：将AdvancedMediaPlayer适配为Player接口
type AdvancedPlayerAdapter struct {
	advancedPlayer *ModernMediaPlayer
	currentFile    string
}

func NewAdvancedPlayerAdapter() *AdvancedPlayerAdapter {
	return &AdvancedPlayerAdapter{
		advancedPlayer: &ModernMediaPlayer{},
	}
}

func (a *AdvancedPlayerAdapter) Play(filename string) string {
	a.advancedPlayer.LoadMedia(filename)
	a.advancedPlayer.Play()
	return fmt.Sprintf("Playing: %s", filename)
}

func (a *AdvancedPlayerAdapter) Pause() string {
	return "Advanced player doesn't support pause"
}

func (a *AdvancedPlayerAdapter) Stop() string {
	a.advancedPlayer.Stop()
	return "Stopped"
}

// ============ 双向适配器 ============

// BidirectionalAdapter 双向适配器，同时实现两个接口
type BidirectionalAdapter struct {
	*OldMP3Player
	*ModernMediaPlayer
	mediaType string
}

func NewBidirectionalAdapter() *BidirectionalAdapter {
	return &BidirectionalAdapter{
		OldMP3Player:     &OldMP3Player{},
		ModernMediaPlayer: &ModernMediaPlayer{},
	}
}

// 实现LegacyMediaPlayer接口
func (b *BidirectionalAdapter) PlayLegacy(filename string) {
	b.OpenFile(filename)
	b.StartPlayback()
}

// 实现ModernMediaPlayer接口
func (b *BidirectionalAdapter) PlayModern(filename string) error {
	b.LoadMedia(filename)
	return b.Play()
}

// ============ 播放器管理器（演示适配器使用）============

type MediaPlayerManager struct {
	players map[string]Player
}

func NewMediaPlayerManager() *MediaPlayerManager {
	return &MediaPlayerManager{
		players: make(map[string]Player),
	}
}

func (m *MediaPlayerManager) RegisterPlayer(format string, player Player) {
	m.players[format] = player
}

func (m *MediaPlayerManager) Play(format, filename string) {
	if player, ok := m.players[format]; ok {
		fmt.Println(player.Play(filename))
	} else {
		fmt.Printf("Unsupported format: %s\n", format)
	}
}

// ============ 使用示例 ============

func AdapterDemo() {
	manager := NewMediaPlayerManager()
	
	// 注册适配后的播放器
	manager.RegisterPlayer("mp3", NewMP3Adapter())
	manager.RegisterPlayer("wav", NewMP3Adapter())
	manager.RegisterPlayer("flac", NewAdvancedPlayerAdapter())
	
	fmt.Println("=== Playing MP3 ===")
	manager.Play("mp3", "song.mp3")
	
	fmt.Println("\n=== Playing FLAC ===")
	manager.Play("flac", "music.flac")
}
```

#### 2.1.5 Java语言实现

```java
package com.designpatterns.adapter;

// ============ 目标接口 ============

/**
 * 播放器接口（客户端期望的接口）
 */
public interface Player {
    String play(String filename);
    String pause();
    String stop();
}

// ============ 被适配者接口 ============

/**
 * 旧式媒体播放器
 */
public interface LegacyMediaPlayer {
    void openFile(String path);
    void startPlayback();
    void endPlayback();
}

// ============ 被适配者实现 ============

/**
 * 旧式MP3播放器
 */
public class OldMP3Player implements LegacyMediaPlayer {
    private String currentFile;
    
    @Override
    public void openFile(String path) {
        this.currentFile = path;
        System.out.println("Legacy: Opening file: " + path);
    }
    
    @Override
    public void startPlayback() {
        System.out.println("Legacy: Starting playback of " + currentFile);
    }
    
    @Override
    public void endPlayback() {
        System.out.println("Legacy: Ending playback");
    }
}

/**
 * 高级媒体播放器
 */
public interface AdvancedMediaPlayer {
    void loadMedia(String path) throws Exception;
    void play() throws Exception;
    void stop() throws Exception;
}

/**
 * 现代媒体播放器实现
 */
public class ModernMediaPlayer implements AdvancedMediaPlayer {
    private String currentMedia;
    
    @Override
    public void loadMedia(String path) throws Exception {
        this.currentMedia = path;
        System.out.println("Modern: Loading media: " + path);
    }
    
    @Override
    public void play() throws Exception {
        System.out.println("Modern: Playing " + currentMedia);
    }
    
    @Override
    public void stop() throws Exception {
        System.out.println("Modern: Stopping playback");
    }
}

// ============ 适配器实现 ============

/**
 * 适配器：将LegacyMediaPlayer适配为Player接口（对象适配器）
 */
public class MP3Adapter implements Player {
    private final LegacyMediaPlayer legacyPlayer;
    private String currentFile;
    
    public MP3Adapter(LegacyMediaPlayer legacyPlayer) {
        this.legacyPlayer = legacyPlayer;
    }
    
    @Override
    public String play(String filename) {
        this.currentFile = filename;
        legacyPlayer.openFile(filename);
        legacyPlayer.startPlayback();
        return "Playing: " + filename;
    }
    
    @Override
    public String pause() {
        legacyPlayer.endPlayback();
        return "Paused";
    }
    
    @Override
    public String stop() {
        legacyPlayer.endPlayback();
        this.currentFile = null;
        return "Stopped";
    }
}

/**
 * 适配器：将AdvancedMediaPlayer适配为Player接口
 */
public class AdvancedPlayerAdapter implements Player {
    private final AdvancedMediaPlayer advancedPlayer;
    private String currentFile;
    
    public AdvancedPlayerAdapter(AdvancedMediaPlayer

我来为您详细讲解Go和Java中最常用的15种设计模式。首先让我搜索一些最新的设计模式应用资料。



# Go与Java中最常用的15种设计模式详解

## 引言

设计模式是软件开发中经过验证的、可重用的解决方案，它们针对软件设计中反复出现的通用问题提供了标准化的解决思路。1994年，由Erich Gamma、Richard Helm、Ralph Johnson和John Vlissides四位学者合著的《设计模式：可复用面向对象软件的基础》一书，系统性地总结了23种经典设计模式，这四位作者因此被业界称为"四人帮"（Gang of Four，GoF）。在当今的Go和Java开发实践中，设计模式依然是提升代码质量、可维护性和扩展性的重要工具。

本文将从创建型、结构型和行为型三个类别，详细讲解15种在Go和Java项目中最常用的设计模式。每种模式将从常见问题、解决方案、代码实现、适用场景以及优缺点等维度进行深入分析，帮助开发者更好地理解和应用这些设计模式。

## 一、创建型设计模式

创建型设计模式专注于对象的创建机制，它们将对象的创建与使用进行分离，从而提高系统的灵活性和可维护性。

### 1.1 单例模式（Singleton Pattern）

#### 常见问题

在软件开发过程中，有些资源在整个应用程序生命周期中只需要存在一个实例，比如数据库连接池、配置管理器、日志记录器、线程池等。如果多次创建这些对象，不仅会造成资源浪费，还可能导致数据不一致或状态混乱的问题。例如，多个线程同时写入同一个日志文件时，如果各自持有独立的文件句柄，可能会导致日志内容交错或文件损坏。

#### 解决方案

单例模式确保一个类只有一个实例，并提供一个全局访问点来获取该实例。通过将类的构造函数私有化，禁止外部直接实例化对象，然后在类内部创建一个静态实例，并提供一个静态方法供外部获取这个唯一实例。

#### 设计模式的解决方案

```go
// Go语言实现 - 懒汉式（线程安全）
package singleton

import (
    "sync"
)

type Singleton struct {
    data string
}

var (
    instance *Singleton
    once     sync.Once
)

// GetInstance 返回单例实例
func GetInstance() *Singleton {
    once.Do(func() {
        instance = &Singleton{data: "Hello, Singleton!"}
    })
    return instance
}
```

```java
// Java语言实现 - 双重检查锁定（线程安全）
public class Singleton {
    private static volatile Singleton instance;
    private String data;
    
    // 私有构造函数
    private Singleton() {
        this.data = "Hello, Singleton!";
    }
    
    // 双重检查锁定
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

#### 适用场景

单例模式适用于以下场景：需要控制资源访问的场景，如数据库连接池；需要共享配置信息的场景，如应用程序配置管理器；需要统一日志输出的场景，如日志记录器；以及需要管理全局状态的场景，如游戏中的分数管理器。

#### 优缺点分析

单例模式的主要优点包括：确保只有一个实例，避免了资源的重复创建和浪费；提供了全局访问点，便于对共享资源进行统一管理；在需要延迟初始化时，可以实现懒加载以提高性能。其主要缺点包括：单例模式增加了类的耦合度，对象的职责过重；单例模式难以进行单元测试，因为私有构造函数限制了mock的可能性；在多线程环境下需要正确实现，否则可能导致多个实例被创建。

### 1.2 工厂方法模式（Factory Method Pattern）

#### 常见问题

在没有工厂方法模式的情况下，客户端代码通常直接依赖具体的类来创建对象。这种做法会导致代码紧耦合，当需要更换产品类型或添加新产品时，必须修改客户端代码。例如，一个图形编辑器需要创建不同类型的形状，如果直接在代码中使用new关键字创建圆形、矩形等对象，那么每次添加新形状或修改现有形状的创建逻辑时，都需要修改客户端代码。

#### 解决方案

工厂方法模式定义一个创建对象的接口，但让实现这个接口的子类决定实例化哪个类。工厂方法让类的实例化过程延迟到子类中进行，从而将对象的创建与使用分离。

#### 设计模式的解决方案

```go
// Go语言实现
package factory

import "fmt"

// Product 接口
type Product interface {
    Use() string
}

// ConcreteProductA
type ConcreteProductA struct{}

func (p *ConcreteProductA) Use() string {
    return "Using Product A"
}

// ConcreteProductB
type ConcreteProductB struct{}

func (p *ConcreteProductB) Use() string {
    return "Using Product B"
}

// Factory 接口
type Factory interface {
    CreateProduct() Product
}

// ConcreteFactoryA
type ConcreteFactoryA struct{}

func (f *ConcreteFactoryA) CreateProduct() Product {
    return &ConcreteProductA{}
}

// ConcreteFactoryB
type ConcreteFactoryB struct{}

func (f *ConcreteFactoryB) CreateProduct() Product {
    return &ConcreteProductB{}
}
```

```java
// Java语言实现
public interface Product {
    String use();
}

public class ConcreteProductA implements Product {
    @Override
    public String use() {
        return "Using Product A";
    }
}

public class ConcreteProductB implements Product {
    @Override
    public String use() {
        return "Using Product B";
    }
}

public interface Factory {
    Product createProduct();
}

public class ConcreteFactoryA implements Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProductA();
    }
}

public class ConcreteFactoryB implements Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProductB();
    }
}
```

#### 适用场景

工厂方法模式适用于以下场景：当类无法预知它必须创建的对象的类时，比如框架需要根据配置创建不同的数据库驱动；当类希望由子类来指定它所创建的对象时，比如日志系统可以根据配置创建文件日志或数据库日志；当需要将对象的创建职责委托给多个辅助类中的一个，并且希望将哪个辅助类是主类的信息局部化时。

#### 优缺点分析

工厂方法模式的主要优点包括：实现了对象创建与使用的分离，降低了系统的耦合度；符合开闭原则，可以在不修改客户端代码的情况下添加新的产品类型；代码结构清晰，易于理解和维护。其主要缺点包括：类的数量增加，每个产品都需要对应的工厂类，增加了系统的复杂度；对于简单对象来说，使用工厂方法模式可能会增加代码的复杂性。

### 1.3 抽象工厂模式（Abstract Factory Pattern）

#### 常见问题

在软件系统中，往往需要创建一系列相互关联的对象，而这些对象属于不同的等级结构。例如，一个跨平台的UI组件库需要同时提供Windows风格和Mac风格的按钮、文本框、菜单等组件。如果使用工厂方法模式，需要为每种组件都创建对应的工厂，这会导致工厂类数量激增，并且无法保证同一产品族的对象被一起使用。

#### 解决方案

抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。它强调的是“产品族”的概念，即属于同一产品族的产品必须一起使用。

#### 设计模式的解决方案

```go
// Go语言实现
package abstractfactory

// 按钮接口
type Button interface {
    Paint() string
}

// 文本框接口
type TextBox interface {
    Render() string
}

// Windows风格按钮
type WindowsButton struct{}

func (b *WindowsButton) Paint() string {
    return "Rendering Windows-style Button"
}

// Mac风格按钮
type MacButton struct{}

func (b *MacButton) Paint() string {
    return "Rendering Mac-style Button"
}

// Windows风格文本框
type WindowsTextBox struct{}

func (t *WindowsTextBox) Render() string {
    return "Rendering Windows-style TextBox"
}

// Mac风格文本框
type MacTextBox struct{}

func (t *MacTextBox) Render() string {
    return "Rendering Mac-style TextBox"
}

// 抽象工厂接口
type GUIFactory interface {
    CreateButton() Button
    CreateTextBox() TextBox
}

// Windows工厂
type WindowsFactory struct{}

func (f *WindowsFactory) CreateButton() Button {
    return &WindowsButton{}
}

func (f *WindowsFactory) CreateTextBox() TextBox {
    return &WindowsTextBox{}
}

// Mac工厂
type MacFactory struct{}

func (f *MacFactory) CreateButton() Button {
    return &MacButton{}
}

func (f *MacFactory) CreateTextBox() TextBox {
    return &MacTextBox{}
}
```

```java
// Java语言实现
public interface Button {
    String paint();
}

public interface TextBox {
    String render();
}

public class WindowsButton implements Button {
    @Override
    public String paint() {
        return "Rendering Windows-style Button";
    }
}

public class MacButton implements Button {
    @Override
    public String paint() {
        return "Rendering Mac-style Button";
    }
}

public class WindowsTextBox implements TextBox {
    @Override
    public String render() {
        return "Rendering Windows-style TextBox";
    }
}

public class MacTextBox implements TextBox {
    @Override
    public String render() {
        return "Rendering Mac-style TextBox";
    }
}

public interface GUIFactory {
    Button createButton();
    TextBox createTextBox();
}

public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public TextBox createTextBox() {
        return new WindowsTextBox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public TextBox createTextBox() {
        return new MacTextBox();
    }
}
```

#### 适用场景

抽象工厂模式适用于以下场景：当系统需要独立于它的产品创建方式时；当系统需要由多个产品系列中的一个来配置时；当需要强调一系列相关产品对象的设计以便进行联合使用时，比如UI主题系统。

#### 优缺点分析

抽象工厂模式的主要优点包括：确保同一产品族的对象能够一起使用，保证了产品之间的兼容性；将产品的实现与使用分离，客户端只依赖抽象接口；易于交换产品系列，因为具体工厂只有在初始化时出现一次。其主要缺点包括：添加新的产品等级结构需要修改抽象工厂接口以及所有的具体工厂类，违背了开闭原则；增加了系统的抽象程度和理解难度。

### 1.4 建造者模式（Builder Pattern）

#### 常见问题

在软件开发中，有些对象的构造过程非常复杂，包含多个可选参数和必选参数。以创建一个用户对象为例，可能需要设置用户名、密码、邮箱、手机号、地址、头像等多个属性。使用传统的构造函数或多个重载构造函数会导致参数列表过长、参数顺序难以记忆、代码可读性差等问题。例如，创建一个包含20个可选参数的配置对象，使用构造函数需要记忆每个参数的位置和含义。

#### 解决方案

建造者模式将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。它通过一步一步地构建一个复杂对象，允许用户通过指定复杂对象的类型和内容来构建它。

#### 设计模式的解决方案

```go
// Go语言实现
package builder

import "fmt"

// Product - 要构建的复杂对象
type Computer struct {
    CPU       string
    RAM       string
    Storage   string
    GPU       string
    OS        string
}

func (c *Computer) Display() string {
    return fmt.Sprintf("Computer: CPU=%s, RAM=%s, Storage=%s, GPU=%s, OS=%s",
        c.CPU, c.RAM, c.Storage, c.GPU, c.OS)
}

// Builder 接口
type Builder interface {
    SetCPU(cpu string) Builder
    SetRAM(ram string) Builder
    SetStorage(storage string) Builder
    SetGPU(gpu string) Builder
    SetOS(os string) Builder
    GetComputer() *Computer
}

// ConcreteBuilder
type ComputerBuilder struct {
    computer *Computer
}

func NewComputerBuilder() *ComputerBuilder {
    return &ComputerBuilder{&Computer{}}
}

func (b *ComputerBuilder) SetCPU(cpu string) Builder {
    b.computer.CPU = cpu
    return b
}

func (b *ComputerBuilder) SetRAM(ram string) Builder {
    b.computer.RAM = ram
    return b
}

func (b *ComputerBuilder) SetStorage(storage string) Builder {
    b.computer.Storage = storage
    return b
}

func (b *ComputerBuilder) SetGPU(gpu string) Builder {
    b.computer.GPU = gpu
    return b
}

func (b *ComputerBuilder) SetOS(os string) Builder {
    b.computer.OS = os
    return b
}

func (b *ComputerBuilder) GetComputer() *Computer {
    return b.computer
}

// Director - 指导构建过程
type Director struct {
    builder Builder
}

func NewDirector(builder Builder) *Director {
    return &Director{builder: builder}
}

func (d *Director) BuildGamingComputer() *Computer {
    return d.builder.
        SetCPU("Intel i9").
        SetRAM("64GB").
        SetStorage("2TB SSD").
        SetGPU("RTX 4090").
        SetOS("Windows 11").
        GetComputer()
}
```

```java
// Java语言实现
public class Computer {
    private String cpu;
    private String ram;
    private String storage;
    private String gpu;
    private String os;
    
    public void setCpu(String cpu) { this.cpu = cpu; }
    public void setRam(String ram) { this.ram = ram; }
    public void setStorage(String storage) { this.storage = storage; }
    public void setGpu(String gpu) { this.gpu = gpu; }
    public void setOs(String os) { this.os = os; }
    
    @Override
    public String toString() {
        return "Computer{cpu='" + cpu + "', ram='" + ram + 
               "', storage='" + storage + "', gpu='" + gpu + "', os='" + os + "'}";
    }
}

public interface ComputerBuilder {
    ComputerBuilder setCpu(String cpu);
    ComputerBuilder setRam(String ram);
    ComputerBuilder setStorage(String storage);
    ComputerBuilder setGpu(String gpu);
    ComputerBuilder setOs(String os);
    Computer build();
}

public class ConcreteComputerBuilder implements ComputerBuilder {
    private Computer computer = new Computer();
    
    @Override
    public ComputerBuilder setCpu(String cpu) {
        computer.setCpu(cpu);
        return this;
    }
    
    @Override
    public ComputerBuilder setRam(String ram) {
        computer.setRam(ram);
        return this;
    }
    
    @Override
    public ComputerBuilder setStorage(String storage) {
        computer.setStorage(storage);
        return this;
    }
    
    @Override
    public ComputerBuilder setGpu(String gpu) {
        computer.setGpu(gpu);
        return this;
    }
    
    @Override
    public ComputerBuilder setOs(String os) {
        computer.setOs(os);
        return this;
    }
    
    @Override
    public Computer build() {
        return computer;
    }
}

public class Director {
    private ComputerBuilder builder;
    
    public Director(ComputerBuilder builder) {
        this.builder = builder;
    }
    
    public Computer buildGamingComputer() {
        return builder
            .setCpu("Intel i9")
            .setRam("64GB")
            .setStorage("2TB SSD")
            .setGpu("RTX 4090")
            .setOs("Windows 11")
            .build();
    }
}
```

#### 适用场景

建造者模式适用于以下场景：当创建复杂对象的算法应该独立于该对象的组成部分时；当构造过程必须允许被构造的对象有不同表示时，比如SQL查询构建器；当需要创建由多个部分组成的产品，并且需要逐步构建它时。

#### 优缺点分析

建造者模式的主要优点包括：代码清晰易读，使用链式调用使构造过程直观；可以分步构建对象，控制构建顺序；可以将构建逻辑封装在Builder中，客户端不需要知道产品内部的细节；支持更精细地控制对象创建过程。其主要缺点包括：增加了类的数量，Builder和Director类的引入会增加系统的复杂度；对于产品变化较大时，需要调整Builder的接口。

### 1.5 原型模式（Prototype Pattern）

#### 常见问题

在软件开发中，有时候需要创建大量相似的对象，这些对象之间只有少数属性不同。如果每次都通过new关键字创建对象并设置属性，会导致大量重复代码。而且，某些对象的创建过程可能非常耗时（比如涉及数据库查询或网络请求），频繁创建会影响系统性能。

#### 解决方案

原型模式通过复制（克隆）现有对象来创建新对象，而不是通过new关键字实例化。这种模式允许动态地添加或删除原型，并且比直接实例化对象更高效。

#### 设计模式的解决方案

```go
// Go语言实现
package prototype

import "fmt"

// Cloneable 接口
type Cloneable interface {
    Clone() Cloneable
}

// ConcretePrototype
type User struct {
    Name     string
    Age      int
    Email    string
    Profile  map[string]string
}

func (u *User) Clone() Cloneable {
    // 深拷贝
    newUser := &User{
        Name:    u.Name,
        Age:     u.Age,
        Email:   u.Email,
        Profile: make(map[string]string),
    }
    for k, v := range u.Profile {
        newUser.Profile[k] = v
    }
    return newUser
}

func (u *User) Display() {
    fmt.Printf("User{Name: %s, Age: %d, Email: %s, Profile: %v}\n",
        u.Name, u.Age, u.Email, u.Profile)
}
```

```java
// Java语言实现
public class User implements Cloneable {
    private String name;
    private int age;
    private String email;
    private Map<String, String> profile;
    
    public User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
        this.profile = new HashMap<>();
    }
    
    @Override
    protected User clone() {
        try {
            User cloned = (User) super.clone();
            // 深拷贝Map
            cloned.profile = new HashMap<>(this.profile);
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
    
    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public Map<String, String> getProfile() { return profile; }
    public void setProfile(Map<String, String> profile) { this.profile = profile; }
}
```

#### 适用场景

原型模式适用于以下场景：当一个系统应该独立于它的产品创建方式时；当需要创建一组对象，而它们的状态几乎相同，只有少量属性不同时，比如游戏中的怪物或道具；当创建对象的过程非常耗时或复杂，而复制现有对象更高效时。

#### 优缺点分析

原型模式的主要优点包括：性能提升，通过克隆比重新创建对象更高效；简化对象创建过程，隐藏具体创建细节；可以动态添加原型类型，通过注册表管理原型。其主要缺点包括：需要为每个类实现Clone方法，对于复杂对象需要处理深拷贝和浅拷贝的问题；如果对象的构造函数有副作用，克隆可能会引入问题；过度使用克隆可能导致对象关系混乱。

## 二、结构型设计模式

结构型设计模式关注类和对象的组合，它们通过继承和组合来形成更大的结构。

### 2.1 适配器模式（Adapter Pattern）

#### 常见问题

在软件开发中，经常会遇到接口不兼容的问题。例如，系统需要对接一个第三方支付接口，但现有的代码已经有一套统一的订单处理接口规范，而第三方接口的响应格式和方法签名与现有系统不一致。这时，直接修改现有代码来适配第三方接口会破坏已有的设计，而重新开发又会浪费资源。

#### 解决方案

适配器模式将一个类的接口转换成客户端期望的另一个接口，使得原本接口不兼容的类可以一起工作。适配器分为类适配器和对象适配器两种类型，前者通过继承实现，后者通过组合实现。

#### 设计模式的解决方案

```go
// Go语言实现 - 对象适配器
package adapter

// 已存在的接口
type Payment interface {
    Pay(amount float64) error
}

// 已存在的支付实现
type LegacyPayment struct{}

func (p *LegacyPayment) Pay(amount float64) error {
    fmt.Printf("Legacy payment: %.2f\n", amount)
    return nil
}

// 第三方支付接口
type ThirdPartyPayment struct {
    APIKey string
}

func (t *ThirdPartyPayment) MakePayment(amount int, currency string) bool {
    fmt.Printf("Third-party payment: %d %s\n", amount, currency)
    return true
}

// 适配器
type PaymentAdapter struct {
    thirdParty *ThirdPartyPayment
}

func NewPaymentAdapter() *PaymentAdapter {
    return &PaymentAdapter{
        thirdParty: &ThirdPartyPayment{APIKey: "api-key-123"},
    }
}

func (a *PaymentAdapter) Pay(amount float64) error {
    // 将金额从元转换为分
    amountInCents := int(amount * 100)
    a.thirdParty.MakePayment(amountInCents, "CNY")
    return nil
}
```

```java
// Java语言实现 - 对象适配器
public interface Payment {
    void pay(double amount);
}

public class LegacyPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.printf("Legacy payment: %.2f%n", amount);
    }
}

public class ThirdPartyPayment {
    private String apiKey;
    
    public ThirdPartyPayment(String apiKey) {
        this.apiKey = apiKey;
    }
    
    public boolean makePayment(int amountInCents, String currency) {
        System.out.printf("Third-party payment: %d %s%n", amountInCents, currency);
        return true;
    }
}

public class PaymentAdapter implements Payment {
    private ThirdPartyPayment thirdPartyPayment;
    
    public PaymentAdapter() {
        this.thirdPartyPayment = new ThirdPartyPayment("api-key-123");
    }
    
    @Override
    public void pay(double amount) {
        int amountInCents = (int) (amount * 100);
        thirdPartyPayment.makePayment(amountInCents, "CNY");
    }
}
```

#### 适用场景

适配器模式适用于以下场景：当需要使用一个已存在的类，但它的接口不符合需求时；当需要复用一些存在的类，但无法修改它们的源代码时；当需要统一多个子类的接口时，通常与策略模式结合使用。

#### 优缺点分析

适配器模式的主要优点包括：提高了类的复用性，无需修改现有代码即可使用新类；实现了客户端与被适配者的解耦；符合开闭原则，可以在不修改现有代码的情况下添加新的适配器。其主要缺点包括：增加了代码复杂度，需要新增适配器类；对于某些语言，可能需要处理异常转换等问题；过度使用适配器可能会导致系统变得混乱。

### 2.2 装饰器模式（Decorator Pattern）

#### 常见问题

在软件开发中，经常需要为对象动态添加额外功能。传统的方式是使用继承，但继承会导致类数量急剧膨胀，而且功能是在编译时静态确定的，无法在运行时动态添加。例如，一个咖啡店系统需要支持多种咖啡类型和配料组合，如果使用继承，每种组合都需要一个子类，会导致类爆炸。

#### 解决方案

装饰器模式动态地给对象添加一些额外的职责，就增加功能来说，装饰器模式比继承更加灵活。它通过创建一个装饰类，将原始对象包装在装饰类中，在保持原始对象接口不变的同时，添加新的行为。

#### 设计模式的解决方案

```go
// Go语言实现
package decorator

import "fmt"

// Component 接口
type Coffee interface {
    GetCost() float64
    GetDescription() string
}

// ConcreteComponent
type SimpleCoffee struct{}

func (c *SimpleCoffee) GetCost() float64 {
    return 10.0
}

func (c *SimpleCoffee) GetDescription() string {
    return "Simple Coffee"
}

// Decorator 基类
type CoffeeDecorator struct {
    coffee Coffee
}

func (d *CoffeeDecorator) GetCost() float64 {
    return d.coffee.GetCost()
}

func (d *CoffeeDecorator) GetDescription() string {
    return d.coffee.GetDescription()
}

// 具体装饰器 - 牛奶
type MilkDecorator struct {
    *CoffeeDecorator
}

func NewMilkDecorator(coffee Coffee) *MilkDecorator {
    return &MilkDecorator{&CoffeeDecorator{coffee: coffee}}
}

func (d *MilkDecorator) GetCost() float64 {
    return d.coffee.GetCost() + 2.0
}

func (d *MilkDecorator) GetDescription() string {
    return d.coffee.GetDescription() + ", Milk"
}

// 具体装饰器 - 糖
type SugarDecorator struct {
    *CoffeeDecorator
}

func NewSugarDecorator(coffee Coffee) *SugarDecorator {
    return &SugarDecorator{&CoffeeDecorator{coffee: coffee}}
}

func (d *SugarDecorator) GetCost() float64 {
    return d.coffee.GetCost() + 1.0
}

func (d *SugarDecorator) GetDescription() string {
    return d.coffee.GetDescription() + ", Sugar"
}
```

```java
// Java语言实现
public interface Coffee {
    double getCost();
    String getDescription();
}

public class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 10.0;
    }
    
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
}

public abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
}

public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 2.0;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", Milk";
    }
}

public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 1.0;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", Sugar";
    }
}
```

#### 适用场景

装饰器模式适用于以下场景：当需要动态地给对象添加功能，并且这些功能可以随时撤销时；当使用继承无法满足需求，而需要扩展类的功能时，比如Java I/O流的设计；当需要为一批兄弟类添加通用的功能时。

#### 优缺点分析

装饰器模式的主要优点包括：可以在运行时动态地添加或删除对象的功能；可以将单一职责的类组合成更复杂的功能；符合开闭原则，可以对现有类进行装饰而无需修改源代码。其主要缺点包括：多层装饰会增加系统的复杂性；如果装饰顺序不当，可能会导致意外的行为；装饰类和被装饰类实现相同的接口，可能会让代码看起来不够直观。

### 2.3 代理模式（Proxy Pattern）

#### 常见问题

在软件开发中，有时候不能或不想直接访问某个对象，需要通过一个中介来处理。比如，访问远程服务时，每次请求都需要处理网络连接、序列化、反序列化等复杂操作；如果创建大对象（如高分辨率图片）的成本很高，可能希望在真正需要时才加载；需要对某些操作进行访问控制，只有满足特定条件的请求才能到达目标对象。

#### 解决方案

代理模式为其他对象提供一种代理以控制对这个对象的访问。代理对象在客户端和目标对象之间起到中介作用，可以在不改变目标对象行为的前提下，对请求进行处理或增强。

#### 设计模式的解决方案

```go
// Go语言实现
package proxy

import "fmt"

// Subject 接口
type Image interface {
    Display()
}

// RealSubject - 真实对象
type RealImage struct {
    filename string
}

func NewRealImage(filename string) *RealImage {
    return &RealImage{filename: filename}
}

func (img *RealImage) Display() {
    fmt.Printf("Displaying image: %s\n", img.filename)
}

func (img *RealImage) loadFromDisk() {
    fmt.Printf("Loading image from disk: %s\n", img.filename)
}

// Proxy - 代理对象
type ImageProxy struct {
    realImage *RealImage
    filename  string
}

func NewImageProxy(filename string) *ImageProxy {
    return &ImageProxy{filename: filename}
}

func (p *ImageProxy) Display() {
    // 懒加载：只在需要时才创建真实对象
    if p.realImage == nil {
        p.realImage = NewRealImage(p.filename)
    }
    p.realImage.Display()
}
```

```java
// Java语言实现
public interface Image {
    void display();
}

public class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

public class ImageProxy implements Image {
    private RealImage realImage;
    private String filename;
    
    public ImageProxy(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void display() {
        // 懒加载
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}
```

#### 适用场景

代理模式适用于以下场景：远程代理，为远程对象提供本地代表，如Web服务的本地代理；虚代理，按需创建开销很大的对象，如图片懒加载；保护代理，控制对原始对象的访问权限；智能引用，访问对象时执行额外操作，如记录访问日志。

#### 优缺点分析

代理模式的主要优点包括：可以在不修改目标对象的情况下添加新功能；可以实现懒加载，提高系统性能；可以对访问进行控制，增强系统安全性；职责清晰，客户端不需要知道具体的实现细节。其主要缺点包括：增加了中间层，可能会影响系统性能；需要为每个真实对象创建代理类，增加代码量；某些类型的代理模式可能会使请求变慢。

### 2.4 外观模式（Facade Pattern）

#### 常见问题

在软件开发中，系统通常由多个子系统组成，每个子系统都有自己的接口和复杂性。客户端如果直接与各个子系统交互，需要了解每个子系统的接口和调用顺序，这会导致客户端代码与子系统高度耦合。当子系统发生变化时，客户端代码也需要相应修改，维护成本很高。

#### 解决方案

外观模式为子系统中的一组接口提供一个一致的界面，定义一个高层接口，使得子系统更容易使用。它隐藏了系统的复杂性，为复杂的子系统提供了一个简化的接口。

#### 设计模式的解决方案

```go
// Go语言实现
package facade

import "fmt"

// 子系统1 - 家庭影院灯光
type Light struct{}

func (l *Light) On() {
    fmt.Println("Lights are ON")
}

func (l *Light) Off() {
    fmt.Println("Lights are OFF")
}

func (l *Light) Dim(level int) {
    fmt.Printf("Lights dimmed to %d%%\n", level)
}

// 子系统2 - 音响系统
type Speaker struct{}

func (s *Speaker) On() {
    fmt.Println("Speaker is ON")
}

func (s *Speaker) Off() {
    fmt.Println("Speaker is OFF")
}

func (s *Speaker) SetVolume(level int) {
    fmt.Printf("Speaker volume set to %d\n", level)
}

// 子系统3 - 投影仪
type Projector struct{}

func (p *Projector) On() {
    fmt.Println("Projector is ON")
}

func (p *Projector) Off() {
    fmt.Println("Projector is OFF")
}

func (p *Projector) SetInput(input string) {
    fmt.Printf("Projector input set to %s\n", input)
}

// 外观类 - 家庭影院控制器
type HomeTheaterFacade struct {
    light     *Light
    speaker   *Speaker
    projector *Projector
}

func NewHomeTheaterFacade() *HomeTheaterFacade {
    return &HomeTheaterFacade{
        light:     &Light{},
        speaker:   &Speaker{},
        projector: &Projector{},
    }
}

func (h *HomeTheaterFacade) WatchMovie() {
    fmt.Println("\n--- Starting Movie Mode ---")
    h.light.Dim(20)
    h.projector.On()
    h.projector.SetInput("HDMI")
    h.speaker.On()
    h.speaker.SetVolume(30)
    fmt.Println("--- Ready to Watch! ---\n")
}

func (h *HomeTheaterFacade) EndMovie() {
    fmt.Println("\n--- Ending Movie Mode ---")
    h.speaker.Off()
    h.projector.Off()
    h.light.On()
    fmt.Println("--- Movie Ended! ---\n")
}
```

```java
// Java语言实现
public class Light {
    public void on() {
        System.out.println("Lights are ON");
    }
    
    public void off() {
        System.out.println("Lights are OFF");
    }
    
    public void dim(int level) {
        System.out.printf("Lights dimmed to %d%%%n", level);
    }
}

public class Speaker {
    public void on() {
        System.out.println("Speaker is ON");
    }
    
    public void off() {
        System.out.println("Speaker is OFF");
    }
    
    public void setVolume(int level) {
        System.out.printf("Speaker volume set to %d%n", level);
    }
}

public class Projector {
    public void on() {
        System.out.println("Projector is ON");
    }
    
    public void off() {
        System.out.println("Projector is OFF");
    }
    
    public void setInput(String input) {
        System.out.printf("Projector input set to %s%n", input);
    }
}

public class HomeTheaterFacade {
    private Light light;
    private Speaker speaker;
    private Projector projector;
    
    public HomeTheaterFacade() {
        this.light = new Light();
        this.speaker = new Speaker();
        this.projector = new Projector();
    }
    
    public void watchMovie() {
        System.out.println("\n--- Starting Movie Mode ---");
        light.dim(20);
        projector.on();
        projector.setInput("HDMI");
        speaker.on();
        speaker.setVolume(30);
        System.out.println("--- Ready to Watch! ---\n");
    }
    
    public void endMovie() {
        System.out.println("\n--- Ending Movie Mode ---");
        speaker.off();
        projector.off();
        light.on();
        System.out.println("--- Movie Ended! ---\n");
    }
}
```

#### 适用场景

外观模式适用于以下场景：当需要为复杂的子系统提供一个简单接口时；当客户端与抽象的实现类之间存在很多依赖时；当需要构建一个层次结构的子系统时。

#### 优缺点分析

外观模式的主要优点包括：简化了客户端代码，无需了解子系统的复杂性；降低了子系统与客户端之间的耦合度；提高了子系统的独立性和可移植性；符合迪米特法则，只与外观类交互。其主要缺点包括：外观类可能成为与所有代码都耦合的"上帝类"；如果需要提供更详细的功能，可能需要绕过外观类直接访问子系统。

### 2.5 组合模式（Composite Pattern）

#### 常见问题

在软件开发中，经常需要处理树形结构的数据。比如，一个组织架构系统包含部门和员工，部门可以包含子部门和员工；一个文件系统包含文件夹和文件，文件夹可以包含子文件夹和文件；一个图形编辑器的图形对象可以包含简单形状和由多个形状组成的复杂图形。如果对每个类型都单独处理，代码会变得复杂且难以维护。

#### 解决方案

组合模式将对象组合成树形结构以表示"部分-整体"的层次结构，使得用户对单个对象和组合对象的使用具有一致性。它让客户端可以统一处理简单元素和复杂元素。

#### 设计模式的解决方案

```go
// Go语言实现
package composite

import "fmt"

// Component 接口
type FileComponent interface {
    GetName() string
    GetSize() int64
    Print(prefix string)
}

// File - 叶子节点
type File struct {
    name string
    size int64
}

func NewFile(name string, size int64) *File {
    return &File{name: name, size: size}
}

func (f *File) GetName() string {
    return f.name
}

func (f *File) GetSize() int64 {
    return f.size
}

func (f *File) Print(prefix string) {
    fmt.Printf("%s[FILE] %s (Size: %d bytes)\n", prefix, f.name, f.size)
}

// Folder - 组合节点
type Folder struct {
    name     string
    children []FileComponent
}

func NewFolder(name string) *Folder {
    return &Folder{name: name, children: make([]FileComponent, 0)}
}

func (f *Folder) Add(child FileComponent) {
    f.children = append(f.children, child)
}

func (f *Folder) Remove(child FileComponent) {
    for i, c := range f.children {
        if c.GetName() == child.GetName() {
            f.children = append(f.children[:i], f.children[i+1:]...)
            break
        }
    }
}

func (f *Folder) GetName() string {
    return f.name
}

func (f *Folder) GetSize() int64 {
    var total int64
    for _, child := range f.children {
        total += child.GetSize()
    }
    return total
}

func (f *Folder) Print(prefix string) {
    fmt.Printf("%s[FOLDER] %s (Total Size: %d bytes)\n", prefix, f.name, f.GetSize())
    newPrefix := prefix + "  "
    for _, child := range f.children {
        child.Print(newPrefix)
    }
}
```

```java
// Java语言实现
public interface FileComponent {
    String getName();
    long getSize();
    void print(String prefix);
}

public class File implements FileComponent {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        return size;
    }
    
    @Override
    public void print(String prefix) {
        System.out.printf("%s[FILE] %s (Size: %d bytes)%n", prefix, name, size);
    }
}

public class Folder implements FileComponent {
    private String name;
    private List<FileComponent> children = new ArrayList<>();
    
    public Folder(String name) {
        this.name = name;
    }
    
    public void add(FileComponent component) {
        children.add(component);
    }
    
    public void remove(FileComponent component) {
        children.remove(component);
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public long getSize() {
        return children.stream().mapToLong(FileComponent::getSize).sum();
    }
    
    @Override
    public void print(String prefix) {
        System.out.printf("%s[FOLDER] %s (Total Size: %d bytes)%n", prefix, name, getSize());
        String newPrefix = prefix + "  ";
        for (FileComponent child : children) {
            child.print(newPrefix);
        }
    }
}
```

#### 适用场景

组合模式适用于以下场景：当需要表示对象的部分-整体层次结构时，如文件系统和组织架构；当希望客户端忽略组合对象与单个对象的差异时；当结构可以递归地组合时。

#### 优缺点分析

组合模式的主要优点包括：定义了包含基本对象和组合对象的类层次结构；简化了客户端代码，可以统一处理简单对象和组合对象；符合开闭原则，可以添加新的组件类型。其主要缺点包括：可能会限制类型约束，在某些情况下可能不希望所有组件都是同质的；设计可能变得复杂，因为链接和职责边界不够清晰。

## 三、行为型设计模式

行为型设计模式关注对象之间的通信和职责分配，它们描述了对象之间的交互模式。

### 3.1 观察者模式（Observer Pattern）

#### 常见问题

在软件开发中，经常需要维护对象之间的一致性关系。例如，股票价格变化时，需要通知所有订阅该股票的投资者；温度传感器数据变化时，需要更新所有依赖该数据的显示设备；当一个对象的状态发生改变时，需要通知其他相关对象并更新它们的状态。如果对象之间直接耦合，当添加或删除观察者时，需要修改主题对象，导致代码难以维护。

#### 解决方案

观察者模式定义了对象之间的一对多依赖关系，当一个对象状态发生改变时，所有依赖它的对象都会收到通知并自动更新。观察者和被观察者之间是松耦合的，被观察者不需要知道观察者的具体类型。

#### 设计模式的解决方案

```go
// Go语言实现
package observer

import "fmt"

// Subject 接口
type Subject interface {
    Attach(observer Observer)
    Detach(observer Observer)
    Notify()
}

// Observer 接口
type Observer interface {
    Update(message string)
}

// ConcreteSubject
type Stock struct {
    symbol    string
    price     float64
    observers []Observer
}

func NewStock(symbol string, price float64) *Stock {
    return &Stock{
        symbol:    symbol,
        price:     price,
        observers: make([]Observer, 0),
    }
}

func (s *Stock) Attach(observer Observer) {
    s.observers = append(s.observers, observer)
}

func (s *Stock) Detach(observer Observer) {
    for i, obs := range s.observers {
        if obs == observer {
            s.observers = append(s.observers[:i], s.observers[i+1:]...)
            break
        }
    }
}

func (s *Stock) Notify() {
    message := fmt.Sprintf("Stock %s price changed to %.2f", s.symbol, s.price)
    for _, observer := range s.observers {
        observer.Update(message)
    }
}

func (s *Stock) SetPrice(price float64) {
    s.price = price
    s.Notify()
}

// ConcreteObserver
type Investor struct {
    name string
}

func NewInvestor(name string) *Investor {
    return &Investor{name: name}
}

func (i *Investor) Update(message string) {
    fmt.Printf("Investor %s received: %s\n", i.name, message)
}
```

```java
// Java语言实现
import java.util.ArrayList;
import java.util.List;

public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

public interface Observer {
    void update(String message);
}

public class Stock implements Subject {
    private String symbol;
    private double price;
    private List<Observer> observers = new ArrayList<>();
    
    public Stock(String symbol, double price) {
        this.symbol = symbol;
        this.price = price;
    }
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        String message = String.format("Stock %s price changed to %.2f", symbol, price);
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
    
    public void setPrice(double price) {
        this.price = price;
        notifyObservers();
    }
    
    public String getSymbol() { return symbol; }
    public double getPrice() { return price; }
}

public class Investor implements Observer {
    private String name;
    
    public Investor(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.printf("Investor %s received: %s%n", name, message);
    }
}
```

#### 适用场景

观察者模式适用于以下场景：当一个对象的改变需要同时改变其他对象，而不知道具体有多少对象有待改变时；当一个对象必须通知其他对象，而它又无法假定其他对象是谁时，如事件驱动系统和消息推送系统。

#### 优缺点分析

观察者模式的主要优点包括：实现了对象之间的松耦合，被观察者和观察者可以独立变化；符合开闭原则，可以添加新的观察者而无需修改被观察者代码；可以建立一套触发机制，实现联动效果。其主要缺点包括：如果观察者数量很多，通知过程可能会影响性能；观察者和被观察者之间可能形成循环依赖；某些编程语言中需要手动处理内存泄漏问题。

### 3.2 策略模式（Strategy Pattern）

#### 常见问题

在软件开发中，算法或行为经常需要根据不同的场景进行切换。例如，支付系统需要支持多种支付方式（微信、支付宝、信用卡等）；排序算法需要根据数据量和特征选择不同的排序方法；如果在主类中使用大量的条件语句来处理不同的行为，会导致代码难以维护和扩展。

#### 解决方案

策略模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换。策略模式使算法可以独立于使用它的客户端而变化。

#### 设计模式的解决方案

```go
// Go语言实现
package strategy

import "fmt"

// PaymentStrategy 接口
type PaymentStrategy interface {
    Pay(amount float64) error
}

// 具体策略 - 微信支付
type WeChatPay struct{}

func (w *WeChatPay) Pay(amount float64) error {
    fmt.Printf("Paid %.2f using WeChat Pay\n", amount)
    return nil
}

// 具体策略 - 支付宝
type Alipay struct{}

func (a *Alipay) Pay(amount float64) error {
    fmt.Printf("Paid %.2f using Alipay\n", amount)
    return nil
}

// 具体策略 - 信用卡
type CreditCardPay struct {
    cardNumber string
}

func (c *CreditCardPay) Pay(amount float64) error {
    fmt.Printf("Paid %.2f using Credit Card %s\n", amount, c.cardNumber)
    return nil
}

// Context
type ShoppingCart struct {
    items    []string
    payment  PaymentStrategy
}

func NewShoppingCart() *ShoppingCart {
    return &ShoppingCart{
        items: make([]string, 0),
    }
}

func (s *ShoppingCart) AddItem(item string) {
    s.items = append(s.items, item)
}

func (s *ShoppingCart) SetPayment(payment PaymentStrategy) {
    s.payment = payment
}

func (s *ShoppingCart) Checkout() error {
    total := float64(len(s.items)) * 10.0 // 假设每个商品10元
    if s.payment == nil {
        return fmt.Errorf("payment method not set")
    }
    return s.payment.Pay(total)
}
```

```java
// Java语言实现
public interface PaymentStrategy {
    void pay(double amount);
}

public class WeChatPay implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.printf("Paid %.2f using WeChat Pay%n", amount);
    }
}

public class Alipay implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.printf("Paid %.2f using Alipay%n", amount);
    }
}

public class CreditCardPay implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPay(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.printf("Paid %.2f using Credit Card %s%n", amount, cardNumber);
    }
}

public class ShoppingCart {
    private List<String> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void addItem(String item) {
        items.add(item);
    }
    
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }
    
    public void checkout() {
        double total = items.size() * 10.0;
        if (paymentStrategy == null) {
            throw new IllegalStateException("Payment method not set");
        }
        paymentStrategy.pay(total);
    }
}
```

#### 适用场景

策略模式适用于以下场景：当一个系统需要在多种算法中选择一种时；当一个类定义了多种行为，并且这些行为以多个条件语句的形式存在时；当需要在一个算法中动态地选择某种行为时。

#### 优缺点分析

策略模式的主要优点包括：提供了算法的替代选择，可以在运行时切换算法；将算法封装在独立的策略类中，可以独立于客户端进行测试；避免了使用大量的条件语句；符合开闭原则，可以添加新的策略而不修改现有代码。其主要缺点包括：客户端必须了解各种策略的不同之处，才能选择合适的策略；策略模式会增加系统中类的数量。

### 3.3 命令模式（Command Pattern）

#### 常见问题

在软件开发中，有时需要将请求发送者和请求接收者解耦。比如，一个菜单系统，用户点击菜单项时需要执行相应的操作，但菜单不知道具体要执行什么操作；如果需要支持撤销和重做功能，需要记录操作的执行历史；需要将操作排队、延迟执行或远程执行。

#### 解决方案

命令模式将请求封装成对象，从而允许使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

#### 设计模式的解决方案

```go
// Go语言实现
package command

import "fmt"

// Command 接口
type Command interface {
    Execute()
    Undo()
}

// Receiver
type Light struct {
    isOn bool
}

func (l *Light) On() {
    l.isOn = true
    fmt.Println("Light is ON")
}

func (l *Light) Off() {
    l.isOn = false
    fmt.Println("Light is OFF")
}

// ConcreteCommand - 开灯命令
type LightOnCommand struct {
    light *Light
}

func NewLightOnCommand(light *Light) *LightOnCommand {
    return &LightOnCommand{light: light}
}

func (c *LightOnCommand) Execute() {
    c.light.On()
}

func (c *LightOnCommand) Undo() {
    c.light.Off()
}

// ConcreteCommand - 关灯命令
type LightOffCommand struct {
    light *Light
}

func NewLightOffCommand(light *Light) *LightOffCommand {
    return &LightOffCommand{light: light}
}

func (c *LightOffCommand) Execute() {
    c.light.Off()
}

func (c *LightOffCommand) Undo() {
    c.light.On()
}

// Invoker
type RemoteControl struct {
    command Command
}

func NewRemoteControl() *RemoteControl {
    return &RemoteControl{}
}

func (r *RemoteControl) SetCommand(command Command) {
    r.command = command
}

func (r *RemoteControl) PressButton() {
    r.command.Execute()
}

func (r *RemoteControl) PressUndo() {
    r.command.Undo()
}
```

```java
// Java语言实现
public interface Command {
    void execute();
    void undo();
}

public class Light {
    private boolean on = false;
    
    public void on() {
        on = true;
        System.out.println("Light is ON");
    }
    
    public void off() {
        on = false;
        System.out.println("Light is OFF");
    }
    
    public boolean isOn() { return on; }
}

public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.on();
    }
    
    @Override
    public void undo() {
        light.off();
    }
}

public class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.off();
    }
    
    @Override
    public void undo() {
        light.on();
    }
}

public class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
    
    public void pressUndo() {
        command.undo();
    }
}
```

#### 适用场景

命令模式适用于以下场景：当需要通过操作来参数化对象时，如菜单项；当需要将操作放入队列、记录操作日志或支持撤销/重做时；当需要支持事务性操作或批处理时。

#### 优缺点分析

命令模式的主要优点包括：将请求封装成对象，可以参数化命令、队列执行和记录日志；支持撤销和重做操作；将调用操作的对象与知道如何执行操作的对象解耦；可以组合多个命令形成复合命令。其主要缺点包括：每个具体命令类都需要创建一个类，可能导致类数量增加；对于简单的命令操作，命令模式可能显得过于复杂。

### 3.4 模板方法模式（Template Method Pattern）

#### 常见问题

在软件开发中，经常遇到这样的情况：多个类实现相似的算法，但某些步骤有所不同。例如，数据导出功能可能支持导出为Excel、CSV、JSON等格式，但打开连接、执行查询、关闭连接的流程是相同的；如果为每种格式都编写完整的导出代码，会造成大量重复代码。

#### 解决方案

模板方法模式定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

#### 设计模式的解决方案

```go
// Go语言实现
package templatemethod

import "fmt"

// AbstractClass - 抽象模板
type DataExporter interface {
    // 模板方法
    Export() {
        fmt.Println("Starting export process...")
        connect()
        queryData()
        formatData()
        writeData()
        disconnect()
        fmt.Println("Export completed!")
    }
    
    connect()
    queryData()
    formatData()
    writeData()
    disconnect()
}

// ConcreteClass - Excel导出
type ExcelExporter struct{}

func (e *ExcelExporter) connect() {
    fmt.Println("Connecting to Excel file...")
}

func (e *ExcelExporter) queryData() {
    fmt.Println("Querying data for Excel...")
}

func (e *ExcelExporter) formatData() {
    fmt.Println("Formatting data as Excel...")
}

func (e *ExcelExporter) writeData() {
    fmt.Println("Writing data to Excel file...")
}

func (e *ExcelExporter) disconnect() {
    fmt.Println("Closing Excel file...")
}

// ConcreteClass - CSV导出
type CSVExporter struct{}

func (c *CSVExporter) connect() {
    fmt.Println("Opening CSV file...")
}

func (c *CSVExporter) queryData() {
    fmt.Println("Querying data for CSV...")
}

func (c *CSVExporter) formatData() {
    fmt.Println("Formatting data as CSV...")
}

func (c *CSVExporter) writeData() {
    fmt.Println("Writing data to CSV file...")
}

func (c *CSVExporter) disconnect() {
    fmt.Println("Closing CSV file...")
}
```

```java
// Java语言实现
public abstract class DataExporter {
    // 模板方法
    public final void export() {
        System.out.println("Starting export process...");
        connect();
        queryData();
        formatData();
        writeData();
        disconnect();
        System.out.println("Export completed!");
    }
    
    protected abstract void connect();
    protected abstract void queryData();
    protected abstract void formatData();
    protected abstract void writeData();
    protected abstract void disconnect();
}

public class ExcelExporter extends DataExporter {
    @Override
    protected void connect() {
        System.out.println("Connecting to Excel file...");
    }
    
    @Override
    protected void queryData() {
        System.out.println("Querying data for Excel...");
    }
    
    @Override
    protected void formatData() {
        System.out.println("Formatting data as Excel...");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing data to Excel file...");
    }
    
    @Override
    protected void disconnect() {
        System.out.println("Closing Excel file...");
    }
}

public class CSVExporter extends DataExporter {
    @Override
    protected void connect() {
        System.out.println("Opening CSV file...");
    }
    
    @Override
    protected void queryData() {
        System.out.println("Querying data for CSV...");
    }
    
    @Override
    protected void formatData() {
        System.out.println("Formatting data as CSV...");
    }
    
    @Override
    protected void writeData() {
        System.out.println("Writing data to CSV file...");
    }
    
    @Override
    protected void disconnect() {
        System.out.println("Closing CSV file...");
    }
}
```

#### 适用场景

模板方法模式适用于以下场景：当多个类的算法骨架相同，但某些步骤不同时；当需要控制子类的扩展点，只允许在特定的方法中进行扩展时；当需要将重复代码抽取到父类中时。

#### 优缺点分析

模板方法模式的主要优点包括：实现了代码复用，将公共行为放在父类中；封装了不变的部分，扩展了可变的部分；符合开闭原则，子类可以重写特定步骤而不改变算法结构。其主要缺点包括：子类可能会增加代码复杂度；父类的某些方法可能是抽象的，要求所有子类都必须实现；如果父类需要添加新的步骤，可能会影响所有子类。

### 3.5 状态模式（State Pattern）

#### 常见问题

在软件开发中，对象的行为取决于它的状态，并且它需要在运行时根据状态改变行为。例如，一个订单系统，订单有新建、处理中、已发货、已完成等状态，不同状态下可以执行的操作不同；如果使用大量的条件语句（如if-else或switch）来处理不同状态下的行为，代码会变得难以维护和扩展。

#### 解决方案

状态模式允许一个对象在其内部状态改变时改变它的行为，看起来好像对象修改了它的类。状态模式将状态相关的行为封装在独立的状态类中，并通过委托将请求发送给当前状态对象。

#### 设计模式的解决方案

```go
// Go语言实现
package state

import "fmt"

// State 接口
type State interface {
    DoAction(context *OrderContext)
    GetStatus() string
}

// Context
type OrderContext struct {
    state State
}

func NewOrderContext() *OrderContext {
    return &OrderContext{}
}

func (c *OrderContext) SetState(state State) {
    c.state = state
}

func (c *OrderContext) GetState() State {
    return c.state
}

func (c *OrderContext) DoAction() {
    c.state.DoAction(c)
}

// ConcreteState - 新建状态
type NewOrderState struct{}

func (s *NewOrderState) DoAction(context *OrderContext) {
    fmt.Println("Order is in NEW state. Awaiting payment...")
}

func (s *NewOrderState) GetStatus() string {
    return "NEW"
}

// ConcreteState - 处理中状态
type ProcessingState struct{}

func (s *ProcessingState) DoAction(context *OrderContext) {
    fmt.Println("Order is being PROCESSED...")
}

func (s *ProcessingState) GetStatus() string {
    return "PROCESSING"
}

// ConcreteState - 已发货状态
type ShippedState struct{}

func (s *ShippedState) DoAction(context *OrderContext) {
    fmt.Println("Order has been SHIPPED. In transit...")
}

func (s *ShippedState) GetStatus() string {
    return "SHIPPED"
}

// ConcreteState - 已完成状态
type CompletedState struct{}

func (s *CompletedState) DoAction(context *OrderContext) {
    fmt.Println("Order is COMPLETED. Thank you!")
}

func (s *CompletedState) GetStatus() string {
    return "COMPLETED"
}
```

```java
// Java语言实现
public interface State {
    void doAction(OrderContext context);
    String getStatus();
}

public class OrderContext {
    private State state;
    
    public void setState(State state) {
        this.state = state;
    }
    
    public State getState() {
        return state;
    }
    
    public void doAction() {
        state.doAction(this);
    }
}

public class NewOrderState implements State {
    @Override
    public void doAction(OrderContext context) {
        System.out.println("Order is in NEW state. Awaiting payment...");
    }
    
    @Override
    public String getStatus() {
        return "NEW";
    }
}

public class ProcessingState implements State {
    @Override
    public void doAction(OrderContext context) {
        System.out.println("Order is being PROCESSED...");
    }
    
    @Override
    public String getStatus() {
        return "PROCESSING";
    }
}

public class ShippedState implements State {
    @Override
    public void doAction(OrderContext context) {
        System.out.println("Order has been SHIPPED. In transit...");
    }
    
    @Override
    public String getStatus() {
        return "SHIPPED";
    }
}

public class CompletedState implements State {
    @Override
    public void doAction(OrderContext context) {
        System.out.println("Order is COMPLETED. Thank you!");
    }
    
    @Override
    public String getStatus() {
        return "COMPLETED";
    }
}
```

#### 适用场景

状态模式适用于以下场景：当对象的行为取决于它的状态，并且必须在运行时根据状态改变行为时；当一个操作包含大量分支语句，且这些分支依赖于对象的状态时，如订单系统、工作流引擎。

#### 优缺点分析

状态模式的主要优点包括：封装了与状态相关的行为，使代码更加清晰；将所有与某个状态相关的代码集中在一个状态类中；使得状态转换更加明确，增加了程序的可读性；符合开闭原则，可以添加新的状态而不修改现有代码。其主要缺点包括：类的数量增加，每个状态都需要一个具体状态类；状态模式增加了系统的复杂度；如果状态之间的关系不清晰，可能会导致调试困难。

## 四、设计模式总结与应用建议

### 4.1 模式分类总览

通过对15种设计模式的详细分析，我们可以将它们按照用途进行分类总结：

**创建型模式（5种）**主要解决对象创建的问题：单例模式确保类只有一个实例；工厂方法模式将对象的创建延迟到子类；抽象工厂模式创建一系列相关对象；建造者模式逐步构建复杂对象；原型模式通过克隆创建对象。

**结构型模式（5种）**关注对象组合形成更大结构：适配器模式使不兼容的接口可以合作；装饰器模式动态添加功能；代理模式控制对象访问；外观模式提供统一接口；组合模式处理树形结构。

**行为型模式（5种）**关注对象间的通信：观察者模式实现一对多依赖；策略模式定义可互换算法；命令模式封装请求；模板方法模式定义算法骨架；状态模式根据状态改变行为。

### 4.2 实际应用建议

在实际项目中应用设计模式时，需要注意以下几点原则：

**适度使用原则**：设计模式是解决问题的工具，而不是目的。不要为了使用模式而使用模式，只有当真正遇到相应的设计问题时，才考虑使用合适的设计模式。

**关注代码质量**：使用设计模式时，应该关注代码的可读性、可维护性和可扩展性。选择能够清晰表达意图的模式，使代码结构更加清晰。

**考虑团队熟悉度**：如果团队成员对某种设计模式不熟悉，强行使用可能会适得其反。建议团队定期进行设计模式的学习和分享，提高整体的设计能力。

**结合项目特点**：不同的项目类型可能适合不同的设计模式。例如，互联网项目可能更关注性能和扩展性，而企业级项目可能更关注稳定性和可维护性。

### 4.3 Go与Java的差异

在Go和Java中实现设计模式时，需要注意两门语言特性和风格上的差异：

Go语言以其简洁性和并发支持著称。在Go中，通常使用接口来定义行为，而不是通过继承。Go的并发原语（goroutines和channels）使得某些模式（如观察者模式）的实现更加自然。Java作为传统的面向对象语言，更加注重类和接口的设计。Java的丰富类库和成熟的生态系统为设计模式的实现提供了更多支持。

无论使用哪种语言，设计模式的核心思想是相同的：将不变的部分封装起来，让变化的部分保持灵活。掌握这些经典的设计模式，将帮助开发者编写出更加优雅、健壮和可维护的代码。

## 结论

设计模式是软件工程领域的宝贵经验总结，它们提供了经过验证的解决方案来应对常见的设计挑战。本文详细介绍了15种在Go和Java项目开发中最常用的设计模式，包括创建型、结构型和行为型三个类别。每种模式都从常见问题、解决方案、代码实现、适用场景以及优缺点等维度进行了深入分析。

通过学习和实践这些设计模式，开发者可以提高代码质量、降低系统复杂度、增强代码的可维护性和可扩展性。然而，设计模式不是银弹，需要根据具体的问题场景和项目需求来选择合适的模式。希望本文能够帮助读者更好地理解和应用这些经典的设计模式，在实际项目中写出更加优秀的代码。
# 构建高效系统：结构型设计模式的力量

结构型模式关注对象的组织方式，主要解决如何更好地组织类和对象，使其更灵活、更高效。

作用：
- 处理对象的组合、继承和接口适配问题。
- 简化系统的结构，增强系统的可维护性和扩展性。

常见结构型模式：
- 适配器模式（Adapter）：将一个类的接口转换成客户期望的另一个接口。
- 桥接模式（Bridge）：将抽象部分与实现部分分离，使它们可以独立变化。
- 组合模式（Composite）：将对象组合成树形结构以表示“整体-部分”层次结构，使客户可以一致地处理单个对象和组合对象。
- 装饰器模式（Decorator）：动态地为对象添加新的职责，而不改变其结构。
- 外观模式（Facade）：为子系统提供一个统一的接口，简化子系统的使用。
- 享元模式（Flyweight）：通过共享尽量多的相似对象，减少内存使用。
- 代理模式（Proxy）：为另一个对象提供一个替身或占位符，以控制对该对象的访问。

## 结构型模式

### 适配器模式

适配器模式的核心作用是将一个类的接口转换为客户端所期望的另一个接口，使得原本接口不兼容的类可以一起工作。其主要作用表现为如下三点：
- 解决接口不兼容的问题
- 重用已有的类
- 增强系统的灵活性和可扩展性

适配器模式可以分为对象适配器模式(has-a)和类适配器模式(is-a)两种。

**1.对象适配器模式（Object Adapter Pattern）**

对象适配器通过组合的方式实现。适配器类内部持有一个需要适配对象的实例，通过将请求委托给该实例来实现适配。对象适配器不涉及继承，适配器和被适配者是两个独立的对象。例如一个日志系统:

假设你有一个旧的日志类 ```OldLogger```，它的接口 ```logMessage(std::string msg)```，但你的系统希望使用新的接口 Logger，该接口提供 ```log(std::string msg, LogLevel level)``` 方法。

```cpp
#include <iostream>
#include <string>

// 旧的日志类
class OldLogger {
public:
    void logMessage(const std::string& msg) {
        std::cout << "Old Logger: " << msg << std::endl;
    }
};

// 新的日志接口
enum class LogLevel {
    INFO,
    WARNING,
    ERROR
};

class Logger {
public:
    virtual void log(const std::string& msg, LogLevel level) = 0;
};

// 适配器类
class LoggerAdapter : public Logger {
private:
    OldLogger* oldLogger;

public:
    LoggerAdapter() {
        oldLogger = new OldLogger();
    }

    void log(const std::string& msg, LogLevel level) override {
        std::string levelStr;
        switch (level) {
            case LogLevel::INFO:
                levelStr = "INFO: ";
                break;
            case LogLevel::WARNING:
                levelStr = "WARNING: ";
                break;
            case LogLevel::ERROR:
                levelStr = "ERROR: ";
                break;
        }
        oldLogger->logMessage(levelStr + msg);
    }

    ~LoggerAdapter() {
        delete oldLogger;
    }
};

int main() {
    Logger* logger = new LoggerAdapter();
    logger->log("This is an information message.", LogLevel::INFO);
    logger->log("This is a warning message.", LogLevel::WARNING);
    logger->log("This is an error message.", LogLevel::ERROR);
    delete logger;
    return 0;
}
```

**2.类适配器模式（Class Adapter Pattern）**

类适配器通过多重继承的方式实现。适配器类同时继承了目标接口和需要适配的类，通过重写目标接口的方法，将调用转发给适配的类。

```cpp
#include <iostream>

// 目标接口
class ITarget {
public:
    virtual void request() = 0;
    virtual ~ITarget() = default;
};

// 需要适配的类
class Adaptee {
public:
    void specificRequest() {
        std::cout << "Adaptee's specific request" << std::endl;
    }
};

// 类适配器类
class ClassAdapter : public ITarget, private Adaptee {
public:
    void request() override {
        specificRequest();  // 直接调用Adaptee的方法
    }
};

int main() {
    ITarget* adapter = new ClassAdapter();

    adapter->request();  // 调用Target接口

    delete adapter;

    return 0;
}
```

在这个例子中，```ClassAdapter``` 类继承了 ```ITarget``` 接口和 ```Adaptee``` 类，并在 ```request()``` 方法中直接调用 ```Adaptee``` 的 ```specificRequest()``` 方法。这种方式使用了多重继承。

**对象适配器与类适配器的区别**
- 对象适配器使用组合方式实现，可以适配多个不同的类，但无法修改适配的类；适配器和适配的类是松散耦合的。
- 类适配器使用多重继承方式实现，继承了适配的类，因此可以直接访问其接口和方法；但由于C++不支持多重继承与抽象类一起使用，会导致继承接口受限，且适配器和适配的类是紧密耦合的。

**选择适配器模式的依据**
- 对象适配器更常用，因为它灵活且可以适配多个不同的类。一般推荐使用对象适配器模式。
- 类适配器更适用于你只需要适配一个类，并且该类的所有方法你都可以直接使用的情况。

### 桥接模式

桥接模式的主要作用是解耦抽象和实现，使得它们可以独立地变化和扩展。通过桥接模式，可以在实现高层结构的灵活性的同时，避免类的爆炸性增长，增强系统的可扩展性和维护性。以下是桥接模式的具体作用：
- 分离抽象与实现。避免了直接依赖具体实现。这样在不影响抽象接口的情况下，可以自由更换实现。
- 支持独立扩展和变化。在桥接模式中，抽象和实现都可以独立扩展，不会相互影响。
- 如果不使用桥接模式，通常需要为每个具体组合创建一个子类，比如"红色圆形"、"绿色矩形"等，可能导致大量的组合类。桥接模式通过将不同维度的实现进行组合，可以大大减少类的数量。
- 提高系统的可维护性和扩展性。桥接模式的松耦合特性使得代码更易维护，因为抽象和实现彼此独立。更新实现或新增实现时，不需要更改抽象部分。
- 实现运行时动态组合。桥接模式支持运行时动态组合，抽象部分可以在运行时选择不同的实现。通过组合不同的实现，系统可以实现不同的行为和效果，具有极大的灵活性。
- 适合跨平台的系统设计。桥接模式可以隐藏不同平台的差异，将平台的具体实现细节封装在实现部分。通过这一模式，可以轻松地在不同平台上使用不同的实现，而不影响系统的抽象部分。

例如，我们有一个图形绘制系统，支持不同的颜色(如绿色和红色)，支持不同类型的形状（如圆形和矩形），支持不同的材料(如木制和金属)。使用桥接模式，可以将颜色、形状以及材料分开，使得三者可以独立变化。

```cpp
#include <iostream>
#include <memory>

// 第一个维度：颜色
class Color {
public:
    virtual void applyColor() = 0;
    virtual ~Color() = default;
};

class RedColor : public Color {
public:
    void applyColor() override {
        std::cout << "Applying Red Color";
    }
};

class GreenColor : public Color {
public:
    void applyColor() override {
        std::cout << "Applying Green Color";
    }
};

// 第二个维度：材质
class Material {
public:
    virtual void applyMaterial() = 0;
    virtual ~Material() = default;
};

class WoodMaterial : public Material {
public:
    void applyMaterial() override {
        std::cout << " with Wood Material" << std::endl;
    }
};

class MetalMaterial : public Material {
public:
    void applyMaterial() override {
        std::cout << " with Metal Material" << std::endl;
    }
};

// 第三个维度：形状
class Shape {
protected:
    std::shared_ptr<Color> color;        // 持有颜色的实现
    std::shared_ptr<Material> material;  // 持有材质的实现

public:
    Shape(std::shared_ptr<Color> c, std::shared_ptr<Material> m) 
        : color(c), material(m) {}

    virtual void draw() = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
public:
    Circle(std::shared_ptr<Color> c, std::shared_ptr<Material> m) 
        : Shape(c, m) {}

    void draw() override {
        std::cout << "Drawing Circle with ";
        color->applyColor();
        material->applyMaterial();
    }
};

class Rectangle : public Shape {
public:
    Rectangle(std::shared_ptr<Color> c, std::shared_ptr<Material> m) 
        : Shape(c, m) {}

    void draw() override {
        std::cout << "Drawing Rectangle with ";
        color->applyColor();
        material->applyMaterial();
    }
};

int main() {
    // 创建颜色和材质的具体实现
    std::shared_ptr<Color> red = std::make_shared<RedColor>();
    std::shared_ptr<Color> green = std::make_shared<GreenColor>();

    std::shared_ptr<Material> wood = std::make_shared<WoodMaterial>();
    std::shared_ptr<Material> metal = std::make_shared<MetalMaterial>();

    // 组合不同的形状、颜色和材质
    std::shared_ptr<Shape> woodenRedCircle = std::make_shared<Circle>(red, wood);
    std::shared_ptr<Shape> metalGreenRectangle = std::make_shared<Rectangle>(green, metal);

    // 使用组合
    woodenRedCircle->draw();
    // 输出：Drawing Circle with Applying Red Color with Wood Material

    metalGreenRectangle->draw();
    // 输出：Drawing Rectangle with Applying Green Color with Metal Material

    // 动态改变材质和颜色
    std::shared_ptr<Shape> metalRedCircle = std::make_shared<Circle>(red, metal);
    metalRedCircle->draw();
    // 输出：Drawing Circle with Applying Red Color with Metal Material

    return 0;
}
```

**代码解析**
从Implementor、ConcreteImplementor、Abstraction、和RefinedAbstraction这四个角色的角度去解析上述代码：
- Implementor： Color类就是Implementor，它定义了applyColor()接口；Material类也是Implementor，它定义了applyMaterial()接口
- ConcreteImplementor： RedColor和GreenColor就是Color类的具体实现；WoodMaterial和MetalMaterial就是Material的具体实现。
- Abstraction：Shape类是Abstraction，它持有一个指向Color的指针和一个指向Material类的指针，并且定义了draw()方法作为抽象接口。
- RefinedAbstraction： Circle和Rectangle类是RefinedAbstraction，它们继承自Shape（Abstraction），并实现了具体的draw()方法。实现了不同颜色和不同材质的绘制效果。

### 组合模式

组合模式（Composite Pattern）允许你将对象组合成树形结构来表示"部分-整体"的层次结构。组合模式使得客户端可以以统一的方式对待单个对象和对象集合，能够让客户端在不关心组合对象内部结构的情况下，像操作单个对象一样操作整个对象树。

组合模式解决的问题
- 统一管理对象： 组合模式将单个对象和由对象组合而成的复合对象统一成相同的接口，允许客户端以相同的方式处理单个对象和对象集合。这样，客户端可以方便地处理树形结构中的所有元素，而无需关心这些元素是简单对象还是复合对象。
- 构建层次结构： 组合模式特别适用于树形结构的对象，如文件系统、UI组件等。通过组合模式，可以方便地创建这种层次结构。
- 递归结构： 组合模式使得递归结构的管理和操作变得非常简单。例如，树形结构的每个节点（无论是叶子节点还是复合节点）都可以统一地进行遍历、添加、删除等操作。

我们以一个文件系统为例，其中包含文件和文件夹。文件夹可以包含多个文件或子文件夹，文件夹和文件都可以统一处理成“文件系统元素”。这个场景适合使用组合模式。

步骤：
- 定义一个 Component 抽象类，表示文件系统中的所有元素（文件或文件夹）。
- 定义 Leaf 类（文件），实现 Component 接口。
- 定义 Composite 类（文件夹），实现 Component 接口，并且可以包含多个子元素（文件或文件夹）。

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>

// Component: 文件系统元素基类
class IFileSystem {
public:
    virtual ~IFileSystem() {}
    virtual void show(int indent = 0) const = 0;
};

// Leaf: 叶子节点类（文件）
class File : public IFileSystem {
private:
    std::string name;
public:
    File(const std::string& name) : name(name) {}
    
    void show(int indent = 0) const override {
        std::string indentation(indent, ' ');
        std::cout << indentation << "File: " << name << std::endl;
    }
};

// Composite: 组合节点类（文件夹）
class Folder : public IFileSystem {
private:
    std::string name;
    std::vector<std::shared_ptr<IFileSystem>> children;  // 子元素

public:
    Folder(const std::string& name) : name(name) {}
    
    void add(std::shared_ptr<FileSystemElement> element) {
        children.push_back(element);
    }

    void show(int indent = 0) const override {
        std::string indentation(indent, ' ');
        std::cout << indentation << "Folder: " << name << std::endl;
        for (const auto& child : children) {
            child->show(indent + 2);  // 子元素的缩进增加
        }
    }
};

int main() {
    // 创建文件和文件夹
    auto file1 = std::make_shared<File>("file1.txt");
    auto file2 = std::make_shared<File>("file2.txt");
    
    auto folder1 = std::make_shared<Folder>("Folder1");
    folder1->add(file1);
    folder1->add(file2);
    
    auto folder2 = std::make_shared<Folder>("Folder2");
    auto file3 = std::make_shared<File>("file3.txt");
    folder2->add(file3);
    
    auto root = std::make_shared<Folder>("RootFolder");
    root->add(folder1);
    root->add(folder2);

    // 展示文件系统结构
    root->show();

    return 0;
}
```

### 装饰模式

装饰模式（Decorator Pattern）是一种结构型设计模式，它允许你动态地给对象添加新的功能，而不改变其结构或影响其他对象。装饰模式通过使用一个或多个装饰器类，将功能包装在原始对象的基础之上。

装饰模式的主要作用
- 动态扩展对象功能：无需修改现有类的定义，便可动态地为对象添加新功能。
- 遵循开闭原则：装饰模式允许你在不修改类代码的情况下扩展功能，符合“对扩展开放，对修改封闭”的设计原则。
- 组合功能：你可以通过多个装饰器来组合对象的功能，这比继承更加灵活。

装饰模式的结构
- 抽象组件（Component）：定义一个接口，用于对象的基本功能声明，可以是具体的类或接口。
- 具体组件（ConcreteComponent）：实现抽象组件的类，表示被装饰的原始对象。
- 装饰器（Decorator）：一个抽象类，继承自组件类，并包含一个指向组件的引用。
- 具体装饰器（ConcreteDecorator）：继承自装饰器类，添加额外的功能。

示例场景
假设我们有一个基础的文本类PlainText，表示一段简单的文本。现在，我们想要动态地为这段文本添加不同的格式（如加粗、斜体、下划线等），而不修改原始文本类的代码。

C++ 实现装饰模式

```cpp
#include <iostream>
#include <string>

// 抽象组件（Component）
class Text {
public:
    virtual std::string getText() const = 0;
    virtual ~Text() = default;
};

// 具体组件（ConcreteComponent）
class PlainText : public Text {
private:
    std::string content;

public:
    PlainText(const std::string &text) : content(text) {}

    std::string getText() const override {
        return content;
    }
};

// 装饰器（Decorator）
class TextDecorator : public Text {
protected:
    Text *wrappedText;  // 被装饰的文本对象

public:
    TextDecorator(Text *text) : wrappedText(text) {}

    virtual ~TextDecorator() {
        delete wrappedText;  // 清理内存
    }
};

// 具体装饰器（ConcreteDecorator）1: 加粗
class BoldDecorator : public TextDecorator {
public:
    BoldDecorator(Text *text) : TextDecorator(text) {}

    std::string getText() const override {
        return "<b>" + wrappedText->getText() + "</b>";  // 用<b>标签包裹文本
    }
};

// 具体装饰器（ConcreteDecorator）2: 斜体
class ItalicDecorator : public TextDecorator {
public:
    ItalicDecorator(Text *text) : TextDecorator(text) {}

    std::string getText() const override {
        return "<i>" + wrappedText->getText() + "</i>";  // 用<i>标签包裹文本
    }
};

// 具体装饰器（ConcreteDecorator）3: 下划线
class UnderlineDecorator : public TextDecorator {
public:
    UnderlineDecorator(Text *text) : TextDecorator(text) {}

    std::string getText() const override {
        return "<u>" + wrappedText->getText() + "</u>";  // 用<u>标签包裹文本
    }
};

// 客户端代码
int main() {
    // 创建一个简单的文本对象
    Text *myText = new PlainText("Hello, World!");
    std::cout << "Plain Text: " << myText->getText() << "\n";

    // 为文本添加加粗效果
    myText = new BoldDecorator(myText);
    std::cout << "Bold Text: " << myText->getText() << "\n";

    // 为文本添加斜体效果
    myText = new ItalicDecorator(myText);
    std::cout << "Bold and Italic Text: " << myText->getText() << "\n";

    // 为文本添加下划线效果
    myText = new UnderlineDecorator(myText);
    std::cout << "Bold, Italic, and Underlined Text: " << myText->getText() << "\n";

    // 清理内存
    delete myText;

    return 0;
}

```

**解释**
- PlainText：这是一个具体组件，实现了Text接口。它代表一个简单的文本，内容是“Hello, World!”。
- TextDecorator：这是装饰器的基类，继承自Text接口。它包含一个指向Text对象的指针，用于包装组件并扩展其功能。
- BoldDecorator：具体装饰器类，给文本添加加粗的效果。它通过在文本内容前后加上<b>和</b>标签来实现。
- ItalicDecorator：具体装饰器类，给文本添加斜体效果。它通过在文本内容前后加上<i>和</i>标签来实现。
- UnderlineDecorator：具体装饰器类，给文本添加下划线效果。它通过在文本内容前后加上<u>和</u>标签来实现。

**输出结果**
```cpp
Plain Text: Hello, World!
Bold Text: <b>Hello, World!</b>
Bold and Italic Text: <b><i>Hello, World!</i></b>
Bold, Italic, and Underlined Text: <b><i><u>Hello, World!</u></i></b>
```

**代码说明**
- 简单文本：首先，我们创建一个PlainText对象，其内容为“Hello, World!”。
- 加粗文本：接着，我们用BoldDecorator装饰PlainText，给文本加粗，输出为<b>Hello, World!</b>。
- 加斜体：然后，我们用ItalicDecorator再装饰之前的结果，给文本加上斜体效果，输出为<b><i>Hello, World!</i></b>。
- 加下划线：最后，我们用UnderlineDecorator再次装饰，给文本加上了下划线效果，输出为<b><i><u>Hello, World!</u></i></b>。

**总结**

这个例子展示了如何使用装饰模式来动态地为文本添加不同的格式（如加粗、斜体、下划线等）。通过装饰模式，我们可以灵活地组合不同的格式，而不需要修改原始的文本类。每个装饰器只负责添加一种特定的格式，这样可以自由组合这些装饰器来扩展文本的功能。这种方法比继承更加灵活和可扩展。

### 外观模式

外观模式主要解决的是简化接口和减少系统复杂性的问题。它通过提供一个统一的接口来访问复杂的子系统，隐藏了子系统的内部细节，使客户端更容易使用子系统的功能。换句话说，外观模式的目标是简化和解耦，提供一个更简单的使用方式。

**外观模式的基本概念**

在外观模式中，外观类（Facade）提供了一个统一的接口，客户端通过这个接口来调用子系统的功能。子系统通常是由多个类组成的，而外观类则将这些复杂的操作封装在一个简单的接口中，隐藏了子系统的复杂性。

**外观模式的优点**
- 简化接口：外观模式将复杂的子系统操作封装在一个统一的接口中，简化了客户端的调用。
- 解耦：客户端与子系统之间的依赖减少了，通过外观类，客户端只需与外观类交互，不直接依赖于子系统的具体实现。
- 便于维护：子系统的改动只需修改外观类的实现，而不必更改客户端的代码，从而减少了系统的维护难度。

外观模式的C++代码示例

假设我们有一个复杂的子系统，其中包含多个类，用于处理不同的功能。我们可以使用外观模式来简化这些功能的调用。

子系统类

```cpp
#include <iostream>

class CPU {
public:
    void freeze() {
        std::cout << "CPU freezing..." << std::endl;
    }
    void jump(int position) {
        std::cout << "CPU jumping to position " << position << std::endl;
    }
    void execute() {
        std::cout << "CPU executing..." << std::endl;
    }
};

class Memory {
public:
    void load(int position, const std::string& data) {
        std::cout << "Memory loading data '" << data << "' at position " << position << std::endl;
    }
};

class HardDrive {
public:
    std::string read(int lba, int size) {
        std::cout << "HardDrive reading " << size << " bytes from LBA " << lba << std::endl;
        return "data";
    }
};
```

外观类

```cpp
class ComputerFacade {
private:
    CPU cpu;
    Memory memory;
    HardDrive hardDrive;

public:
    void startComputer() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
};
```

客户端代码

```cpp
int main() {
    ComputerFacade computer;
    computer.startComputer();
    return 0;
}
```

解释

**1.子系统类**：

- CPU, Memory, 和 HardDrive 类分别处理计算机的不同方面，如处理器的冻结和执行、内存的加载、硬盘的读取等。

**2.外观类**：

- ComputerFacade 类将这些子系统的复杂操作封装在一个简单的 startComputer 方法中。这个方法调用了子系统中的多个操作，简化了客户端的操作流程。

**3.客户端代码**：

- 在 main 函数中，客户端只需与 ComputerFacade 类交互，通过调用 startComputer 方法启动计算机，而无需直接操作 CPU, Memory, 和 HardDrive 类。这使得客户端的代码更加简洁，且与子系统解耦。

使用外观模式可以有效地将系统的复杂性隐藏在一个简单的接口后面，减少了客户端的复杂度并提高了系统的可维护性。

### 享元模式

享元模式（Flyweight Pattern）是一种结构型设计模式，旨在减少程序中创建对象的数量，从而减少内存的占用。它通过共享已经存在的对象来避免重复创建相同对象的代价，尤其在大量细粒度对象被频繁使用的场景下，这种模式非常有用。

**适用场景**

享元模式特别适用于以下场景：

- 程序中有大量相似的对象，并且这些对象的大部分状态是相同的，可以共享。
- 系统需要节省内存，并且对象的数量可能很大。

使用享元模式来管理 ConsoleLogger 和 FileLogger 可以更好地展示如何通过共享内部状态来减少内存开销和对象创建的成本。我们可以让这两种不同类型的日志记录器共享相同的日志级别（或其他配置），而输出到控制台还是文件作为具体实现的细节。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <map>
#include <memory>

// 抽象享元类
class Logger {
public:
    virtual void log(const std::string& message, const std::string& context) const = 0;
    virtual ~Logger() = default;
};

// 具体享元类 - 控制台日志记录器
class ConsoleLogger : public Logger {
private:
    std::string logLevel; // 共享的内部状态

public:
    ConsoleLogger(const std::string& logLevel) : logLevel(logLevel) {}

    void log(const std::string& message, const std::string& context) const override {
        std::cout << "[" << logLevel << "] " << context << ": " << message << std::endl;
    }
};

// 具体享元类 - 文件日志记录器
class FileLogger : public Logger {
private:
    std::string logLevel; // 共享的内部状态
    std::string filename; // 文件名 - 共享的内部状态

public:
    FileLogger(const std::string& logLevel, const std::string& filename) 
        : logLevel(logLevel), filename(filename) {}

    void log(const std::string& message, const std::string& context) const override {
        std::ofstream ofs(filename, std::ios_base::app);
        if (ofs.is_open()) {
            ofs << "[" << logLevel << "] " << context << ": " << message << std::endl;
            ofs.close();
        }
    }
};

// 享元工厂类，用于管理和创建享元对象
class LoggerFactory {
private:
    std::map<std::string, std::shared_ptr<Logger>> loggers;

    std::string getKey(const std::string& logType, const std::string& logLevel, const std::string& filename = "") const {
        return logType + "-" + logLevel + "-" + filename;
    }

public:
    std::shared_ptr<Logger> getConsoleLogger(const std::string& logLevel) {
        std::string key = getKey("Console", logLevel);
        auto it = loggers.find(key);
        if (it != loggers.end()) {
            return it->second;
        } else {
            auto logger = std::make_shared<ConsoleLogger>(logLevel);
            loggers[key] = logger;
            return logger;
        }
    }

    std::shared_ptr<Logger> getFileLogger(const std::string& logLevel, const std::string& filename) {
        std::string key = getKey("File", logLevel, filename);
        auto it = loggers.find(key);
        if (it != loggers.end()) {
            return it->second;
        } else {
            auto logger = std::make_shared<FileLogger>(logLevel, filename);
            loggers[key] = logger;
            return logger;
        }
    }
};

int main() {
    LoggerFactory factory;

    // 创建不同类型和级别的日志记录器
    auto consoleErrorLogger = factory.getConsoleLogger("ERROR");
    auto consoleInfoLogger = factory.getConsoleLogger("INFO");

    auto fileErrorLogger = factory.getFileLogger("ERROR", "error.log");
    auto fileInfoLogger = factory.getFileLogger("INFO", "info.log");

    // 使用日志记录器记录日志信息
    consoleErrorLogger->log("This is an error message", "Main");
    consoleInfoLogger->log("This is an info message", "Main");

    fileErrorLogger->log("This is an error message", "Main");
    fileInfoLogger->log("This is an info message", "Main");

    // 使用同一个日志类型和级别的记录器
    auto anotherConsoleErrorLogger = factory.getConsoleLogger("ERROR");
    anotherConsoleErrorLogger->log("This is another error message", "Helper");

    auto anotherFileErrorLogger = factory.getFileLogger("ERROR", "error.log");
    anotherFileErrorLogger->log("This is another error message", "Helper");

    // 注意：consoleErrorLogger 和 anotherConsoleErrorLogger 是同一个实例
    // fileErrorLogger 和 anotherFileErrorLogger 也是同一个实例

    return 0;
}

```


### 代理模式

代理模式（Proxy Pattern）是一种结构型设计模式，它为其他对象提供一种代理以控制对这个对象的访问。代理模式可以在不改变原始对象的前提下，为其提供额外的功能或控制。代理模式的作用主要有以下几个方面：

**代理模式的主要作用**
- 控制访问：代理可以限制或控制对某个对象的访问。例如，在远程代理中，客户端无法直接访问远程对象，代理控制这个访问并管理网络通信。

- 延迟加载：也称为“懒加载”（Lazy Initialization），在对象的初始化开销较大时，可以通过代理延迟对象的创建，只有在实际需要时才创建对象。

- 权限管理：在保护代理中，可以通过代理来检查客户端的访问权限，从而决定是否允许访问实际对象。

- 日志记录：代理可以在调用真实对象的操作之前或之后记录日志信息，用于调试或审计。

- 远程代理：代理处理客户端与远程服务之间的通信，隐藏复杂的网络细节。

**代理模式的结构**
- 抽象接口（Subject）：定义了代理类和实际对象类的共同接口。
- 真实对象（RealSubject）：实现了抽象接口，是真正需要代理的对象。
- 代理类（Proxy）：实现了抽象接口，内部持有真实对象的引用，负责控制对真实对象的访问。

示例场景

假设我们有一个银行账户类BankAccount，客户可以通过该类来存款、取款和查看余额。但是为了增加安全性，我们可以引入一个代理类BankAccountProxy，它在执行这些操作之前检查用户的权限。

C++ 实现代理模式

```cpp
#include <iostream>
#include <string>

// 抽象接口（Subject）
class BankAccount {
public:
    virtual void deposit(double amount) = 0;
    virtual void withdraw(double amount) = 0;
    virtual double getBalance() const = 0;
};

// 真实对象（RealSubject）
class RealBankAccount : public BankAccount {
private:
    double balance;

public:
    RealBankAccount() : balance(0.0) {}

    void deposit(double amount) override {
        balance += amount;
        std::cout << "Deposited: " << amount << ", New Balance: " << balance << "\n";
    }

    void withdraw(double amount) override {
        if (amount <= balance) {
            balance -= amount;
            std::cout << "Withdrew: " << amount << ", New Balance: " << balance << "\n";
        } else {
            std::cout << "Insufficient funds. Withdrawal failed.\n";
        }
    }

    double getBalance() const override {
        return balance;
    }
};

// 代理类（Proxy）
class BankAccountProxy : public BankAccount {
private:
    RealBankAccount *realAccount;
    std::string userRole;

public:
    BankAccountProxy(const std::string &role) : realAccount(new RealBankAccount()), userRole(role) {}

    ~BankAccountProxy() {
        delete realAccount;
    }

    void deposit(double amount) override {
        if (userRole == "Admin" || userRole == "User") {
            realAccount->deposit(amount);
        } else {
            std::cout << "Access denied. Only Admin or User can deposit funds.\n";
        }
    }

    void withdraw(double amount) override {
        if (userRole == "Admin") {
            realAccount->withdraw(amount);
        } else {
            std::cout << "Access denied. Only Admin can withdraw funds.\n";
        }
    }

    double getBalance() const override {
        if (userRole == "Admin" || userRole == "User") {
            return realAccount->getBalance();
        } else {
            std::cout << "Access denied. Only Admin or User can view the balance.\n";
            return -1;
        }
    }
};

// 客户端代码
int main() {
    // 创建一个代理对象，角色为普通用户
    BankAccount *userAccount = new BankAccountProxy("User");

    userAccount->deposit(100.0);  // 正常存款
    userAccount->withdraw(50.0);  // 用户不能取款
    std::cout << "Balance: " << userAccount->getBalance() << "\n";  // 查看余额

    delete userAccount;

    // 创建一个代理对象，角色为管理员
    BankAccount *adminAccount = new BankAccountProxy("Admin");

    adminAccount->deposit(500.0);  // 管理员存款
    adminAccount->withdraw(200.0);  // 管理员取款
    std::cout << "Balance: " << adminAccount->getBalance() << "\n";  // 查看余额

    delete adminAccount;

    return 0;
}
```


好的，让我们用一个不同的例子来说明C++中的代理模式。这次我们模拟一个银行账户的场景，通过代理来控制对账户的访问权限。

示例场景
假设我们有一个银行账户类BankAccount，客户可以通过该类来存款、取款和查看余额。但是为了增加安全性，我们可以引入一个代理类BankAccountProxy，它在执行这些操作之前检查用户的权限。

C++ 实现代理模式

```cpp
#include <iostream>
#include <string>

// 抽象接口（Subject）
class BankAccount {
public:
    virtual void deposit(double amount) = 0;
    virtual void withdraw(double amount) = 0;
    virtual double getBalance() const = 0;
};

// 真实对象（RealSubject）
class RealBankAccount : public BankAccount {
private:
    double balance;

public:
    RealBankAccount() : balance(0.0) {}

    void deposit(double amount) override {
        balance += amount;
        std::cout << "Deposited: " << amount << ", New Balance: " << balance << "\n";
    }

    void withdraw(double amount) override {
        if (amount <= balance) {
            balance -= amount;
            std::cout << "Withdrew: " << amount << ", New Balance: " << balance << "\n";
        } else {
            std::cout << "Insufficient funds. Withdrawal failed.\n";
        }
    }

    double getBalance() const override {
        return balance;
    }
};

// 代理类（Proxy）
class BankAccountProxy : public BankAccount {
private:
    RealBankAccount *realAccount;
    std::string userRole;

public:
    BankAccountProxy(const std::string &role) : realAccount(new RealBankAccount()), userRole(role) {}

    ~BankAccountProxy() {
        delete realAccount;
    }

    void deposit(double amount) override {
        if (userRole == "Admin" || userRole == "User") {
            realAccount->deposit(amount);
        } else {
            std::cout << "Access denied. Only Admin or User can deposit funds.\n";
        }
    }

    void withdraw(double amount) override {
        if (userRole == "Admin") {
            realAccount->withdraw(amount);
        } else {
            std::cout << "Access denied. Only Admin can withdraw funds.\n";
        }
    }

    double getBalance() const override {
        if (userRole == "Admin" || userRole == "User") {
            return realAccount->getBalance();
        } else {
            std::cout << "Access denied. Only Admin or User can view the balance.\n";
            return -1;
        }
    }
};

// 客户端代码
int main() {
    // 创建一个代理对象，角色为普通用户
    BankAccount *userAccount = new BankAccountProxy("User");

    userAccount->deposit(100.0);  // 正常存款
    userAccount->withdraw(50.0);  // 用户不能取款
    std::cout << "Balance: " << userAccount->getBalance() << "\n";  // 查看余额

    delete userAccount;

    // 创建一个代理对象，角色为管理员
    BankAccount *adminAccount = new BankAccountProxy("Admin");

    adminAccount->deposit(500.0);  // 管理员存款
    adminAccount->withdraw(200.0);  // 管理员取款
    std::cout << "Balance: " << adminAccount->getBalance() << "\n";  // 查看余额

    delete adminAccount;

    return 0;
}
```

**解释**
- RealBankAccount：这是实际的银行账户类，包含存款、取款和获取余额的功能。
- BankAccountProxy：代理类负责控制对RealBankAccount的访问。它接收用户的角色信息，根据用户的角色决定是否允许进行某些操作。
  - 如果用户角色是"Admin"（管理员），代理类允许执行所有操作。
  - 如果用户角色是"User"（普通用户），代理类只允许存款和查看余额，不允许取款。
  - 如果用户角色是其他角色，代理类拒绝所有操作。
- 客户端代码：通过代理类来操作银行账户，而不是直接操作RealBankAccount类。这样可以在不修改RealBankAccount的情况下，控制对账户的访问。

输出结果

```shell
Deposited: 100, New Balance: 100
Access denied. Only Admin can withdraw funds.
Balance: 100
Deposited: 500, New Balance: 600
Withdrew: 200, New Balance: 400
Balance: 400
```

**总结**

在这个例子中，代理模式的主要作用是控制访问权限。通过代理类BankAccountProxy，我们可以在执行账户操作之前检查用户的权限，防止未授权的操作。这种方式提高了系统的安全性，同时保持了系统设计的灵活性。
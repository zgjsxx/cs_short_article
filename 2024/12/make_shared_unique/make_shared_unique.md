# `std::make_shared` 和 `std::make_unique`

`std::make_shared` 和 `std::make_unique` 是 C++11 引入的两个模板函数，分别用来创建 `std::shared_ptr` 和 `std::unique_ptr` 的实例。它们提供了一种安全、高效且易于使用的方式来管理动态分配的内存。

## **1. std::make_shared**

`std::make_shared` 用来创建一个 `std::shared_ptr` 对象，同时分配和初始化共享对象的内存。

### **优点**
- **性能优化**：  
  `std::make_shared` 在一个内存块中同时分配共享对象和控制块，减少了动态内存分配的次数，提升了性能。
- **安全性**：  
  避免了手动调用 `new` 时可能产生的异常安全问题。  
  如果构造函数抛出异常，`std::make_shared` 会保证不会泄漏内存。
- **简洁**：  
  代码更清晰，无需显式使用 `new`。

对于性能上：

使用 std::shared_ptr 直接管理裸指针的内存分配问题

```cpp
std::shared_ptr<Widget> spw(new Widget);
```

两次内存分配：
- 第一次分配：为 Widget 对象分配内存，调用 new Widget。
- 第二次分配：为 std::shared_ptr 的控制块分配内存。控制块包含以下信息：
  - 引用计数：用于跟踪当前有多少 std::shared_ptr 和 std::weak_ptr 指向该对象。
  - 弱引用计数：跟踪弱引用的数量。
  - 其他簿记信息：可能包含删除器（deleter）的指针等。

由于控制块和对象的内存分配是独立的，这种方式会增加程序的运行开销：
- 更多的内存分配调用：额外的分配操作不仅增加了程序的代码量，还可能引起堆碎片化。
- 降低内存局部性：控制块和对象存储在不同的内存区域，可能导致缓存命中率降低。

对于安全性，典型的例子是下面的函数调用：

```cpp
processWidget(std::shared_ptr<Widget>(new Widget),  // 潜在的资源泄漏！
              computePriority());
```

使用 `new Widget` 创建一个动态分配的对象，并将其交由 `std::shared_ptr` 管理。根据 C++ 的函数调用规则，参数的求值顺序未定义。如果 `computePriority()` 在 `std::shared_ptr` 的构造完成之前抛出异常，那么 `new Widget` 创建的裸指针就永远不会被 `std::shared_ptr` 管理，导致内存泄漏。

`std::make_shared` 是推荐的解决方案，可以将对象的创建、内存分配和智能指针的管理整合到一个步骤中。

改进后的代码：
```cpp
processWidget(std::make_shared<Widget>(),  // 异常安全
              computePriority());
```

对于其简洁性，使用make_shared可以少敲几个字母，可以有效减少你得腱鞘炎的概率。

```cpp
auto spw1(std::make_shared<Widget>());      //使用make函数
std::shared_ptr<Widget> spw2(new Widget);   //不使用make函数
```

### **语法**
```cpp
template<class T, class... Args>
std::shared_ptr<T> make_shared(Args&&... args);
```

### **示例**
```cpp
#include <memory>
#include <iostream>

struct MyClass {
    MyClass(int x, int y) : a(x), b(y) {}
    int a, b;
};

int main() {
    auto ptr = std::make_shared<MyClass>(10, 20); // 创建并初始化共享指针
    std::cout << "a: " << ptr->a << ", b: " << ptr->b << '\n';
    return 0;
}
```

---

## **2. std::make_unique**

`std::make_unique` 用来创建一个 `std::unique_ptr` 对象，同时分配和初始化动态分配的对象。

### **优点**
- **安全性**：  
  和 `std::make_shared` 类似，避免了直接使用 `new` 时的异常安全问题。
- **简洁性**：  
  提供了比直接使用 `std::unique_ptr` 构造函数更简洁的写法。
- **推荐使用**：  
  对于所有权独占的场景，推荐使用 `std::make_unique`。

### **语法**
```cpp
template<class T, class... Args>
std::unique_ptr<T> make_unique(Args&&... args);
```

### **示例**
```cpp
#include <memory>
#include <iostream>

struct MyClass {
    MyClass(int x, int y) : a(x), b(y) {}
    int a, b;
};

int main() {
    auto ptr = std::make_unique<MyClass>(10, 20); // 创建并初始化独占指针
    std::cout << "a: " << ptr->a << ", b: " << ptr->b << '\n';
    return 0;
}
```

---

## **对比 std::make_shared 和 std::make_unique**
| 特性               | `std::make_shared`                        | `std::make_unique`                        |
|--------------------|------------------------------------------|------------------------------------------|
| **指针类型**        | `std::shared_ptr`（共享所有权）           | `std::unique_ptr`（独占所有权）           |
| **内存分配效率**    | 高效（单次内存分配，包含控制块和对象）      | 效率稍低（仅分配对象，不涉及控制块）       |
| **线程安全性**      | 支持多线程的引用计数                     | 不涉及引用计数，线程安全性依赖对象本身    |
| **使用场景**        | 多个指针共享同一个动态对象                | 对象所有权独占                            |

---

## **使用注意事项**
1. **避免手动使用 `new`**  
   手动分配内存容易导致内存泄漏，建议优先使用 `std::make_shared` 和 `std::make_unique`。

2. **生命周期管理**  
   - 使用 `std::shared_ptr` 时要注意避免循环引用（例如 `std::weak_ptr` 的辅助使用）。
   - `std::unique_ptr` 不允许拷贝，只允许转移所有权。

3. **C++14 支持**  
   `std::make_unique` 在 C++14 中引入，如果使用 C++11，可以自己实现一个简单版本：
   ```cpp
   template<class T, class... Args>
   std::unique_ptr<T> make_unique(Args&&... args) {
       return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
   }
   ```

总结来说，`std::make_shared` 和 `std::make_unique` 是现代 C++ 中动态内存管理的最佳实践，大大简化了指针使用的复杂性，同时提升了代码的安全性和可维护性。


## 不适合使用 `make` 函数的场景

虽然 `std::make_shared` 和 `std::make_unique` 提供了许多优点（如一次性内存分配和异常安全性），但在某些特定场景下，它们并不是最适合的选择。以下是几个常见的 **不适合使用 `make` 函数的场景**：

### **1. 自定义删除器**
`std::make_shared` 和 `std::make_unique` 不支持为对象指定自定义删除器。

#### 示例：
如果需要在对象销毁时执行特定的操作（如释放非堆资源），必须使用显式构造：
```cpp
std::shared_ptr<FILE> filePtr(fopen("example.txt", "r"), fclose); // 自定义删除器
```
此处不能使用 `std::make_shared`，因为无法直接传递自定义删除器。

---

### **2. 动态数组**
`std::make_shared` 和 `std::make_unique` 不支持直接创建动态数组。

#### 示例：
```cpp
auto arr = std::make_unique<int[]>(10); // 仅 `std::make_unique` 支持动态数组
```
但对于 `std::shared_ptr`，需要显式指定删除器，否则会导致未定义行为：
```cpp
std::shared_ptr<int> arr(new int[10], [](int* p) { delete[] p; }); // 显式删除器
```

### **3. 对象的大小非常大**
当被管理的对象特别大时，`std::make_shared` 将对象和控制块存储在同一内存块中，这可能导致堆碎片问题。如果将对象与控制块分开存储，可以更好地控制内存布局。

#### 示例：
```cpp
std::shared_ptr<BigObject> sp(new BigObject); // 控制块和对象分离
```

### **4.花括号初始化**

std::make_shared和std::make_unique只能处理圆括号初始化（传递多个参数），而不能处理花括号初始化（用于初始化容器等）。对于需要使用花括号初始化的情景，仍需使用new。


## 实践建议：

尽量使用```std::make_shared```和```std::make_unique```来构造对象。即使在一些特殊情况下你不得不使用new，确保在没有其他代码依赖的情况下直接将结果传递给智能指针构造函数，从而避免潜在的内存泄漏和异常安全问题。如果需要高效的异常安全代码，可以使用```std::move```来转化```std::shared_ptr```（或者```std::unique_ptr```）为右值传递，这可以避免不必要的复制操作，从而提高性能。
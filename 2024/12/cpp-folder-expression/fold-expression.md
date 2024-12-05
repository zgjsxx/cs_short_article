# c++折叠表达式

折叠表达式是 C++17 引入的一种语法，专门用于简化和统一对 **参数包（parameter pack）** 的操作。它能够在编译时展开参数包，并对其中的每个元素应用指定的操作。这种语法既简洁又高效，非常适合处理变参模板中的逻辑。

---

## 折叠表达式的语法

折叠表达式有以下几种形式：

1. **一元左折叠**
   ```cpp
   (... op pack)
   ```
   这表示操作符 `op` 应用于参数包中的所有元素，从左到右进行计算。

2. **一元右折叠**
   ```cpp
   (pack op ...)
   ```
   这表示操作符 `op` 应用于参数包中的所有元素，从右到左进行计算。

3. **二元左折叠**
   ```cpp
   (init op ... op pack)
   ```
   这是带初始值的左折叠。初始值 `init` 会作为左侧的第一个操作数。

4. **二元右折叠**
   ```cpp
   (pack op ... op init)
   ```
   这是带初始值的右折叠。初始值 `init` 会作为右侧的最后一个操作数。

---

## 示例讲解

### 1. 一元左折叠
计算参数包的和：
```cpp
#include <iostream>

template<typename... Args>
auto sum(Args... args) {
    return (... + args); // 左折叠：args1 + args2 + args3 + ...
}

int main() {
    std::cout << sum(1, 2, 3, 4) << std::endl; // 输出 10
    return 0;
}
```

折叠过程为：
```
(((1 + 2) + 3) + 4) = 10
```

### 2. 一元右折叠
类似于上例，但计算方向从右到左：
```cpp
#include <iostream>

template<typename... Args>
auto subtract(Args... args) {
    return (args - ...); // 右折叠：args1 - (args2 - (args3 - ...))
}

int main() {
    std::cout << subtract(10, 3, 2) << std::endl; // 输出 5
    return 0;
}
```

折叠过程为：
```
10 - (3 - 2) = 10 - 1 = 9
```

### 3. 二元左折叠
结合初始值计算参数包的和：
```cpp
#include <iostream>

template<typename... Args>
auto sum_with_init(int init, Args... args) {
    return (init + ... + args); // 左折叠，初始值为 init
}

int main() {
    std::cout << sum_with_init(10, 1, 2, 3) << std::endl; // 输出 16
    return 0;
}
```

折叠过程为：
```
((10 + 1) + 2) + 3 = 16
```

### 4. 二元右折叠
带初始值从右到左计算：
```cpp
#include <iostream>

template<typename... Args>
auto divide_with_init(int init, Args... args) {
    return (args / ... / init); // 右折叠，初始值为 init
}

int main() {
    std::cout << divide_with_init(2, 8, 16) << std::endl; // 输出 1
    return 0;
}
```

折叠过程为：
```
(8 / (16 / 2)) = (8 / 8) = 1
```

---

## 常见用途

1. **递归替代**
   折叠表达式消除了手动展开参数包或编写递归函数的需求。

2. **逻辑运算**
   利用折叠表达式对多个布尔值进行操作：
   ```cpp
   template<typename... Args>
   bool all(Args... args) {
       return (... && args); // 所有元素为真时返回 true
   }

   template<typename... Args>
   bool any(Args... args) {
       return (... || args); // 任意一个元素为真时返回 true
   }

   int main() {
       std::cout << all(true, true, false) << std::endl; // 输出 0 (false)
       std::cout << any(false, false, true) << std::endl; // 输出 1 (true)
   }
   ```

3. **打印变长参数**
   利用逗号表达式实现打印：
   ```cpp
   template<typename... Args>
   void print(Args... args) {
       (..., (std::cout << args << " ")); // 左折叠逐一打印
       std::cout << std::endl;
   }

   int main() {
       print(1, 2.5, "Hello", 'C'); // 输出：1 2.5 Hello C 
   }
   ```

4. **自定义操作**
   你可以在折叠中嵌入任意的逻辑，例如统计特定条件满足的数量：
   ```cpp
   template<typename... Args>
   int count_positives(Args... args) {
       return ((args > 0) + ...); // 左折叠统计正数个数
   }

   int main() {
       std::cout << count_positives(-1, 2, 0, 5, -3) << std::endl; // 输出 2
   }
   ```

---

## 优势

1. **简洁性**
   - 取代手动展开参数包或编写递归逻辑，代码更清晰。

2. **编译期展开**
   - 折叠表达式在编译期处理，效率高，没有运行时开销。

3. **灵活性**
   - 支持多种操作符和逻辑，适合处理复杂的变长参数。

---

折叠表达式是一种非常强大、灵活的工具，尤其适合需要操作变长参数的泛型编程场景。通过掌握它，你可以简化代码、提升可读性，并最大化利用编译期优化。
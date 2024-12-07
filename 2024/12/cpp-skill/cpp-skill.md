# c++隐藏魔法： 那些鲜为人知的奇淫技巧

##  1.通过 std::enable_if 实现条件模板实例化

```std::enable_if``` 可以帮助你根据类型特征启用或禁用特定的模板函数。它是C++中一种非常强大的SFINAE（Substitution Failure Is Not An Error）技巧，能够根据传入的类型条件选择函数重载。
示例：

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_integral<T>::value>::type print(T value) {
    std::cout << "Integral: " << value << std::endl;
}

template <typename T>
typename std::enable_if<std::is_floating_point<T>::value>::type print(T value) {
    std::cout << "Floating point: " << value << std::endl;
}

int main() {
    print(42);       // 输出 Integral: 42
    print(3.14);     // 输出 Floating point: 3.14
}
```


## 通过 2.std::move_iterator 优化容器间的转移操作

```std::move_iterator``` 可以在遍历容器时避免不必要的拷贝，通过移动元素来提高效率。这在大量数据移动（而非拷贝）时非常有用。

```cpp
#include <iostream>
#include <vector>
#include <iterator>
#include <string>

int main() {
    std::vector<std::string> src = {"1", "2", "3", "4"};
    std::vector<std::string> dest;
    dest.reserve(src.size());

    std::move_iterator<std::vector<std::string>::iterator> begin(src.begin());
    std::move_iterator<std::vector<std::string>::iterator> end(src.end());
    std::copy(begin, end, std::back_inserter(dest));

    for (auto& x : src) {
        std::cout << x << " ";  // 
    }
    
    for (auto& x : dest) {
        std::cout << x << " ";  // 输出 1 2 3 4
    }
}
```

## 使用 3.std::source_location 提供丰富的调试信息

C++20引入了 std::source_location，它允许在程序运行时获取代码的文件名、行号和函数名等信息。你可以用它来方便地调试，特别是在日志输出时提供详细的源代码信息。

```cpp
#include <iostream>
#include <source_location>

void log(const std::string& message, const std::source_location location = std::source_location::current()) {
    std::cout << "File: " << location.file_name()
              << ", Line: " << location.line()
              << ", Function: " << location.function_name() << "\n"
              << message << std::endl;
}

int main() {
    log("An error occurred!");
}
```

## 通过 4.std::tuple 和 std::index_sequence 实现变参模板优化

std::tuple 和 std::index_sequence 允许你在模板中传递和操作多个参数。通过结合这些工具，可以实现变参模板的高效处理，特别是在需要逐一操作每个参数时。

```cpp
#include <iostream>
#include <tuple>
#include <utility>

// 用来打印单个元组元素
template <typename Tuple, std::size_t... Indices>
void print_tuple(const Tuple& t, std::index_sequence<Indices...>) {
    // 展开每个索引，调用 std::get<Indices>(t) 获取元组的元素
    (..., (std::cout << std::get<Indices>(t) << " ")); // C++17 的折叠表达式
}

// 对外接口：隐藏了索引生成的逻辑
template <typename... Args>
void print(Args&&... args) {
    // 创建元组，自动生成索引序列，并调用 print_tuple
    print_tuple(std::make_tuple(std::forward<Args>(args)...), 
                std::index_sequence_for<Args...>{});
}

int main() {
    print(1, 2.5, "Hello", 'C'); // 输出：1 2.5 Hello C
    return 0;
}

```

## 5.在模板元编程中使用if constexpr进行编译时switch

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void printType(T value) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "Integer: " << value << '\n';
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Float: " << value << '\n';
    } else {
        std::cout << "Other: " << value << '\n';
    }
}

int main() {
    printType(42);        // Integer
    printType(3.14);      // Float
    printType("C++!");    // Other
}
```

## 6.SFINAE：检测类是否有某成员函数

通过 SFINAE（Substitution Failure Is Not An Error）机制，可以检测某个类是否有特定的成员函数。

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
class HasToString {
private:
    template <typename U>
    static auto test(int) -> decltype(std::declval<U>().toString(), std::true_type{});

    template <typename>
    static std::false_type test(...);

public:
    static constexpr bool value = decltype(test<T>(0))::value;
};

struct A {
    void toString() {}
};

struct B {};

int main() {
    std::cout << "A has toString: " << HasToString<A>::value << "\n"; // true
    std::cout << "B has toString: " << HasToString<B>::value << "\n"; // false
    return 0;
}

```

## 7.RAII：std::shared_ptr 自定义析构函数

可以利用 std::shared_ptr 的自定义析构函数，管理复杂资源，比如文件句柄、锁等。

```cpp
#include <iostream>
#include <memory>
#include <cstdio>

struct FileDeleter {
    void operator()(FILE* file) const {
        if (file) {
            std::cout << "Closing file\n";
            fclose(file);
        }
    }
};

int main() {
    std::shared_ptr<FILE> file(fopen("example.txt", "w"), FileDeleter{});
    if (file) {
        fputs("Hello, World!", file.get());
    }
    return 0;
}

```

## 8.编译期计算：constexpr 求解递归问题

使用 constexpr 编写可以在编译期执行的递归算法。

```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

int main() {
    constexpr int result = factorial(5);
    std::cout << "Factorial of 5: " << result << "\n";
    return 0;
}

```

## 9.隐藏模板参数中的数据

通过类型参数“隐藏”编译期数据。

```cpp
#include <iostream>

template <int N>
struct HiddenData {
    static constexpr int value = N;
};

template <typename T>
void reveal() {
    std::cout << "Hidden value: " << T::value << "\n";
}

int main() {
    reveal<HiddenData<42>>(); // 输出：Hidden value: 42
    return 0;
}

```

## 10. 编译期字符串计算

通过模板元编程，在编译期计算字符串的长度或其他属性。

```cpp
#include <iostream>
#include <array>

template <std::size_t N>
constexpr std::size_t constexpr_strlen(const char (&str)[N]) {
    return N - 1; // 去掉 '\0'
}

int main() {
    constexpr auto len = constexpr_strlen("Hello, World!");
    static_assert(len == 13, "Length mismatch");
    std::cout << "Length: " << len << "\n";
    return 0;
}
```
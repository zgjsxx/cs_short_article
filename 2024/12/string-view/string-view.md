# string_view

`std::string_view` 是 C++17 中引入的一个轻量级的字符串视图类，它提供了一种无需拷贝的方式来操作字符串数据。通过 `std::string_view`，你可以在不修改原始数据的情况下访问和操作一个字符串的片段或整段内容。它主要用于优化性能，避免在字符串传递过程中不必要的内存拷贝。

## 基本概念

`std::string_view` 并不拥有其指向的字符数据。它只是指向一个已有的字符串数据的视图（即一个轻量级的非拥有型指针），并且持有字符串的大小信息。这样你可以在不持有实际数据的情况下对字符串进行操作。

## 1. 定义和构造

`std::string_view` 是一个模板类，通常用于 `char` 类型的字符串：

```cpp
#include <string_view>

std::string_view sv("Hello, World!");
```

这里，`sv` 是一个指向字面量字符串 `"Hello, World!"` 的视图。它实际上并没有存储这个字符串的数据，只是通过一个指针指向它，并持有字符串的长度信息。

`std::string_view` 的构造函数支持从以下几种类型创建：

- **C 风格字符串** (`const char*`)
- **`std::string`**
- **`std::string_view`** 本身
- **字符数组和子数组**

例如：

```cpp
const char* cstr = "Hello";
std::string_view sv1(cstr);    // 从 C 风格字符串构造

std::string str = "World";
std::string_view sv2(str);     // 从 std::string 构造

std::string_view sv3(sv2);     // 从另一个 string_view 构造
```

## 2. 访问字符

`std::string_view` 提供了与普通字符串类似的接口来访问字符数据：

```cpp
std::string_view sv("Hello, World!");
char ch = sv[0];  // 获取第一个字符，'H'
```

你可以像普通数组一样使用索引访问字符串内容，但请注意，`std::string_view` 不会对越界访问进行检查。

## 3. 切割字符串

`std::string_view` 允许你创建字符串的子视图而无需实际拷贝数据。你可以使用 `substr` 来获取一个子字符串视图：

```cpp
std::string_view sv("Hello, World!");
std::string_view sv2 = sv.substr(7, 5);  // 获取 "World"
```

这里，`substr` 创建了一个从位置 7 开始、长度为 5 的子视图。重要的是，`sv2` 并没有存储任何数据，只是指向原字符串数据的一个新视图。

## 4. 转换为其他类型

虽然 `std::string_view` 是一个非拥有类型，它可以方便地与其他类型进行转换。例如，使用 `std::string_view` 创建 `std::string` 或 C 风格字符串：

```cpp
std::string_view sv("Hello, World!");
std::string str(sv);  // 转换为 std::string
const char* cstr = sv.data();  // 获取 C 风格字符串指针
```

## 5. 比较操作

`std::string_view` 支持标准的比较操作符，如 `==`, `!=`, `<`, `>`, `<=`, `>=`：

```cpp
std::string_view sv1("Hello");
std::string_view sv2("Hello");
std::string_view sv3("World");

if (sv1 == sv2) {
    std::cout << "sv1 and sv2 are equal\n";
}

if (sv1 != sv3) {
    std::cout << "sv1 and sv3 are not equal\n";
}
```

## 6. 优势

### 6.1. 性能

`std::string_view` 的最大优势在于避免了字符串拷贝。当你需要传递一个字符串或者操作字符串的某个子区域时，使用 `std::string_view` 可以避免不必要的内存分配和拷贝，尤其是在处理大量字符串时，性能优势更为明显。

`std::string_view` 是 C++ 中非常强大的工具，它提供了一个非拥有的、轻量级的方式来查看和操作字符串数据。它可以提高性能，避免不必要的内存拷贝，特别是在大型文本处理或者需要频繁访问字符串的场景中非常有用。不过，使用时需要小心确保字符串数据的生命周期，以及了解它并不支持修改内容。

下面这个例子通过性能测试比较了 ```std::string_view``` 和 ```std::string``` 在进行字符串截取 (substr) 操作时的效率。程序创建了一个包含100万字符的字符串，并对其进行多次截取操作。分别使用 ```std::string_view``` 和 ```std::string``` 进行截取，并记录每种方式所需的时间，最后输出两者的性能对比。```std::string_view``` 在执行过程中表现得更为高效，因为它避免了额外的内存分配。

```cpp
#include <iostream>
#include <string>
#include <string_view>
#include <chrono>

int main() {
    // 创建一个大字符串
    const std::string largeString(1'000'000, 'A'); // 长度为100万的字符串
    const size_t substrStart = 500'000;           // 截取的起始位置
    const size_t substrLength = 100;              // 截取的长度

    // 使用 std::string_view 进行 substr
    auto start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < 1'000'000; ++i) {
        std::string_view sv(largeString);
        auto svSub = sv.substr(substrStart, substrLength);
    }
    auto end = std::chrono::high_resolution_clock::now();
    auto stringViewTime = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
    std::cout << "std::string_view substr total time: " << stringViewTime << " microseconds" << std::endl;

    // 使用 std::string 进行 substr
    start = std::chrono::high_resolution_clock::now();
    for (int i = 0; i < 1'000'000; ++i) {
        auto stringSub = largeString.substr(substrStart, substrLength);
    }
    end = std::chrono::high_resolution_clock::now();
    auto stringTime = std::chrono::duration_cast<std::chrono::microseconds>(end - start).count();
    std::cout << "std::string substr total time: " << stringTime << " microseconds" << std::endl;

    // 打印性能对比
    std::cout << "Performance comparison (string_view vs string): "
              << (double)stringTime / stringViewTime << "x slower for std::string" << std::endl;

    return 0;
}
```


```shell
std::string_view substr total time: 27214 microseconds
std::string substr total time: 53279 microseconds
Performance comparison (string_view vs string): 1.95778x slower for std::string
```

### 6.2. 灵活性

`std::string_view` 允许你访问常量字符串、`std::string` 或其他字符数组而无需拷贝数据。这使得它成为库函数设计中的一个非常有用的工具。

### 6.3. 适用场景

- **作为函数参数**：传递大型字符串时，通过 `std::string_view` 传递可以避免拷贝数据，同时允许函数灵活处理各种字符串类型。
- **子字符串提取**：你可以高效地提取原字符串的子视图，而不必进行拷贝操作。
- **非拥有的视图**：当你不需要拥有字符串数据，仅仅需要读取和查看数据时，`std::string_view` 是理想的选择。

## 7. 注意事项

尽管 `std::string_view` 提供了很多优点，但在使用时需要特别小心：

### 7.1. 数据生命周期

`std::string_view` 本身并不拥有它指向的数据，因此，使用时需要确保数据的生命周期足够长。例如，如果你创建了一个 `std::string_view`，并指向一个局部变量或临时字符串，确保该字符串的生命周期在 `std::string_view` 使用期间没有结束。

```cpp
std::string_view get_view() {
    std::string str = "temporary";
    return std::string_view(str);  // 错误：str 可能在返回后被销毁
}
```

在这个例子中，`str` 是一个局部变量，`std::string_view` 会在 `str` 被销毁后变成悬空指针。

### 7.2. 不支持修改内容

`std::string_view` 是不可修改的视图。你不能通过 `string_view` 来修改原始数据。如果你需要修改字符串内容，应该使用 `std::string` 或者其他可修改的容器。

### 7.3. 需要显式转化为 `std::string` 时才会拷贝

虽然 `std::string_view` 提供了高效的操作和视图，但当你确实需要在某些操作中对数据进行修改或需要一个拥有数据的字符串时，仍然需要将 `std::string_view` 转换为 `std::string`，而这将触发数据的拷贝。

## 8. 代码示例

```cpp
#include <iostream>
#include <string_view>

void print_view(std::string_view sv) {
    std::cout << "String view: " << sv << std::endl;
}

int main() {
    std::string_view sv1 = "Hello, ";
    std::string_view sv2 = "World!";
    
    print_view(sv1);  // 输出：Hello, 
    print_view(sv2);  // 输出：World!

    std::string_view sv3 = sv1.substr(0, 5);  // 获取 "Hello"
    print_view(sv3);  // 输出：Hello
    
    // 合并两个 string_view
    std::string_view sv4 = sv1.substr(0, 5) + sv2.substr(0, 5);  // 注意：这会创建一个新的 string
    print_view(sv4);  // 输出：HelloW
}
```

### 总结

```std::string_view``` 是 C++ 中非常强大的工具，它提供了一个非拥有的、轻量级的方式来查看和操作字符串数据。它可以提高性能，避免不必要的内存拷贝，特别是在大型文本处理或者需要频繁访问字符串的场景中非常有用。不过，使用时需要小心确保字符串数据的生命周期，以及了解它并不支持修改内容。
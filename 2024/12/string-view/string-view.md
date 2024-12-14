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
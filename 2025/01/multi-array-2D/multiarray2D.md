# 用一维向量玩转二维世界：解密高效矩阵模拟的神器 `MultiArray2D`

## 引言

在实际开发中，二维数组或矩阵是一个常见的数据结构，尤其是在图像处理、数值计算等领域。然而，C++ 原生的二维数组使用起来较为繁琐，尤其是在动态分配大小或需要进行高级操作时。因此，为了提高灵活性和性能，使用一维 `std::vector` 模拟二维数组是一种常见且高效的实现方式。

本文将介绍一个基于一维 `std::vector` 模拟二维矩阵的实现——`MultiArray2D`，并探讨这种实现方式的优点和设计背后的思考。

## 代码结构概述

### 1. 基础实现

代码实现了一个模板类 `MultiArray2D`，可以用于存储任意类型的数据（包括 `bool` 类型的特化版本）。其核心思想是使用一个一维向量 `std::vector<T>` 来存储二维矩阵的数据，通过行列索引的数学映射将二维坐标转换为一维索引，从而实现对二维数据的模拟。

```cpp
size_t index = row * cols_ + col;
```

通过这种映射，任意位置 `(row, col)` 的元素可以直接映射到 `std::vector` 的第 `index` 个位置。这种方式既简单又高效。

### 2. 功能实现
`MultiArray2D` 提供了一些重要的功能，包括：
- **动态调整矩阵大小**：支持重新调整矩阵行列数，同时可以选择是否保留旧数据。
- **矩阵清理和压缩**：提供清理数据（`clear`）和释放多余内存（`shrink_to_fit`）的功能。
- **值填充和遍历**：可以将整个矩阵填充为一个指定值，并通过重载的 `[]` 操作符支持直观的访问方式。
- **行代理类 `RowProxy`**：通过 `RowProxy` 提供类似二维数组的访问语法，例如 `array[row][col]`。

### 3. `bool` 特化版本

为了优化 `bool` 类型的存储和访问，代码对 `MultiArray2D<bool>` 进行了特化。`std::vector<bool>` 实现了一种位图存储方式，因此需要单独处理访问和操作逻辑。

## 为什么选择一维 `std::vector` 实现二维矩阵？

### 1. 内存连续性
- 使用一维 `std::vector`，所有数据存储在连续的内存区域中。这种存储方式对性能非常友好，尤其是在需要大量数据访问或操作时，可以充分利用 CPU 缓存。
- 与传统的二维动态数组相比，避免了额外的内存碎片问题。

### 2. 灵活性与可扩展性
- 动态调整矩阵大小时，一维 `std::vector` 的行为非常直观。通过 `resize` 方法，可以方便地扩展或缩减矩阵，而不需要显式地分配和释放内存。
- 可以轻松实现跨平台支持，而无需担心底层内存分配细节。

### 3. 符合现代 C++ 设计
- 现代 C++ 鼓励使用标准库容器代替原生指针管理内存。`std::vector` 提供了强大的功能，如自动管理内存、支持移动语义和异常安全性。
- 使用 `std::vector` 还可以借助 STL 提供的算法（如 `std::fill` 和 `std::copy`）实现常见的操作，大幅减少开发工作量。

### 4. 支持泛型和特化
- 通过模板实现，`MultiArray2D` 可以支持任意数据类型，具有良好的通用性。
- 对于特殊类型（如 `bool`），可以通过特化提供针对性的优化，进一步提升效率和灵活性。

## 代码亮点

### 1. 简洁高效的索引映射
核心逻辑是通过行列索引的简单数学计算完成一维到二维的映射：

```cpp
size_t index = row * cols_ + col;
```

这种计算方式的时间复杂度是 \(O(1)\)，非常高效。

### 2. 动态调整矩阵大小
`resize` 方法允许用户在不固定矩阵大小的情况下，根据需求动态调整行列数。可以选择是否保留旧数据：

```cpp
void resize(size_t new_rows, size_t new_cols, const T& default_value = T(), bool keep_old_data = false);
```

如果选择保留旧数据，代码会将原矩阵中的数据复制到新的矩阵中：

```cpp
for(size_t i = 0; i < std::min(rows_, new_rows); ++i) {
    for(size_t j = 0; j < std::min(cols_, new_cols); ++j) {
        new_data[i * new_cols + j] = data_[i * cols_ + j];
    }
}
```

### 3. 特化优化 `bool` 类型

`std::vector<bool>` 是一个特殊的 STL 实现，使用位图存储 `bool` 值。虽然节省了空间，但也引入了一些访问复杂性。因此，`RowProxy` 对 `bool` 类型进行了单独处理：

```cpp
std::vector<bool>::reference operator[](size_t col);
```

通过这种设计，既保持了操作的一致性，又充分利用了位图存储的优点。

## 应用场景与性能分析

### 应用场景
- **图像处理**：矩阵通常用于存储像素值，可以利用 `MultiArray2D` 高效操作图像数据。
- **科学计算**：例如存储和操作稀疏矩阵或密集矩阵。
- **表格数据**：可以方便地实现高效存储和访问。

### 性能分析
1. **时间复杂度**：所有基本操作（如元素访问、填充、调整大小）均为 \(O(1)\) 或 \(O(n)\)，与数据量成线性比例。
2. **空间效率**：连续存储方式不仅减少了内存开销，还提升了缓存命中率，从而提高了访问速度。

---

## 示例代码运行说明

在主函数中，代码演示了 `MultiArray2D` 的基本用法，包括创建矩阵、设置元素值、访问元素值以及检查结果的正确性。

```cpp
MultiArray2D<bool> array(65536, 1024);
for(size_t i = 0; i < array.rows(); ++i) {
    for(size_t j = 0; j < array.cols(); ++j) {
        array[i][j] = (i + j) % 2 == 1;
    }
}
```

通过简单的循环填充和验证，展示了其高效性和正确性。

## 总结

`MultiArray2D` 使用一维 `std::vector` 模拟二维矩阵，为现代 C++ 提供了一种高效、灵活的矩阵实现方式。通过优化存储和访问方式，它在内存管理和性能上优于传统的二维数组。同时，模板化设计和特化支持不同类型的数据需求，极大地增强了代码的可重用性和适用性。

这种实现方式非常适合需要大规模矩阵操作、对性能要求高的场景，是现代 C++ 编程中值得推荐的解决方案。


## 附完整代码

```cpp
#include <stdint.h>
#include <vector>
#include <stdexcept>
#include <iostream>

template <typename T>
class MultiArray2D {
  public:
    MultiArray2D() = default;
    MultiArray2D(size_t rows, size_t cols, const T& default_value = T())
    : rows_(rows), cols_(cols), data_(rows * cols, default_value) {}
    MultiArray2D(const MultiArray2D& other) : rows_(other.rows_), cols_(other.cols_), data_(other.data_){}

    MultiArray2D& operator=(const MultiArray2D& other){
        if(this != &other) {
            rows_ = other.rows_;
            cols_ = other.cols_;
            data_ = other.data_;
        }
        return *this;
    }

    MultiArray2D(MultiArray2D&& other) noexcept
    : rows_(other.rows_), cols_(other.cols_),data_(std::move(other.data_)){
        other.rows_ = 0;
        other.cols_ = 0;
    }

    MultiArray2D& operator=(MultiArray2D&& other) noexcept{
        if(this != &other) {
            rows_ = other.rows_;
            cols_ = other.cols_;
            data_ = std::move(other.data_);
            other.rows_ = 0;
            other.cols_ = 0;
        }
        return *this;
    }
    [[nodiscard]] size_t rows() const{
        return rows_;
    }
    [[nodiscard]] size_t cols() const {
        return cols_;
    }
    [[nodiscard]] bool empty() const {
        return data_.empty();
    }

    void fill(const T& value) {
        std::fill(data_.begin(), data_.end(), value);
    }
    void resize(size_t new_rows, size_t new_cols, const T& default_value = T(), bool keep_old_data = false){
        std::vector<T> new_data(new_rows * new_cols, default_value);
        if(!keep_old_data){
            for(size_t i = 0;i < std::min(rows_, new_rows); ++i) {
                for(size_t j = 0;j < std::min(cols_, new_cols); ++j) {
                    new_data[i * new_cols + j] = data_[i * cols_ + j];
                }
            }
        }
        rows_ = new_rows;
        cols_ = new_cols;
        data_ = std::move(new_data);
    }

    void clear() {
        rows_ = 0;
        cols_ = 0;
        data_.clear();
    }

    void shrink_to_fit() {
        data_.shrink_to_fit();
    }

    class RowProxy {
       public:
        RowProxy(T* row_data, size_t cols) : row_data_(row_data), cols_(cols){}
        T& operator[](size_t col) {
            if(col >= cols_) {
                throw std::out_of_range("Column index out of bounds");
            }
            return row_data_[col];
        }
        private:
            T* row_data_{nullptr};
            size_t cols_{0};
    };

    RowProxy operator[](size_t row) {
        if(row >= rows_) {
            throw std::out_of_range("Row index out of bounds");
        }
        return RowProxy(&data_[row * cols_], cols_);
    }

  private:
     size_t rows_{0}, cols_{0};
     std::vector<T> data_;
};

template <>
class MultiArray2D<bool> {
  public:
    MultiArray2D() = default;
    MultiArray2D(size_t rows, size_t cols, bool default_value = false)
    : rows_(rows), cols_(cols), data_(rows * cols, default_value) {}
    MultiArray2D(const MultiArray2D& other) : rows_(other.rows_), cols_(other.cols_), data_(other.data_){}

    MultiArray2D& operator=(const MultiArray2D& other){
        if(this != &other) {
            rows_ = other.rows_;
            cols_ = other.cols_;
            data_ = other.data_;
        }
        return *this;
    }

    MultiArray2D(MultiArray2D&& other)  noexcept
    : rows_(other.rows_), cols_(other.cols_),data_(std::move(other.data_)){
        other.rows_ = 0;
        other.cols_ = 0;
    }

    MultiArray2D& operator=(MultiArray2D&& other) noexcept{
        if(this != &other) {
            rows_ = other.rows_;
            cols_ = other.cols_;
            data_ = std::move(other.data_);
            other.rows_ = 0;
            other.cols_ = 0;
        }
        return *this;
    }
    [[nodiscard]] size_t rows() const{
        return rows_;
    }
    [[nodiscard]] size_t cols() const {
        return cols_;
    }
    [[nodiscard]] bool empty() const {
        return data_.empty();
    }

    void fill(bool value) {
        std::fill(data_.begin(), data_.end(), value);
    }
    void resize(size_t new_rows, size_t new_cols, bool default_value = false, bool keep_old_data = false){
        std::vector<bool> new_data(new_rows * new_cols, default_value);
        if(!keep_old_data){
            for(size_t i = 0;i < std::min(rows_, new_rows); ++i) {
                for(size_t j = 0;j < std::min(cols_, new_cols); ++j) {
                    new_data[i * new_cols + j] = data_[i * cols_ + j];
                }
            }
        }
        rows_ = new_rows;
        cols_ = new_cols;
        data_ = std::move(new_data);
    }

    void clear() {
        rows_ = 0;
        cols_ = 0;
        data_.clear();
    }

    void shrink_to_fit() {
        data_.shrink_to_fit();
    }

    class RowProxy {
       public:
        RowProxy(std::vector<bool>& row_data, size_t offset, size_t cols)
         : row_data_(row_data), offset_(offset), cols_{cols} {}

        std::vector<bool>::reference operator[](size_t col) {
            if(col >= cols_) {
                throw std::out_of_range("Column index out of bounds");
            }
            return row_data_[offset_ + col];
        }
        private:
            std::vector<bool>& row_data_;
            size_t offset_{0}, cols_{0};
    };
    RowProxy operator[](size_t row) {
        if(row >= rows_) {
            throw std::out_of_range("Row index out of bounds");
        }
        return RowProxy(data_, row * cols_, cols_);
    }
  private:
     size_t rows_{0}, cols_{0};
     std::vector<bool> data_;
};


int main() {
    MultiArray2D<bool> array(65536, 1024);
    for(size_t i = 0; i < array.rows(); ++i) {
        for(size_t j = 0; j < array.cols(); ++j) {
            if((i + j) % 2 == 1) {
                array[i][j] = true;
            } else {
                array[i][j] = false;
            }
        }
    }

    for(size_t i = 0; i < array.rows(); ++i) {
        for(size_t j = 0; j < array.cols(); ++j) {
            if((i + j) % 2 == 1) {
                if(array[i][j] != true){
                    std::cout << "error"<< std::endl;
                }
            } else {
                if(array[i][j] != false){
                     std::cout << "error"<< std::endl;
                }
            }
        }
    }

    MultiArray2D<int> array2(65536, 1024);
    for(size_t i = 0; i < array2.rows(); ++i) {
        for(size_t j = 0; j < array2.cols(); ++j) {
            if((i + j) % 2 == 1) {
                array2[i][j] = 10;
            } else {
                array2[i][j] = 0;
            }
        }
    }

    for(size_t i = 0; i < array2.rows(); ++i) {
        for(size_t j = 0; j < array2.cols(); ++j) {
            if((i + j) % 2 == 1) {
                if(array2[i][j] != 10){
                    std::cout << "error"<< std::endl;
                }
            } else {
                if(array2[i][j] != 0){
                     std::cout << "error"<< std::endl;
                }
            }
        }
    }
    std::cout << "finish" << std::endl;
}
```
# ks 库编码规范（正式定稿·含附录示例）

## 1. 头文件
- 使用 `#pragma once` 作为头文件保护，不使用 `#ifndef` 宏保护。

## 2. 缩进与格式
- 命名空间 `ks` 内部**不缩进**。
- 左大括号**不换行**。
- 条件、循环、关键字与括号之间留空格。
- 函数与函数之间空一行。

## 3. 函数写法
- 除 `void` 外，统一使用**后置返回类型**：
  ```cpp
  auto func() -> int;
  ```
- `void` 函数保持传统写法：
  ```cpp
  void func();
  ```

## 4. 命名规则
- 变量、函数：小写 + 下划线 `snake_case`
- 类型、类、结构体：大驼峰 `PascalCase`
- 私有成员变量：末尾加下划线 `data_`

## 5. 变量与类型
- 优先使用 `auto` 推导变量类型，不使用冗余的 `auto*`。
- 固定大小数组**优先使用 C 风格数组**，不建议使用 `std::array`。
- 尽量不使用全局变量。

## 6. 类与成员布局
- 类内部按**逻辑顺序**排列，不按 `public/private` 分段：
  1. 私有字段
  2. 构造函数 / 析构函数
  3. 公开接口
  4. 私有方法
- **不推荐使用虚函数**，优先使用静态派发。

## 7. 内存与所有权
- 使用**智能指针/所有权指针**管理动态内存。
- 尽量减少裸指针；如必须使用，明确是观察而非持有。

## 8. 类型转换（重要）
- **禁止使用 C 风格强制转换 `(Type)expr`**。
- 统一使用 C++ 显式转换：
  - `static_cast`：安全、普通类型转换
  - `reinterpret_cast`：底层指针/内存重解释（高危，需注释）
  - `const_cast`：仅用于兼容 C 接口等极端场景

## 9. 错误处理
- **禁止使用 C++ 异常与 try-catch**。
- 所有可失败操作统一返回 `ks::Result`。

## 10. 其他约束
- 不使用宏定义常量或函数，尽量使用 `constexpr`/`enum`。
- 减少模板元编程深度，保持编译速度与可读性。
- 接口保持简洁稳定，内部实现可自由优化。

---

# 附录 A：完整示例代码（严格遵循本规范）

## example.h
```cpp
#pragma once

#include <cstddef>
#include <memory>

namespace ks {

struct Result {
    bool ok;
    int  code;

    static auto make_ok() -> Result;
    static auto make_err(int code) -> Result;
};

class Buffer {
    // 1. 私有字段（在前）
    std::byte* data_;
    size_t     size_;
    size_t     capacity_;

    // 2. 构造 / 析构
public:
    Buffer();
    explicit Buffer(size_t init_capacity);
    ~Buffer();

    // 禁止拷贝，简化所有权
    Buffer(const Buffer&)            = delete;
    auto operator=(const Buffer&) -> Buffer& = delete;

    // 允许移动
    Buffer(Buffer&& other) noexcept;
    auto operator=(Buffer&& other) noexcept -> Buffer&;

    // 3. 公开接口
    auto write(size_t offset, const std::byte* src, size_t len) -> Result;
    auto resize(size_t new_size) -> Result;
    auto data() const -> const std::byte*;
    auto size() const -> size_t;

    // 4. 私有方法
private:
    auto reallocate(size_t new_capacity) -> bool;
};

auto add(int a, int b) -> int;
void print_hello();

} // namespace ks
```

## example.cpp
```cpp
#include "example.h"
#include <cstdlib>
#include <cstring>
#include <iostream>

namespace ks {

// Result
auto Result::make_ok() -> Result {
    return {true, 0};
}

auto Result::make_err(int code) -> Result {
    return {false, code};
}

// Buffer
Buffer::Buffer()
    : data_(nullptr), size_(0), capacity_(0) {}

Buffer::Buffer(size_t init_capacity)
    : data_(nullptr), size_(0), capacity_(init_capacity) {
    if (capacity_ > 0) {
        data_ = static_cast<std::byte*>(std::malloc(capacity_));
    }
}

Buffer::~Buffer() {
    if (data_) {
        std::free(data_);
    }
}

Buffer::Buffer(Buffer&& other) noexcept
    : data_(other.data_),
      size_(other.size_),
      capacity_(other.capacity_) {
    other.data_     = nullptr;
    other.size_     = 0;
    other.capacity_ = 0;
}

auto Buffer::operator=(Buffer&& other) noexcept -> Buffer& {
    if (this == &other) {
        return *this;
    }
    if (data_) {
        std::free(data_);
    }
    data_     = other.data_;
    size_     = other.size_;
    capacity_ = other.capacity_;

    other.data_     = nullptr;
    other.size_     = 0;
    other.capacity_ = 0;
    return *this;
}

auto Buffer::write(size_t offset, const std::byte* src, size_t len) -> Result {
    if (!data_ || offset + len > size_) {
        return Result::make_err(1);
    }
    std::memcpy(data_ + offset, src, len);
    return Result::make_ok();
}

auto Buffer::resize(size_t new_size) -> Result {
    if (new_size > capacity_ && !reallocate(new_size)) {
        return Result::make_err(2);
    }
    size_ = new_size;
    return Result::make_ok();
}

auto Buffer::data() const -> const std::byte* {
    return data_;
}

auto Buffer::size() const -> size_t {
    return size_;
}

auto Buffer::reallocate(size_t new_capacity) -> bool {
    auto* new_data = static_cast<std::byte*>(std::malloc(new_capacity));
    if (!new_data) {
        return false;
    }
    if (data_) {
        std::memcpy(new_data, data_, size_);
        std::free(data_);
    }
    data_      = new_data;
    capacity_  = new_capacity;
    return true;
}

// free function
auto add(int a, int b) -> int {
    return a + b;
}

void print_hello() {
    std::cout << "hello from ks\n";
}

} // namespace ks
```
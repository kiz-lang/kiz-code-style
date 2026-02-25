# kiz modern C++ 库编码规范

## 1. 头文件
- 使用 `#pragma once` 作为头文件保护，不使用 `#ifndef` 宏保护。

## 2. 缩进与格式
- 命名空间 `ks` 内部**不缩进**。
- 左大括号**不换行**。
- 条件、循环、关键字与括号之间留空格。
- 函数与函数之间空一行。

## 3. 命名空间与模块
- **不允许使用 `inline namespace`**，会降低代码可读性与结构清晰度。

## 4. 函数写法
- 除 `void` 外，统一使用**后置返回类型**：
```cpp
auto func() -> int;
```
- `void` 函数可以保持传统写法：
```cpp
void func();
```

## 5. 命名规则
- 变量、函数：小写 + 下划线 `snake_case`
- 类型、类、结构体：大驼峰 `PascalCase`
- 私有成员变量：末尾加下划线 `data_`

## 6. 变量与类型
- 优先使用 `auto` 推导变量类型，不使用冗余的 `auto*`。
- 固定大小数组**优先使用 C 风格数组**，不建议使用 `std::array`。
- 尽量不使用全局变量。
- **优先使用定长/明确大小类型**：
  `uint8_t`、`int32_t`、`size_t`、`uintptr_t` 等，
  **不建议使用模糊长度的原始类型**：`int`、`long`、`long long` 等。

## 7. 类与成员布局
- 类内部按**逻辑顺序**排列，不按 `public/private/protected` 分段：
  1. 私有字段
  2. 公开字段和常量
  3. 构造函数 / 析构函数
  4. 公开接口
  5. 私有方法
- **不推荐使用虚函数**，优先使用静态派发。

## 8. 内存与所有权
- 使用**智能指针/所有权指针**管理动态内存。
- 尽量减少裸指针；如必须使用，明确是观察而非持有。

## 9. 类型转换（重要）
- **禁止使用 C 风格强制转换 `(Type)expr`**。
- 统一使用 C++ 显式转换：
  - `static_cast`：安全、普通类型转换
  - `reinterpret_cast`：底层指针/内存重解释（高危，需注释）
  - `const_cast`：仅用于兼容 C 接口等极端场景

## 10. 错误与安全
- **不推荐使用 C++ 异常与 try-catch**。
- 所有可失败操作统一返回 `ks::Result`。
- **不推荐使用 RTTI**：不推荐使用 `dynamic_cast`、`typeid`。
- **优先使用断言保证前提**，而不是运行时检查。
- 数组访问优先 `a[0]`，**不使用 `a.at(0)`**（效率低）。

## 11. 常量与枚举
- **多使用 `enum class` / `constexpr`**，
  禁止魔法数字、禁止滥用宏定义常量/函数。

## 12. 注释与文档
- 文档统一使用 `///` 精简注释，
  只写**作用 + 副作用 + 关键前提**，不写冗余长文本。
- 不使用冗长 `/**/` 块注释。

```cpp
/// 把 x 加到内部计数器，无溢出检查
auto add_count(uint32_t x) -> uint32_t;
```

---

# 附录 A：完整示例代码（严格遵循本规范）

## example.h
```cpp
#pragma once

#include <cstddef>
#include <cstdint>
#include <memory>
#include <cassert>

namespace ks {

/// 操作结果：成功/错误码
struct Result {
    bool ok;
    int  code;

    static auto make_ok() -> Result;
    static auto make_err(int code) -> Result;
};

/// 错误类型枚举
enum class ErrorCode : int32_t {
    None      = 0,
    BadOffset = 1,
    NoMemory  = 2,
};

/// 固定容量字节缓冲区
class Buffer {
    // 1. 私有字段
    std::byte* data_;
    size_t     size_;
    size_t     capacity_;

    // 2. 构造 / 析构
public:
    Buffer();
    explicit Buffer(size_t init_capacity);
    ~Buffer();

    Buffer(const Buffer&)            = delete;
    auto operator=(const Buffer&) -> Buffer& = delete;

    Buffer(Buffer&& other) noexcept;
    auto operator=(Buffer&& other) noexcept -> Buffer&;

    // 3. 公开接口
    /// 写入数据到 offset，不扩容；越界返回错误
    auto write(size_t offset, const std::byte* src, size_t len) -> Result;

    /// 调整逻辑大小，必要时重分配
    auto resize(size_t new_size) -> Result;

    /// 获取底层数据指针
    auto data() const -> const std::byte*;

    /// 有效字节数
    auto size() const -> size_t;

    // 4. 私有方法
private:
    auto reallocate(size_t new_capacity) -> bool;
};

/// 两数相加（纯函数）
auto add(uint32_t a, uint32_t b) -> uint32_t;

/// 打印欢迎信息
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
    if (offset + len > size_) {
        return Result::make_err(static_cast<int>(ErrorCode::BadOffset));
    }
    // 断言保证内部合法性，不使用 .at()
    assert(data_ != nullptr);
    std::memcpy(data_ + offset, src, len);
    return Result::make_ok();
}

auto Buffer::resize(size_t new_size) -> Result {
    if (new_size > capacity_ && !reallocate(new_size)) {
        return Result::make_err(static_cast<int>(ErrorCode::NoMemory));
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
    auto new_data = static_cast<std::byte*>(std::malloc(new_capacity));
    if (!new_data) {
        return false;
    }
    if (data_) {
        std::memcpy(new_data, data_, size_);
        std::free(data_);
    }
    data_     = new_data;
    capacity_ = new_capacity;
    return true;
}

// free function
auto add(uint32_t a, uint32_t b) -> uint32_t {
    return a + b;
}

void print_hello() {
    std::cout << "hello from ks\n";
}

} // namespace ks
```

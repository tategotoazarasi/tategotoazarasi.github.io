---
date: '2025-04-06T16:34:33+08:00'
draft: false
summary: '一篇详细的技术指南，介绍如何使用 Wasmtime 运行时在 C++ 宿主应用程序与 Rust WebAssembly 模块之间实现复杂的双向通信、共享内存访问和结构体传递。'
title: '深入探索 Wasmtime：C++ 与 Rust Wasm 模块的双向通信与内存共享'
tags: [ "wasm", "wasmtime", "cpp", "rust", "ffi", "shared-memory", "bidirectional-communication", "host-guest", "linear-memory", "struct-passing", "state-management", "wasi", "runtime", "sandboxing", "wasmtime-api", "c-plus-plus" ]
---

今天我们来聊一个越来越火热的技术：WebAssembly（简称 Wasm）。不过，我们不把它局限在浏览器里，而是探讨如何在服务器端或者桌面应用中，利用
Wasmtime 这个运行时，让 C++ 程序能够加载和运行 Rust 编译的 Wasm 模块，并且实现它们之间复杂的交互，比如双向函数调用、共享内存、传递结构体，甚至相互修改状态。

## WebAssembly 与 Wasmtime 简介

首先，简单说说 WebAssembly 是什么。你可以把它想象成一种为现代 Web 设计的、可移植的二进制指令格式。它不是用来取代 JavaScript
的，而是作为一种强大的补充，让那些性能敏感或者需要利用底层能力的 C/C++/Rust 等语言编写的代码，也能在 Web 环境（以及其他支持
Wasm 的环境）中以接近本地的速度运行。Wasm 的核心优势在于其**沙箱化**的安全模型和**平台无关**的特性。

而 Wasmtime，则是由 Bytecode Alliance（一个由 Mozilla、Fastly、Intel、Red Hat 等公司组成的联盟）推出的一个**独立、高效、安全**的
WebAssembly 运行时。它允许你在浏览器之外的环境（比如服务器、命令行工具、嵌入式设备）中运行 Wasm 模块。Wasmtime 提供了
C、C++、Python、Rust、Go 等多种语言的 API，方便我们将 Wasm 集成到现有的应用程序中。

## 为什么选择 C++ Host + Rust Wasm？

这种组合有几个吸引人的地方：很多成熟的项目拥有庞大的 C++ 基础。通过 Wasm，可以在不重写核心逻辑的情况下，将其部分功能模块化、沙箱化，或者提供插件系统。Rust
语言以其内存安全和并发安全著称，非常适合编写需要高度可靠性的 Wasm 模块。在 Wasm 的沙箱之上，Rust 又加了一层保障。 C++ 和
Rust 都是高性能语言，编译成 Wasm 后，借助 Wasmtime 这样的 JIT 运行时，可以获得接近本地代码的执行效率。 Wasm
模块和宿主之间的交互必须通过明确定义的接口（导入/导出），这有助于维持清晰的架构。

本文的目标就是通过一个具体的例子，展示如何使用 Wasmtime 的 C++ API，搭建一个 C++ 宿主程序，加载一个用 Rust 编写的 Wasm
模块，并实现两者之间各种有趣的互动。

## 核心概念：连接 C++ 与 Wasm 的桥梁

在深入代码之前，我们需要理解几个关键概念：

### 宿主（Host）与访客（Guest）

在这个场景中，C++ 应用程序是**宿主**，它负责加载、管理和运行 Wasm 模块。Rust 编译成的 Wasm 模块则是**访客**，它运行在宿主提供的
Wasmtime 运行时环境中，受到沙箱的限制。

### Wasm 的导入（Imports）与导出（Exports）

Wasm 模块与外界通信的主要方式就是通过导入和导出。

Wasm 模块可以**导出**函数、内存、全局变量或表，供宿主或其他 Wasm 模块调用或访问。在 Rust 中，我们通常使用
`#[no_mangle] pub extern "C"` 来标记需要导出的函数。

Wasm 模块可以声明它需要从宿主环境**导入**哪些功能（通常是函数）。宿主在实例化 Wasm 模块时，必须提供这些导入项的实现。在 Rust
中，我们使用 `extern "C" { ... }` 块配合 `#[link(wasm_import_module = "...")]` 来声明导入。

这个导入/导出的机制构成了宿主与 Wasm 模块之间的接口契约。

### 线性内存（Linear Memory）

每个 Wasm 实例（通常）都有自己的一块**线性内存**。这是一块连续的、可由 Wasm 代码和宿主代码共同读写的字节数组。Wasm
代码中的指针，实际上就是这块内存区域的**偏移量**（通常是 32 位或 64 位整数）。

关键点在于，Wasm 本身是沙箱化的，它不能直接访问宿主的内存。宿主也不能随意访问 Wasm 内部的变量。但是，宿主可以通过 Wasmtime
提供的 API 获取到 Wasm 实例导出的线性内存的**访问权**（通常是一个指向内存起始位置的指针或 Span），然后直接读写这块内存。同样，Wasm
代码也可以通过调用宿主提供的函数（导入函数），间接地操作宿主的状态或资源。

这种通过共享线性内存进行数据交换的方式是 Wasm 交互的核心。传递复杂数据结构（如 C++ 的 struct 或 Rust 的
struct）通常就是通过将它们序列化到这块内存中，然后传递指向内存的指针（偏移量）来实现的。

### WASI (WebAssembly System Interface)

WASI 是一套标准化的系统接口，旨在让 Wasm 模块能够以安全、可移植的方式与底层操作系统进行交互，比如文件系统访问、网络通信、标准输入输出等。虽然我们的例子不涉及复杂的文件操作，但
Rust 标准库中的 `println!` 宏依赖于底层的标准输出功能。为了让 Wasm 模块中的 `println!` 能正常工作（将内容打印到宿主的控制台），我们需要在宿主中配置并链接
WASI 支持。

## 构建 C++ 宿主：Wasmtime 的舞台搭建者

现在，我们来看看 C++ 宿主端都需要做些什么。为了更好地组织代码，我们通常会创建一个类（比如 `WasmHost`）来封装与 Wasmtime
的交互逻辑。

### 加载与编译 Wasm 模块

第一步是读取 Wasm 模块文件（`.wasm` 二进制文件）的内容，然后使用 Wasmtime 的 `Engine` 来编译它。`Engine` 可以看作是 Wasmtime
的核心编译和执行引擎，它负责将 Wasm 字节码转换为可执行的机器码。编译的结果是一个 `Module` 对象。这个 `Module`
对象是线程安全的，可以被多个 `Store` 重用。

```cpp
// 伪代码示例 (实际代码在 wasm_host.cpp)
#include "wasmtime.hh" // 包含 Wasmtime C++ 头文件
#include <vector>
#include <fstream>
#include <stdexcept>

using namespace wasmtime;

// ... WasmHost 类的定义 ...

std::vector<uint8_t> WasmHost::readWasmFile() {
    std::ifstream file(wasm_path_, std::ios::binary | std::ios::ate);
    // ... 错误处理 ...
    std::streamsize size = file.tellg();
    file.seekg(0, std::ios::beg);
    std::vector<uint8_t> buffer(static_cast<size_t>(size));
    // ... 读取文件内容到 buffer ...
    return buffer;
}

void WasmHost::loadAndCompile() {
    std::vector<uint8_t> wasm_bytes = readWasmFile();
    std::cout << "[Host Setup] Compiling WASM module..." << std::endl;
    // engine_ 是 WasmHost 的成员变量，类型为 wasmtime::Engine
    Result<Module> module_res = Module::compile(engine_, wasm_bytes);
    if (!module_res) {
        throw std::runtime_error("Module compilation failed: " + module_res.err().message());
    }
    // module_ 也是 WasmHost 的成员变量，类型为 std::optional<wasmtime::Module>
    module_ = std::move(module_res.ok());
    std::cout << "[Host Setup] Module compiled successfully." << std::endl;
}

// 在 WasmHost 构造函数或初始化函数中调用 loadAndCompile()
```

### Engine 与 Store

`Engine` 负责编译代码，而 `Store` 则代表了一个 Wasm 实例的“世界”或者说“上下文”。所有与 Wasm 实例相关的数据，比如内存、全局变量、表，以及实例本身，都
**属于**一个特定的 `Store`。一个 `Engine` 可以关联多个 `Store`，但一个 `Store` 只与一个 `Engine` 关联。`Store`
不是线程安全的，通常一个线程对应一个 `Store`。

```cpp
// WasmHost 类成员变量
Engine engine_;
Store store_;

// WasmHost 构造函数
WasmHost::WasmHost(std::string wasm_path) : wasm_path_(std::move(wasm_path)),
                                            engine_(), // 创建默认 Engine
                                            store_(engine_) // 基于 Engine 创建 Store
{
    // ...
}
```

### 配置 WASI

如前所述，如果 Wasm 模块需要进行系统调用（比如 `println!`），我们需要为 `Store` 配置 WASI。这通常在实例化模块**之前**
完成。Wasmtime 提供了 `WasiConfig` 类来配置 WASI 的行为，比如是否继承宿主的标准输入/输出/错误流、环境变量、命令行参数等。配置好的
`WasiConfig` 需要设置到 `Store` 的上下文中。

```cpp
// WasmHost::setupWasi() 方法
void WasmHost::setupWasi() {
    // ... 检查是否已初始化或已配置 ...
    std::cout << "[Host Setup] Configuring WASI..." << std::endl;
    WasiConfig wasi;
    wasi.inherit_stdout(); // 让 Wasm 的 stdout 输出到宿主的 stdout
    wasi.inherit_stderr(); // 同上，stderr
    // store_ 是 WasmHost 的成员变量
    auto wasi_set_res = store_.context().set_wasi(std::move(wasi));
    if (!wasi_set_res) {
        throw std::runtime_error("Failed setting WASI config in store: " + wasi_set_res.err().message());
    }
    wasi_configured_ = true;
    std::cout << "[Host Setup] WASI configured for Store." << std::endl;
    // 还需要在 Linker 中定义 WASI 导入
    linkWasiImports();
}

// WasmHost::linkWasiImports() 方法
void WasmHost::linkWasiImports() {
    // ... 检查 WASI 是否配置 ...
    std::cout << "[Host Setup] Defining WASI imports in linker..." << std::endl;
    // linker_ 是 WasmHost 的成员变量，类型为 wasmtime::Linker
    auto linker_define_wasi_res = linker_.define_wasi();
     if (!linker_define_wasi_res) {
        throw std::runtime_error("Failed defining WASI imports in linker: " + linker_define_wasi_res.err().message());
    }
    std::cout << "[Host Setup] WASI imports defined." << std::endl;
}
```

### Linker：连接宿主与 Wasm 的桥梁

`Linker` 是 Wasmtime 中用于解析模块导入并将它们链接到宿主提供的实现的工具。在实例化模块之前，我们需要告诉 `Linker` 如何满足
Wasm 模块的所有导入需求。

这包括两个主要部分：

1. **链接 WASI 导入：** 如果我们配置了 WASI，需要调用 `linker_.define_wasi()`，它会自动将标准的 WASI 函数实现添加到
   `Linker` 中。
2. **链接自定义宿主函数导入：** Wasm 模块可能需要调用我们自己定义的宿主函数。我们需要将这些 C++ 函数（或 lambda）包装成
   Wasmtime 能理解的形式，并使用 `linker_.define()` 或 `linker_.func_wrap()` 将它们注册到 `Linker` 中，指定它们对应的 Wasm
   模块名（在 Rust 代码中 `#[link(wasm_import_module = "...")]` 指定的）和函数名。

### 定义可被 Wasm 调用的宿主函数

这是实现 Wasm 调用 Host 功能的关键。我们需要在 C++ 中编写实现函数，它们的签名需要与 Rust 中声明的 `extern "C"` 函数相匹配（或者
Wasmtime C++ API 可以通过模板推断进行适配）。

例如，Rust 中声明了导入：

```rust
// src/ffi.rs
#[link(wasm_import_module = "env")] // 模块名是 "env"
unsafe extern "C" {
    fn host_log_value(value: i32);
    fn host_get_shared_value() -> i32;
    fn host_set_shared_value(value: i32);
}
```

那么在 C++ 宿主中，我们需要提供这三个函数的实现，并将它们注册到 `Linker` 中，关联到 "env" 模块。

```cpp
// host.cpp
#include <iostream>
#include <cstdint>

// 宿主状态
int32_t shared_host_value = 42;

// C++ 实现函数
void host_log_value_impl_target(int32_t value) {
    std::cout << "[Host Target] host_log_value called by WASM with value: " << value << std::endl;
}

int32_t host_get_shared_value_impl_target() {
    std::cout << "[Host Target] host_get_shared_value called by WASM. Returning: " << shared_host_value << std::endl;
    return shared_host_value;
}

void host_set_shared_value_impl_target(int32_t new_value) {
    std::cout << "[Host Target] host_set_shared_value called by WASM. Old host value: "
              << shared_host_value << ", New host value: " << new_value << std::endl;
    shared_host_value = new_value; // 修改宿主状态
}

// 在 WasmHost 类或主函数中，使用 Linker 注册这些函数
// (这是 WasmHost 类中的简化版包装函数)
template <typename FuncPtr>
void WasmHost::defineHostFunction(std::string_view module_name, std::string_view func_name, FuncPtr func_ptr)
{
    if (is_initialized_) {
        throw std::logic_error("Cannot define host functions after initialization.");
    }
    std::cout << "[Host Setup] Defining host function: " << module_name << "::" << func_name << "..." << std::endl;
    // linker_ 是 WasmHost 成员变量
    auto result = linker_.func_wrap(module_name, func_name, func_ptr);
    if (!result) {
        throw std::runtime_error("Failed to define host function '" + std::string(func_name) + "': " + result.err().message());
    }
}

// 在 main 函数中调用
host.defineHostFunction("env", "host_log_value", host_log_value_impl_target);
host.defineHostFunction("env", "host_get_shared_value", host_get_shared_value_impl_target);
host.defineHostFunction("env", "host_set_shared_value", host_set_shared_value_impl_target);

```

`linker_.func_wrap()` 是一个方便的模板函数，它可以自动推断 C++ 函数的参数和返回类型，并将其转换为对应的 Wasm
函数类型，然后进行注册。这通常比手动创建 `FuncType` 并使用 `linker_.define()` 更简单。

### 实例化模块

当所有导入项（WASI 和自定义函数）都在 `Linker` 中定义好之后，我们就可以使用 `linker_.instantiate()` 来创建 Wasm
模块的一个实例 (`Instance`) 了。实例化过程会将 Wasm 代码与宿主提供的实现连接起来，并在 `Store` 中分配内存、全局变量等资源。

```cpp
// WasmHost::instantiateModule() 方法
void WasmHost::instantiateModule() {
    // ... 检查 module_ 是否有效 ...
    std::cout << "[Host Setup] Instantiating module..." << std::endl;
    // store_ 是 WasmHost 成员变量
    TrapResult<Instance> instance_res = linker_.instantiate(store_.context(), module_.value());
    if (!instance_res) {
         // 处理实例化错误（可能是链接错误或 Wasm 启动陷阱）
        throw std::runtime_error("Module instantiation failed: " + instance_res.err().message());
    }
    // instance_ 是 WasmHost 成员, 类型 std::optional<wasmtime::Instance>
    instance_ = std::move(instance_res.ok());
    std::cout << "[Host Setup] Module instantiated successfully." << std::endl;
}
```

### 访问 Wasm 线性内存

为了与 Wasm 模块交换复杂数据或直接读写其内存状态，宿主需要获取对 Wasm 实例线性内存的访问权限。Wasm
模块通常会导出一个名为 "memory" 的内存对象。我们可以通过 `instance_.get()` 来获取它。

```cpp
// WasmHost::getMemory() 方法
void WasmHost::getMemory() {
    // ... 检查 instance_ 是否有效 ...
    std::cout << "[Host Setup] Getting exported memory 'memory'..." << std::endl;
    // store_ 是 WasmHost 成员变量
    auto memory_export_opt = instance_.value().get(store_.context(), "memory");

    if (memory_export_opt && std::holds_alternative<Memory>(*memory_export_opt)) {
        // memory_ 是 WasmHost 成员, 类型 std::optional<wasmtime::Memory>
        memory_ = std::get<Memory>(*memory_export_opt);
        std::cout << "[Host Setup] Found exported memory. Size: "
                  << memory_.value().data(store_.context()).size() << " bytes." << std::endl;
    } else {
        std::cout << "[Host Setup] Export 'memory' not found or not a memory. Proceeding without memory access." << std::endl;
    }
}

// 获取内存的 Span<uint8_t>，它提供了对内存区域的视图
Span<uint8_t> WasmHost::getMemorySpan() {
    if (!is_initialized_ || !memory_.has_value()) {
        throw std::logic_error("Memory not available or host not initialized.");
    }
    return memory_.value().data(store_.context());
}
```

获取到的 `wasmtime::Memory` 对象有一个 `data()` 方法，它返回一个 `wasmtime::Span<uint8_t>`（如果 C++20 可用，就是
`std::span<uint8_t>`）。这个 Span 提供了对 Wasm 线性内存区域的直接、底层的访问接口（一个指针和大小）。有了这个
Span，我们就可以在宿主端直接读写 Wasm 的内存了。

## 构建 Wasm 模块：Rust 的安全地带

现在切换到 Rust 这边，看看 Wasm 模块是如何构建的。

### 项目结构

通常我们会将 FFI（Foreign Function Interface）相关的代码放在一个独立的模块（如 `src/ffi.rs`）中，而将核心的、安全的 Rust
逻辑放在另一个模块（如 `src/core.rs` 或直接在 `src/lib.rs` 中定义）。

`src/lib.rs` 作为库的入口，会声明并导出 `ffi` 模块中需要暴露给外部（宿主）的接口，并可能包含或调用 `core` 模块的逻辑。

```rust
// src/lib.rs
mod ffi; // 声明 ffi 模块
pub(crate) mod core; // 声明内部的 core 模块

// 重新导出 FFI 层中需要被宿主调用的函数和类型
pub use ffi::{
    Point, get_plugin_shared_value_ptr, just_add, point_add, simple_add, trigger_host_calls,
};
```

### FFI 层 (`src/ffi.rs`)

这是 Rust 与外部世界（C++ 宿主）交互的边界。

1. **声明宿主函数导入：** 使用 `extern "C"` 块和 `#[link(wasm_import_module = "env")]` 来告诉 Rust 编译器和 Wasm
   运行时，存在一些由名为 "env" 的模块提供的外部函数。这些函数的签名必须与 C++ 宿主提供的实现相匹配。注意 `extern "C"`
   块内部通常是 `unsafe` 的，因为调用外部函数无法保证 Rust 的内存安全规则。

   ```rust
   // src/ffi.rs
   #[link(wasm_import_module = "env")]
   unsafe extern "C" {
	   fn host_log_value(value: i32);
	   fn host_get_shared_value() -> i32;
	   fn host_set_shared_value(value: i32);
   }
   ```

2. **提供安全封装：** 为了避免在业务逻辑代码中到处写 `unsafe`，通常会为导入的 `unsafe` 函数提供安全的 Rust 包装函数。

   ```rust
   // src/ffi.rs
   pub fn log_value_from_host(value: i32) {
	   unsafe { host_log_value(value) } // unsafe 调用被封装在内部
   }
   // ... 其他包装函数 ...
   ```

3. **导出 Wasm 函数：** 使用 `#[no_mangle]` 防止 Rust 编译器对函数名进行混淆，并使用 `pub extern "C"` 指定 C
   语言的调用约定，使得这些函数可以被 C++ 宿主按名称查找和调用。

   ```rust
   // src/ffi.rs
   #[no_mangle] // 防止名称混淆
   pub extern "C" fn just_add(left: u64, right: u64) -> u64 {
	   println!("[WASM FFI] just_add called..."); // 使用 WASI 的 println!
	   core::perform_basic_add(left, right) // 调用核心逻辑
   }

   #[no_mangle]
   pub extern "C" fn trigger_host_calls(input_val: i32) {
	   println!("[WASM FFI] trigger_host_calls called...");
	   core::perform_host_calls_test(input_val); // 调用核心逻辑
   }
   // ... 其他导出函数 ...
   ```

### 核心逻辑层 (`src/core.rs`)

这里是实现 Wasm 模块具体功能的地方，应该尽量使用安全的 Rust 代码。它会调用 FFI 层提供的安全包装函数来与宿主交互。

```rust
// src/lib.rs (core 模块)
pub(crate) mod core {
    use crate::ffi::{ // 导入 FFI 层的安全包装器
        Point,
        get_shared_value_from_host,
        log_value_from_host,
        set_shared_value_in_host,
        // ...
    };

    pub fn perform_basic_add(left: u64, right: u64) -> u64 {
        println!("[WASM Core] perform_basic_add: {} + {}", left, right);
        left.wrapping_add(right) // 安全的加法
    }

    pub fn perform_host_calls_test(input_val: i32) {
        println!("[WASM Core] perform_host_calls_test with input: {}", input_val);
        // 调用宿主函数 (通过安全包装器)
        log_value_from_host(input_val * 2);
        let host_val = get_shared_value_from_host();
        set_shared_value_in_host(host_val + input_val + 5);
        // ...
    }
    // ... 其他核心逻辑函数 ...
}
```

### 定义共享数据结构

如果需要在 C++ 和 Rust 之间传递复杂的数据结构，必须确保两者对该结构的内存布局有相同的理解。在 Rust 中，使用 `#[repr(C)]`
属性可以强制结构体使用 C 语言兼容的内存布局。在 C++ 中，虽然编译器通常会按顺序布局，但为了绝对保险，可以使用
`#pragma pack(push, 1)` 和 `#pragma pack(pop)` 来确保紧凑（无填充）的布局，或者确保两边的对齐方式一致。

```rust
// src/ffi.rs
#[repr(C)] // 关键：保证 C 兼容布局
#[derive(Debug, Copy, Clone, Default)]
pub struct Point {
    pub x: i32,
    pub y: i32,
}
```

```cpp
// host.cpp
#pragma pack(push, 1) // 建议：确保与 Rust 端一致的紧凑布局
struct Point {
    int32_t x;
    int32_t y;
};
#pragma pack(pop)
```

### 管理 Wasm 内部状态

Wasm 模块有时也需要维护自己的状态。一种方法是使用 Rust 的 `static mut` 变量。但是，访问 `static mut` 需要 `unsafe`
块，因为它可能引入数据竞争（虽然在单线程 Wasm 环境中风险较小，但 Rust 依然要求 `unsafe`）。

```rust
// src/ffi.rs
static mut PLUGIN_SHARED_VALUE: i32 = 100; // Wasm 模块内部状态

// FFI 内部帮助函数，用于安全地读取（仍然需要 unsafe 块）
pub(crate) fn read_plugin_value_internal() -> i32 {
    unsafe { PLUGIN_SHARED_VALUE }
}

// 在 core 模块中使用
// use crate::ffi::read_plugin_value_internal;
// let val = read_plugin_value_internal();
```

如果需要让宿主能够直接修改这个状态，可以导出一个函数，返回该 `static mut` 变量的**指针**（内存偏移量）。

```rust
// src/ffi.rs
#[no_mangle]
pub unsafe extern "C" fn get_plugin_shared_value_ptr() -> *mut i32 {
    // 注意：这里需要 `unsafe` fn 并且内部还需要 `unsafe` 块
    // 使用 `&raw mut` (较新 Rust 语法) 或直接转换来获取原始指针
    // let ptr = unsafe { &mut PLUGIN_SHARED_VALUE as *mut i32 };
    let ptr = { &raw mut PLUGIN_SHARED_VALUE as *mut i32 }; // 使用 &raw mut 避免 Miri 抱怨
    println!("[WASM FFI] get_plugin_shared_value_ptr() -> {:?}", ptr);
    ptr
}
```

**警告：** 直接向宿主暴露内部可变状态的指针是一种非常危险的做法！这打破了 Wasm 的封装性，宿主可以直接修改 Wasm
内部的数据，可能导致意想不到的后果或破坏 Wasm
内部的不变性。在实际应用中应极力避免这种模式，除非有非常明确和受控的理由。更好的方式是通过导出的函数来间接、安全地修改内部状态。这里展示它主要是为了演示内存操作的可能性。

## 交互模式详解

现在我们结合 C++ 宿主和 Rust Wasm 模块的代码，来看看具体的交互流程是如何实现的。

### 模式一：宿主调用简单 Wasm 函数 (`just_add`)

这是最基本的交互。宿主需要调用 Wasm 模块导出的一个纯计算函数。

**C++ 宿主端 (`host.cpp`):**

1. **获取函数：** 通过 `WasmHost` 封装的方法（内部调用 `instance_.get()` 和 `func.typed()`）获取类型安全的 Wasm 函数代理
   `TypedFunc`。
2. **准备参数：** 将 C++ 的 `uint64_t` 参数包装在 `std::tuple` 中。
3. **调用：** 使用 `typed_func.call()` 方法调用 Wasm 函数。Wasmtime C++ API 会处理参数和返回值的传递。
4. **处理结果：** 从返回的 `Result` 中获取结果 `std::tuple`，并提取出 `uint64_t` 的返回值。

```cpp
// host.cpp (main 函数内, Test 1)
uint64_t arg1 = 15, arg2 = 27;
auto args = std::make_tuple(arg1, arg2);
std::cout << "[Host Main] Calling Wasm function 'just_add(" << arg1 << ", " << arg2 << ")'..." << std::endl;

// host 是 WasmHost 实例
// 类型推导：返回值是 tuple<u64>, 参数是 tuple<u64, u64>
auto result_tuple = host.callFunction<std::tuple<uint64_t>, std::tuple<uint64_t, uint64_t>>(
    "just_add", args);
// result_tuple 是 Result<std::tuple<uint64_t>, TrapError>
if (!result_tuple) { /* 错误处理 */ }
uint64_t result_val = std::get<0>(result_tuple.ok());
std::cout << "[Host Main] 'just_add' Result: " << result_val << std::endl;
```

这里 `host.callFunction` 是 `WasmHost` 类中对 Wasmtime API 的封装，它隐藏了获取函数、类型检查和调用的细节。

**Rust Wasm 端 (`src/ffi.rs` 和 `src/lib.rs::core`):**

1. `#[no_mangle] pub extern "C" fn just_add` 函数被导出。
2. 它接收两个 `u64` 参数，调用 `core::perform_basic_add` 进行计算。
3. 返回 `u64` 结果。

```rust
// src/ffi.rs
#[no_mangle]
pub extern "C" fn just_add(left: u64, right: u64) -> u64 {
    println!("[WASM FFI] just_add called with: {} + {}", left, right);
    let result = crate::core::perform_basic_add(left, right); // 调用核心逻辑
    println!("[WASM FFI] just_add result: {}", result);
    result
}

// src/lib.rs::core
pub fn perform_basic_add(left: u64, right: u64) -> u64 {
    println!("[WASM Core] perform_basic_add: {} + {}", left, right);
    left.wrapping_add(right) // 使用安全加法
}
```

这个流程展示了从 C++ 到 Rust 的基本函数调用和简单数据类型传递。

### 模式二：Wasm 调用宿主函数 (`trigger_host_calls`)

这个模式反过来，Wasm 模块需要调用宿主提供的功能。

**C++ 宿主端:**

1. **实现宿主函数：** 如 `host_log_value_impl_target`, `host_get_shared_value_impl_target`,
   `host_set_shared_value_impl_target`。这些函数可以直接访问和修改宿主的状态（如 `shared_host_value`）。
2. **注册到 Linker：** 使用 `host.defineHostFunction("env", ...)` 将这些 C++ 函数与 Wasm 模块期望导入的 "env"
   模块下的函数名关联起来。
3. **调用 Wasm 入口：** 宿主调用 Wasm 导出的 `trigger_host_calls` 函数，这个函数会触发 Wasm
   内部对宿主函数的调用。这里调用的是一个无返回值的函数，可以使用 `host.callFunctionVoid`。

```cpp
// host.cpp (main 函数内, Test 2)
int32_t trigger_arg = 7;
int32_t host_value_before = shared_host_value; // 记录调用前状态
std::cout << "[Host Main] Calling Wasm function 'trigger_host_calls(" << trigger_arg << ")'..." << std::endl;

// host.callFunctionVoid 封装了调用无返回值 Wasm 函数的逻辑
// 参数是 tuple<i32>
host.callFunctionVoid<std::tuple<int32_t>>(
    "trigger_host_calls", std::make_tuple(trigger_arg));

std::cout << "[Host Main] Returned from 'trigger_host_calls'." << std::endl;
// 检查调用后宿主状态是否被 Wasm 修改
// ... 比较 shared_host_value 与预期值 ...
```

**Rust Wasm 端:**

1. **声明导入：** 在 `src/ffi.rs` 中使用 `extern "C"` 和 `#[link(wasm_import_module = "env")]` 声明需要从宿主导入的函数。
2. **提供安全包装：** 在 `src/ffi.rs` 中提供如 `log_value_from_host`, `get_shared_value_from_host`,
   `set_shared_value_in_host` 的安全包装器。
3. **导出触发函数：** `trigger_host_calls` 函数被导出。
4. **调用宿主函数：** 在 `core::perform_host_calls_test`（被 `trigger_host_calls` 调用）中，通过调用 FFI 层的安全包装器来间接调用
   C++ 宿主函数，从而读取和修改宿主状态。

```rust
// src/ffi.rs - 导入声明和安全包装 (前面已展示)

// src/ffi.rs - 导出触发函数
#[no_mangle]
pub extern "C" fn trigger_host_calls(input_val: i32) {
    println!("[WASM FFI] trigger_host_calls called with input: {}", input_val);
    crate::core::perform_host_calls_test(input_val); // 调用核心逻辑
    println!("[WASM FFI] trigger_host_calls finished.");
}

// src/lib.rs::core - 核心逻辑，调用宿主函数
pub fn perform_host_calls_test(input_val: i32) {
    println!("[WASM Core] perform_host_calls_test with input: {}", input_val);

    // 1. 调用 host_log_value
    log_value_from_host(input_val * 2);

    // 2. 调用 host_get_shared_value
    let host_val = get_shared_value_from_host();
    println!("[WASM Core] Received value from host: {}", host_val);

    // 3. 调用 host_set_shared_value (修改宿主状态)
    let new_host_val = host_val.wrapping_add(input_val).wrapping_add(5);
    set_shared_value_in_host(new_host_val);
    // ...
}
```

这个流程展示了从 Wasm 到 C++ 的调用，以及 Wasm 如何通过调用宿主函数来影响宿主的状态。

### 模式三：通过内存共享结构体 (`point_add`)

这是更复杂的交互，涉及到在宿主和 Wasm 之间传递结构体数据。由于不能直接传递 C++ 或 Rust 对象，我们利用共享的线性内存。

**C++ 宿主端 (`host.cpp`, Test 3):**

1. **定义结构体：** 定义 `Point` 结构体，并使用 `#pragma pack` 确保布局可控。
2. **计算内存偏移量：** 在 Wasm 线性内存中选择几个地址（偏移量）用于存放输入点 `p1`, `p2` 和结果点 `result`
   。需要确保这些地址不会冲突，并且有足够的空间。
3. **写入内存：** 创建 C++ `Point` 对象 `host_p1`, `host_p2`。使用 `host.writeMemory()` 方法将这两个对象的数据按字节复制到
   Wasm 线性内存中对应的偏移量 `offset_p1`, `offset_p2` 处。`writeMemory` 内部会获取内存 Span 并执行 `memcpy`。
4. **调用 Wasm 函数：** 调用 Wasm 导出的 `point_add` 函数。注意，传递给 Wasm 的参数是之前计算好的**内存偏移量**（作为
   `int32_t` 指针）。
5. **读取内存：** Wasm 函数执行完毕后，结果已经写回到了 Wasm 内存的 `offset_result` 位置。宿主使用
   `host.readMemory<Point>()` 方法从该偏移量读取数据，并将其解析为一个 C++ `Point` 对象。`readMemory` 内部同样会获取内存
   Span 并执行 `memcpy`。
6. **验证结果：** 比较从 Wasm 内存读回的结果与预期结果。

```cpp
// host.cpp (main 函数内, Test 3)
const size_t point_size = sizeof(Point);
const int32_t offset_p1 = 2048; // 示例偏移量
const int32_t offset_p2 = offset_p1 + point_size;
const int32_t offset_result = offset_p2 + point_size;

Point host_p1 = {100, 200};
Point host_p2 = {30, 70};

std::cout << "[Host Main] Writing points to WASM memory..." << std::endl;
// host.writeMemory 封装了获取 Span 和 memcpy 的逻辑
host.writeMemory(offset_p1, host_p1); // 将 host_p1 写入 Wasm 内存
host.writeMemory(offset_p2, host_p2); // 将 host_p2 写入 Wasm 内存

std::cout << "[Host Main] Calling Wasm function 'point_add' with offsets..." << std::endl;
// 参数是偏移量 (i32)，代表指针
auto point_add_args = std::make_tuple(offset_result, offset_p1, offset_p2);
host.callFunctionVoid<std::tuple<int32_t, int32_t, int32_t>>("point_add", point_add_args);

std::cout << "[Host Main] Reading result struct from WASM memory..." << std::endl;
// host.readMemory 封装了获取 Span 和 memcpy 的逻辑
Point result_point = host.readMemory<Point>(offset_result); // 从 Wasm 内存读取结果
std::cout << "[Host Main] 'point_add' Result read from memory: { x: " << result_point.x
          << ", y: " << result_point.y << " }" << std::endl;
// ... 验证结果 ...

// WasmHost 类中的 writeMemory/readMemory 简化实现：
template <typename T>
void WasmHost::writeMemory(int32_t offset, const T& data) {
    auto memory_span = getMemorySpan();
    size_t data_size = sizeof(T);
    if (offset < 0 || static_cast<size_t>(offset) + data_size > memory_span.size()) {
        throw std::out_of_range("Memory write out of bounds");
    }
    std::memcpy(memory_span.data() + offset, &data, data_size);
}

template <typename T>
T WasmHost::readMemory(int32_t offset) {
    auto memory_span = getMemorySpan();
    size_t data_size = sizeof(T);
     if (offset < 0 || static_cast<size_t>(offset) + data_size > memory_span.size()) {
        throw std::out_of_range("Memory read out of bounds");
    }
    T result;
    std::memcpy(&result, memory_span.data() + offset, data_size);
    return result;
}
```

**Rust Wasm 端:**

1. **定义结构体：** 定义 `Point` 结构体，并使用 `#[repr(C)]` 确保布局与 C++ 端兼容。
2. **导出函数：** 导出 `point_add` 函数。它的参数是 `*mut Point` 和 `*const Point` 类型，这些实际上接收的是宿主传来的 32
   位整数（内存偏移量），Wasmtime 会将它们解释为指向 Wasm 线性内存的指针。
3. **使用 `unsafe`：** 在函数体内部，必须使用 `unsafe` 块来解引用这些原始指针 (`*result_ptr`, `*p1_ptr`, `*p2_ptr`)。Rust
   编译器无法保证这些指针的有效性（它们来自外部世界），所以需要开发者承担责任。
4. **执行操作：** 从指针读取输入的 `Point` 数据，调用 `core::add_points` 计算结果。
5. **写入内存：** 将计算得到的 `result` 通过 `*result_ptr = result;` 写回到宿主指定的内存位置。

```rust
// src/ffi.rs - Point struct 定义 (前面已展示)

// src/ffi.rs - 导出 point_add 函数
#[no_mangle]
pub extern "C" fn point_add(result_ptr: *mut Point, p1_ptr: *const Point, p2_ptr: *const Point) {
    println!("[WASM FFI] point_add called with pointers...");
    unsafe { // 必须使用 unsafe 来解引用原始指针
        if result_ptr.is_null() || p1_ptr.is_null() || p2_ptr.is_null() {
            println!("[WASM FFI] Error: Received null pointer.");
            return;
        }
        // 解引用输入指针，读取数据
        let p1 = *p1_ptr;
        let p2 = *p2_ptr;
        // 调用核心逻辑计算
        let result = crate::core::add_points(p1, p2);
        // 解引用输出指针，写入结果
        *result_ptr = result;
        println!("[WASM FFI] Wrote result to address {:?}", result_ptr);
    }
}

// src/lib.rs::core - 核心加法逻辑
pub fn add_points(p1: Point, p2: Point) -> Point {
    println!("[WASM Core] add_points called with p1: {:?}, p2: {:?}", p1, p2);
    Point {
        x: p1.x.wrapping_add(p2.x),
        y: p1.y.wrapping_add(p2.y),
    }
}
```

这个模式是 Wasm 与宿主进行复杂数据交换的基础。关键在于内存布局的约定和通过指针（偏移量）进行访问，以及在 Rust 中正确使用
`unsafe`。

### 模式四：宿主直接读写 Wasm 内部状态

这个模式演示了（但不推荐）宿主如何直接修改 Wasm 模块内部的 `static mut` 状态。

**C++ 宿主端 (`host.cpp`, Test 4):**

1. **获取状态指针：** 调用 Wasm 导出的 `get_plugin_shared_value_ptr` 函数。这个函数返回一个 `int32_t`，它代表
   `PLUGIN_SHARED_VALUE` 在 Wasm 线性内存中的偏移量。
2. **读取初始值：** 使用 `host.readMemory<int32_t>()` 从获取到的偏移量读取 Wasm 状态的当前值。
3. **写入新值：** 使用 `host.writeMemory()` 向该偏移量写入一个新的 `int32_t` 值。
4. **再次读取验证：** 再次使用 `host.readMemory<int32_t>()` 读取，确认写入成功。

```cpp
// host.cpp (main 函数内, Test 4)
int32_t plugin_value_offset = -1;
// ...

std::cout << "[Host Main] Calling Wasm 'get_plugin_shared_value_ptr'..." << std::endl;
// getPluginDataOffset 封装了调用 Wasm 函数获取偏移量的逻辑
plugin_value_offset = host.getPluginDataOffset("get_plugin_shared_value_ptr");
std::cout << "[Host Main] Received offset: " << plugin_value_offset << std::endl;

if (plugin_value_offset > 0) { // 基本有效性检查
    // 读取 Wasm 状态
    int32_t value_from_plugin_before = host.readMemory<int32_t>(plugin_value_offset);
    std::cout << "[Host Main] Value read from plugin: " << value_from_plugin_before << std::endl;

    // 写入新值到 Wasm 状态
    const int32_t new_value_for_plugin = 777;
    std::cout << "[Host Main] Writing new value (" << new_value_for_plugin << ") to plugin state..." << std::endl;
    host.writeMemory(plugin_value_offset, new_value_for_plugin);

    // 再次读取验证
    int32_t value_from_plugin_after = host.readMemory<int32_t>(plugin_value_offset);
    std::cout << "[Host Main] Value read after host write: " << value_from_plugin_after << std::endl;
    // ... 验证 value_from_plugin_after == new_value_for_plugin ...
}

// WasmHost::getPluginDataOffset 实现
int32_t WasmHost::getPluginDataOffset(std::string_view func_name) {
    std::cout << "[Host] Getting plugin data offset via '" << func_name << "'..." << std::endl;
    // Wasm 函数无参数，返回 i32 (偏移量)
    auto result_tuple = callFunction<std::tuple<int32_t>>(func_name);
    if (!result_tuple) { /* 错误处理 */ return -1; }
    int32_t offset = std::get<0>(result_tuple.ok());
    std::cout << "[Host] Received offset from plugin: " << offset << std::endl;
    return offset;
}
```

**Rust Wasm 端:**

1. **定义 `static mut` 状态：** `static mut PLUGIN_SHARED_VALUE: i32 = 100;`
2. **导出指针函数：** 导出 `get_plugin_shared_value_ptr` 函数，它在 `unsafe` 上下文中返回 `PLUGIN_SHARED_VALUE`
   的原始指针（偏移量）。

```rust
// src/ffi.rs
static mut PLUGIN_SHARED_VALUE: i32 = 100;

#[no_mangle]
pub unsafe extern "C" fn get_plugin_shared_value_ptr() -> *mut i32 {
    let ptr = { &raw mut PLUGIN_SHARED_VALUE as *mut i32 };
    println!("[WASM FFI] get_plugin_shared_value_ptr() -> {:?}", ptr);
    ptr
}
```

这个模式展示了内存操作的强大能力，但也突显了潜在的风险。宿主现在可以直接干预 Wasm 的内部实现细节。

### 模式五：Wasm 验证内部状态被宿主修改

为了确认模式四中宿主的写入确实生效了，我们让 Wasm 模块自己检查一下那个 `static mut` 变量的值。

**C++ 宿主端 (`host.cpp`, Test 5):**

在模式四修改了 Wasm 状态后，调用另一个 Wasm 函数（比如 `simple_add`，虽然名字不符，但可以复用）。我们不关心这个函数的返回值，而是关心它在
Wasm 内部执行时打印的日志。

```cpp
// host.cpp (main 函数内, Test 5, 假设 plugin_value_offset > 0)
std::cout << "[Host Main] Calling Wasm 'simple_add' to verify internal state..." << std::endl;
// 调用一个 Wasm 函数，让它有机会读取并打印自己的状态
auto args = std::make_tuple(1ULL, 1ULL);
host.callFunction<std::tuple<uint64_t>, std::tuple<uint64_t, uint64_t>>(
    "simple_add", args);
std::cout << "[Host Main] Returned from 'simple_add'. Check WASM output above." << std::endl;
```

**Rust Wasm 端:**

我们需要修改 `simple_add` 函数（或其调用的核心逻辑 `perform_simple_add_and_read_internal_state`），让它在执行主要任务之前，先读取
`PLUGIN_SHARED_VALUE` 的值并打印出来。

```rust
// src/ffi.rs
#[no_mangle]
pub extern "C" fn simple_add(left: u64, right: u64) -> u64 {
    println!("[WASM FFI] simple_add (verification step) called...");
    crate::core::perform_simple_add_and_read_internal_state(left, right)
}

// 内部帮助函数，读取 static mut (需要 unsafe)
pub(crate) fn read_plugin_value_internal() -> i32 {
    unsafe { PLUGIN_SHARED_VALUE }
}

// src/lib.rs::core
pub fn perform_simple_add_and_read_internal_state(left: u64, right: u64) -> u64 {
    // 读取并打印自己的内部状态
    let current_plugin_val = read_plugin_value_internal(); // 调用 FFI 辅助函数
    println!(
        "[WASM Core] Current plugin's internal shared value: {}", // 期望这里打印 777
        current_plugin_val
    );

    println!("[WASM Core] Performing simple add: {} + {}", left, right);
    // ... 执行原本的加法逻辑 ...
    left + right // 假设简单返回
}
```

当宿主执行 Test 5 时，我们应该能在控制台看到来自 `[WASM Core]` 的输出，显示 `Current plugin's internal shared value: 777`
（或者模式四中写入的任何值），这就验证了宿主确实成功修改了 Wasm 的内部状态。

## 关键要点与思考

通过这个实例，我们可以总结出使用 Wasmtime 进行 C++/Rust Wasm 交互的几个关键点：

1. **清晰的接口定义:** FFI 层是核心。Rust 的 `extern "C"`（导入/导出）和 C++ 的函数签名/链接必须精确匹配。
2. **内存操作是基础:** 复杂数据的传递依赖于对 Wasm 线性内存的读写。理解指针即偏移量、确保数据结构布局一致 (`#[repr(C)]`,
   `#pragma pack`) 至关重要。
3. **`unsafe` 的必要性:** 在 Rust Wasm 模块中，与 FFI 和 `static mut` 交互几乎不可避免地需要 `unsafe` 块。必须谨慎使用，并尽量将其限制在
   FFI 边界层。
4. **状态管理需谨慎:** 宿主和 Wasm 都可以有自己的状态。可以通过函数调用相互影响对方的状态。直接暴露 Wasm
   内部状态的指针给宿主虽然技术上可行，但破坏了封装，应尽量避免，优先选择通过接口函数进行状态管理。
5. **WASI 的作用:** 对于需要标准 I/O 或其他系统交互的 Wasm 模块（即使只是 `println!`），宿主需要配置并链接 WASI。
6. **Wasmtime API:** Wasmtime 提供了相当完善的 C++ API (`wasmtime.hh`)，包括 `Engine`, `Store`, `Module`, `Linker`,
   `Instance`, `Memory`, `Func`, `TypedFunc`, `Val` 等核心类，以及用于错误处理的 `Result` 和 `Trap`。理解这些类的作用和关系是成功使用的关键。

## 结语

WebAssembly 和 Wasmtime 为我们提供了一种强大的方式来扩展现有应用程序，实现高性能、安全、可移植的模块化。C++ 与 Rust 的结合，既能利用
C++ 的生态和性能，又能享受 Rust 带来的安全保证，尤其适合构建插件系统、处理性能关键任务或需要强沙箱隔离的场景。

虽然本文涉及的交互模式已经比较丰富，但这仅仅是冰山一角。Wasmtime 还支持更高级的特性，如抢占式中断（epoch
interruption）、燃料计量（fuel metering）、引用类型（reference types）、多内存、线程等。

希望这篇详细的演练能帮助你理解 C++ 宿主与 Rust Wasm 模块通过 Wasmtime 进行交互的基本原理和实践方法。如果你对这个领域感兴趣，不妨亲自动手尝试一下，将
Wasm 融入到你的下一个项目中去！

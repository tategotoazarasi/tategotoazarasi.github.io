---
date: '2025-04-06T16:42:41+08:00'
draft: false
summary: 'A detailed technical guide on using the Wasmtime runtime to enable complex bidirectional communication, shared memory access, and struct passing between C++ host applications and Rust WebAssembly modules.'
title: 'Deep Dive into Wasmtime: Bidirectional Communication and Memory Sharing Between C++ and Rust Wasm Modules'
tags: [ "wasm", "wasmtime", "cpp", "rust", "ffi", "shared-memory", "bidirectional-communication", "host-guest", "linear-memory", "struct-passing", "state-management", "wasi", "runtime", "sandboxing", "wasmtime-api", "c-plus-plus" ]
---

Today, let's talk about an increasingly popular technology: WebAssembly (Wasm). However, we won't confine it to the
browser. Instead, we'll explore how, on the server-side or in desktop applications, we can use the Wasmtime runtime to
allow C++ programs to load and execute Rust-compiled Wasm modules. We'll also delve into enabling complex interactions
between them, such as bidirectional function calls, shared memory, passing structs, and even modifying each other's
state.

## A Brief Introduction to WebAssembly and Wasmtime

First, let's briefly explain what WebAssembly is. You can think of it as a portable binary instruction format designed
for the modern web. It's not meant to replace JavaScript but rather to act as a powerful complement, allowing code
written in performance-sensitive or low-level languages like C, C++, or Rust to run in web environments (and other
Wasm-supporting environments) at near-native speeds. Wasm's core strengths lie in its **sandboxed** security model and *
*platform-agnostic** nature.

Wasmtime, on the other hand, is a **standalone, efficient, and secure** WebAssembly runtime developed by the Bytecode
Alliance (a consortium including companies like Mozilla, Fastly, Intel, and Red Hat). It enables you to run Wasm modules
outside the browser – for instance, on servers, in command-line tools, or on embedded devices. Wasmtime provides APIs
for various languages, including C, C++, Python, Rust, and Go, making it convenient to integrate Wasm into existing
applications.

## Why Choose a C++ Host + Rust Wasm Combination?

This combination offers several compelling advantages:

Many mature projects have extensive C++ foundations. Wasm allows parts of these projects to be modularized, sandboxed,
or exposed as a plugin system without rewriting the core logic. Rust is renowned for its memory and concurrency safety,
making it an excellent choice for writing highly reliable Wasm modules. Rust adds another layer of assurance on top of
Wasm's sandbox. Both C++ and Rust are high-performance languages. When compiled to Wasm and executed with a JIT runtime
like Wasmtime, they can achieve performance close to native code. Interaction between the Wasm module and the host must
occur through explicitly defined interfaces (imports/exports), which helps maintain a clean architecture.

The goal of this article is to demonstrate, through a concrete example, how to use Wasmtime's C++ API to build a C++
host application that loads a Rust-written Wasm module and facilitates various interesting interactions between them.

## Core Concepts: Bridging C++ and Wasm

Before diving into the code, we need to understand a few key concepts:

### Host and Guest

In this scenario, the C++ application is the **host**. It is responsible for loading, managing, and running the Wasm
module. The Rust-compiled Wasm module is the **guest**. It runs within the Wasmtime runtime environment provided by the
host, constrained by the sandbox.

### Wasm Imports and Exports

The primary way Wasm modules communicate with the outside world is through imports and exports.

A Wasm module can **export** functions, memory, global variables, or tables, making them available for the host or other
Wasm modules to call or access. In Rust, we typically use `#[no_mangle] pub extern "C"` to mark functions intended for
export.

A Wasm module can declare which functionalities (usually functions) it needs to **import** from the host environment.
When the host instantiates the Wasm module, it must provide implementations for these imports. In Rust, we use an
`extern "C" { ... }` block combined with `#[link(wasm_import_module = "...")]` to declare imports.

This import/export mechanism forms the interface contract between the host and the Wasm module.

### Linear Memory

Each Wasm instance (usually) has its own **linear memory**. This is a contiguous, mutable array of bytes that can be
read and written by both the Wasm code and the host code. Pointers within Wasm code are essentially **offsets** (
typically 32-bit or 64-bit integers) into this memory region.

Crucially, Wasm itself is sandboxed; it cannot directly access the host's memory. Likewise, the host cannot arbitrarily
access variables internal to the Wasm instance. However, the host *can* obtain **access** to the Wasm instance's
exported linear memory via Wasmtime APIs (often as a pointer or Span to the memory's start). Once access is granted, the
host can directly read from or write to this memory block. Similarly, Wasm code can indirectly interact with the host's
state or resources by calling host-provided functions (imported functions).

This method of data exchange via shared linear memory is central to Wasm interaction. Passing complex data structures (
like C++ `struct`s or Rust `struct`s) is typically achieved by serializing them into this memory and then passing
pointers (offsets) to that location.

### WASI (WebAssembly System Interface)

WASI is a set of standardized system interfaces designed to allow Wasm modules to interact with the underlying operating
system in a secure and portable manner, covering functionalities like file system access, network communication, and
standard I/O. While our example doesn't involve complex file operations, Rust's standard `println!` macro relies on
underlying standard output capabilities. To make `println!` within the Wasm module work correctly (printing output to
the host's console), we need to configure and link WASI support in the host.

## Building the C++ Host: Setting the Stage with Wasmtime

Now, let's examine what the C++ host side needs to do. For better code organization, we often create a class (e.g.,
`WasmHost`) to encapsulate the interaction logic with Wasmtime.

### Loading and Compiling the Wasm Module

The first step is to read the contents of the Wasm module file (the `.wasm` binary) and then use Wasmtime's `Engine` to
compile it. The `Engine` acts as Wasmtime's core compilation and execution engine, responsible for transforming Wasm
bytecode into executable machine code. The compilation result is a `Module` object. This `Module` object is thread-safe
and can be reused by multiple `Store`s.

```cpp
// Pseudo-code example (Actual code in wasm_host.cpp)
#include "wasmtime.hh" // Include Wasmtime C++ header
#include <vector>
#include <fstream>
#include <stdexcept>

using namespace wasmtime;

// ... WasmHost class definition ...

std::vector<uint8_t> WasmHost::readWasmFile() {
    std::ifstream file(wasm_path_, std::ios::binary | std::ios::ate);
    // ... Error handling ...
    std::streamsize size = file.tellg();
    file.seekg(0, std::ios::beg);
    std::vector<uint8_t> buffer(static_cast<size_t>(size));
    // ... Read file contents into buffer ...
    return buffer;
}

void WasmHost::loadAndCompile() {
    std::vector<uint8_t> wasm_bytes = readWasmFile();
    std::cout << "[Host Setup] Compiling WASM module..." << std::endl;
    // engine_ is a member variable of WasmHost, type wasmtime::Engine
    Result<Module> module_res = Module::compile(engine_, wasm_bytes);
    if (!module_res) {
        throw std::runtime_error("Module compilation failed: " + module_res.err().message());
    }
    // module_ is also a WasmHost member, type std::optional<wasmtime::Module>
    module_ = std::move(module_res.ok());
    std::cout << "[Host Setup] Module compiled successfully." << std::endl;
}

// Call loadAndCompile() in the WasmHost constructor or an initialization function
```

### Engine and Store

The `Engine` handles code compilation, while the `Store` represents the "world" or "context" of a Wasm instance. All
data associated with a Wasm instance, such as its memory, global variables, tables, and the instance itself, **belongs**
to a specific `Store`. One `Engine` can be associated with multiple `Store`s, but a `Store` is linked to only one
`Engine`. `Store`s are not thread-safe; typically, one thread corresponds to one `Store`.

```cpp
// WasmHost class members
Engine engine_;
Store store_;

// WasmHost constructor
WasmHost::WasmHost(std::string wasm_path) : wasm_path_(std::move(wasm_path)),
                                            engine_(), // Create default Engine
                                            store_(engine_) // Create Store based on Engine
{
    // ...
}
```

### Configuring WASI

As mentioned, if the Wasm module requires system interactions (like `println!`), we need to configure WASI for the
`Store`. This is usually done **before** instantiating the module. Wasmtime provides the `WasiConfig` class to configure
WASI behavior, such as inheriting the host's standard input/output/error streams, environment variables, and
command-line arguments. The configured `WasiConfig` must be set into the `Store`'s context.

```cpp
// WasmHost::setupWasi() method
void WasmHost::setupWasi() {
    // ... Check if already initialized or configured ...
    std::cout << "[Host Setup] Configuring WASI..." << std::endl;
    WasiConfig wasi;
    wasi.inherit_stdout(); // Make Wasm's stdout go to host's stdout
    wasi.inherit_stderr(); // Same for stderr
    // store_ is a WasmHost member variable
    auto wasi_set_res = store_.context().set_wasi(std::move(wasi));
    if (!wasi_set_res) {
        throw std::runtime_error("Failed setting WASI config in store: " + wasi_set_res.err().message());
    }
    wasi_configured_ = true;
    std::cout << "[Host Setup] WASI configured for Store." << std::endl;
    // Also need to define WASI imports in the Linker
    linkWasiImports();
}

// WasmHost::linkWasiImports() method
void WasmHost::linkWasiImports() {
    // ... Check if WASI is configured ...
    std::cout << "[Host Setup] Defining WASI imports in linker..." << std::endl;
    // linker_ is a WasmHost member variable, type wasmtime::Linker
    auto linker_define_wasi_res = linker_.define_wasi();
     if (!linker_define_wasi_res) {
        throw std::runtime_error("Failed defining WASI imports in linker: " + linker_define_wasi_res.err().message());
    }
    std::cout << "[Host Setup] WASI imports defined." << std::endl;
}
```

### Linker: The Bridge Connecting Host and Wasm

The `Linker` is a Wasmtime utility for resolving module imports and connecting them to host-provided implementations.
Before instantiating a module, we need to inform the `Linker` how to satisfy all of the Wasm module's import
requirements.

This involves two main parts:

1. **Linking WASI Imports:** If we've configured WASI, we need to call `linker_.define_wasi()`. This automatically adds
   implementations for standard WASI functions to the `Linker`.
2. **Linking Custom Host Function Imports:** The Wasm module might need to call our custom host functions. We must wrap
   these C++ functions (or lambdas) into a form Wasmtime understands and register them with the `Linker` using
   `linker_.define()` or `linker_.func_wrap()`. We specify the corresponding Wasm module name (defined by
   `#[link(wasm_import_module = "...")]` in the Rust code) and the function name.

### Defining Host Functions Callable by Wasm

This is crucial for enabling Wasm-to-Host calls. We need to write the implementation functions in C++. Their signatures
must match the `extern "C"` function declarations in Rust (or be adaptable by Wasmtime C++ API template deduction).

For example, if Rust declares imports like this:

```rust
// src/ffi.rs
#[link(wasm_import_module = "env")] // Module name is "env"
unsafe extern "C" {
    fn host_log_value(value: i32);
    fn host_get_shared_value() -> i32;
    fn host_set_shared_value(value: i32);
}
```

Then, in the C++ host, we provide implementations for these three functions and register them with the `Linker`,
associated with the "env" module.

```cpp
// host.cpp
#include <iostream>
#include <cstdint>

// Host state
int32_t shared_host_value = 42;

// C++ implementation functions
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
    shared_host_value = new_value; // Modify host state
}

// In the WasmHost class or main function, register these using the Linker
// (Simplified wrapper function within WasmHost class)
template <typename FuncPtr>
void WasmHost::defineHostFunction(std::string_view module_name, std::string_view func_name, FuncPtr func_ptr)
{
    if (is_initialized_) {
        throw std::logic_error("Cannot define host functions after initialization.");
    }
    std::cout << "[Host Setup] Defining host function: " << module_name << "::" << func_name << "..." << std::endl;
    // linker_ is a WasmHost member variable
    auto result = linker_.func_wrap(module_name, func_name, func_ptr);
    if (!result) {
        throw std::runtime_error("Failed to define host function '" + std::string(func_name) + "': " + result.err().message());
    }
}

// Called from main function
host.defineHostFunction("env", "host_log_value", host_log_value_impl_target);
host.defineHostFunction("env", "host_get_shared_value", host_get_shared_value_impl_target);
host.defineHostFunction("env", "host_set_shared_value", host_set_shared_value_impl_target);
```

`linker_.func_wrap()` is a convenient template function. It automatically deduces the parameter and return types of the
C++ function, converts them to the corresponding Wasm function type, and registers the function. This is often simpler
than manually creating a `FuncType` and using `linker_.define()`.

### Instantiating the Module

Once all imports (WASI and custom functions) are defined in the `Linker`, we can use `linker_.instantiate()` to create
an instance (`Instance`) of the Wasm module. The instantiation process connects the Wasm code with the host-provided
implementations and allocates resources like memory and globals within the `Store`.

```cpp
// WasmHost::instantiateModule() method
void WasmHost::instantiateModule() {
    // ... Check if module_ is valid ...
    std::cout << "[Host Setup] Instantiating module..." << std::endl;
    // store_ is a WasmHost member variable
    TrapResult<Instance> instance_res = linker_.instantiate(store_.context(), module_.value());
    if (!instance_res) {
         // Handle instantiation error (could be linking error or Wasm start trap)
        throw std::runtime_error("Module instantiation failed: " + instance_res.err().message());
    }
    // instance_ is a WasmHost member, type std::optional<wasmtime::Instance>
    instance_ = std::move(instance_res.ok());
    std::cout << "[Host Setup] Module instantiated successfully." << std::endl;
}
```

### Accessing Wasm Linear Memory

To exchange complex data with the Wasm module or directly read/write its memory state, the host needs access to the Wasm
instance's linear memory. Wasm modules typically export a memory object named "memory". We can retrieve it using
`instance_.get()`.

```cpp
// WasmHost::getMemory() method
void WasmHost::getMemory() {
    // ... Check if instance_ is valid ...
    std::cout << "[Host Setup] Getting exported memory 'memory'..." << std::endl;
    // store_ is a WasmHost member variable
    auto memory_export_opt = instance_.value().get(store_.context(), "memory");

    if (memory_export_opt && std::holds_alternative<Memory>(*memory_export_opt)) {
        // memory_ is a WasmHost member, type std::optional<wasmtime::Memory>
        memory_ = std::get<Memory>(*memory_export_opt);
        std::cout << "[Host Setup] Found exported memory. Size: "
                  << memory_.value().data(store_.context()).size() << " bytes." << std::endl;
    } else {
        std::cout << "[Host Setup] Export 'memory' not found or not a memory. Proceeding without memory access." << std::endl;
    }
}

// Get a Span<uint8_t> for the memory, providing a view into the memory region
Span<uint8_t> WasmHost::getMemorySpan() {
    if (!is_initialized_ || !memory_.has_value()) {
        throw std::logic_error("Memory not available or host not initialized.");
    }
    return memory_.value().data(store_.context());
}
```

The obtained `wasmtime::Memory` object has a `data()` method that returns a `wasmtime::Span<uint8_t>` (or
`std::span<uint8_t>` if C++20 is available). This Span provides direct, low-level access (a pointer and size) to the
Wasm linear memory region. With this Span, the host can directly read from and write to the Wasm's memory.

## Building the Wasm Module: Rust's Safe Territory

Now let's switch to the Rust side and see how the Wasm module is constructed.

### Project Structure

Typically, FFI (Foreign Function Interface) related code is placed in a separate module (e.g., `src/ffi.rs`), while the
core, safe Rust logic resides in another module (e.g., `src/core.rs` or directly within `src/lib.rs`).

`src/lib.rs` serves as the library's entry point. It declares and exports the interfaces from the `ffi` module needed by
the host and might contain or invoke logic from the `core` module.

```rust
// src/lib.rs
mod ffi; // Declare the ffi module
pub(crate) mod core; // Declare the internal core module

// Re-export functions and types from the FFI layer needed by the host
pub use ffi::{
    Point, get_plugin_shared_value_ptr, just_add, point_add, simple_add, trigger_host_calls,
};
```

### FFI Layer (`src/ffi.rs`)

This is the boundary where Rust interacts with the external world (the C++ host).

1. **Declare Host Function Imports:** Use `extern "C"` blocks and `#[link(wasm_import_module = "env")]` to inform the
   Rust compiler and Wasm runtime about external functions provided by a module named "env". The signatures must match
   the implementations provided by the C++ host. Note that `extern "C"` blocks are inherently `unsafe` because calling
   external functions cannot guarantee Rust's memory safety rules.

   ```rust
   // src/ffi.rs
   #[link(wasm_import_module = "env")]
   unsafe extern "C" {
	   fn host_log_value(value: i32);
	   fn host_get_shared_value() -> i32;
	   fn host_set_shared_value(value: i32);
   }
   ```

2. **Provide Safe Wrappers:** To avoid scattering `unsafe` blocks throughout the business logic, it's common practice to
   provide safe Rust wrapper functions for the imported `unsafe` functions.

   ```rust
   // src/ffi.rs
   pub fn log_value_from_host(value: i32) {
	   unsafe { host_log_value(value) } // The unsafe call is encapsulated inside
   }
   // ... other wrapper functions ...
   ```

3. **Export Wasm Functions:** Use `#[no_mangle]` to prevent the Rust compiler from mangling function names, and use
   `pub extern "C"` to specify the C calling convention. This allows the C++ host to find and call these functions by
   name.

   ```rust
   // src/ffi.rs
   #[no_mangle] // Prevent name mangling
   pub extern "C" fn just_add(left: u64, right: u64) -> u64 {
	   println!("[WASM FFI] just_add called..."); // Using WASI's println!
	   core::perform_basic_add(left, right) // Call core logic
   }

   #[no_mangle]
   pub extern "C" fn trigger_host_calls(input_val: i32) {
	   println!("[WASM FFI] trigger_host_calls called...");
	   core::perform_host_calls_test(input_val); // Call core logic
   }
   // ... other exported functions ...
   ```

### Core Logic Layer (`src/core.rs`)

This is where the actual functionality of the Wasm module is implemented, ideally using safe Rust code. It calls the
safe wrappers provided by the FFI layer to interact with the host.

```rust
// src/lib.rs (core module)
pub(crate) mod core {
    use crate::ffi::{ // Import safe wrappers from the FFI layer
        Point,
        get_shared_value_from_host,
        log_value_from_host,
        set_shared_value_in_host,
        // ...
    };

    pub fn perform_basic_add(left: u64, right: u64) -> u64 {
        println!("[WASM Core] perform_basic_add: {} + {}", left, right);
        left.wrapping_add(right) // Safe addition
    }

    pub fn perform_host_calls_test(input_val: i32) {
        println!("[WASM Core] perform_host_calls_test with input: {}", input_val);
        // Call host functions (via safe wrappers)
        log_value_from_host(input_val * 2);
        let host_val = get_shared_value_from_host();
        set_shared_value_in_host(host_val + input_val + 5);
        // ...
    }
    // ... other core logic functions ...
}
```

### Defining Shared Data Structures

If complex data structures need to be passed between C++ and Rust, both sides must agree on the memory layout. In Rust,
use the `#[repr(C)]` attribute to enforce a C-compatible memory layout for the struct. In C++, while compilers often lay
out structs sequentially, using `#pragma pack(push, 1)` and `#pragma pack(pop)` ensures a packed (no padding) layout for
absolute certainty, or ensures consistent alignment between both sides.

```rust
// src/ffi.rs
#[repr(C)] // Crucial: guarantees C-compatible layout
#[derive(Debug, Copy, Clone, Default)]
pub struct Point {
    pub x: i32,
    pub y: i32,
}
```

```cpp
// host.cpp
#pragma pack(push, 1) // Recommended: ensures packed layout consistent with Rust
struct Point {
    int32_t x;
    int32_t y;
};
#pragma pack(pop)
```

### Managing Wasm Internal State

Wasm modules sometimes need to maintain their own state. One way is using Rust's `static mut` variables. However,
accessing `static mut` requires an `unsafe` block because it can potentially introduce data races (though the risk is
lower in single-threaded Wasm environments, Rust still mandates `unsafe`).

```rust
// src/ffi.rs
static mut PLUGIN_SHARED_VALUE: i32 = 100; // Wasm module's internal state

// Internal FFI helper function for safe reading (still needs unsafe block)
pub(crate) fn read_plugin_value_internal() -> i32 {
    unsafe { PLUGIN_SHARED_VALUE }
}

// Used in the core module
// use crate::ffi::read_plugin_value_internal;
// let val = read_plugin_value_internal();
```

If the host needs to modify this state directly, an exported function can return a **pointer** (memory offset) to the
`static mut` variable.

```rust
// src/ffi.rs
#[no_mangle]
pub unsafe extern "C" fn get_plugin_shared_value_ptr() -> *mut i32 {
    // Note: Requires `unsafe fn` and an inner `unsafe` block
    // Use `&raw mut` (newer Rust syntax) or direct cast to get the raw pointer
    // let ptr = unsafe { &mut PLUGIN_SHARED_VALUE as *mut i32 };
    let ptr = { &raw mut PLUGIN_SHARED_VALUE as *mut i32 }; // Using &raw mut avoids Miri complaints
    println!("[WASM FFI] get_plugin_shared_value_ptr() -> {:?}", ptr);
    ptr
}
```

**Warning:** Exposing a pointer to internal mutable state directly to the host is a very dangerous practice! It breaks
Wasm's encapsulation, allowing the host to modify internal Wasm data directly, potentially leading to unexpected
consequences or violating internal invariants. This pattern should be strongly avoided in practice unless there's a very
specific and controlled reason. A better approach is to modify internal state indirectly and safely via exported
functions. It's shown here primarily to demonstrate the possibilities of memory manipulation.

## Detailed Interaction Patterns

Now let's combine the C++ host and Rust Wasm module code to see how specific interaction flows are implemented.

### Pattern One: Host Calls a Simple Wasm Function (`just_add`)

This is the most basic interaction. The host needs to call a pure computation function exported by the Wasm module.

**C++ Host Side (`host.cpp`):**

1. **Get Function:** Obtain a type-safe Wasm function proxy (`TypedFunc`) using a method encapsulated in `WasmHost` (
   which internally calls `instance_.get()` and `func.typed()`).
2. **Prepare Arguments:** Wrap the C++ `uint64_t` arguments in an `std::tuple`.
3. **Call:** Invoke the Wasm function using the `typed_func.call()` method. The Wasmtime C++ API handles argument and
   return value marshalling.
4. **Process Result:** Extract the `std::tuple` containing the `uint64_t` return value from the returned `Result`.

```cpp
// host.cpp (inside main, Test 1)
uint64_t arg1 = 15, arg2 = 27;
auto args = std::make_tuple(arg1, arg2);
std::cout << "[Host Main] Calling Wasm function 'just_add(" << arg1 << ", " << arg2 << ")'..." << std::endl;

// host is the WasmHost instance
// Type deduction: Return is tuple<u64>, Params are tuple<u64, u64>
auto result_tuple = host.callFunction<std::tuple<uint64_t>, std::tuple<uint64_t, uint64_t>>(
    "just_add", args);
// result_tuple is Result<std::tuple<uint64_t>, TrapError>
if (!result_tuple) { /* Error handling */ }
uint64_t result_val = std::get<0>(result_tuple.ok());
std::cout << "[Host Main] 'just_add' Result: " << result_val << std::endl;
```

Here, `host.callFunction` is a wrapper within the `WasmHost` class that hides the details of getting the function,
type-checking, and calling.

**Rust Wasm Side (`src/ffi.rs` and `src/lib.rs::core`):**

1. The `#[no_mangle] pub extern "C" fn just_add` function is exported.
2. It receives two `u64` parameters and calls `core::perform_basic_add` for the computation.
3. It returns the `u64` result.

```rust
// src/ffi.rs
#[no_mangle]
pub extern "C" fn just_add(left: u64, right: u64) -> u64 {
    println!("[WASM FFI] just_add called with: {} + {}", left, right);
    let result = crate::core::perform_basic_add(left, right); // Call core logic
    println!("[WASM FFI] just_add result: {}", result);
    result
}

// src/lib.rs::core
pub fn perform_basic_add(left: u64, right: u64) -> u64 {
    println!("[WASM Core] perform_basic_add: {} + {}", left, right);
    left.wrapping_add(right) // Use safe addition
}
```

This flow demonstrates the basic function call from C++ to Rust and the passing of simple data types.

### Pattern Two: Wasm Calls Host Functions (`trigger_host_calls`)

This pattern reverses the direction: the Wasm module needs to invoke functionality provided by the host.

**C++ Host Side:**

1. **Implement Host Functions:** Such as `host_log_value_impl_target`, `host_get_shared_value_impl_target`,
   `host_set_shared_value_impl_target`. These functions can directly access and modify the host's state (like
   `shared_host_value`).
2. **Register with Linker:** Use `host.defineHostFunction("env", ...)` to associate these C++ functions with the
   function names the Wasm module expects to import from the "env" module.
3. **Call Wasm Entry Point:** The host calls the Wasm-exported `trigger_host_calls` function. This function will, in
   turn, trigger calls from within Wasm back to the host functions. Since this Wasm function returns void,
   `host.callFunctionVoid` can be used.

```cpp
// host.cpp (inside main, Test 2)
int32_t trigger_arg = 7;
int32_t host_value_before = shared_host_value; // Record state before call
std::cout << "[Host Main] Calling Wasm function 'trigger_host_calls(" << trigger_arg << ")'..." << std::endl;

// host.callFunctionVoid wraps calling void Wasm functions
// Params are tuple<i32>
host.callFunctionVoid<std::tuple<int32_t>>(
    "trigger_host_calls", std::make_tuple(trigger_arg));

std::cout << "[Host Main] Returned from 'trigger_host_calls'." << std::endl;
// Check if host state was modified by Wasm after the call
// ... Compare shared_host_value with the expected value ...
```

**Rust Wasm Side:**

1. **Declare Imports:** Use `extern "C"` and `#[link(wasm_import_module = "env")]` in `src/ffi.rs` to declare the
   functions needed from the host.
2. **Provide Safe Wrappers:** Offer safe wrappers like `log_value_from_host`, `get_shared_value_from_host`,
   `set_shared_value_in_host` in `src/ffi.rs`.
3. **Export Trigger Function:** The `trigger_host_calls` function is exported.
4. **Call Host Functions:** In `core::perform_host_calls_test` (called by `trigger_host_calls`), invoke the C++ host
   functions indirectly by calling the safe wrappers from the FFI layer, thereby reading and modifying the host's state.

```rust
// src/ffi.rs - Import declarations and safe wrappers (shown previously)

// src/ffi.rs - Export trigger function
#[no_mangle]
pub extern "C" fn trigger_host_calls(input_val: i32) {
    println!("[WASM FFI] trigger_host_calls called with input: {}", input_val);
    crate::core::perform_host_calls_test(input_val); // Call core logic
    println!("[WASM FFI] trigger_host_calls finished.");
}

// src/lib.rs::core - Core logic calling host functions
pub fn perform_host_calls_test(input_val: i32) {
    println!("[WASM Core] perform_host_calls_test with input: {}", input_val);

    // 1. Call host_log_value
    log_value_from_host(input_val * 2);

    // 2. Call host_get_shared_value
    let host_val = get_shared_value_from_host();
    println!("[WASM Core] Received value from host: {}", host_val);

    // 3. Call host_set_shared_value (modifying host state)
    let new_host_val = host_val.wrapping_add(input_val).wrapping_add(5);
    set_shared_value_in_host(new_host_val);
    // ...
}
```

This flow demonstrates calls from Wasm to C++ and how Wasm can influence the host's state by invoking host functions.

### Pattern Three: Sharing Structs via Memory (`point_add`)

This is a more complex interaction involving passing struct data between the host and Wasm. Since C++ or Rust objects
cannot be passed directly, we utilize the shared linear memory.

**C++ Host Side (`host.cpp`, Test 3):**

1. **Define Struct:** Define the `Point` struct, using `#pragma pack` to ensure a controlled layout.
2. **Calculate Memory Offsets:** Choose addresses (offsets) within the Wasm linear memory to store the input points
   `p1`, `p2`, and the result `result`. Ensure these addresses don't overlap and have sufficient space.
3. **Write to Memory:** Create C++ `Point` objects `host_p1`, `host_p2`. Use the `host.writeMemory()` method to copy the
   byte representation of these objects into the Wasm linear memory at the corresponding offsets `offset_p1`,
   `offset_p2`. `writeMemory` internally gets the memory Span and performs `memcpy`.
4. **Call Wasm Function:** Invoke the Wasm-exported `point_add` function. Importantly, the arguments passed to Wasm are
   the previously calculated **memory offsets** (as `int32_t` pointers).
5. **Read from Memory:** After the Wasm function executes, the result is written back to `offset_result` in Wasm memory.
   The host uses `host.readMemory<Point>()` to read the bytes from that offset and interpret them as a C++ `Point`
   object. `readMemory` also gets the memory Span and uses `memcpy`.
6. **Verify Result:** Compare the result read back from Wasm memory with the expected result.

```cpp
// host.cpp (inside main, Test 3)
const size_t point_size = sizeof(Point);
const int32_t offset_p1 = 2048; // Example offset
const int32_t offset_p2 = offset_p1 + point_size;
const int32_t offset_result = offset_p2 + point_size;

Point host_p1 = {100, 200};
Point host_p2 = {30, 70};

std::cout << "[Host Main] Writing points to WASM memory..." << std::endl;
// host.writeMemory encapsulates getting Span and memcpy
host.writeMemory(offset_p1, host_p1); // Write host_p1 to Wasm memory
host.writeMemory(offset_p2, host_p2); // Write host_p2 to Wasm memory

std::cout << "[Host Main] Calling Wasm function 'point_add' with offsets..." << std::endl;
// Args are offsets (i32), representing pointers
auto point_add_args = std::make_tuple(offset_result, offset_p1, offset_p2);
host.callFunctionVoid<std::tuple<int32_t, int32_t, int32_t>>("point_add", point_add_args);

std::cout << "[Host Main] Reading result struct from WASM memory..." << std::endl;
// host.readMemory encapsulates getting Span and memcpy
Point result_point = host.readMemory<Point>(offset_result); // Read result from Wasm memory
std::cout << "[Host Main] 'point_add' Result read from memory: { x: " << result_point.x
          << ", y: " << result_point.y << " }" << std::endl;
// ... Verify result ...

// Simplified implementation of writeMemory/readMemory in WasmHost:
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

**Rust Wasm Side:**

1. **Define Struct:** Define the `Point` struct using `#[repr(C)]` to ensure layout compatibility with the C++ side.
2. **Export Function:** Export the `point_add` function. Its parameters are `*mut Point` and `*const Point`. These
   receive the 32-bit integers (memory offsets) from the host, which Wasmtime interprets as pointers into the Wasm
   linear memory.
3. **Use `unsafe`:** Inside the function body, an `unsafe` block is mandatory to dereference these raw pointers (
   `*result_ptr`, `*p1_ptr`, `*p2_ptr`). The Rust compiler cannot guarantee the validity of these pointers (they
   originate from the external world), so the developer must take responsibility.
4. **Perform Operation:** Read the input `Point` data from the pointers, call `core::add_points` to compute the result.
5. **Write to Memory:** Write the calculated `result` back to the memory location specified by the host using
   `*result_ptr = result;`.

```rust
// src/ffi.rs - Point struct definition (shown previously)

// src/ffi.rs - Export point_add function
#[no_mangle]
pub extern "C" fn point_add(result_ptr: *mut Point, p1_ptr: *const Point, p2_ptr: *const Point) {
    println!("[WASM FFI] point_add called with pointers...");
    unsafe { // Must use unsafe to dereference raw pointers
        if result_ptr.is_null() || p1_ptr.is_null() || p2_ptr.is_null() {
            println!("[WASM FFI] Error: Received null pointer.");
            return;
        }
        // Dereference input pointers to read data
        let p1 = *p1_ptr;
        let p2 = *p2_ptr;
        // Call core logic for calculation
        let result = crate::core::add_points(p1, p2);
        // Dereference output pointer to write the result
        *result_ptr = result;
        println!("[WASM FFI] Wrote result to address {:?}", result_ptr);
    }
}

// src/lib.rs::core - Core addition logic
pub fn add_points(p1: Point, p2: Point) -> Point {
    println!("[WASM Core] add_points called with p1: {:?}, p2: {:?}", p1, p2);
    Point {
        x: p1.x.wrapping_add(p2.x),
        y: p1.y.wrapping_add(p2.y),
    }
}
```

This pattern forms the basis for complex data exchange between Wasm and the host. Key elements are agreed-upon memory
layouts, access via pointers (offsets), and the correct use of `unsafe` in Rust.

### Pattern Four: Host Directly Reads/Writes Wasm Internal State

This pattern demonstrates (but does not recommend) how the host can directly modify internal `static mut` state within
the Wasm module.

**C++ Host Side (`host.cpp`, Test 4):**

1. **Get State Pointer:** Call the Wasm-exported `get_plugin_shared_value_ptr` function. This function returns an
   `int32_t`, representing the offset of `PLUGIN_SHARED_VALUE` within Wasm linear memory.
2. **Read Initial Value:** Use `host.readMemory<int32_t>()` to read the current value of the Wasm state from the
   obtained offset.
3. **Write New Value:** Use `host.writeMemory()` to write a new `int32_t` value to that offset.
4. **Read Again to Verify:** Use `host.readMemory<int32_t>()` again to confirm the write was successful.

```cpp
// host.cpp (inside main, Test 4)
int32_t plugin_value_offset = -1;
// ...

std::cout << "[Host Main] Calling Wasm 'get_plugin_shared_value_ptr'..." << std::endl;
// getPluginDataOffset wraps calling the Wasm function to get the offset
plugin_value_offset = host.getPluginDataOffset("get_plugin_shared_value_ptr");
std::cout << "[Host Main] Received offset: " << plugin_value_offset << std::endl;

if (plugin_value_offset > 0) { // Basic validity check
    // Read Wasm state
    int32_t value_from_plugin_before = host.readMemory<int32_t>(plugin_value_offset);
    std::cout << "[Host Main] Value read from plugin: " << value_from_plugin_before << std::endl;

    // Write new value to Wasm state
    const int32_t new_value_for_plugin = 777;
    std::cout << "[Host Main] Writing new value (" << new_value_for_plugin << ") to plugin state..." << std::endl;
    host.writeMemory(plugin_value_offset, new_value_for_plugin);

    // Read again to verify
    int32_t value_from_plugin_after = host.readMemory<int32_t>(plugin_value_offset);
    std::cout << "[Host Main] Value read after host write: " << value_from_plugin_after << std::endl;
    // ... Verify value_from_plugin_after == new_value_for_plugin ...
}

// WasmHost::getPluginDataOffset implementation
int32_t WasmHost::getPluginDataOffset(std::string_view func_name) {
    std::cout << "[Host] Getting plugin data offset via '" << func_name << "'..." << std::endl;
    // Wasm function takes no args, returns i32 (offset)
    auto result_tuple = callFunction<std::tuple<int32_t>>(func_name);
    if (!result_tuple) { /* Error handling */ return -1; }
    int32_t offset = std::get<0>(result_tuple.ok());
    std::cout << "[Host] Received offset from plugin: " << offset << std::endl;
    return offset;
}
```

**Rust Wasm Side:**

1. **Define `static mut` State:** `static mut PLUGIN_SHARED_VALUE: i32 = 100;`
2. **Export Pointer Function:** Export the `get_plugin_shared_value_ptr` function, which, within an `unsafe` context,
   returns the raw pointer (offset) to `PLUGIN_SHARED_VALUE`.

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

This pattern showcases the power of memory manipulation but also highlights the potential risks. The host can now
directly interfere with Wasm's internal implementation details.

### Pattern Five: Wasm Verifies Internal State Change by Host

To confirm that the host's write in Pattern Four actually took effect, we let the Wasm module itself check the value of
that `static mut` variable.

**C++ Host Side (`host.cpp`, Test 5):**

After modifying the Wasm state in Pattern Four, call another Wasm function (e.g., `simple_add`, repurposed here). We
aren't interested in this function's return value, but rather in the log output it generates from within Wasm.

```cpp
// host.cpp (inside main, Test 5, assuming plugin_value_offset > 0)
std::cout << "[Host Main] Calling Wasm 'simple_add' to verify internal state..." << std::endl;
// Call a Wasm function, allowing it to read and print its own state
auto args = std::make_tuple(1ULL, 1ULL);
host.callFunction<std::tuple<uint64_t>, std::tuple<uint64_t, uint64_t>>(
    "simple_add", args);
std::cout << "[Host Main] Returned from 'simple_add'. Check WASM output above." << std::endl;
```

**Rust Wasm Side:**

We need to modify the `simple_add` function (or the core logic it calls, `perform_simple_add_and_read_internal_state`)
so that before performing its main task, it reads the value of `PLUGIN_SHARED_VALUE` and prints it.

```rust
// src/ffi.rs
#[no_mangle]
pub extern "C" fn simple_add(left: u64, right: u64) -> u64 {
    println!("[WASM FFI] simple_add (verification step) called...");
    crate::core::perform_simple_add_and_read_internal_state(left, right)
}

// Internal helper function to read static mut (requires unsafe)
pub(crate) fn read_plugin_value_internal() -> i32 {
    unsafe { PLUGIN_SHARED_VALUE }
}

// src/lib.rs::core
pub fn perform_simple_add_and_read_internal_state(left: u64, right: u64) -> u64 {
    // Read and print its own internal state
    let current_plugin_val = read_plugin_value_internal(); // Call FFI helper
    println!(
        "[WASM Core] Current plugin's internal shared value: {}", // Expecting 777 here
        current_plugin_val
    );

    println!("[WASM Core] Performing simple add: {} + {}", left, right);
    // ... Perform original addition logic ...
    left + right // Assuming simple return
}
```

When the host executes Test 5, we should see output from `[WASM Core]` in the console showing
`Current plugin's internal shared value: 777` (or whatever value was written in Pattern Four). This verifies that the
host successfully modified the Wasm's internal state.

## Key Takeaways and Considerations

This example highlights several crucial points when using Wasmtime for C++/Rust Wasm interactions:

1. **Clear Interface Definition:** The FFI layer is central. Rust's `extern "C"` (for both imports and exports) and the
   C++ function signatures/linking must match precisely.
2. **Memory Operations are Fundamental:** Passing complex data relies on reading and writing to Wasm's linear memory.
   Understanding pointers as offsets and ensuring consistent data structure layouts (`#[repr(C)]`, `#pragma pack`) is
   vital.
3. **Necessity of `unsafe`:** In the Rust Wasm module, interacting with the FFI and `static mut` almost inevitably
   requires `unsafe` blocks. Use them cautiously and confine them to the FFI boundary layer whenever possible.
4. **Careful State Management:** Both the host and Wasm can maintain state. They can influence each other's state
   through function calls. Directly exposing pointers to Wasm's internal state to the host, while technically feasible,
   breaks encapsulation and should generally be avoided. Prefer managing state through interface functions.
5. **Role of WASI:** For Wasm modules needing standard I/O or other system interactions (even just `println!`), the host
   must configure and link WASI.
6. **Wasmtime API:** Wasmtime provides a comprehensive C++ API (`wasmtime.hh`) featuring core classes like `Engine`,
   `Store`, `Module`, `Linker`, `Instance`, `Memory`, `Func`, `TypedFunc`, `Val`, and error handling mechanisms like
   `Result` and `Trap`. Understanding the roles and relationships of these classes is key to successful implementation.

## Conclusion

WebAssembly and Wasmtime offer a powerful way to extend existing applications and achieve high-performance, secure, and
portable modularity. The combination of C++ and Rust leverages C++'s ecosystem and performance while benefiting from
Rust's safety guarantees, making it particularly suitable for building plugin systems, handling performance-critical
tasks, or scenarios requiring strong sandboxing.

While the interaction patterns covered here are quite comprehensive, they represent just the tip of the iceberg.
Wasmtime also supports more advanced features like epoch-based interruption, fuel metering for resource control,
reference types, multiple memories, threading, and more.

Hopefully, this detailed walkthrough has helped you grasp the fundamental principles and practical methods for enabling
interaction between a C++ host and a Rust Wasm module using Wasmtime. If this area interests you, I encourage you to
experiment and integrate Wasm into your next project!
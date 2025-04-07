---
date: '2025-04-07T21:50:58+08:00'
draft: false
summary: 'Refactor a C++ EnTT host and Rust WASM plugin, replacing custom event triggers with Boost.Signals2 via Wasmtime for robust, decoupled FFI communication and advanced host-plugin interaction.'
title: 'Beyond Basic Bridging: Robust Eventing Between C++ EnTT and Rust WASM with Boost.Signals2'
tags: [ "cpp", "rust", "wasm", "webassembly", "wasmtime", "entt", "boost-signals2", "ffi", "host-plugin", "plugin-architecture", "ecs", "entity-component-system", "event-handling", "signals-and-slots", "decoupling", "refactoring", "cross-language-communication", "game-development", "simulation", "interoperability", "c-api", "bridge" ]
---

Let's dive back into our evolving C++/Rust/WASM project. In our previous explorations, we successfully:

1. Established robust methods
   for [managing entity relationships (1:1, 1:N, N:N) within the C++ EnTT ECS framework](/en/posts/weaving-the-web-managing-entity-relationships-in-entt/).
2. Built a bridge using Wasmtime
   for [bidirectional communication and memory sharing between a C++ host and a Rust WASM module](/en/posts/deep-dive-into-wasmtime-bidirectional-communication-and-memory-sharing-between-cpp-and-rust-wasm-modules/).
3. Combined these
   concepts, [creating a stable C FFI layer to allow a Rust WASM plugin to manage EnTT entity relationships residing in the C++ host](/en/posts/bridging-the-gap-flexible-relationship-management-between-cpp-host-and-rust-wasm-plugins-using-entt/).

This layered architecture, leveraging EnTT's data-oriented nature and a carefully crafted C FFI, proved effective in
overcoming the inherent limitations of the WASM boundary. However, as projects grow, the need for more sophisticated
interaction patterns emerges. Our previous solution relied on the WASM module *calling* host functions to perform
actions. What if we need the *host* to notify the WASM plugin when certain events occur within the EnTT world? What if
the WASM plugin needs to *intercept* or *modify* the behaviour of host operations?

Our initial foray into this involved creating custom "trigger" and "patching" mechanisms. While these solutions
functioned, their ad-hoc nature, often depending on string-based function lookups and requiring manual management of
callbacks, revealed significant drawbacks, rapidly leading to systems that were complex, brittle, and difficult to
maintain. We specifically encountered a number of challenges. A primary concern was type safety; the reliance on
function names represented as strings provided absolutely no compile-time guarantee that a given WASM function's
signature would actually match what the host expected for a particular trigger or patch point. Another difficulty arose
in connection management: manually keeping track of which WASM functions were registered to handle which specific events
became increasingly cumbersome, and tasks like disconnecting or updating these registrations necessitated meticulous,
error-prone bookkeeping. Furthermore, our custom system offered no inherent capability to control the execution order or
apply prioritization when multiple WASM callbacks were registered for the very same event. The handling of results
presented yet another significant problem: determining how results from potentially multiple WASM "patch" functions
should be combined, or even whether one WASM plugin should possess the ability to entirely prevent an action initiated
by the host, was left without any standard or well-defined approach within our custom framework. Lastly, a considerable
amount of boilerplate code was required; implementing the necessary registration, lookup, and invocation logic for every
single trigger or patch point involved substantial and repetitive coding effort on both the C++ host and the Rust WASM
sides of the system.

It became clear that we needed a more robust, standardized, and feature-rich eventing system. Enter `Boost.Signals2`.

This post chronicles the refactoring journey, replacing our custom trigger and patching mechanisms with the powerful and
flexible `Boost.Signals2` library. We'll explore how this transition simplifies the architecture, enhances type safety (
as much as possible across FFI), provides sophisticated features like automatic connection management, prioritization,
and result combination ("combiners"), and ultimately leads to a more maintainable and extensible host-plugin interaction
model.

We'll dissect the significant changes on both the C++ host side (introducing a `SignalManager`, adapting `WasmHost` and
`EnttManager`, and leveraging C++ macros for signal emission) and the Rust WASM side (implementing signal slots and a
new connection mechanism). Prepare for a deep dive into leveraging a mature signaling library to orchestrate complex
events across the WASM boundary.

## The Case for Signals: Why `Boost.Signals2`?

Before dismantling our existing trigger/patch system, let's understand why `Boost.Signals2` is a compelling alternative.
At its core, `Boost.Signals2` implements the **signals and slots** programming pattern, a powerful mechanism for
decoupled communication.

At its core, `Boost.Signals2` implements the signals and slots programming pattern, a potent mechanism facilitating
decoupled communication within an application. You can conceptualize signals as event broadcasters. Whenever a
particular event takes place in the system, such as an entity being on the verge of creation or a name component having
just been added, a corresponding signal object is formally "emitted" or "fired," announcing the event occurrence.

Complementing signals are the slots, which function as the designated event receivers. These slots are typically
functions or callable function objects, like C++ lambdas, that are explicitly registered or "connected" to one or more
specific signals. The crucial behavior is that when a signal is emitted, the framework automatically invokes all the
slots currently connected to that specific signal.

The link established between a particular signal and a slot is represented by a connection object. A critically
important feature offered by `Boost.Signals2`, setting it apart from many manual systems, is its provision of automatic
connection management. This means that if either the signal itself or a connected slot object ceases to exist (for
instance, by going out of scope) or if the connection is explicitly severed, the library automatically breaks the link.
This robust management prevents the common and problematic issue of dangling callbacks, where the system might attempt
to invoke a function that no longer exists, which is a significant advantage when compared to manually managed callback
lists.

Where `Boost.Signals2` particularly demonstrates its power, especially for our integration scenario, is through its
concept of combiners. A combiner is essentially a rule or a policy that dictates how the return values generated by
multiple slots, all connected to the same signal, should be aggregated or processed into a single outcome. For example,
when dealing with "before" events, like `before_create_entity`, we might desire a behavior where any single connected
slot has the power to veto or prevent the original operation from proceeding. This can be effectively achieved by
implementing a custom combiner that intelligently stops the invocation sequence and returns immediately as soon as any
slot returns `true`, thereby signaling that the operation should be skipped. Conversely, for "after" events where
connected slots might intend to modify a result, such as in the `after_get_name` scenario, we could employ a standard
combiner like `boost::signals2::optional_last_value`. This specific combiner conveniently returns the value that was
produced by the very last slot executed in the sequence, a behavior that becomes particularly useful when slots are
assigned different priorities. It's also worth noting that the default combiner behavior simply returns `void` if the
slots have no return value, or it returns a `boost::optional<ResultType>` containing the result from the last slot that
returned a non-void value.

Furthermore, `Boost.Signals2` allows slots to be connected with associated group priorities. This feature provides
developers with fine-grained control over the precise order in which slots connected to the same signal are executed
relative to one another, enabling more complex interaction sequences.

Finally, the library offers various configurable levels of thread safety. While our current host application operates in
a single thread, this capability is a crucial consideration for potentially multi-threaded host environments, ensuring
that signal emissions and slot connections can be handled safely under concurrent conditions.

By adopting `Boost.Signals2`, we replace our bespoke, error-prone system with a well-tested, feature-rich library
designed specifically for this kind of event handling, significantly improving robustness and maintainability.

## Host-Side Revolution: The `SignalManager` and Macro Magic

The most significant changes occur on the C++ host side. We need a central place to define our signals and manage
connections to WASM slots, and we need a non-intrusive way to emit these signals when our existing C API functions are
called.

### Introducing `SignalManager`

This new class becomes the heart of our host-side eventing system.

**Signal Definitions:** Inside `signal_manager.h`, we define specific signal types using `boost::signals2::signal`. The
template arguments define the signature of the slots that can connect to it (return type and parameter types).
Critically, we also specify a *combiner* type.

```cpp
// signal_manager.h (Illustrative Snippets)
#include <boost/signals2.hpp>
#include <cstdint>
#include <optional> // For optional_last_value combiner

namespace WasmHostSignals {

// Custom Combiner: Stops invocation if any slot returns true.
// Useful for "before" signals to allow skipping the original action.
struct StopOnTrueCombiner {
    typedef bool result_type; // The combiner returns bool

    template<typename InputIterator>
    result_type operator()(InputIterator first, InputIterator last) const {
        while (first != last) {
            // Dereference the iterator to get the slot's return value
            // Assuming slots connected to signals using this combiner return bool
            if (*first) { // If the slot returned true...
                return true; // ...stop and return true (indicating skip)
            }
            ++first;
        }
        return false; // No slot returned true, return false (don't skip)
    }
};

// --- Signal Type Definitions ---

// Example: Entity Creation
// bool(): Return true to skip creation.
using SignalBeforeCreateEntity = boost::signals2::signal<bool(), StopOnTrueCombiner>;
// uint32_t(uint32_t original_id): Can modify the returned ID.
using SignalAfterCreateEntity = boost::signals2::signal<uint32_t(uint32_t), boost::signals2::optional_last_value<uint32_t>>;

// Example: Entity Destruction
// bool(uint32_t entity_id): Return true to skip destruction.
using SignalBeforeDestroyEntity = boost::signals2::signal<bool(uint32_t), StopOnTrueCombiner>;
// void(uint32_t entity_id): Just a notification.
using SignalAfterDestroyEntity = boost::signals2::signal<void(uint32_t)>;

// Example: Get Name (Complex due to buffer)
// bool(uint32_t id, char* buffer, size_t buffer_len): Can skip original get.
// Note: WASM slot won't easily access the host buffer content here.
// Signature might be simplified in practice.
using SignalBeforeGetName = boost::signals2::signal<bool(uint32_t, char*, size_t), StopOnTrueCombiner>;
// size_t(uint32_t id, char* buffer, size_t buffer_len, size_t original_req_len): Can modify required_len.
using SignalAfterGetName = boost::signals2::signal<size_t(uint32_t, char*, size_t, size_t), boost::signals2::optional_last_value<size_t>>;

// ... Define signal types for all relevant host operations ...


class WasmHost; // Forward declaration

class SignalManager {
public:
    // Signals are public members for macros to access easily
    // Could be private with accessor methods too.
    SignalBeforeCreateEntity signal_before_create_entity;
    SignalAfterCreateEntity signal_after_create_entity;
    SignalBeforeDestroyEntity signal_before_destroy_entity;
    SignalAfterDestroyEntity signal_after_destroy_entity;
    // ... Other signal members ...
    SignalBeforeGetName signal_before_get_name;
    SignalAfterGetName signal_after_get_name;
    // ... and many more ...

    explicit SignalManager(WasmHost* host);
    ~SignalManager();

    // Deleted copy/move constructors/assignment operators
    SignalManager(const SignalManager&) = delete;
    SignalManager& operator=(const SignalManager&) = delete;
    // ...

    // Connects a WASM function (by name) to a specific signal (by name)
    bool connectWasmSlot(const std::string& signal_name, const std::string& wasm_func_name, int priority);

private:
    WasmHost* wasm_host_ptr_; // Needed to call back into WASM

    // Type definition for the factory function
    using WasmSlotConnectorFactory = std::function<boost::signals2::connection(
        WasmHost* host,              // Pointer to WasmHost
        boost::signals2::signal_base& signal, // Reference to the specific signal object
        const std::string& wasm_func_name,   // Name of the WASM function
        int priority                     // Priority for connection
    )>;

    // Map from signal name (string) to a factory that creates the connection lambda
    std::map<std::string, WasmSlotConnectorFactory> slot_connector_factories_;

    // Initializes the slot_connector_factories_ map
    void initializeConnectorFactories();

    // Structure to potentially track connection info (optional)
    struct WasmSlotInfo {
        std::string wasm_function_name;
        int priority = 0;
        boost::signals2::connection connection; // Stores the connection object
    };

    // Store connections grouped by signal name (optional, for management)
    std::map<std::string, std::vector<std::shared_ptr<WasmSlotInfo>>> wasm_connections_;
};

} // namespace WasmHostSignals
```

Several critical design decisions shape the effectiveness of the `SignalManager`. The choice of combiners is fundamental
to defining the interaction logic for different event types. For instance, we specifically define our custom
`StopOnTrueCombiner` for signals intended to run *before* an operation (like `before_create_entity`), enabling any
connected slot to prevent the original action simply by returning `true`. For signals emitted *after* an operation,
especially those where slots might wish to modify a return value (such as `after_create_entity` potentially altering the
returned ID), we utilize the standard `boost::signals2::optional_last_value<T>` combiner. This combiner has the behavior
of returning the value produced by the very last slot that executed in the sequence, a feature that integrates naturally
with the priority system. In cases where the signal serves purely as a notification (like `after_destroy_entity`), the
default combiner, which simply returns `void`, is perfectly adequate.

The definition of signal signatures, such as `bool()`, `uint32_t(uint32_t)`, `void(uint32_t)`, and so forth, plays a
crucial role in establishing the contract for any slots wishing to connect. These signatures dictate the exact parameter
types and the return type that a compliant slot function must adhere to, which is essential for maintaining type safety
across the system. It's noteworthy that even complex scenarios, like the `before_get_name` signal, initially include
buffer details (`char*`, `size_t`) in their signature to match the underlying C API. However, we recognize the practical
difficulties of WASM slots directly manipulating host memory buffers via these parameters and anticipate that the actual
WASM slot implementation might simplify its approach, perhaps ignoring these buffer arguments and opting to call back
into the host via another FFI function if the buffer content is needed.

Connecting WASM functions is facilitated by the `connectWasmSlot` public method. This function serves as the designated
entry point that the WASM module will ultimately invoke, using the intermediary `host_connect_signal` FFI function, to
register its handlers as slots. `connectWasmSlot` requires the string name of the target signal on the host and the
string name of the function exported by the WASM module that should be connected to it.

Internally, the setup relies heavily on the `initializeConnectorFactories` private method, which is executed within the
`SignalManager`'s constructor. This method's responsibility is to populate the `slot_connector_factories_` map. This map
uses the string name of a signal (e.g., the literal string `"before_create_entity"`) as its key. The corresponding value
for each key is a C++ lambda function, which we term a "lambda factory."

Each lambda factory stored within the `slot_connector_factories_` map is precisely engineered to perform a single,
specific task: it knows how to connect a WASM function, identified by its name string, to one particular, hardcoded
`Boost.Signals2` signal member within the `SignalManager` instance (e.g., the factory associated with the key
`"before_create_entity"` knows it must operate on the `signal_before_create_entity` member). To achieve this, the
factory lambda typically captures the `this` pointer of the `SignalManager` or sometimes directly captures the specific
signal member it targets. It's designed to accept several arguments: a pointer to the `WasmHost` instance (necessary for
invoking WASM functions), a reference to the specific target signal object (passed as a `signal_base&` for polymorphism
within the factory signature, requiring a `static_cast` back to the concrete signal type inside), the string name of the
WASM function to connect, and the desired connection priority. The core action within the factory lambda is the call
`signal.connect(priority, [host, wasm_func] (...) { ... })`. The crucial element here is the *second* lambda passed to
`signal.connect` – this inner lambda is the *actual slot wrapper*. This wrapper lambda is precisely what the
`Boost.Signals2` framework will execute whenever the specific Boost signal it's connected to is emitted. The logic
embedded within this slot wrapper lambda is responsible for bridging the gap to Wasmtime. It receives arguments directly
from the Boost signal emission, matching the Boost signal's defined signature (for example, the `original_id` parameter
for `signal_after_create_entity`). Its first task is to marshal these incoming C++ arguments into the format Wasmtime
expects, typically a `std::vector<wasmtime::Val>`. Next, it invokes the target WASM function by name using the
`WasmHost` pointer and its `callFunction` method (e.g., `host->callFunction<ReturnType>(wasm_func, args)`), carefully
specifying the expected `ReturnType` based on the WASM function's FFI signature (like `int32_t` for a WASM function
returning a boolean, or `uint32_t` for one returning an entity ID). This call inherently involves handling potential
Wasmtime traps, usually by checking the `Result` returned by `callFunction`. If the WASM call is successful, the wrapper
then unmarshals the resulting `wasmtime::Val` back into the C++ data type that is expected by the *combiner* associated
with the Boost signal (for instance, converting an `int32_t` result back to a `bool` for signals using the
`StopOnTrueCombiner`, or to a `uint32_t` for those using `optional_last_value<uint32_t>`). Finally, this unmarshalled
C++ value is returned by the slot wrapper lambda, feeding it back into the Boost signal's processing mechanism (
specifically, its combiner).

To correctly route the connection request, the `connectWasmSlot` method must determine the actual
`boost::signals2::signal` member object corresponding to the provided `signal_name` string. The current implementation
employs a straightforward, albeit potentially lengthy, `if/else if` cascade to perform this mapping. It compares the
input string against known signal names and, upon finding a match, passes a reference to the appropriate signal member (
like `signal_before_create_entity`) into the corresponding factory lambda retrieved from the `slot_connector_factories_`
map.

Finally, robust connection management is implicitly handled by `Boost.Signals2`. While the code includes an optional
mechanism to store the `boost::signals2::connection` object returned by `connect` within a `wasm_connections_` map (
keyed by signal name), which could facilitate more granular future management like targeted disconnections, the primary
benefit comes from the `SignalManager`'s destructor. Within the destructor, all stored connections are explicitly
disconnected. More importantly, even without this explicit storage, Boost guarantees that connections are automatically
broken if either the signal or the slot's context (which isn't directly applicable here since our slots are host-side
lambdas calling WASM) is destroyed, significantly mitigating the risk of dangling pointers or callbacks.

`WasmHost` now creates and owns both the `SignalManager` and the `EnttManager`, passing the `SignalManager` pointer to
the `EnttManager` constructor. `EnttManager` itself is simplified – it no longer manages triggers directly but uses its
`SignalManager` pointer to emit signals where appropriate (primarily in the `onEntityDestroyedSignalHook`).

### Emitting Signals via Macros (`host_macros.h`)

We need to trigger these signals whenever the corresponding host C API functions are called *from WASM*. We could
manually insert signal emission code into every host function lambda in `host.cpp`, but that's repetitive and
error-prone. Instead, we use C++ macros defined in `host_macros.h`.

```cpp
// host_macros.h (Illustrative Snippet)
#pragma once

#include "entt_api.h"
#include "signal_manager.h"
#include "wasm_host.h"
#include <wasmtime.hh>
#include <vector>
#include <string>
#include <optional>
#include <stdexcept> // For runtime_error

// Helper within namespace to avoid polluting global scope
namespace WasmHostUtils {
// (Keep read_wasm_string_helper, check_result, handle_wasm_trap helpers here)
} // namespace WasmHostUtils

// Macro to define a host function taking 0 arguments and returning a value
#define DEFINE_HOST_FUNC_0_ARGS_RET(LINKER, HOST_PTR, MGR_HANDLE, NAME, C_API_FUNC, WASM_TYPE, RET_TYPE, WASM_RET_TYPE, DEFAULT_RET) \
    (LINKER).func_new(                                                                 \
        "env", (NAME), (WASM_TYPE),                                                    \
        [(HOST_PTR), (MGR_HANDLE)](                                                    \
            wasmtime::Caller caller,                                                   \
            wasmtime::Span<const wasmtime::Val> args,                                  \
            wasmtime::Span<wasmtime::Val> results                                      \
        ) -> wasmtime::Result<std::monostate, wasmtime::Trap> {                        \
            using namespace WasmHostSignals;                                           \
            using namespace WasmHostUtils;                                             \
            SignalManager& sig_mgr = (HOST_PTR)->getSignalManager();                   \
            RET_TYPE final_result = (DEFAULT_RET); /* Initialize with default */      \
            try {                                                                      \
                /* --- Before Signal --- */                                            \
                /* Assuming signal names match: before_NAME */                         \
                bool skip = sig_mgr.signal_##before_##C_API_FUNC();                    \
                if (skip) {                                                            \
                    std::cout << "[Host Signal] Skipping " << (NAME) << " due to before_ signal." << std::endl; \
                } else {                                                               \
                    /* --- Original C API Call --- */                                  \
                    RET_TYPE original_result = C_API_FUNC((MGR_HANDLE));               \
                                                                                       \
                    /* --- After Signal --- */                                         \
                    /* Assuming signal names match: after_NAME */                      \
                    /* Pass original result, combiner decides final result */          \
                    final_result = sig_mgr.signal_##after_##C_API_FUNC(original_result); \
                }                                                                      \
                /* --- Marshall result for WASM --- */                                 \
                results[0] = wasmtime::Val(static_cast<WASM_RET_TYPE>(final_result));  \
                return std::monostate();                                               \
            } catch (const wasmtime::Trap& trap) {                                     \
                 std::cerr << "[Host Function Error] " << (NAME) << " trapped: " << trap.message() << std::endl; \
                 return wasmtime::Trap(trap.message()); /* Create new trap */          \
            } catch (const std::exception& e) {                                        \
                 std::cerr << "[Host Function Error] " << (NAME) << " exception: " << e.what() << std::endl; \
                 return wasmtime::Trap(std::string("Host function ") + (NAME) + " failed: " + e.what()); \
            } catch (...) {                                                            \
                 std::cerr << "[Host Function Error] " << (NAME) << " unknown exception." << std::endl; \
                 return wasmtime::Trap(std::string("Host function ") + (NAME) + " failed with unknown exception."); \
            }                                                                          \
        }                                                                              \
    ).unwrap() /* Use unwrap() for example, check Result in prod */


// Other macros for different signatures (e.g., U32_VOID, U32_STR_VOID, U32_GET_STR...)
// Example: Macro for uint32_t argument, void return
#define DEFINE_HOST_FUNC_U32_VOID(LINKER, HOST_PTR, MGR_HANDLE, NAME, C_API_FUNC, WASM_TYPE) \
    (LINKER).func_new(                                                                   \
        "env", (NAME), (WASM_TYPE),                                                      \
        /* Lambda implementation similar to above */                                     \
        [(HOST_PTR), (MGR_HANDLE)](/* ... */) -> wasmtime::Result<std::monostate, wasmtime::Trap> { \
            /* ... extract uint32_t arg ... */                                           \
            uint32_t arg0_u32 = /* ... */;                                               \
            try {                                                                        \
                bool skip = sig_mgr.signal_##before_##C_API_FUNC(arg0_u32);              \
                if (!skip) {                                                             \
                    C_API_FUNC((MGR_HANDLE), arg0_u32);                                  \
                    sig_mgr.signal_##after_##C_API_FUNC(arg0_u32);                       \
                } else { /* Log skip */ }                                                \
                return std::monostate(); /* Void return */                               \
            } catch(/* ... trap/exception handling ... */) { /* ... */ }                 \
        }                                                                                \
    ).unwrap()

// Example: Macro for uint32_t argument, getting a string
#define DEFINE_HOST_FUNC_U32_GET_STR(LINKER, HOST_PTR, MGR_HANDLE, NAME, C_API_FUNC, WASM_TYPE) \
    (LINKER).func_new(                                                                    \
        "env", (NAME), (WASM_TYPE),                                                       \
        /* Lambda implementation */                                                       \
        [(HOST_PTR), (MGR_HANDLE)](/* ... */) -> wasmtime::Result<std::monostate, wasmtime::Trap> { \
            /* ... extract uint32_t id, char* buffer_ptr_offset, size_t buffer_len ... */ \
            uint32_t entity_id = /* ... */;                                               \
            int32_t buffer_offset = /* ... */;                                            \
            size_t buffer_len = /* ... */;                                                \
            char* wasm_buffer = nullptr;                                                  \
            try {                                                                         \
                /* Get memory and calculate wasm_buffer pointer safely */                 \
                auto mem_span_opt = WasmHostUtils::get_wasm_memory_span_helper(caller);   \
                if (!mem_span_opt) return wasmtime::Trap("Failed to get WASM memory");     \
                auto& mem_span = mem_span_opt.value();                                    \
                if (buffer_offset >= 0 && buffer_len > 0 /* ... more bounds checks ... */){ \
                    wasm_buffer = reinterpret_cast<char*>(mem_span.data() + buffer_offset);\
                } else if (buffer_offset != 0 || buffer_len > 0) { /* Invalid buffer args */ } \
                                                                                          \
                size_t final_req_len = 0; /* Default */                                   \
                bool skip = sig_mgr.signal_##before_##C_API_FUNC(entity_id, wasm_buffer, buffer_len); \
                if (!skip) {                                                              \
                    size_t original_req_len = C_API_FUNC((MGR_HANDLE), entity_id, wasm_buffer, buffer_len); \
                    /* Pass original_req_len to after signal */                           \
                    final_req_len = sig_mgr.signal_##after_##C_API_FUNC(entity_id, wasm_buffer, buffer_len, original_req_len); \
                } else { /* Log skip, return 0 */ final_req_len = 0; }                  \
                results[0] = wasmtime::Val(static_cast<int32_t>(final_req_len)); /* Return size_t as i32 */ \
                return std::monostate();                                                  \
            } catch(/* ... trap/exception handling ... */) { /* ... */ }                  \
        }                                                                                 \
    ).unwrap()

// ... More macros for other patterns ...
```

The C++ macros defined in `host_macros.h` encapsulate several key elements essential for integrating the host's C API
functions with the `Boost.Signals2` event system when exposing them to the WASM module via Wasmtime. Their primary
function is boilerplate reduction; they conveniently wrap the necessary `linker.func_new` call required by Wasmtime and
construct the complex lambda function that serves as the actual host function implementation callable by WASM.

These macros are highly parameterized to handle different function signatures. They typically accept arguments such as
the Wasmtime linker object, a pointer to the `WasmHost` instance, the opaque handle for the `EnttManager`, the specific
name the WASM module will use to import the function (referred to as `NAME`), a pointer to the underlying C API function
being wrapped (`C_API_FUNC`), the corresponding Wasmtime function type definition, the expected C++ return type of the C
API function, the corresponding WASM ABI type for the return value (e.g., `int32_t` for a C `int` or `uint32_t`), and a
default return value to use if the operation is skipped by a signal.

Within the lambda generated by the macro, specific captures are essential. The lambda captures the `HOST_PTR`, which is
crucial for gaining access to the `SignalManager` instance needed to emit signals, and it also captures the
`MGR_HANDLE`, the opaque pointer required to invoke the original C API function.

The lambda implementation handles the intricate details of argument and result marshalling across the WASM boundary.
It's responsible for extracting incoming arguments from the `Span<const wasmtime::Val>` provided by Wasmtime, converting
them to the types expected by the C API function. For functions dealing with buffers or strings, it performs necessary
bounds checking, often using helper functions, to ensure memory safety when interacting with WASM's linear memory. After
the operation and potential signal handling, it marshals the final computed result back into the `Span<wasmtime::Val>`
for the WASM caller.

A core responsibility of the macro-generated lambda is signal emission. It first retrieves the `SignalManager` instance
via the captured `HOST_PTR`. Then, *before* invoking the wrapped C API function, it emits the corresponding "before"
signal. This emission uses C++ preprocessor token pasting (`##`) to dynamically construct the correct signal member name
based on the C API function's name (for example, combining `signal_##before_##` with `entt_manager_create_entity`
results in `signal_before_entt_manager_create_entity`). The lambda carefully checks the return value provided by the "
before" signal's combiner (e.g., the boolean result from `StopOnTrueCombiner`). If this return value indicates that the
operation should be skipped (typically `true`), the lambda logs a message and immediately returns the predefined default
value to WASM, bypassing the call to the original C API function and the emission of the "after" signal. If the "before"
signal does not indicate a skip, the lambda proceeds to call the original C API function (`C_API_FUNC`) using the
captured manager handle and extracted arguments. Following the C API call, it emits the corresponding "after" signal,
passing any relevant original arguments along with the result obtained from the C API call. Finally, it captures the
return value generated by the "after" signal's combiner (which might have been modified by WASM slots, for example,
using `optional_last_value`) and uses this value as the `final_result` that is ultimately marshalled and returned to the
WASM caller.

Lastly, robust error handling is built into the generated lambda. It includes comprehensive `try-catch` blocks designed
to catch standard C++ exceptions (`std::exception`) as well as Wasmtime-specific traps (`wasmtime::Trap`) that might
occur during the execution of the C API function, the signal emissions, or the slot invocations within WASM. These
caught exceptions or traps are then safely converted into new `wasmtime::Trap` objects, ensuring that host-side errors
are propagated back to the WASM runtime gracefully without crashing the host process. Special care is taken to correctly
handle the move-only semantics of `wasmtime::Trap` when re-throwing or constructing new traps.

In `host.cpp`, we now replace the direct lambda definitions with calls to these macros for each host function we want to
expose *with signal support*.

```cpp
// host.cpp (main, illustrative usage)

// ... includes, setup ...

// Get pointers and references
WasmHost host(wasm_path);
EnttManager* manager_raw_ptr = &host.getEnttManager();
EnttManagerHandle* manager_handle = reinterpret_cast<EnttManagerHandle*>(manager_raw_ptr);
Linker& linker = host.getLinker();
Store& store = host.getStore();
WasmHost* host_ptr = &host; // For macro capture
// SignalManager& signal_manager = host.getSignalManager(); // Not directly needed here

host.setupWasi();

// Define function types...

// Use the macros to define host functions
DEFINE_HOST_FUNC_0_ARGS_RET(linker, host_ptr, manager_handle,
                            "host_create_entity", entt_manager_create_entity, void_to_i32_type,
                            uint32_t, int32_t, FFI_NULL_ENTITY);

DEFINE_HOST_FUNC_U32_VOID(linker, host_ptr, manager_handle,
                          "host_destroy_entity", entt_manager_destroy_entity, i32_to_void_type);

DEFINE_HOST_FUNC_U32_GET_STR(linker, host_ptr, manager_handle,
                             "host_get_name", entt_manager_get_name, i32ptrlen_to_size_type);

// ... Define ALL other host functions using the appropriate macros ...

// Define the signal connection function (doesn't need a macro as it doesn't wrap a C API call)
linker.func_new( "env", "host_connect_signal", /* type */ ...,
    // Capture signal_manager by reference
    [&signal_manager = host.getSignalManager()](...) { // Note capture detail
         // ... implementation using signal_manager.connectWasmSlot ...
    }
).unwrap();


host.initialize(); // Instantiate

// ... Call connect_all_signals in WASM ...
// ... Call test_relationships_oo in WASM ...

// ... Manual test section calling linker.get() to ensure signal firing ...

```

### Connecting WASM Slots: `host_connect_signal`

How does the WASM module tell the host, "Please connect my `wasm_before_create_entity` function to your
`before_create_entity` signal"? We provide *one more* host function specifically for this: `host_connect_signal`.

This specific host function, `host_connect_signal`, is defined directly within `host.cpp` using `linker.func_new` and a
lambda, rather than relying on one of the host function macros, primarily because it doesn't wrap an existing C API
function but instead provides new functionality specific to the signal system. The lambda implementation performs
several distinct steps when invoked by the WASM module.

First, it receives its necessary input arguments directly from the WASM caller via the `Span<const Val> args`. These
arguments consist of pointers and lengths representing the signal name (`signal_name_ptr`, `signal_name_len`) and the
WASM function name (`wasm_func_name_ptr`, `wasm_func_name_len`), along with an integer representing the desired
connection `priority`.

Next, to safely retrieve the actual string values from the potentially insecure pointers provided by WASM, the lambda
utilizes the `WasmHostUtils::read_wasm_string_helper` utility function. This helper reads the specified number of bytes
from the WASM linear memory at the given offsets, performing necessary bounds checks and returning the strings.

Crucially, the lambda is defined in a way that it captures a reference to the host's central `SignalManager` instance.
This captured reference provides the context needed to interact with the signal system.

With the signal and function names successfully read and the `SignalManager` accessible, the core logic of the lambda is
executed: it invokes the `connectWasmSlot` method on the captured `signal_manager`, passing the retrieved `signal_name`,
`wasm_func_name`, and `priority` as arguments. This call delegates the actual task of creating and registering the
signal-slot connection to the `SignalManager`.

Finally, after the connection attempt, the lambda returns the outcome back to the WASM module. It takes the boolean
success status returned by `connectWasmSlot` and marshals it into the expected FFI format, typically an `int32_t` (1 for
success, 0 for failure), which is placed into the `Span<Val> results` for the WASM caller to interpret.

This provides the crucial link, allowing the WASM module to dynamically register its handlers during its initialization
phase.

## WASM-Side Adaptation: Becoming a Signal Client

The Rust WASM module now needs to adapt to this new signal-based system.

The first step in adapting the Rust WASM module involves dismantling the previous custom eventing infrastructure. This
cleanup requires removing the remnants of the now-obsolete trigger and patching systems. Specifically, the
`src/patch_handler.rs` file, along with the `PatchHandler` trait defined within it, must be entirely deleted from the
project. Correspondingly, within the FFI layer defined in `src/ffi.rs`, the `extern "C"` import declarations that
previously brought in the host functions related to registering patches and triggers, namely `host_register_patch` and
`host_register_trigger`, need to be removed. Finally, the exported WASM functions that served as the initialization
entry points for these old systems, `init_patches` and `init_triggers`, must also be removed from the exports list, as
the host will no longer call them.

With the old plumbing removed, a new mechanism must be established to allow the WASM module to initiate the connection
of its handlers to the host's signals. This new process involves several coordinated steps within the Rust code. First,
the necessary FFI import declaration for the new host function responsible for handling connections,
`host_connect_signal`, must be added to the `extern "C"` block located in `src/ffi.rs`, mirroring the function signature
defined on the host side. Second, to encapsulate the unsafe FFI interaction, a safe Rust wrapper function,
`ffi::connect_signal`, needs to be created. This wrapper function should accept standard Rust string slices (`&str`) for
the signal name and the WASM function name, along with an integer priority. Its implementation will handle the necessary
conversions of these Rust strings into null-terminated CStrings suitable for the FFI call and will contain the `unsafe`
block required to invoke the imported `host_connect_signal` function, returning a boolean indicating the success or
failure of the connection attempt. Third, the responsibility for orchestrating all necessary connections from the WASM
side is centralized within a new function, `core::connect_all_signals`, implemented in `src/core.rs`. This function's
sole purpose is to repeatedly call the safe `ffi::connect_signal` wrapper, systematically pairing the known string names
of the signals exposed by the host (such as `"before_create_entity"`) with the string names of the corresponding
exported WASM functions designed to handle those signals (like `"wasm_before_create_entity"`), along with their desired
priorities. Fourth and finally, to expose this connection logic to the host, a C-compatible function named
`connect_all_signals` needs to be exported from `src/ffi.rs` using `#[no_mangle] pub unsafe extern "C"`. The
implementation of this exported function simply delegates the actual work by calling `core::connect_all_signals()`. The
C++ host application will then be responsible for invoking this single exported `connect_all_signals` function exactly
once, typically right after the WASM module has been successfully instantiated, thereby triggering the registration of
all defined WASM signal handlers with the host's `SignalManager`.

```rust
// src/ffi.rs (Snippets)

// ... other imports ...
#[link(wasm_import_module = "env")]
unsafe extern "C" {
    // --- Signal Connection Import (NEW) ---
    fn host_connect_signal(
        signal_name_ptr: *const c_char,
        signal_name_len: usize,
        wasm_func_name_ptr: *const c_char,
        wasm_func_name_len: usize,
        priority: c_int,
    ) -> c_int; // Returns bool (0 or 1) for success

    // ... other host function imports remain ...
}

// --- Signal Connection Wrapper (NEW) ---
pub fn connect_signal(
    signal_name: &str,
    wasm_func_name: &str,
    priority: i32,
) -> bool {
    // ... (Implementation as shown previously, using CString::new and unsafe call) ...
    let success_code = unsafe { host_connect_signal(...) };
    success_code != 0
}

// --- Exported Function for Host to Trigger Connections ---
#[no_mangle]
pub unsafe extern "C" fn connect_all_signals() {
    println!("[WASM Export] connect_all_signals called. Connecting handlers via core...");
    crate::core::connect_all_signals(); // Delegate to core logic
}

// --- Exported Signal Handler Implementations (Slots) ---
// ... (Functions like wasm_before_create_entity as defined previously) ...

// --- Test Runner Export ---
#[no_mangle]
pub unsafe extern "C" fn test_relationships_oo() {
     // ... runs core::run_entt_relationship_tests_oo ...
}
```

```rust
// src/core.rs (Snippet)

use crate::ffi::{connect_signal, DEFAULT_SIGNAL_PRIORITY /* ... */};

/// Connects all WASM signal handlers (slots) to the corresponding host signals.
/// Called by the host via the exported `connect_all_signals` function in ffi.rs.
pub fn connect_all_signals() {
    println!("[WASM Core] Connecting WASM functions to host signals...");
    let mut success = true;

    // Connect slots for host_create_entity
    success &= connect_signal(
        "before_create_entity",      // Host signal name (string)
        "wasm_before_create_entity", // Exported WASM function name (string)
        DEFAULT_SIGNAL_PRIORITY,
    );
    success &= connect_signal(
        "after_create_entity",       // Host signal name
        "wasm_after_create_entity",  // Exported WASM function name
        DEFAULT_SIGNAL_PRIORITY,
    );
    // ... connect ALL other slots similarly ...
     success &= connect_signal(
        "after_get_profile_for_player",
        "wasm_after_get_profile_for_player_high_prio", // Name matches exported function
        DEFAULT_SIGNAL_PRIORITY + 100,                 // Higher priority number
    );


    if success { /* Log success */ } else { /* Log failure */ }
}

// ... run_entt_relationship_tests_oo() remains largely the same ...
```

### Implementing WASM Signal Slots

The Rust functions that were previously designated for the custom patching mechanism, such as `prefix_create_entity`,
are now either repurposed or replaced by new functions specifically designed to serve as the signal slots within the
`Boost.Signals2` framework. For these functions to correctly receive signals emitted by the host, they must adhere to
two fundamental requirements.

Firstly, they must be properly exported from the WASM module so that the host's `SignalManager` (via Wasmtime) can
locate and invoke them when connecting or firing signals. This necessitates marking each slot function with
`#[no_mangle]` to prevent Rust's name mangling and declaring it as `pub unsafe extern "C"` to ensure C-compatible
linkage and calling conventions. Critically, the exact name assigned to each exported slot function in the Rust code
must perfectly match the string literal used when connecting it within the `core::connect_all_signals` function; any
discrepancy will result in a connection failure.

Secondly, and equally crucial, the function signature of each WASM slot – encompassing both its parameters and its
return type – must precisely align with the expectations hardcoded into the corresponding host-side slot wrapper lambda.
These wrapper lambdas are defined within the `SignalManager::initializeConnectorFactories` method in the C++ host. Any
mismatch in the number of parameters, their types, or the return type will lead to undefined behavior or runtime traps
when the host attempts to call the WASM slot. For instance, the slot `wasm_before_create_entity()` is expected by the
host wrapper to take no arguments and return a `c_int`, where `0` signifies continuation and `1` indicates the operation
should be skipped. Similarly, `wasm_after_create_entity(original_id: u32)` must accept a `u32` representing the original
entity ID and return a `u32`, allowing it the opportunity to modify the ID passed back through the signal chain. A slot
like `wasm_after_destroy_entity(entity_id: u32)` is expected to accept the ID but return `void`, as it functions purely
as a notification. More complex cases like `wasm_before_get_name(entity_id: u32, buffer_len: u32)` demonstrate a
simplification in the FFI signature; the host wrapper expects it to receive the entity ID and the intended buffer length
but *not* the host-side buffer pointer itself, returning a `c_int` (0 or 1) to potentially veto the `get_name`
operation. This design choice avoids the complexity and potential unsafety of the WASM slot directly accessing the host
buffer; should the slot require the actual string content during this "before" phase, it would need to initiate a
separate call back into the host (e.g., using `host_get_name` itself). Correspondingly, the
`wasm_after_get_name(entity_id: u32, buffer_len: u32, original_req_len: u32)` slot receives the ID, buffer length, and
the original required length calculated by the C API, and is expected to return a `u32` representing the potentially
adjusted required length. This pattern of precisely matching the parameter list and return type defined implicitly by
the host's slot wrapper lambda must be rigorously applied to all other WASM functions intended to serve as signal slots
for the various host events.

```rust
// src/ffi.rs (Slot Implementation Snippets)

// --- host_create_entity ---
#[no_mangle]
pub unsafe extern "C" fn wasm_before_create_entity() -> c_int {
    println!("[WASM Slot] <<< before_create_entity called");
    0 // Allow creation
}

#[no_mangle]
pub unsafe extern "C" fn wasm_after_create_entity(original_id: u32) -> u32 {
    println!("[WASM Slot] <<< after_create_entity called (Original ID: {})", original_id);
    original_id // Return original ID
}

// --- host_get_profile_for_player ---
#[no_mangle]
pub unsafe extern "C" fn wasm_before_get_profile_for_player(player_id: u32) -> c_int {
    println!("[WASM Slot] <<< before_get_profile_for_player called (P: {})", player_id);
    0 // Allow get
}

// Default priority postfix slot
#[no_mangle]
pub unsafe extern "C" fn wasm_after_get_profile_for_player(player_id: u32, original_profile_id: u32) -> u32 {
    println!("[WASM Slot] <<< after_get_profile_for_player called (P: {}, OrigProf: {})", player_id, original_profile_id);
    // This one just observes
    original_profile_id
}

// High priority postfix slot (runs AFTER the default one)
#[no_mangle]
pub unsafe extern "C" fn wasm_after_get_profile_for_player_high_prio(player_id: u32, current_profile_id: u32) -> u32 {
    println!(
        "[WASM Slot][HIGH PRIO] <<< after_get_profile_for_player_high_prio called (P: {}, CurrentProf: {})",
        player_id, current_profile_id // current_profile_id is the result from the previous slot/original call
    );
    // Example: Override profile ID for player 2
    if player_id == 2 {
        println!("    > [HIGH PRIO] Changing profile for player 2 to 888!");
        return 888; // Override the value
    }
    current_profile_id // Otherwise, return the value passed in
}

// ... Implement ALL other exported slot functions ...
```

Now, when the host emits `signal_before_create_entity`, the `wasm_before_create_entity` function in the WASM module will
be executed. When the host emits `signal_after_get_profile_for_player`, both `wasm_after_get_profile_for_player` and
`wasm_after_get_profile_for_player_high_prio` will run (in priority order), and the `optional_last_value` combiner will
ensure the final result seen by the host macro is the value returned by the high-priority slot.

## The Full Picture: Execution Flow with Signals

To understand the interplay between the WASM module, the host, and the signal system, let's trace the sequence of events
when the WASM module initiates an entity creation by calling `host_create_entity`. We assume the WASM slots
`wasm_before_create_entity` and `wasm_after_create_entity` have already been successfully connected to the corresponding
host signals via `connect_all_signals`.

The process begins within the WASM module. A call to the higher-level `entity::create_entity()` function occurs, which
in turn invokes the lower-level FFI wrapper `ffi::create_entity()`. Inside this FFI wrapper, the `unsafe` block executes
the actual call across the boundary: `host_create_entity()`.

Control now transfers to the C++ host. The specific lambda wrapper function, previously generated by the
`DEFINE_HOST_FUNC_0_ARGS_RET` macro and registered with Wasmtime's linker for the name `host_create_entity`, receives
this incoming call. The first action within this host lambda is to obtain a reference to the `SignalManager`. Following
this, the lambda emits the `signal_before_create_entity` signal, passing no arguments as per the signal's definition.

The `Boost.Signals2` framework intercepts this signal emission and proceeds to invoke any slots connected to
`signal_before_create_entity`. In our scenario, this triggers the execution of the host-side slot wrapper lambda that
was created specifically for the WASM function `"wasm_before_create_entity"`. This slot wrapper lambda prepares its
arguments (none in this case) and executes a Wasmtime call back into the module:
`host->callFunction<int32_t>("wasm_before_create_entity", ...)`.

Execution jumps back to the WASM module, specifically to the `wasm_before_create_entity()` function. This function runs
its logic, typically printing a log message indicating it was called, and then returns its result, which is `0` (
representing false in C ABI boolean convention), signaling that the operation should proceed.

Back in the host, the slot wrapper lambda receives the `int32_t` result (`0`) from the Wasmtime call and unmarshals it
into a C++ `bool` (`false`). This boolean result is then passed back to the `Boost.Signals2` framework. The
`StopOnTrueCombiner` associated with `signal_before_create_entity` receives this `false` value. Since it's not `true`,
the combiner allows processing to continue (if other slots were connected, they would run now). Ultimately, the combiner
returns `false` to the original host function lambda that emitted the signal.

The host lambda checks the `skip` flag returned by the combiner. Since it's `false`, the lambda determines that the
operation should not be skipped and proceeds with the core logic. It now calls the underlying C API function:
`entt_manager_create_entity(manager_handle)`. This C API function, in turn, calls the `EnttManager::createEntity()`
method on the C++ manager object. Inside the manager, `registry_.create()` is invoked, a new EnTT entity is created, its
ID is converted to `uint32_t`, a creation log message is printed, and this `uint32_t` ID is returned.

The ID (`original_result`) travels back up the call stack from `EnttManager` to the C API function, and then to the host
lambda. Now, the host lambda emits the second signal: `signal_after_create_entity(original_result)`, passing the newly
created entity's ID.

Again, `Boost.Signals2` takes over, invoking the slots connected to `signal_after_create_entity`. This leads to the
execution of the slot wrapper lambda associated with `"wasm_after_create_entity"`, which is called with the
`original_id`. This wrapper lambda prepares its arguments (packing the `original_id` into a `wasmtime::Val`) and calls
back into the module: `host->callFunction<int32_t>("wasm_after_create_entity", ...)`. Note the expected return is
`int32_t` because the WASM function returns `u32`, which fits in `i32`.

Execution returns to WASM's `wasm_after_create_entity(original_id)` function. It executes its logic (e.g., logging) and,
in this example, simply returns the `original_id` it received.

The host slot wrapper receives this ID as an `int32_t` result from Wasmtime and unmarshals it back into a `uint32_t`.
This value is passed to the `Boost.Signals2` framework. The `optional_last_value<uint32_t>` combiner associated with
`signal_after_create_entity` receives this result. As it's the only (or the last) slot executed, the combiner wraps this
value and returns `boost::optional<uint32_t>(result)` to the host lambda.

The host lambda receives the combiner's result (`boost::optional<uint32_t>`). It extracts the contained value (or would
use a default if the optional were empty, though not expected here). This extracted value becomes the `final_result`.
The lambda then marshals this `final_result` (the entity ID) into the `results` span as a `wasmtime::Val` of kind `I32`
for the original WASM caller.

Finally, the host lambda completes its execution by returning success (`std::monostate()`) to the Wasmtime runtime.
Wasmtime then returns control back to the point where the initial `host_create_entity()` call was made within WASM's
`ffi::create_entity` function. This function receives the ID and returns it up to `entity::create_entity`, which then
uses `Entity::new(id)` to create the final Rust wrapper object for the newly created entity. This completes the entire
cross-boundary call sequence, including signal interceptions.

This detailed flow illustrates the powerful orchestration provided by `Boost.Signals2`, handling slot invocation,
argument passing (from signal to slot wrapper), return value combination, and allowing interception points before and
after the core C API logic, all while integrating with Wasmtime calls across the FFI boundary.

## Benefits and Considerations Revisited

This significant refactoring effort yields substantial benefits for the overall architecture and maintainability of the
C++/Rust/WASM integration. Foremost among these is the establishment of a unified mechanism; the `Boost.Signals2` system
now replaces both the previous custom trigger implementation and the separate patching framework, providing a single,
consistent model for handling events between the host and the plugin. This contributes significantly to the system's
robustness. `Boost.Signals2` inherently manages signal-slot connections automatically, effectively preventing the common
issue of dangling callbacks that could arise in manual systems. Furthermore, its built-in combiner concept offers
standard and predictable methods for aggregating or processing results when multiple listeners (WASM slots) respond to
the same host signal. The refactoring also promotes better decoupling within the host application. The C API
implementation layer (`entt_api.cpp`), for instance, becomes considerably simpler as it no longer needs any intrinsic
awareness of the trigger or patching logic. The `EnttManager` class is similarly streamlined, offloading event
management responsibilities. Instead, the newly introduced C++ macros and the dedicated `SignalManager` now cleanly
encapsulate the logic related to signal emission and connection management. The system gains considerable flexibility
through the features offered by Boost.Signals2; assignable priorities allow for precise control over the execution order
of different WASM slots connected to the same signal, while the availability of various combiners enables the
implementation of diverse interaction patterns, such as allowing WASM to veto host actions, modify return values, or
simply receive notifications. Ultimately, this leads to improved maintainability. The clearer separation of concerns
between core logic, the C API, the signal management infrastructure, and the WASM FFI/slot implementation, combined with
the reliance on a well-established standard library like Boost.Signals2, makes the entire codebase easier for developers
to understand, debug, and modify safely over time.

However, adopting this approach also introduces several considerations that must be acknowledged. The most obvious is
the introduction of a new external dependency on the Boost library, specifically requiring `Boost.Signals2` which,
depending on the build system and configuration, might implicitly pull in other Boost components. There is also an
inherent increase in conceptual complexity; developers working with the system now need to understand the core concepts
of `Boost.Signals2`, including signals, slots, combiners, connection management, and the specific factory pattern used
within our `SignalManager`, which represents an initial learning curve compared to the simpler, albeit less robust,
custom solutions. Additionally, the C++ macro magic employed in `host_macros.h`, while effective at reducing repetitive
boilerplate code for signal emission, can also introduce a layer of opacity, potentially making it harder to debug the
exact flow of control within the host function wrappers without understanding the macro expansions. A critical point of
potential fragility remains in the FFI signature matching: the contract between the C++ host's slot wrapper lambda (
defined within the signal connector factory) and the signature of the exported Rust WASM slot function it intends to
call must be manually synchronized with extreme care. Any mismatch in parameter types, number of parameters, or return
types will not be caught at compile time but will manifest as difficult-to-diagnose runtime traps or undefined behavior.
Lastly, the reliance on string-based names persists during the crucial connection phase. Both the host-side
`connectWasmSlot` method and the WASM-side `connect_signal` wrapper function operate using string literals to identify
signals and WASM functions. Simple typographical errors in these string names will result in silent connection failures,
which can be challenging to track down without careful logging or debugging procedures on both sides of the FFI
boundary.

## Conclusion: A More Elegant Bridge

By replacing our custom eventing system with `Boost.Signals2`, we've significantly elevated the sophistication and
robustness of the interaction between our C++ EnTT host and the Rust WASM plugin. We now have a unified, flexible, and
more maintainable mechanism for the host and plugin to react to each other's actions, intercept operations, and modify
results in a controlled manner.

The `SignalManager` centralizes signal definition and connection logic, while the C++ macros provide a clean way to
instrument our existing host C API functions with signal emissions. On the WASM side, exporting dedicated slot functions
and using a single host call (`host_connect_signal`) to register them simplifies the plugin's responsibility. Features
like combiners (`StopOnTrueCombiner`, `optional_last_value`) and priorities unlock powerful patterns like vetoing
actions or overriding results, all managed by the `Boost.Signals2` framework.

While it introduces a Boost dependency and requires understanding its concepts, the payoff in terms of reduced custom
code complexity, automatic connection management, and standardized event handling is substantial. This architecture
provides a solid foundation for building even more intricate and dynamic interactions across the WASM boundary, proving
that even complex event-driven communication is achievable with the right tools and design patterns.

Our journey continues, but this refactoring marks a significant step towards a more mature and production-ready
C++/Rust/WASM integration.

# vk
[Autogenerated](https://github.com/JolifantoBambla/vk-generator) Common Lisp/CFFI bindings for the Vulkan API.


## Motivation
The goal of this project is to make Vulkan feel lispy without bringing in too much abstraction.
This is achieved by adding a thin layer of CLOS wrappers and functions atop CFFI-bindings to the Vulkan shared library that gets rid
of redundant struct members and output parameters of functions as much as possible.

E.g. where you would have to write the following in C++ (without VulkanHpp) to get all GPUs on a computer:

```cpp
std::vector<VkPhysicalDevice> devices;
uint32_t count = 0;

// first get the number of available devices
VkResult res = vkEnumeratePhysicalDevices(instance, &count, nullptr);

// then get the devices
devices.resize(count);
res = vkEnumeratePhysicalDevices(instance, &count, devices.data());
```

You can just write the following with `vk`:

```cl
(let ((devices (vk:enumerate-physical-devices instance)))
  ;; do something with your devices
  )
```


## Requirements

### Supported CL implementations
`vk` has been mostly tested on SBCL.

Minimal tests (loading the system and running a dummy test) suggest that `vk` works on:

* ABCL
* Clozure CL
* ECL
* SBCL

Check out the results of the latest [test actions](https://github.com/JolifantoBambla/vk/actions).

Unfortunately CLISP fails to install using Roswell (at least via GitHub Actions), so it remains untested.

Allegro is installed in a 32 bit version by Roswell (at least via GitHub Actions) which does not support `:long-long`. 64 bit versions are untested.

CMUCL fails to find `libvulkan.so` in the test action.

### Supported operating systems
`vk` has currently only been tested on linux (Ubuntu 20.04) and Windows (SBCL).

MacOS might also work if [MoltenVK](https://github.com/KhronosGroup/MoltenVK) is set up correctly.

### Supported Vulkan API versions
**The current version of `vk` is based on version `v1.2.178`.**

`vk` targets Vulkan 1.2, so all versions support the core API of version 1.2.x.
The main branch is always generated from the most recent version of the [Vulkan API XML registry](https://github.com/KhronosGroup/Vulkan-Docs)
supported by [vk-generator](https://github.com/JolifantoBambla/vk-generator).
Other versions are available on other branches named after the version (e.g. `v1.2.153`).

The following versions are not supported by the generator:

* Version `1.2.164` is currently not supported by the generator, because of a bug in the Vulkan API XML registry (missing `len` attribute in `VkDescriptorSetAllocateInfo`).
* Every version below `1.2.153` is currently not available.

#### Versioning of `vk`
`vk` uses the following versioning scheme: `major.minor.patch-<Vulkan API verion>`.

Since there are a lot of different Vulkan API versions, when there's a bug fix for the current version older versions might not
receive bug fixes right away.
If you absolutely have to work with a specific version and it seems like a bug fix just won't come for that version, feel free to
open an issue in the GitHub repository.


### CL dependencies
* `alexandria`
* `cffi`

#### Test dependencies
* `rove`

### Other dependencies
* [Vulkan SDK](https://vulkan.lunarg.com/sdk/home)

#### MacOS only
* [MoltenVK](https://github.com/KhronosGroup/MoltenVK)


## Installation
As of the may 2021 dist update `vk` is available on Quicklisp.
Just make sure to have Vulkan installed (see [Vulkan SDK](https://vulkan.lunarg.com/sdk/home)), and then run 

```cl
(ql:quickload :vk)
```

### Samples and Usage
Check out the [documentation](https://jolifantobambla.github.io/vk).

Check out the project [vk-samples](https://github.com/JolifantoBambla/vk-samples) for sample usage of `vk` as well as `vk-utils`.


## Packages and Subsystems
### vk
Contains class wrappers around all structs and unions defined in the Vulkan API as well as all functions getting rid of redundancy in class slots/function parameters and hiding all usages of `cffi` as much as possible.

Wherever a slot or parameter of a function specifies the length of a list or array of another slot or parameter it is omitted from the respective class or function.
The exception to this are `void` pointers to arbitrary data, for which the size can not be determined without any knowledge about the type and number of elements in the array/buffer the pointer points to (e.g. the slot `initial-data` of the class `vk:pipeline-cache-create-info` which wraps `VkPipelineCacheCreateInfo`).
Another exception are cases where a slot specifies the length of an optional array (which can be null) but is not optional itself (e.g. `descriptor-count` in `vk:descriptor-set-layout-binding` and `swapchain-count` in `vk:present-regions-khr` or `vk:present-times-info-google`).

Parameters of wrapped functions that are only used as output parameters in the C API (and the bindings in the `vulkan` package) are omitted from the lambda list of the wrapper function. Instead wrapper functions will return the output as `cl:values`, where the output parameters are in order of their appearence in the lambda list of the wrapped function followed by its return value (e.g. a wrapped/translated `VkResult`) if it returns a value (e.g. `vk:create-instance` returns `(cl:values <the created instance> <a translated VkResult value>)`). 
The exception to this are again `void` pointers (e.g. the parameter `data` in `vk:get-query-pool-results` which wraps `vkGetQueryPoolResults` in the C API).

For some of the mentioned exceptions making assumptions based on the XML API registry are probably possible, so this might change in the future.

#### Shadowed Symbols
`vk` is not meant do be `:use`d by packages directly, since it shadows symbols from `cl` that clash with function and/or slot names from the Vulkan API.

`vk` shadows the following symbols:

* `format`
* `set`
* `stream`
* `type`
* `values`

#### Naming conventions
All names in the Vulkan API have been stripped of their prefixes (i.e. `Vk` for types and `vk` for commands) and lispified (e.g. `VkInstanceCreateInfo` is `vk:instance-create-info`).

Struct and union member names as well as command arguments designating pointers in the C API by being prefixes with either `p` or `pp` have also been stripped of those.

Enum and bitmask value names have been stripped of any redundant prefixes denoting the enum or bitmask they belong to and are represented as keywords in `vk` (e.g. `VK_FORMAT_R4G4_UNORM_PACK8` is just `:r4g4-unorm-pack8`).

##### Exceptions
There are a few name clashes in the C API which break the naming conventions.
Currently, they all are between functions and slot accessors of the same name.
As a general rule, function names take precedence over slot accessors.
Slots and their `:initarg`s still have the same name, but the accessors use the lispified names of their corresponding struct members in the C API.

* The accessors to all slots named `wait-semaphores` are named `p-wait-semaphores` because it clashes with the name of the function `vk:wait-semaphores`.
* Types from external headers (e.g. OS specific types) are not modified.
* Slots and accessors with the name `function` or `pFunction` in the C API are called `function-handle`. 

### vulkan (Nickname: %vk)
Contains the actual `cffi` bindings for the Vulkan API.

The naming conventions are the same as in `vk` except for struct/union member names and command arguments.

### vk-alloc
Contains utilities for allocating resources and translating classes/structs to/from foreign memory.

#### Multithreading
When translating class instances the pointers to all translated struct members which are non-primitive types (e.g of `vk:instance-create-info` if it is bound to an instance of `vk:debug-utils-messenger-create-info-ext`) are stored in the hash table `vk-alloc:*allocated-foreign-objects*` and are freed before the pointer to the translated class instance is freed.
Since hash tables are not thread-safe and there should be no case where type translation needs to span multiple threads, each thread can and should have its own `vk-alloc:*allocated-foreign-objects*` that is independent of those of other threads.

### vk-utils
Contains utils for `vk`.
This package doesn't shadow any symbols.

Aside from utilities for `vk`, this package also contains the function `memcpy`.

*The following is not yet generated, but a roadmap:*

* `vk-utils` will provide autogenerated `with-` style wrappers for allocated resources (e.g. `with-instance`) as well as other utilites that make `vk` more lispy.
* `vk-utils` will also provide `make`-style constructors for all classes defined in `vk`.

## Caveats
### Validation Errors & Slime
Validation errors produced by `VK_LAYER_KHRONOS_validation` are not logged via a debug utils messengers, but directly to `stdout`.
This means that for slime users validation errors will be logged to `*inferior-lisp*` by default.
See [this stackoverflow answer](https://stackoverflow.com/a/40180199) for more information.

### pNext-member of VkBaseOutStructure
Due to how foreign structs with `pNext` members are translated from foreign memory, a translated `vk:base-out-structure` will always have a foreign pointer or `nil` bound to its `next`-slot.
This should not be a problem however, since there is almost no use for instances of `vk:base-out-structure` aside from determining the actual type of the instance by reading its `s-type`-slot.

### VkShaderModule
The `VkShaderModule` struct has a member called `codeSize` which is the number of bytes in its `code` member.
You might be tempted to read your shaders byte by byte, but `VkShaderModule` actually expects an array of `uint32_t`.
As with other `*Count`-members in the Vulkan API, `vk` determines the value to set for `codeSize` automatically.
For this to work properly, the `code` slot of a `vk:shader-module` also needs to be a sequence of 32-bit integers.

`vk-utils:read-shader-source` exists exactly for this purpose: it reads a SPIR-V binary and spits out a vector of 32-bit integers.


## Contributing
Found a bug? Please open a bug report in the GitHub repository.


## Acknowledgements
The whole project is autogenerated by [vk-generator](https://github.com/JolifantoBambla/vk-generator) which has been forked from [cl-vulkan](https://github.com/3b/cl-vulkan) and is partially ported from [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp).

The documentation is autogenerated using [staple](https://github.com/Shinmera/staple).

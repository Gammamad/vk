# vk
[Autogenerated](https://github.com/JolifantoBambla/vk-generator) Common Lisp/CFFI bindings for the Vulkan API.

**This project is still under development - wait for the first release.**

## Requirements

### Supported CL implementations
`vk` has currently only been tested on `SBCL 2.0.0`, though other implementations might work as well.

### Versioning
**There are no releases yet!**

Releases will follow realeases of [vk-generator](https://github.com/JolifantoBambla/vk-generator). 
The `main` branch of this repository will always contain the most recent version of `vk` for the most recent version of the Vulkan SDK that can be generated by [vk-generator](https://github.com/JolifantoBambla/vk-generator). There will be separate branches for older versions of the Vulkan SDK where the systems (and `asd` files) will be named `<system>-<Vulkan-SDK-version-tag>` instead of just `<system>` (e.g. `vk-v1.2.153.asd` instead of `vk.asd`).

### Unsupported Vulkan API versions
* Version `1.2.164` is currently not supported, because of a bug in the Vulkan API XML registry (missing `len` attribute in `VkDescriptorSetAllocateInfo`).
* Every version below `1.2.153` is currently not supported.


## Installation
*This project is not on quicklisp yet - a quicklisp release will be requested with version 1.0.0* 

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

##### Exceptions
There are a few name clashes in the C API which break the naming conventions.
Currently, they all are between functions and slot accessors of the same name.
As a general rule, function names take precedence over slot accessors.
Slots and their `:initarg`s still have the same name, but the accessors use the lispified names of their corresponding struct members in the C API.

- The accessors to all slots named `wait-semaphores` are named `p-wait-semaphores` because it clashes with the name of the function `vk:wait-semaphores`.

### vulkan (Nickname: %vk)
Contains the actual `cffi` bindings for the Vulkan API. 

### vk-alloc
Contains utilities for allocating resources and translating classes/structs to/from foreign memory.

#### Multithreading
*TODO: describe vk-alloc stuff, special care when multithreading (e.g. bordeaux-threads:default-special-bindings)*

### vk-utils
*Not yet generated*

`vk-utils` will provide autogenerated `with-` style wrappers for allocated resources (e.g. `with-instance`) as well as other utilites that make `vk` more lispy.

## Caveats
### Validation Errors & Slime
Validation errors produced by `VK_LAYER_KHRONOS_validation` are not logged via a debug utils messengers, but directly to `stdout`.
This means that for slime users validation errors will be logged to `*inferior-lisp*` by default.
See [this stackoverflow answer](https://stackoverflow.com/a/40180199) for more information.

## Acknowledgements
The whole project is autogenerated by [vk-generator](https://github.com/JolifantoBambla/vk-generator) which has been forked from [cl-vulkan](https://github.com/3b/cl-vulkan) and is partially ported from [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp).

The documentation is autogenerated using [staple](https://github.com/Shinmera/staple).

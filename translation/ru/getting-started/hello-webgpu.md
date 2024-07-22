Привет, WebGPU
============

```{lit-setup}
:tangle-root: 001 - Hello WebGPU
:parent: 000 - Project setup
:fetch-files: ../../data/webgpu-distribution-v0.2.0-beta2.zip
```

*Итоговый код:* [`step001`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step001)

WebGPU это *Render Hardware Interface* (RHI, *Аппаратный интерфейс рендеринга*), что означает, что это библиотека, которая дает **универсальный интерфейс** для целого множества графической аппаратуры и операционных систем.

Для вашего C++ кода WebGPU не более чем **одиночный заголовочный файл**, который описывает все доступные процедуры и структуры данных: [`webgpu.h`](https://github.com/webgpu-native/webgpu-headers/blob/main/webgpu.h).

Однако, во время сборки программы, компилятор должен знать в самом конце (на финальном этапе *линковки*) **где найти** реализацию этих функций. В отличие от нативных API, имплементация WebGPU не предоставляется драйвером, поэтому мы должны явно предоставить ее.

```{figure} /images/rhi-vs-opengl.png
:align: center

*Render Hardware Interface* (RHI), как WebGPU, **не предоставлен напрямую драйверами**: мы должны слинковать библиотеку, которая реализует API поверх низкоуровневых, поддерживаемых системой.

```

Установка WebGPU
-----------------

Существует две основные реализации нативного заголовка WebGPU:

 - [wgpu-native](https://github.com/gfx-rs/wgpu-native), дающий нативный интерфейс [`wgpu`](https://github.com/gfx-rs/wgpu) Библиотеки на Rust, разработанной для Firefox.
 - Google's [Dawn](https://dawn.googlesource.com/dawn), разработанной для Chrome.

```{figure} /images/different-backend.png
:align: center

Существуют (как минимум) две реализации WebGPU, разработанных для двух главных web движков.
```

Эти две реализации до сих пор имеют **некоторые расхождения**, но они исчезнуть когда спецификация WebGPU станет стабильной. Я пытаюсь писать этот гайд так, чтобы все **работало для обоих из них**.

Чтобы упростить интеграцию любого из них, я сделал репозиторий [WebGPU-distribution](https://github.com/eliemichel/WebGPU-distribution), который позволяет выбрать одну из следующих опций:

`````{admonition} Слишком много опций? (Кликни)
:class: foldable quickstart

*Ты предпочитаешь быструю сборку детализированным сообщениям об ошибках?*

````{admonition} Ага, **быстрая сборка** и **ненадобность** интернет соединения для первой сборки
:class: foldable yes

[**Опция A**](#option-a-the-lightness-of-wgpu-native) (wgpu-native) для тебя!

````

````{admonition} Нет, я предпочитаю **подробные сообщения об ошибках**.
:class: foldable no

Тогда [**Опция B**](#option-b-the-comfort-of-dawn) (Dawn)!

````

```{admonition} Без понятия, не хочу выбирать...
:class: foldable warning

Попробуй [**Опцию C**](#option-c-the-flexibility-of-both), которая позволит переключаться с одного бэкенда на другой в любое время!
```

`````

### Опция A: Легковесный wgpu-native

Так как `wgpu-native` написан на rust, мы не сможем собрать все с нуля, поэтому существуют прекомпилированные (pre-compiled) библиотеки:

```{important}
**WIP:** Используй ссылку "для любой платформы", а не специфичные для какой-либо платформы. Я так и не автоматизировал их генерацию, поэтому они обычно отстают от главного.
```

 - [wgpu-native для любых платформ](https://github.com/eliemichel/WebGPU-distribution/archive/refs/tags/wgpu-v0.19.4.1.zip) (немного более тяжелый поскольку содержит все возможные платформы)
 - [wgpu-native для Linux](#)
 - [wgpu-native для Windows](#)
 - [wgpu-native для MacOS](#)

```{note}
Прекомпилированные бинарники даны проектом `wgpu-native`, поэтому смело доверяйте им. Единственное что я добавил это `CMakeLists.txt`, который позволяет их легко подключить.
```

**Pros**
 - Это самый легковесный вариант для сборки.


**Cons**
 - Не нужно собирать из исходников.
 - `wgpu-native` не дает debug инфу как Dawn.

### Option B: The comfort of Dawn

Dawn gives much better error messages, and since it is written in C++ we can build it from source and thus inspect more deeply the stack trace in case of crash:

 - [Dawn for any platform](https://github.com/eliemichel/WebGPU-distribution/archive/refs/tags/dawn-6512.zip)

```{note}
The Dawn-based distribution I provide here fetches the source code of Dawn from its original repository, but in an as shallow as possible way, and pre-sets some options to avoid building parts that we do not use.
```

**Pros**

 - Dawn is much more comfortable to develop with, because it gives more detailed error messages.
 - It is in general ahead of `wgpu-native` regarding the progress of implementation (but `wgpu-native` will catch up eventually).

**Cons**
 - Although I reduced the need for extra dependencies, you still need to [install Python](https://www.python.org/) and [git](https://git-scm.com/download).
 - The distribution fetches Dawn's source code and its dependencies so the first time you build you need an **Internet connection**.
 - The initial build takes significantly longer, and occupies more disk space overall.

````{note}
On Linux check out [Dawn's build documentation](https://dawn.googlesource.com/dawn/+/HEAD/docs/building.md) for the list of packages to install. As of April 7, 2024, the list is the following (for Ubuntu):

```bash
sudo apt-get install libxrandr-dev libxinerama-dev libxcursor-dev mesa-common-dev libx11-xcb-dev pkg-config nodejs npm
```
````

### Option C: The flexibility of both

In this option, we only include a couple of CMake files in our project, which then dynamically fetch either `wgpu-native` or Dawn depending on a configuration option:

```
cmake -B build -DWEBGPU_BACKEND=WGPU
# or
cmake -B build -DWEBGPU_BACKEND=DAWN
```

```{note}
The **accompanying code** uses this Option C.
```

This is given by the `main` branch of my distribution repository:

 - [WebGPU any distribution](https://github.com/eliemichel/WebGPU-distribution/archive/refs/tags/main-v0.2.0-beta1.zip)

**Pros**
 - You can have two `build` at the same time, one that uses Dawn and one that uses `wgpu-native`

**Cons**
 - This is a "meta-distribution" that fetches the one you want at configuration time (i.e., when calling `cmake` the first time) so you need an **Internet connection** and **git** at that time.

And of course depending on your choice the pros and cons of *Option A* and *Option B* apply.

### Integration

Whichever distribution you choose, the integration is the same:

 1. Download the zip of your choice.
 2. Unzip it at the root of the project, there should be a `webgpu/` directory containing a `CMakeLists.txt` file and some other (.dll or .so).
 3. Add `add_subdirectory(webgpu)` in your `CMakeLists.txt`.

```{lit} CMake, Dependency subdirectories ru (insert in {{Define app target}} before "add_executable")
# Include webgpu directory, to define the 'webgpu' target
add_subdirectory(webgpu)
```

```{important}
The name 'webgpu' here designate the directory where GLFW is located, so there should be a file `webgpu/CMakeLists.txt`. Otherwise it means that `webgpu.zip` was not decompressed in the correct directory; you may either move it or adapt the `add_subdirectory` directive.
```

 4. Add the `webgpu` target as a dependency of our app, using the `target_link_libraries` command (after `add_executable(App main.cpp)`).

```{lit} CMake, Link libraries ru (insert in {{Define app target}} after "add_executable")
# Add the 'webgpu' target as a dependency of our App
target_link_libraries(App PRIVATE webgpu)
```

```{tip}
This time, the name 'webgpu' is one of the *target* defined in `webgpu/CMakeLists.txt` by calling `add_library(webgpu ...)`, it is not related to a directory name.
```

One additional step when using pre-compiled binaries: call the function `target_copy_webgpu_binaries(App)` at the end of `CMakeLists.txt`, this makes sure that the .dll/.so file that your binary depends on at runtime is copied next to it. Whenever you distribute your application, make sure to also distribute this dynamic library file as well.

```{lit} CMake, Link libraries ru (append)
# The application's binary must find wgpu.dll or libwgpu.so at runtime,
# so we automatically copy it (it's called WGPU_RUNTIME_LIB in general)
# next to the binary.
target_copy_webgpu_binaries(App)
```

```{note}
In the case of Dawn, there is no precompiled binaries to copy but I define the `target_copy_webgpu_binaries` function anyway (it does nothing) so that you can really use the same CMakeLists with both distributions.
```

Testing the installation
------------------------

To test the implementation, we simply create the WebGPU **instance**, i.e., the equivalent of the `navigator.gpu` we could get in JavaScript. We then check it and destroy it.

```{important}
Make sure to include `<webgpu/webgpu.h>` before using any WebGPU function or type!
```

```{lit} C++, Includes ru
// Includes
#include <webgpu/webgpu.h>
#include <iostream>
```

```{lit} C++, file: main.cpp ru
{{Includes}}

int main (int, char**) {
    {{Create WebGPU instance}}

    {{Check WebGPU instance}}

    {{Destroy WebGPU instance}}

    return 0;
}
```

### Descriptors and Creation

The instance is created using the `wgpuCreateInstance` function. Like all WebGPU functions meant to **create** an entity, it takes as argument a **descriptor**, which we can use to specify options regarding how to set up this object.

```{lit} C++, Create WebGPU instance ru
// We create a descriptor
WGPUInstanceDescriptor desc = {};
desc.nextInChain = nullptr;

// We create the instance using this descriptor
WGPUInstance instance = wgpuCreateInstance(&desc);
```

```{note}
The descriptor is a kind of way to **pack many function arguments** together, because some descriptors really have a lot of fields. It can also be used to write utility functions that take care of populating the arguments, to ease the program's architecture.
```

We meet another WebGPU **idiom** in the `WGPUInstanceDescriptor` structure: the first field of a descriptor is always a pointer called `nextInChain`. This is a generic way for the API to enable **custom extensions** to be added in the future, or to return multiple entries of data. In a lot of cases, we set it to `nullptr`.


### Check

A WebGPU entity created with a `wgpuCreateSomething` function is technically **just a pointer**. It is a blind handle that identifies the actual object, which lives on the backend side and to which we never need direct access.

To check that an object is valid, we can just compare it with `nullptr`, or use the boolean operator:

```{lit} C++, Check WebGPU instance ru
// We can check whether there is actually an instance created
if (!instance) {
    std::cerr << "Could not initialize WebGPU!" << std::endl;
    return 1;
}

// Display the object (WGPUInstance is a simple pointer, it may be
// copied around without worrying about its size).
std::cout << "WGPU instance: " << instance << std::endl;
```

This should display something like `WGPU instance: 000001C0D2637720` at startup.

### Destruction and lifetime management

All the entities that can be **created** using WebGPU must eventually be **released**. A procedure that creates an object always looks like `wgpuCreateSomething`, and its equivalent for releasing it is `wgpuSomethingRelease`.

Note that each object internally holds a reference counter, and releasing it only frees related memory if no other part of your code still references it (i.e., the counter falls to 0):

```C++
WGPUSomething sth = wgpuCreateSomething(/* descriptor */);

// This means "increase the ref counter of the object sth by 1"
wgpuSomethingReference(sth);
// Now the reference is 2 (it is set to 1 at creation)

// This means "decrease the ref counter of the object sth by 1
// and if it gets down to 0 then destroy the object"
wgpuSomethingRelease(sth);
// Now the reference is back to 1, the object can still be used

// Release again
wgpuSomethingRelease(sth);
// Now the reference is down to 0, the object is destroyed and
// should no longer be used!
```

In particular, we need to release the global WebGPU instance:

```{lit} C++, Destroy WebGPU instance ru
// We clean up the WebGPU instance
wgpuInstanceRelease(instance);
```

### Implementation-specific behavior

In order to handle the slight differences between implementations, the distributions I provide also define the following preprocessor variables:

```C++
// If using Dawn
#define WEBGPU_BACKEND_DAWN

// If using wgpu-native
#define WEBGPU_BACKEND_WGPU

// If using emscripten
#define WEBGPU_BACKEND_EMSCRIPTEN
```

### Building for the Web

The WebGPU distribution listed above are readily compatible with [Emscripten](https://emscripten.org/docs/getting_started/downloads.html) and if you have trouble with building your application for the web, you can consult [the dedicated appendix](../appendices/building-for-the-web.md).

As we will add a few options specific to the web build from time to time, we can add a section at the end of our `CMakeLists.txt`:

```{lit} CMake, file: CMakeLists.txt ru (append)
# Options that are specific to Emscripten
if (EMSCRIPTEN)
    {{Emscripten-specific options}}
endif()
```

For now we only change the output extension so that it is an HTML web page (rather than a WebAssembly module or JavaScript library):

```{lit} CMake, Emscripten-specific options ru
# Generate a full web page rather than a simple WebAssembly module
set_target_properties(App PROPERTIES SUFFIX ".html")
```

For some reason the instance descriptor **must be null** (which means "use default") when using Emscripten, so we can already use our `WEBGPU_BACKEND_EMSCRIPTEN` macro:

```{lit} C++, Create WebGPU instance ru (replace)
// We create a descriptor
WGPUInstanceDescriptor desc = {};
desc.nextInChain = nullptr;

// We create the instance using this descriptor
#ifdef WEBGPU_BACKEND_EMSCRIPTEN
WGPUInstance instance = wgpuCreateInstance(nullptr);
#else //  WEBGPU_BACKEND_EMSCRIPTEN
WGPUInstance instance = wgpuCreateInstance(&desc);
#endif //  WEBGPU_BACKEND_EMSCRIPTEN
```

Conclusion
----------

In this chapter we set up WebGPU and learnt that there are **multiple backends** available. We also saw the basic idioms of **object creation and destruction** that will be used all the time in WebGPU API!

*Resulting code:* [`step001`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step001)

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

**Плюсы**
 - Это самый легковесный вариант для сборки.


**Минусы**
 - Не нужно собирать из исходников.
 - `wgpu-native` не дает debug инфу как Dawn.

### Опция B: Удобство Dawn

Dawn дает гораздо лучшие сообщения об ошибках, и так как он написан на C++, мы можем его собрать из исходников и потому более тщательно проверять stack trace в случае краша:

 - [Dawn для любой платформы](https://github.com/eliemichel/WebGPU-distribution/archive/refs/tags/dawn-6512.zip)

```{note}
Пакет основанный на Dawn, который я даю, берет исходники Dawn из оригинального репозитория, но как можно более поверхностно, и устанавливает некоторые настройки так, чтобы не собирать части, которые мы не будем использовать.
```

**Плюсы**

 - С Dawn гораздо более удобно разрабатывать, потому что он дает более подробные сообщения об ошибки.
 - Он опережает `wgpu-native` в разработке (но `wgpu-native` скоро догонит).

**Минусы**
 - Хотя я избавил тебя от установки некоторых дополнительных зависимостей, все еще нужно [установить Python](https://www.python.org/) и [git](https://git-scm.com/download).
 - Пакет скачивает исходники Dawn и его зависимости, поэтому нужно **интернет соединение**.
 - Первая сборка занимает значительное количество времени, и занимает больше места на диске.

````{note}
На Linux чекни [документацию по сборке Dawn](https://dawn.googlesource.com/dawn/+/HEAD/docs/building.md) для списка зависимостей, которые нужно установить. На момент 7 Апреля, 2024, список следующий (для Ubuntu):

```bash
sudo apt-get install libxrandr-dev libxinerama-dev libxcursor-dev mesa-common-dev libx11-xcb-dev pkg-config nodejs npm
```
````

### Опция C: Гибкость обоих

Здесь мы просто подключаем несколько файлов CMake в наш проект, которые динамически будут подруба либо wgpu-native`, либо Dawn, в зависимости от настройки конфигурации:

```
cmake -B build -DWEBGPU_BACKEND=WGPU
# или
cmake -B build -DWEBGPU_BACKEND=DAWN
```

```{note}
 **Сопровождающий (типо дальнейший в гайде) код** использует эту Опцию C.
```

Он использует в ветке `main` моего репозитория:

 - [WebGPU любой бэк](https://github.com/eliemichel/WebGPU-distribution/archive/refs/tags/main-v0.2.0-beta1.zip)

**Плюсы**
 - Ты можешь иметь два `билда` в одно и то же время, один использующий Dawn, другой - `wgpu-native`

**Минусы**
 - Это "мета-пакет", который выбирает один из билдов во время конфигурации (тобиш, когда делаешь `cmake` в первый раз), поэтому нужно и **интернет соединение**, и **git**.

Ну и понятно, в зависимости от выбора, получаешь плюсы и минусы *Опции A* и *Опции B*.

### Подрубаем

В независимости от опции, которую ты выбрал, подрубка одинаковая:

 1. Скачай zip архив, выбранной опции.
 2. Разархивируй в корень проекта, там должна быть папка `webgpu/`, содержащая файл `CMakeLists.txt` и некоторые другие (.dll или .so).
 3. Добавь `add_subdirectory(webgpu)` в твой `CMakeLists.txt`.

```{lit} CMake, Dependency subdirectories ru (insert in {{Define app target}} before "add_executable")
# Добавь папку webgpu, чтобы определить 'webgpu' target
add_subdirectory(webgpu)
```

```{important}
Имя 'webgpu' обозначает папку, где располагается GLFW, поэтому должен быть файл `webgpu/CMakeLists.txt`. Иначе, означает, что `webgpu.zip` был разархивирован в не ту директорию; тогда нужно либо переместить папку куда надо, либо изменить команду `add_subdirectory`.
```

 4. Добавь таргет `webgpu`  как зависимость в твое приложение, используя команду `target_link_libraries` (после `add_executable(App main.cpp)`).


```{lit} CMake, Link libraries ru (insert in {{Define app target}} after "add_executable")
# Добавить таргет 'webgpu' как зависимость для App
target_link_libraries(App PRIVATE webgpu)
```

```{tip}
Теперь, имя 'webgpu' это один из *таргетов*, определенных в `webgpu/CMakeLists.txt` вызовом `add_library(webgpu ...)`, и оно никак не связано с названием папки. 
```

Еще один дополнительный шаг, если используешь прекомпилированные библиотеки: вызови функцию `target_copy_webgpu_binaries(App)` в конце `CMakeLists.txt`, благодаря этому файл .dll/.so, от которого зависит бинарник, будет скопирован прямо к нему. Всякий раз когда ты будешь распространять свое приложение, убедись, что файл динамической библиотеки лежит рядом с бинарником.


```{lit} CMake, Link libraries ru (append)
# Бинарник приложения должен найти wgpu.dll или libwgpu.so в рантайме,
# поэтому мы автоматически копируем его (вообще штука называется WGPU_RUNTIME_LIB)
# к бинарнику.
target_copy_webgpu_binaries(App)
```

```{note}
В случае Dawn, нет прекомпилированного бинарника для копирования, но я все равно определяю функцию `target_copy_webgpu_binaries` (она ничего не будет делать), для того, чтобы один и тот же CMakeLists работал с обоими пакетами (`wgpu-native` и Dawn).
```

Проверяем установку
------------------------

Чтобы проверить, что все ок, создадим WebGPU **instance**, то есть, по сути это то же самое, что `navigator.gpu` из JavaScript. После мы проверим его и уничтожим.

```{important}
Убедись, что за инклудил `<webgpu/webgpu.h>` перед тем как использовать функции и объекты WebGPU!
```

```{lit} C++, Includes ru
// Инклудим
#include <webgpu/webgpu.h>
#include <iostream>
```

```{lit} C++, file: main.cpp ru
{{Includes ru}}

int main (int, char**) {
    {{Create WebGPU instance ru}}

    {{Check WebGPU instance ru}}

    {{Destroy WebGPU instance ru}}

    return 0;
}
```

### Дескрипторы(Descriptors) и Создание


Инстанс (instance, экземляр) создается функцией `wgpuCreateInstance`. Как и все функции WebGPU, предназначенные для **создания** какой-либо сущности, в качестве аргумента она берет **дескриптор**, который мы используем, чтобы задать определенные свойства для создания объекта.

```{lit} C++, Create WebGPU instance ru
// Создаем дескриптор
WGPUInstanceDescriptor desc = {};
desc.nextInChain = nullptr;

// Создаем инстанс, используя дескриптор
WGPUInstance instance = wgpuCreateInstance(&desc);
```

```{note}
Дескриптор это такой способ **упаковать большое количество аргументов функции** вместе, ибо некоторые дескрипторы имеют огромное количество полей. Он также может быть использован, чтобы писать вспомогательные функции для заполнения (populating) этих аргументов, упрощая таким образом архитектуру программы.
```

Также тут мы встречаем другую **идиому** WebGPU  в структуре `WGPUInstanceDescriptor`: первое поле дескриптора это всегда указатель с названием `nextInChain`. Это общий способ для API добавлять **пользовательские дополнения**, или для возврата нескольких записей данных (entries of data). В большинстве случаев мы просто делаем его `nullptr`.


### Проверка

Сущность WebGPU, созданная функцией `wgpuCreateSomething`, является **обычным указателем**. Это 'слепой' хэндл (handle), который определяет объект, живущий где-то на бэкенде и к которому нам никогда не понадобится прямого доступа.

Чтобы проверить является ли объект валидным, мы можем просто сравнит его с `nullptr`, или использовать булевый оператор:

```{lit} C++, Check WebGPU instance ru
// Мы можем проверить создался ли инстанс
if (!instance) {
    std::cerr << "Could not initialize WebGPU!" << std::endl;
    return 1;
}

// Отобразить объект (WGPUInstance это обычный указатель, он может быть
// скопирован, не заботясь о его размере).
std::cout << "WGPU instance: " << instance << std::endl;
```

Все это должно вывести при запуске `WGPU instance: 000001C0D2637720`.

### Уничтожение и управление временем жизни

Все сущности, которые могут быть **созданы**, используя WebGPU, должны быть и **освобождены**. Процедура, создающая объект, всегда выглядит типа `wgpuCreateSomething`, и ее эквивалент для 'освобождения' - `wgpuSomethingRelease`.

Заметьте, что каждый объект внутри хранит счетчик ссылок (reference counter), и освобождение памяти происходит только если никакая другая часть когда не ссылает на нее (иначе говоря, только если счетчик равен 0):

```C++
WGPUSomething sth = wgpuCreateSomething(/* дескриптор */);

// Это означает "увеличить ref counter объекта на 1"
wgpuSomethingReference(sth);
// Now the reference is 2 (it is set to 1 at creation)
// Теперь ссылок 2 (так как 1 при создании)

// Это означает "уменьшить ref counter объекта на 1
// и если он станет 0 тогда уничтожь объект"
wgpuSomethingRelease(sth);
// Теперь ссылок 1, объект все еще может быть использован

// Освобождаем опять
wgpuSomethingRelease(sth);
// Теперь ссылок 0, объект уничтожен
// и не может больше быть использованным!
```

В частности, мы должны освободить и глобальный инстанс WebGPU:

```{lit} C++, Destroy WebGPU instance ru
// Уничтожаем инстанс WebGPU
wgpuInstanceRelease(instance);
```

### Поведение, зависимое от реализации

Для того, чтобы обрабатывать некоторые различия между реализациями, репа, которую я предоставил, также определяет следующие дефайны:

```C++
// Если используешь Dawn
#define WEBGPU_BACKEND_DAWN

// Если используешь wgpu-native
#define WEBGPU_BACKEND_WGPU

// Если используешь emscripten
#define WEBGPU_BACKEND_EMSCRIPTEN
```

### Сборка под Web

Репа WebGPU, описанная выша, также совместима с [Emscripten](https://emscripten.org/docs/getting_started/downloads.html) и если у вас проблемы со сборкой проекта для веба, обратись к [the dedicated appendix](../appendices/building-for-the-web.md).

Так как мы будем иногда добавлять специфичные для веба параметры, мы можем добавить для этого спец секцию в конце `CMakeLists.txt`:

```{lit} CMake, file: CMakeLists.txt ru (append)
# Параметры специфичные для Emscripten
if (EMSCRIPTEN)
    {{Emscripten-specific options}}
endif()
```

Теперь нам нужно поменять формат выходного файла, поскольку теперь это HTML веб страница (а не модуль WebAssembly или библиотека JavaScript):

```{lit} CMake, Emscripten-specific options ru
# Сгенерировать полноценную html веб страницу, а не модуль вебасм
set_target_properties(App PROPERTIES SUFFIX ".html")
```

По какой-то причине дескриптор инстанса **должен быть null** (что означает "использовать по умолчанию") при использовании Emscripten, поэтому воспользуемся нашим дефайном `WEBGPU_BACKEND_EMSCRIPTEN`:


```{lit} C++, Create WebGPU instance ru (replace)
// Создаем дескриптор
WGPUInstanceDescriptor desc = {};
desc.nextInChain = nullptr;

// Создаем инстанс с помощью дескриптора
#ifdef WEBGPU_BACKEND_EMSCRIPTEN
WGPUInstance instance = wgpuCreateInstance(nullptr);
#else //  WEBGPU_BACKEND_EMSCRIPTEN
WGPUInstance instance = wgpuCreateInstance(&desc);
#endif //  WEBGPU_BACKEND_EMSCRIPTEN
```

Заключение
----------

В этой главе мы настроили WebGPU и узнали, что существует **несколько бэкендов**. Также мы увидели основные идиомы **создания и уничтожения объекта**, которые мы будем использовать все время в WebGPU API!


*Итоговый код:* [`step001`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step001)

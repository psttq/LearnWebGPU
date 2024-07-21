Настройка проекта
=============

```{lit-setup}
:tangle-root: 000 - Project setup
```

*Итоговый код:* [`step000`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step000)

В нашем примере мы используем [CMake](https://cmake.org/), чтобы организовать компиляцию кода. Это стандарт для кроссплатформенных проектов, и также мы следуем идиомам [modern cmake](https://cliutils.gitlab.io/modern-cmake/).

Требования
------------

Все что нам нужно это CMake и компилятор C++, инструкции для каждой ОС приведены ниже.


```{hint}
После установки ты можешь написать `which cmake` (linux/macOS) или `where cmake` (Windows), чтобы убедиться, что командная строка находит полный путь до команды `cmake`. Если нет убедись, что переменная окружения `PATH` содержит папку, куда был установлен CMake.

```

### Linux

Если ты используешься дистрибутив Ubuntu/Debian, установи следующие пакеты:

```bash
sudo apt install cmake build-essential
```

Другие дистрибутивы имеют похожие пакеты, убедитесь что команды `cmake`, `make` and `g++` работают.
### Windows

Скачайте и установите CMake со [страницы загрузки](https://cmake.org/download/). Ты можешь использовать [Visual Studio](https://visualstudio.microsoft.com/downloads/) или [MinGW](https://www.mingw-w64.org/) для компиляции.

### MacOS

Можешь установить CMake, выполнив команду `brew install cmake`, и [XCode](https://developer.apple.com/xcode/) для сборки проекта.

Минимальный проект
---------------

Самый минимальный проект состоит из `main.cpp` и `CMakeLists.txt`.

Начнем с классического hello world в `main.cpp`:

```{lit} C++, file: main.cpp ru
#include <iostream>

int main (int, char**) {
	std::cout << "Hello, world!" << std::endl;
	return 0;
}
```

В `CMakeLists.txt` мы указываем что хотим создать *target* типа *executable*, называемый "App" (это также будет названием исполняемого файла), и чей исходный код записан в `main.cpp`:

```{lit} CMake, Define app target ru
add_executable(App main.cpp)
```

CMake also expects at the beginning of `CMakeLists.txt` to know the version of CMake the file is written for (minimum supported...your version) and some information about the project:

CMake также ожидает в начале файла `CMakeLists.txt` версию CMake (минимально поддерживаемая...твоя версия) и некоторую информацию о проекте:


```{lit} CMake, file: CMakeLists.txt ru
cmake_minimum_required(VERSION 3.0...3.25)
project(
	LearnWebGPU # название проекта, который также будет названием проекта visual studio, если вы его используете
	VERSION 0.1.0 # версия вашего проекта
	LANGUAGES CXX C # используемые языки программирования в проекте
)
{{Define app target ru}}
{{Recommended extras ru}}
```

Сборка
--------

Теперь мы готовы, чтобы собрать наш минимальный проект. Открой терминал и перейди в папку, содержащую файлы `CMakeLists.txt` и `main.cpp`:


```bash
cd your/project/directory
```

```{hint}
В проводнике Windows, открыв папку с проектом, нажми Ctrl+L, а затем напиши `cmd` и нажми return. Так ты откроешь текущую папку в терминале.
```

Теперь попросим CMake создать файлы для сборки нашего проекта. Чтобы CMake создал их отдельно от исходных файлов в папке *build/*, используй флаг `-B build`. Это очень рекомендуется сделать, чтобы отличать файлы сгенерированные CMake и файлы, созданные тобой (a.k.a. исходные файлы):

```bash
cmake . -B build
```

В итоге ты должен был получить файлы для сборки для `make`, Visual Studio или XCode в зависимости от твоей системы (ты можешь использовать флаг `-G` чтобы задать систему сборки, используй `cmake -h` для большей информации). И, наконец, чтобы собрать программу и сгенерировать исполняемый файл `App` (или `App.exe`), ты можешь либо открыть сгенерированный проект Visual Studio или XCode, либо набрать в терминале:


```bash
cmake --build build
```

Затем запусти, полученную программу:

```bash
build/App  # linux/macOS
build\Debug\App.exe  # Windows
```

Рекомендуемые дополнения
------------------

Настроим некоторые свойства *target* `App`, написав где-нибудь после `add_executable` команду `set_target_properties`.


```{lit} CMake, Recommended extras ru
set_target_properties(App PROPERTIES
	CXX_STANDARD 17
	CXX_STANDARD_REQUIRED ON
	CXX_EXTENSIONS OFF
	COMPILE_WARNING_AS_ERROR ON
)
```

Свойство `CXX_STANDARD` имеет значение 17, что означает что мы используем C++17 (это позволит нам использовать некоторые синтаксические приколы в будущем). Свойство `CXX_STANDARD_REQUIRED` гарантирует, что сборка зафейлится если C++17 не поддерживается.

Свойство `CXX_EXTENSIONS` назначено в `OFF`, чтобы отключить compiler specific extensions (например, GCC использует `-std=c++17`, а не `-std=gnu++17` в списке флагов компиляции).

Параметр `COMPILE_WARNING_AS_ERROR` включен как хорошая практика, чтобы быть уверенным, что ни один ворнинг не был проигнорен. Ворнинги на самом деле очень важны, особенно когда учишь новый язык или библиотеку. Для максимально возможного количества ворнингов, добавим следующие параметры компиляции:


```{lit} CMake, Recommended extras ru (append)
if (MSVC)
	target_compile_options(App PRIVATE /W4)
else()
	target_compile_options(App PRIVATE -Wall -Wextra -pedantic)
endif()
```

```{note}
В прилагаемом коде я скрыл эти данные в функции `target_treat_all_warnings_as_errors()`, определенной в  `utils.cmake` и включенной в начале `CMakeLists.txt`.
```

В MacOS, CMake генерирует файлы проекта XCode. Но, по умолчанию, ни одной *schemes* не будет создано, и XCode сам генерирует *scheme* для каждого *target* CMake  -- обычно нам нужна *scheme* только для нашего главного *target*. Поэтому, включим параметр `XCODE_GENERATE_SCHEME`.
Также, мы сразу включим frame capture для дебага GPU.

```{lit} CMake, Recommended extras ru (append)
if (XCODE)
	set_target_properties(App PROPERTIES
		XCODE_GENERATE_SCHEME ON
		XCODE_SCHEME_ENABLE_GPU_FRAME_CAPTURE_MODE "Metal"
	)
endif()
```

Заключение
----------

Теперь у нас есть **базовая конфигурация проекта**, которую мы и будем использовать в следующих главах. В них, мы увидим как [подключить WebGPU](hello-webgpu.md) в наш проект, как [инициализировать его](adapter-and-device/index.md), и как [открыть окно](opening-a-window.md), в которое мы и будем рисовать.


*Итоговый код:* [`step000`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step000)

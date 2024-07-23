# Адаптер

```{lit-setup}
:tangle-root: 005 - The Adapter
:parent: 001 - Hello WebGPU
```

_Итоговый код_: [`step005`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step005)

Перед тем как начать работать с **девайсом**, мы должны выбрать **адаптер**. Одна система может иметь **несколько адаптеров**, если у нее есть доступ к нескольким GPU. Также может быть адаптер, который представляет эмулированное/виртуальное устройство.

```{note}
Часто бывает, что топовые ноутбуки имеют **два GPU**, с **высокопроизводительным** и **экономным** энергопотреблением (которое интегрировано в чип CPU).

```

Каждый адаптер предлагает список необязательных **фич (возможностей)** и **поддерживаемые ограничения**, с которыми он может справиться. Все это используется, чтобы определить возможности системы перед тем как **запрашивать девайс**.

> 🤔 Но зачем нам и **адаптер**, и **девайс**?

Идея заключается в том, чтобы "работает на машине" стало "работает на чужой машине тоже". **Адаптер** нужен, чтобы получить **доступ к возможностям** пользовательского устройства, которые используются для выбора поведения твоего приложения среди самых разных путей кода. Когда путь кода выбран, **устройство** создается с **выбранными возможностями**.

Только выбранные возможности для устройства разрешены в дальнейшем в приложении. Таким образом, **невозможно использовать возможности специфичные только для твоего устройства**.

```{figure} /images/device-creation.png
:align: center
Дополнительно, используя разделение адаптер/девайс, мы можем настроить различные пресеты для ограничений и выбирать нужный в зависимости от адаптера. В нашем случае, мы будем использовать один пресет и абортить если он не поддерживается.
```

## Получение адаптера

Адаптер это не то, что мы _создаем_, но то что мы _запрашиваем_, используя функцию `wgpuInstanceRequestAdapter`.

````{note}
Название процедур, предоставляемых `webgpu.h`, всегда следуют одной и той же конструкции:

```C
wgpuSomethingSomeAction(something, ...)
             ^^^^^^^^^^ // Что делать...
    ^^^^^^^^^ // ...над каким типом объекта
^^^^ // (префикс, чтобы избежать коллизии имён)
```

Первый аргумент функции это всегда "хэндл" (слепой указатель) представляющий объект типа "Something".

````

Поэтому, как подсказывает имя, первый аргумент это `WGPUInstance`, который мы создали в предыдущей главе. Что насчет остальных?

```C++
// Сигнатура функции wgpuInstanceRequestAdapter, описанная в webgpu.h
void wgpuInstanceRequestAdapter(
	WGPUInstance instance,
	WGPU_NULLABLE WGPURequestAdapterOptions const * options,
	WGPURequestAdapterCallback callback,
	void * userdata
);
```

```{note}
Всегда очень полезно посмотреть как определена функция в `webgpu.h`!
```

Второй аргумент это множество **параметров**, которые похожи на **дескриптор**, который мы видели в функциях `wgpuCreateSomething`, мы опишем это множество параметров ниже. Флаг `WGPU_NULLABLE` это пустой дефайн, который дает читателям (то есть, нам), оставить аргумент `nullptr`, чтобы использовать **настройки по умолчанию**.

### Асинхронные функции

Последние два аргумента идут вместе, и представляют собой еще одну **идиому WebGPU**. Функция `wgpuInstanceRequestAdapter` **асинхронная**. Это значит, что вместо того, чтобы напрямую возвращать объект `WGPUAdapter`, эта функция запоминает **коллбэк**, тобиш функцию, которая будет вызвана когда запрос будет выполнен.

```{note}
Асинхронные функции используются в нескольких местах WebGPU API, там где операция может занять некоторое время. На самом деле **ни одна из функций WebGPU** не занимает времени для возврата. Так, программа CPU, которую мы пишем, никогда не заблокируется длинной операцией!
```

Чтобы понять этот механизм коллбэков, вот определение типа функции `WGPURequestAdapterCallback`:

```C++
// Определение типа функции WGPURequestAdapterCallback, определенной в webgpu.h
typedef void (*WGPURequestAdapterCallback)(
	WGPURequestAdapterStatus status,
	WGPUAdapter adapter,
	char const * message,
	void * userdata
);
```

Коллбэк это **функция**, которая получает **запрошенный адаптер** как аргумент, вместе с **статусом** (который говорит зафейлился ли реквест и почему если да), и мистический **указатель** `userdata`.

Этот указатель `userdata` может быть чем угодно, он не описан в WebGPU, и только **передается** из изначального вызова `wgpuInstanceRequestAdapter` в коллбэк, как средство для **передачи контекстной информации**:

```C++
void onAdapterRequestEnded(
	WGPURequestAdapterStatus status, // статус реквеста
	WGPUAdapter adapter, // получаемый адаптер
	char const* message, // сообщение об ошибке, или nullptr
	void* userdata // кастомная инфа юзера, передается при реквесте адаптера
) {
	// [...] Сделать что-то с адаптером

	// Взаимодействие с user data
	bool* pRequestEnded = reinterpret_cast<bool*>(userdata);
	*pRequestEnded = true;
}

// [...]

// В main():
bool requestEnded = false;
wgpuInstanceRequestAdapter(
	instance /* эквивалент navigator.gpu */,
	&options,
	onAdapterRequestEnded,
	&requestEnded // в данном случае кастомная инфа юзера это просто указатель на булеан
);
```

В следующих главах мы увидим более продвинутое использование контекста для получения адаптера, как только реквест будет выполнен.

````{admonition} Заметка - JavaScript API
:class: foldable note

В **API JavaScript** WebGPU, используют механизм [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise):


```js
const adapterPromise = navigator.gpu.requestAdapter(options);
// "промис" еще не имеет значения, он скорее является средством для подключением коллбэков:
adapterPromise.then(onAdapterRequestEnded).catch(onAdapterRequestFailed);

// [...]

// Вместо аргумента 'status', мы имеем множество коллбэков:
function onAdapterRequestEnded(adapter) {
	// сделать что-то с адаптером
}
function onAdapterRequestFailed(error) {
	// отобразить ошибку
}
```

В языке JavaScript позже появился механизм [`async` функций](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function), который позволяет **"ждать"** асинхронные функции без создания коллбэков:

```js
// (Внутри асинхронной функции)
const adapter = await navigator.gpu.requestAdapter(options);
// что-то делаем с адаптером
```
Этот механизм также существуют и в других языка, например, [Python](https://docs.python.org/3/library/asyncio-task.html), и даже был представлен в C++20 с [корутинами](https://en.cppreference.com/w/cpp/language/coroutines).

Тем не менее я пытаюсь **избегать большого уровня абстракций** в этом гайде, поэтому мы не будем их использовать (и ограничимся C++17), но продвинутые читатели могут создать свой враппер WebGPU, который полагается на корутины.
````

### Request

Мы можем обернуть весь реквест адаптера в функцию `requestAdapterSync()`, которую я просто сразу вам дам, чтобы не тратить слишком много времени на коде **бойлерплейта** (важная часть была про идею асинхронных коллбэков, описанная выше):

```{lit} C++, Includes ru (append)
#include <cassert>
```

```{lit} C++, Request adapter function ru
/**
 * Вспомогательная функция для получения адаптера WebGPU, так чтобы
 *     WGPUAdapter adapter = requestAdapterSync(options);
 * было эквивалентно
 *     const adapter = await navigator.gpu.requestAdapter(options);
 */
WGPUAdapter requestAdapterSync(WGPUInstance instance, WGPURequestAdapterOptions const * options) {
	// Простенькая структура, содержащая информацию для
	// коллбэка onAdapterRequestEnded.
	struct UserData {
		WGPUAdapter adapter = nullptr;
		bool requestEnded = false;
	};
	UserData userData;

	// Коллбэк вызовется wgpuInstanceRequestAdapter, когда реквест будет выполнен
	// Это лямбда функция C++, однако ей может быть любая функция определенная
	// в global scope. Это должна быть non-capturing лямбда (скобки [] пусты),
	// для того чтобы она вела себя как простой сишный указатель на функцию, именно этого
	// и ожидает wgpuInstanceRequestAdapter (ибо WebGPU это C API). Идея в том,
	// чтобы использовать для получения нужной информации указатель pUserData,
	// переданный последним аргументом в wgpuInstanceRequestAdapter и полученный
	// коллбэком последним аргументом.
	auto onAdapterRequestEnded = [](WGPURequestAdapterStatus status, WGPUAdapter adapter, char const * message, void * pUserData) {
		UserData& userData = *reinterpret_cast<UserData*>(pUserData);
		if (status == WGPURequestAdapterStatus_Success) {
			userData.adapter = adapter;
		} else {
			std::cout << "Could not get WebGPU adapter: " << message << std::endl;
		}
		userData.requestEnded = true;
	};

	// Вызываем процедуру реквеста адаптера WebGPU
	wgpuInstanceRequestAdapter(
		instance /* эквивалент navigator.gpu */,
		options,
		onAdapterRequestEnded,
		(void*)&userData
	);

	// ждем пока userData.requestEnded не станет true
	{{Wait for request to end ru}}

	assert(userData.requestEnded);

	return userData.adapter;
}
```

```{lit} C++, Utility functions ru (hidden)
// Все вспомогательные функции перегруппированы здесь
{{Request adapter function ru}}
```

В основной функции, после создания инстанса WebGPU, можем получить адаптер:

```{lit} C++, Request adapter ru
std::cout << "Requesting adapter..." << std::endl;

WGPURequestAdapterOptions adapterOpts = {};
adapterOpts.nextInChain = nullptr;
WGPUAdapter adapter = requestAdapterSync(instance, &adapterOpts);

std::cout << "Got adapter: " << adapter << std::endl;
```

#### Ждем окончания реквеста

Ты возможно заметил коммент выше **нужно дождаться** окончания реквеста, то есть вызова коллбэка, перед возвратом функции.

Когда мы используем **нативный** API (Dawn или `wgpu-native`), что на практике **не нужно**, мы знаем что функция `wgpuInstanceRequestAdapter` сделает возврат, только когда его коллбэк будет вызван.

Однако, когда мы используем **Emscripten**, мы должны вернуть управление **обратно браузеру** до тех пор пока адаптер не будет готов. В JavaScript, это возможно благодаря ключевому слову `await`. В свою очередь, Emscripten дает функцию `emscripten_sleep`, которая прерывает модуль C++ на несколько миллисекунд:

```{lit} C++, Wait for request to end ru
#ifdef __EMSCRIPTEN__
	while (!userData.requestEnded) {
		emscripten_sleep(100);
	}
#endif // __EMSCRIPTEN__
```

Для того ,чтобы использовать это, нам нужно добавить **кастомную опцию линковки** в `CMakeLists.txt`, в блок `if (EMSCRIPTEN)`:

```{lit} CMake, Emscripten-specific options ru (append)
# Позволяет использовать emscripten_sleep()
target_link_options(App PRIVATE -sASYNCIFY)
```

Также не забудьте заинклудить `emscripten.h` для использования `emscripten_sleep`:

```{lit} C++, Includes ru (append)
#ifdef __EMSCRIPTEN__
#  include <emscripten.h>
#endif // __EMSCRIPTEN__
```

### Уничтожение

Как и для инстанса WebGPU, мы должны освободить память адаптера:

```{lit} C++, Destroy adapter ru
wgpuAdapterRelease(adapter);
```

````{note}
We will no longer need to use the `instance` once we have selected our **adapter**, so we can call `wgpuInstanceRelease(instance)` right after the adapter request **instead of at the very end**. The **underlying instance** object will keep on living until the adapter gets released but we do not need to manager this.

Нам больше не нужно использовать `instance` после выбора нашего **адаптера**, поэтому можем вызвать`wgpuInstanceRelease(instance)` сразу после реквеста адаптера **вместо самого конца**. **Инстанс** будет жить где-то там внутри, пока адаптер не будет освобожден и потому нам не нужно контролировать его освобождение.

```{lit} C++, Create things ru (hidden)
{{Create WebGPU instance ru}}
{{Check WebGPU instance ru}}
{{Request adapter ru}}
// Нам больше не нужен инстанс после того как получили адаптер
{{Destroy WebGPU instance ru}}
```
````

```{lit} C++, file: main.cpp ru (replace, hidden)
{{Includes ru}}

{{Utility functions in main.cpp ru}}

int main() {
	{{Create things ru}}

	{{Main body ru}}

	{{Destroy things ru}}

	return 0;
}
```

```{lit} C++, Utility functions in main.cpp ru (hidden)
{{Utility functions ru}}
```

```{lit} C++, Main body ru (hidden)

```

```{lit} C++, Destroy things ru (hidden)
{{Destroy adapter ru}}
```

## Рассмотрим адаптер

Адаптер дает **информацию об внутренней реализации** и устройствах, и о том, что можно сделать, а что нельзя. Он дает следующую информацию:

-   **Limits (ограничения)** содержит все **максимальные и минимальные** значения которые ограничивают поведение GPU и его драйвера. Например, максимальный размер текстуры. Доступные ограничения можно получить используя `wgpuAdapterGetLimits`.
-   **Features (фичи)** не обязательные **расширения** WebGPU, которые адаптера могут или нет могут поддерживать. Можно получить используя `wgpuAdapterEnumerateFeatures` или тестируя индивидуально с помощью `wgpuAdapterHasFeature`.
-   **Properties (свойства) ** дополнительная информация об адаптере, типо имени, производителя, и тд. Свойства получаем с помощью `wgpuAdapterGetProperties`.

```{note}
В приведенном коде, рассмотрение возможностей адаптера находятся в функции `inspectAdapter()`.
```

```{lit} C++, Utility functions ru (append, hidden)
void inspectAdapter(WGPUAdapter adapter) {
	{{Inspect adapter ru}}
}
```

```{lit} C++, Request adapter ru (append, hidden)
inspectAdapter(adapter);
```

### Ограничения

We can first list the limits that our adapter supports with `wgpuAdapterGetLimits`. This function takes as argument a `WGPUSupportedLimits` object where it writes the limits:

```{lit} C++, Inspect adapter ru
#ifndef __EMSCRIPTEN__
WGPUSupportedLimits supportedLimits = {};
supportedLimits.nextInChain = nullptr;

#ifdef WEBGPU_BACKEND_DAWN
bool success = wgpuAdapterGetLimits(adapter, &supportedLimits) == WGPUStatus_Success;
#else
bool success = wgpuAdapterGetLimits(adapter, &supportedLimits);
#endif

if (success) {
	std::cout << "Adapter limits:" << std::endl;
	std::cout << " - maxTextureDimension1D: " << supportedLimits.limits.maxTextureDimension1D << std::endl;
	std::cout << " - maxTextureDimension2D: " << supportedLimits.limits.maxTextureDimension2D << std::endl;
	std::cout << " - maxTextureDimension3D: " << supportedLimits.limits.maxTextureDimension3D << std::endl;
	std::cout << " - maxTextureArrayLayers: " << supportedLimits.limits.maxTextureArrayLayers << std::endl;
}
#endif // NOT __EMSCRIPTEN__
```

```{admonition} Implementation divergences
The procedure `wgpuAdapterGetLimits` returns a boolean in `wgpu-native` but a `WGPUStatus` in Dawn.

Also, as of April 1st, 2024, `wgpuAdapterGetLimits` is not implemented yet on Google Chrome, hence the `#ifndef __EMSCRIPTEN__` above.
```

Here is an example of what you could see:

```
Adapter limits:
 - maxTextureDimension1D: 32768
 - maxTextureDimension2D: 32768
 - maxTextureDimension3D: 16384
 - maxTextureArrayLayers: 2048
```

This means for instance that my GPU can handle 2D textures up to 32k, 3D textures up to 16k and texture arrays up to 2k layers.

```{note}
There are **many more limits**, that we will progressively introduce in the next chapters. The **full list** is [available in the spec](https://www.w3.org/TR/webgpu/#limits), together with their **default values**, which is also expected to be the minimum for an adapter to claim support for WebGPU.
```

### Features

Let us now focus on the `wgpuAdapterEnumerateFeatures` function, which enumerates the features of the WebGPU implementation, because its usage is very typical from WebGPU native.

We call the function **twice**. The **first time**, we provide a null pointer as the return, and as a consequence the function only returns the **number of features**, but not the features themselves.

We then dynamically **allocate memory** for storing this many items of result, and call the same function a **second time**, this time with a pointer to where the result should store its result.

```{lit} C++, Includes ru (append)
#include <vector>
```

```{lit} C++, Inspect adapter ru (append)
std::vector<WGPUFeatureName> features;

// Call the function a first time with a null return address, just to get
// the entry count.
size_t featureCount = wgpuAdapterEnumerateFeatures(adapter, nullptr);

// Allocate memory (could be a new, or a malloc() if this were a C program)
features.resize(featureCount);

// Call the function a second time, with a non-null return address
wgpuAdapterEnumerateFeatures(adapter, features.data());

std::cout << "Adapter features:" << std::endl;
std::cout << std::hex; // Write integers as hexadecimal to ease comparison with webgpu.h literals
for (auto f : features) {
	std::cout << " - 0x" << f << std::endl;
}
std::cout << std::dec; // Restore decimal numbers
```

The features are numbers corresponding to the enum `WGPUFeatureName` defined in `webgpu.h`. We use `std::hex` to display them as hexadecimal values, because this is how they are listed in `webgpu.h`.

You may notice very high numbers apparently not defined in this enum. These are **extensions** provided by our native implementation (e.g., defined in `wgpu.h` instead of `webgpu.h` in the case of `wgpu-native`).

### Properties

Lastly we can have a look at the adapter's properties, that contain information that we may want to display to the end user:

```{lit} C++, Inspect adapter ru (append)
WGPUAdapterProperties properties = {};
properties.nextInChain = nullptr;
wgpuAdapterGetProperties(adapter, &properties);
std::cout << "Adapter properties:" << std::endl;
std::cout << " - vendorID: " << properties.vendorID << std::endl;
if (properties.vendorName) {
	std::cout << " - vendorName: " << properties.vendorName << std::endl;
}
if (properties.architecture) {
	std::cout << " - architecture: " << properties.architecture << std::endl;
}
std::cout << " - deviceID: " << properties.deviceID << std::endl;
if (properties.name) {
	std::cout << " - name: " << properties.name << std::endl;
}
if (properties.driverDescription) {
	std::cout << " - driverDescription: " << properties.driverDescription << std::endl;
}
std::cout << std::hex;
std::cout << " - adapterType: 0x" << properties.adapterType << std::endl;
std::cout << " - backendType: 0x" << properties.backendType << std::endl;
std::cout << std::dec; // Restore decimal numbers
```

Here is a sample result with my nice Titan RTX:

```
Adapter properties:
 - vendorID: 4318
 - vendorName: NVIDIA
 - architecture:
 - deviceID: 7682
 - name: NVIDIA TITAN RTX
 - driverDescription: 536.23
 - adapterType: 0x0
 - backendType: 0x5
```

## Conclusion

-   The very first thing to do with WebGPU is to get the **adapter**.
-   Once we have an adapter, we can inspect its **capabilities** (limits, features) and properties.
-   We learned to use **asynchronous functions** and **double call** enumeration functions.

_Resulting code:_ [`step005`](https://github.com/eliemichel/LearnWebGPU-Code/tree/step005)

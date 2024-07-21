Learn WebGPU
============

*Для нативной графики на C++.*

Эта документация научит вас использовать [WebGPU](https://www.w3.org/TR/webgpu), чтобы создавать **нативные 3D приложения** на C++ с нуля для Windows, Linux и macOS. 

`````{admonition} Быстрый старт! (Кликни)
:class: foldable quickstart

*Хочешь понять каждый кусок GPU кода, который ты напишешь?*

````{admonition} Да, написать WebGPU код **с нуля**!
:class: foldable yes

Супер! Ты просто можешь начать с [вступления](introduction.md) и **прочитать все главы** последовательно.

````

````{admonition} Нет, я бы лучше **пропустил начальный шаблон**.
:class: foldable no

Без проблем! Если что, ты всегда можешь **вернуться к [начальным шагам](getting-started/index.md) позже**.

Возможно тебе пригодится ссылка на _**Итоговый код**_  в начале и конце **каждой страницы**, например:


```{image} /images/intro/resulting-code-light.png
:class: only-light with-shadow
```

```{image} /images/intro/resulting-code-dark.png
:class: only-dark with-shadow
```

*Тебе хочется использовать высокоуровневую обертку для более легкого чтения?*

```{admonition} Да, Я предпочитаю **C++ styled** код.
:class: foldable yes

Тогда используй вкладку "**С webgpu.hpp**".
```

```{admonition} Нет, покажи мне **сырой C WebGPU API**!
:class: foldable no

Используй вкладку "**Vanilla webgpu.h**". *Итоговый код* для vanilla WebGPU несколько устаревший, но эта вкладка также переключает **все блоки кода** внутри гайда, и вот они **самые актуальные**.
```

Чтобы **сбилдить этот базовый код**, обратись к параграфу [Сборка](getting-started/project-setup.md#building) главы настройки проекта. Ты можешь добавить `-DWEBGPU_BACKEND=WGPU` (по умолчанию) или `-DWEBGPU_BACKEND=DAWN` в строчку `cmake -B build`, чтобы выбрать либо [`wgpu-native`](https://github.com/gfx-rs/wgpu-native), либо [Dawn](https://dawn.googlesource.com/dawn/) в качестве бэкенда соответственно.


*На сколько ты хочешь расширить базовый код?*

```{admonition} Простой треугольник
:class: foldable quickstart

Зацени главу [Привет Треугольник](basic-3d-rendering/hello-triangle.md).
```

```{admonition} 3D mesh viewer с базовым взаимодействием
:class: foldable quickstart

Тогда рекомендую начать с конца главы [Управление светом](basic-3d-rendering/some-interaction/lighting-control.md).
```

````

```{admonition} Хочу чтобы штука **работала в вебе**.
:class: foldable warning

Основной текст гайда пропускает несколько дополнительных строк для этого, обратись к приложению [Сборка для Web](appendices/building-for-the-web.md) чтобы **адаптировать примеры** для работы в Web!
```

`````

Содержание
--------

```{toctree}
:titlesonly:

introduction
getting-started/index
basic-3d-rendering/index
basic-compute/index
advanced-techniques/index
appendices/index
```

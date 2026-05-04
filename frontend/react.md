Рекомендую предварительно посмотреть три лекции Темы Сенюкова:

[https://www.youtube.com/live/5LH5M5wvz34?si=OX7qXQgbAm3edch9](https://www.youtube.com/live/5LH5M5wvz34?si=OX7qXQgbAm3edch9)

[https://www.youtube.com/live/nja8YT4-fyQ?si=X2r5aX95r9_9ZgJ7](https://www.youtube.com/live/nja8YT4-fyQ?si=X2r5aX95r9_9ZgJ7)

[https://www.youtube.com/live/HDajDASxn-w?si=59Fl4zHs_E2omiWe](https://www.youtube.com/live/HDajDASxn-w?si=59Fl4zHs_E2omiWe)

# Общая теория

<details>
<summary><b>Что такое React? Для чего он нужен?</b></summary>

React — это JavaScript библиотека для создания пользовательских интерфейсов.

Предназначена для рендеринга интерфейсов.

Основные преимущества React:

- компонентный подход

- декларативность

- простота входа для начинающих

- популярность и стабильность

- детерминированность, за счет FLUX-архитектуры (меняется состояние → происходит рендеринг)

---

### Дополнительные материалы

- [https://habr.com/ru/companies/ruvds/articles/519902/](https://habr.com/ru/companies/ruvds/articles/519902/)

</details>

<details>
<summary><b>Что такое Virtual DOM?</b></summary>

В современном мире, когда React стал мультиплатформенным и запускается не только в браузере, разработчики React рекомендуют отходить от понятия Virtual DOM, к понятию дерева React элементов (Render Tree).

Дерево React элементов — это древовидная структура, состоящая из JS-объектов, описывающих структуру React приложения. Все элементы в ней иммутабельны и не хранят свое состояние.

При каждом изменении React сравнивает новое дерево React-элементов со старым деревом React-элементов, находит diff и точечно применяет его к реальному DOM. Это позволяет приложению работать быстрее, так как основные манипуляции происходят с деревом React элементов, которое является более легковесным по сравнению с реальным DOM.

---

### Дополнительные материалы

- [https://react.dev/learn/understanding-your-ui-as-a-tree#the-render-tree](https://react.dev/learn/understanding-your-ui-as-a-tree#the-render-tree)

- Тема Сенюков [https://www.youtube.com/live/HDajDASxn-w?si=XgureyCcU8h6zOkO](https://www.youtube.com/live/HDajDASxn-w?si=XgureyCcU8h6zOkO)

</details>

<details>
<summary><b>Почему алгоритм сравнения и построения Virtual DOM быстрый? O(n) вместо O(n^3)?</b></summary>

Если кратко, то React применяет две эвристики при сравнении старого Virtual DOM с новым:

1. Если тип React элемента отличается, то React считает, что его надо уничтожить и создать заново. При проверке он не идет вниз по дереву и не сравнивает дочерние элементы компонента, если сам компонент поменялся.

2. Если key изменился, то React уничтожает и создает элемент заново. Если не изменился — то не трогает, максимум перерендер.

---

### Дополнительные материалы

- ОБЯЗАТЕЛЬНО ПОСМОТРИ: [https://www.youtube.com/live/HDajDASxn-w?si=59Fl4zHs_E2omiWe](https://www.youtube.com/live/HDajDASxn-w?si=59Fl4zHs_E2omiWe)

</details>

<details>
<summary><b>Что такое React порталы и для чего они нужны?</b></summary>

React портал — это инструмент, позволяющий отрендерить элемент в любом месте DOM, несмотря на его расположение в дереве React элементов. Используется с помощью функции `createPortal`, которая первым аргументом принимает React элемент, а вторым аргументом — DOM элемент, в который его необходимо отрендерить.

Порталы используются, когда нужно отрендерить компонент вне текущей иерархии компонентов, но при этом сохранить доступ к контексту и состоянию родительского компонента. Например, для модальных окон, попапов, тостов, хинтов и прочего.

---

### Дополнительные материалы

- [https://react.dev/reference/react-dom/createPortal](https://react.dev/reference/react-dom/createPortal)

</details>

<details>
<summary><b>Что такое JSX?</b></summary>

JSX — это расширение синтаксиса JavaScript, которое позволяет писать HTML-подобный код внутри JavaScript.

Основные правила использования JSX:

1. Компонент должен возвращать один корневой элемент

2. Все теги должны закрываться

3. Названия практически всех атрибутов пишутся через camelCase

---

### Дополнительные материалы

- [https://react.dev/learn/writing-markup-with-jsx](https://react.dev/learn/writing-markup-with-jsx)

</details>

<details>
<summary><b>В чем разница между контролируемыми (controlled) и неконтролируемыми (uncontrolled) компонентами в React?</b></summary>

Контролируемые компоненты управляются стейтом React (useState), неконтролируемые компоненты используют сам DOM для обработки данных.

---

### Дополнительные материалы

- Дока ([https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components))

</details>

<details>
<summary><b>Для чего нужен React.lazy?</b></summary>

`React.lazy` используется для ленивой загрузки кода компонентов, что улучшает производительность приложения, разбивая его на части и загружая их по требованию.

---

### Дополнительные материалы

- Дока ([https://react.dev/reference/react/lazy](https://react.dev/reference/react/lazy))

- Статья ([https://habr.com/ru/sandbox/183594/](https://habr.com/ru/sandbox/183594/))

- YouTube ([https://www.youtube.com/watch?v=ANQYKGUcLMg](https://www.youtube.com/watch?v=ANQYKGUcLMg))

</details>

<details>
<summary><b>Что такое React.Fragment и для чего он нужен?</b></summary>

`React.Fragment` — это специальный React элемент, который нужен для того, чтобы вернуть из компонента несколько элементов без добавления лишнего родительского DOM узла.

---

### Дополнительные материалы

- [https://react.dev/reference/react/Fragment](https://react.dev/reference/react/Fragment)

</details>

<details>
<summary><b>Чем отличаются нативные браузерные события от событий в React?</b></summary>

Нативные события обрабатываются непосредственно в DOM-дереве. Когда происходит событие (например, клик на кнопку), оно распространяется через DOM-дерево в две фазы: **фаза захвата** (event capturing) и **фаза всплытия** (event bubbling). Обработчики событий добавляются с помощью методов вроде `addEventListener` и могут быть привязаны к любой фазе (захват или всплытие).

React использует **систему событий "синтетических событий" (Synthetic Events)**, которая является кросс-браузерной абстракцией над нативными событиями. React не использует нативное DOM API для назначения событий на каждый элемент. Вместо этого он добавляет один обработчик для всего приложения (обычно на уровне корневого элемента), который управляет всеми событиями через механизм **делегирования** (event delegation).

Преимущества событий в React:

- Кроссбраузерность

- За счет делегирования событий снижается количество обработчиков событий и улучшается производительность в больших приложениях

- React автоматически удаляет события при размонтировании компонента, что упрощает управление жизненным циклом событий

- Обработка событий в соответствии с деревом React элементов, а не реальным DOM (удобно при использовании порталов)

---

### Дополнительные материалы

- [https://medium.com/trabe/events-in-react-what-do-they-do-do-they-do-things-lets-find-out-9f1ac743b4c7](https://medium.com/trabe/events-in-react-what-do-they-do-do-they-do-things-lets-find-out-9f1ac743b4c7)

- [https://youtu.be/edHN-8hZvcU?si=FUBEGeQNT-8WppS2](https://youtu.be/edHN-8hZvcU?si=FUBEGeQNT-8WppS2)

</details>

<details>
<summary><b>Что такое и зачем нужен StrictMode в React?</b></summary>

React Strict Mode - это компонент-обертка, которая помогает обнаруживать потенциальные проблемы в приложении React. Она включает дополнительные проверки и предупреждения в режиме разработки, которые могут помочь выявить проблемы, такие как устаревшие методы жизненного цикла, непреднамеренное изменение состояния и другие возможные ошибки.

Использование StrictMode приводит к следующим эффектам:

- Компоненты будут повторно ререндериться, чтобы найти ошибки, вызванные неконсистеностью состояния

- Компоненты будут повторно запускать эффекты, чтобы найти ошибки, вызванные отсутствием очистки эффектов

- Компоненты будут проверяться на использование устаревших API

---

### Дополнительные материалы

- [https://react.dev/reference/react/StrictMode](https://react.dev/reference/react/StrictMode)

- [https://reactdev.ru/reference/react/StrictMode/](https://reactdev.ru/reference/react/StrictMode/)

</details>

# Хуки

<details>
<summary><b>Какие хуки знаешь?</b></summary>

Здесь лучше рассказать про то, какие хуки ты используешь на своей практике.

Основные React хуки:

- useState

- useEffect

- useContext

- useRef

- useMemo

- useCallback

- useDefferedValue

- useImperativeHandle

- useLayoutEffect

- useId

- useInsertionEffect

- useTransition

---

### Дополнительные материалы

- [https://react.dev/reference/react/hooks](https://react.dev/reference/react/hooks)

</details>

<details>
<summary><b>Для чего нужен хук useEffect?</b></summary>

Хук `useEffect` нужен, чтобы выполнять действия после того, как компонент отрендерился на странице или обновился. Это полезно для таких вещей, как загрузка данных, настройка подписок или взаимодействие с другими частями браузера.

Как это работает:

1. `useEffect` принимает функцию-колбэк, которая запускается каждый раз, когда компонент рендерится.

2. Можно указать второй аргумент — массив зависимостей. Тогда эффект сработает только при изменении значений в этом массиве.

    - Пустой массив `[]` означает, что эффект выполнится один раз, когда компонент впервые появился на странице.

    - Если второй аргумент не указан, функция будет выполняться при каждом рендере.

Также `useEffect` может возвращать функцию для очистки (клинап-функция), которая запускается перед повторным вызовом эффекта или при удалении компонента (в случае пустого массива зависимостей) — это удобно для остановки подписок и других задач очистки.

---

### Дополнительные материалы

- [https://react.dev/reference/react/useEffect](https://react.dev/reference/react/useEffect)

</details>

<details>
<summary><b>Как работает useCallback и для чего он нужен?</b></summary>

Хук `useCallback` нужен, чтобы сохранить функцию и не создавать её заново при каждом рендере. Он возвращает ту же самую версию функции (мемоизирует её), пока не изменятся указанные зависимости. Зависимости передаются вторым аргументов в виде массива.

Для чего это нужно? Если мы передаём функцию в дочерний компонент, `useCallback` помогает избежать ненужных перерисовок, потому что сохраняет ту же ссылку на функцию, если зависимости не меняются. Это полезно для оптимизации производительности.

---

### Дополнительные материалы

- [https://react.dev/reference/react/useCallback#usecallback](https://react.dev/reference/react/useCallback#usecallback)

</details>

<details>
<summary><b>Как работает useMemo и для чего он нужен?</b></summary>

`useMemo` используется для того, чтобы закешировать результат вычислений. Первым аргументом принимает вычисляющую функцию, вторым — массив зависимостей. Вычисляющая функция вызывается только при изменении одной из зависимостей. Эта оптимизация помогает избежать дорогостоящих вычислений при каждом рендере.

---

### Дополнительные материалы

- [https://react.dev/reference/react/useMemo](https://react.dev/reference/react/useMemo)

</details>

<details>
<summary><b>Для чего нужен хук useRef?</b></summary>

`useRef` возвращает объект с полем `current`, в котором можно хранить состояние, которое при изменении не вызывает ререндеры компонента.

Используется в двух случаях:

1. Для хранения данных без реактивности (например, для сохранения ссылки на интервал)

2. Для хранения ссылки на React DOM элемент

---

### Дополнительные материалы

- [https://react.dev/reference/react/useRef#useref](https://react.dev/reference/react/useRef#useref)

</details>

<details>
<summary><b>Какие требования есть к хукам в React?</b></summary>

1. Название начинается с `use`

2. Хуки могут вызываться только внутри функциональных React компонентов или других хуков

3. Хуки нельзя вызывать условно, в циклах и во вложенных функциях

---

### Дополнительные материалы

- [https://react.dev/reference/rules/rules-of-hooks](https://react.dev/reference/rules/rules-of-hooks)

</details>

<details>
<summary><b>Что сработает раньше useEffect или useLayoutEffect?</b></summary>

useLayoutEffect выполнится раньше, так как он работает синхронно в момент, когда компоненты уже находятся на Virtual DOM (в памяти и можно прочитать/установить различные свойств), но еще не были отрисованы браузером.

useEffect работает асинхронно и выполнится тогда, когда страница уже будет отрисована браузером

---

### Дополнительные материалы

- [https://habr.com/ru/companies/otus/articles/668700/](https://habr.com/ru/companies/otus/articles/668700/)

</details>

<details>
<summary><b>Что такое ленивая инициализация useState?</b></summary>

`useState` принимает в виде аргумента начальное значение состояния. Его можно передать как в виде просто значения, так и функции-колбэка, возвращающей это значение. Второй вариант называется ленивой инициализацией и полезен в случаях, когда первоначальное состояние создается на основе каких-либо затратных вычислений. Чтоб не выполнять эти вычисления при каждом рендере компонента, они оборачиваются в колбэк, который будет вызван только один раз при монтировании компонента.

Пример:

```TypeScript
const [state, setState] = useState<string>(() => {
    const initialState: string = someExpensiveComputation(props);
  
    return initialState;
})
```

---

### Дополнительные материалы

- [https://dev.to/msshanmukh/lazy-initialization-and-storing-functions-with-react-usestate-hook-1ahj](https://dev.to/msshanmukh/lazy-initialization-and-storing-functions-with-react-usestate-hook-1ahj)

</details>

<details>
<summary><b>Для чего нужен хук useDeferredValue?</b></summary>

`useDeferredValue` в React используется для отложенного обновления значения состояния, что помогает улучшить производительность компонентов при частых изменениях данных.

---

### Дополнительные материалы

- Дока ([https://react.dev/reference/react/useDeferredValue](https://react.dev/reference/react/useDeferredValue))

- Статья ([https://www.geeksforgeeks.org/what-is-usedeferredvalue-hook-and-how-to-use-it/](https://www.geeksforgeeks.org/what-is-usedeferredvalue-hook-and-how-to-use-it/))

- YouTube ([https://www.youtube.com/watch?v=AdH46TVjpBE](https://www.youtube.com/watch?v=AdH46TVjpBE))

</details>

<details>
<summary><b>Для чего нужен хук useTransition?</b></summary>

`useTransition` в React используется для обновления стейта в неблокирующем работу интерфейса режиме.

---

### Дополнительные материалы

- [https://react.dev/reference/react/useTransition](https://react.dev/reference/react/useTransition)

- [https://youtu.be/hWUlv7M7rR8?si=9E3fac7A-FeHBDz-](https://youtu.be/hWUlv7M7rR8?si=9E3fac7A-FeHBDz-)

</details>

<details>
<summary><b>Для чего нужен хук useReducer, если есть useState?</b></summary>

`useReducer` помогает улучшить читаемость и связность кода в случаях, когда происходит работа со сложным состоянием (например, объект с большим кол-вом полей), части которого взаимосвязаны и требуют синхронизации при изменении.

</details>

<details>
<summary><b>Для чего нужен хук useImperativeHandle?</b></summary>

`useImperativeHandle` - это хук в React, который позволяет родительскому компоненту получать доступ к определенным методам или значениям дочернего компонента и контролировать их использование.

---

### Дополнительные материалы

- Дока ([https://react.dev/reference/react/useImperativeHandle](https://react.dev/reference/react/useImperativeHandle))

- Статья ([https://blog.webdevsimplified.com/2022-06/use-imperative-handle/](https://blog.webdevsimplified.com/2022-06/use-imperative-handle/))

- YouTube ([https://www.youtube.com/watch?v=ndVIEMasBl8&t=3s](https://www.youtube.com/watch?v=ndVIEMasBl8&t=3s))

</details>

# Паттерны

<details>
<summary><b>Что такое React Context и для чего он нужен?</b></summary>

Классический паттерн передачи данных от родительского компонента дочернему — это передача данных через `props`. Однако такой способ может быть чересчур громоздким в ситуациях, когда большому количеству глубоко вложенных компонентов необходимы одни и те же данные. Появляются компоненты, которые принимают пропсы, которые им нужны, только для того, чтобы передать их дочерним. Этот анти-паттерн называется Prop Drilling.

Одним из решений этой проблемы является использование React Context. Он предоставляет способ делиться данными с компонентами без необходимости явно передавать пропсы через каждый уровень дерева.

Дерево компонентов оборачиваются в Context Provider. Любой из вложенных компонентов сможет получить доступ к значение из контекста через хук `useContext`.

---

### Дополнительные материалы

- [https://dev.to/codeofrelevancy/what-is-prop-drilling-in-react-3kol](https://dev.to/codeofrelevancy/what-is-prop-drilling-in-react-3kol)

- [https://react.dev/reference/react/createContext](https://react.dev/reference/react/createContext)

- [https://react.dev/reference/react/useContext](https://react.dev/reference/react/useContext)

</details>

<details>
<summary><b>Что такое паттерн Compound Components?</b></summary>

Паттерн Compound Components используется для сложных компонентов, состоящих из нескольких частей. Он предлагает подход, в котором несколько компонентов объединяются одной общей сущностью и одним общим состоянием.

На практике часто выглядит следующим образом: создается родительский компонент, который провайдит контекст с некоторой общей логикой всем дочерним компонентам. Дочерние компоненты создаются отдельно и передаются в качестве `children` родительскому. Часто создают в виде свойства родительского компонента, чтобы подчеркнуть связь между ними.

---

### Дополнительные материалы

- [https://habr.com/ru/companies/alfa/articles/647013/](https://habr.com/ru/companies/alfa/articles/647013/)

- [https://www.patterns.dev/react/compound-pattern/](https://www.patterns.dev/react/compound-pattern/)

</details>

<details>
<summary><b>Что такое паттерн Render Props?</b></summary>

Render Prop — это паттерн в React, при использовании которого в компонент через пропсы передаются функции рендера дочерних компонентов.

Например, есть некий компонент `Layout`, в который передаются рендер-функции для отрисовки компонентов внутри этого лэйаута (header, content, footer и тд).

---

### Дополнительные материалы

- [https://www.patterns.dev/react/render-props-pattern/](https://www.patterns.dev/react/render-props-pattern/)

</details>

<details>
<summary><b>Что такое HOC (High Order Component)?</b></summary>

HOC — это компонент высшего порядка. Представляет из себя функцию, которая принимает один или несколько компонентов и возвращает React компонент, обогащенный дополнительной функциональностью. Название HOC, как правило, начинается с `with`.

Например: `withAuthorization`, `withStyles` и тд

---

### Дополнительные материалы

- [https://www.patterns.dev/react/hoc-pattern](https://www.patterns.dev/react/hoc-pattern)

</details>

<details>
<summary><b>Что такое виртуализация?</b></summary>

Виртуализация в JavaScript — это метод оптимизации работы с большими объемами данных или динамическим контентом путем эффективного управления отображением элементов на странице только при необходимости.

Другими словами, подход заключается в том, чтоб не рендерить контент, который находится вне вьюпорта (видимой зоны) пользователя.

---

### Дополнительные материалы

- [https://youtu.be/1JoEuJQIJbs?si=cCFTKqBRfWxxCeZr](https://youtu.be/1JoEuJQIJbs?si=cCFTKqBRfWxxCeZr)

- [https://medium.com/@atulbanwar/list-virtualization-in-react-3db491346af4](https://medium.com/@atulbanwar/list-virtualization-in-react-3db491346af4)

</details>

# Рендеринг

<details>
<summary><b>В каких случаях происходит ререндер компонента?</b></summary>

1. Изменение состояние

2. Изменение пропсов

3. Изменение контекста (можно объединить с состоянием)

4. Ререндер родительского компонента

---

### Дополнительные материалы

- [https://habr.com/ru/companies/timeweb/articles/684718/](https://habr.com/ru/companies/timeweb/articles/684718/)

</details>

<details>
<summary><b>Для чего нужно свойство key в React?</b></summary>

Чаще всего атрибут `key` используется для оптимизации рендеринга списков. Если `key` элемента изменился, то React ремаунтит компонент. Если `key` не изменился, то React считает, что компонент тоже не изменился не ремаунтит его.

Чаще всего используется при рендеринге списков элементов, чтобы указать React, какие элементы будут стабильны между рендерами. Это позволяет сильно оптимизировать скорость отрисовки интерфейса за счет избавления от лишних ремаунтов.

---

### Дополнительные материалы

- [https://habr.com/ru/companies/otus/articles/750542/](https://habr.com/ru/companies/otus/articles/750542/)

- [https://react.dev/learn/rendering-lists](https://react.dev/learn/rendering-lists)

- Тема Сенюков [https://www.youtube.com/live/HDajDASxn-w?si=XgureyCcU8h6zOkO](https://www.youtube.com/live/HDajDASxn-w?si=XgureyCcU8h6zOkO)

</details>

<details>
<summary><b>Для чего нужен React.memo?</b></summary>

`React.memo` предназначен для оптимизации количества ререндеров компонента. Если обернуть компонент в `React.memo`, то он будет ререндерится только в том случае, если изменились значения пропсов. Важно помнить, что происходит поверхностное сравнение старых и новых пропсов. То есть объекты и функции сравниваются по ссылке.

`React.memo` часто используют в связке с `useMemo` и `useCallback`, чтоб сохранить стабильную ссылку на переданные в компонент объекты и функции.

---

### Дополнительные материалы

- [https://react.dev/reference/react/memo](https://react.dev/reference/react/memo)

</details>

<details>
<summary><b>За что отвечает второй параметр в React.memo?</b></summary>

Второй параметр `React.memo` — это функция сравнения (`areEqual`), которая позволяет определить, следует ли повторно рендерить компонент. Она принимает два аргумента: текущие и следующие пропсы, и должна возвращать `true`, если пропсы равны (и рендеринг не требуется), или `false`, если пропсы различаются (и рендеринг необходим).

---

### Дополнительные материалы

- Дока ([https://reactdev.ru/reference/react/memo/](https://reactdev.ru/reference/react/memo/))

- Статья ([https://refine.dev/blog/react-memo-guide/#reactmemo-prop-comparison](https://refine.dev/blog/react-memo-guide/#reactmemo-prop-comparison))

- YouTube ([https://www.youtube.com/watch?v=Hm769uj6WPo](https://www.youtube.com/watch?v=Hm769uj6WPo))

</details>

<details>
<summary><b>Что такое flushSync?</b></summary>

`flushSync` — это метод из React, который позволяет принудительно выполнить обновление состояния и немедленно рендерить компонент сразу после выполнения переданного в него колбэка, минуя батчинг. Обычно используется, когда необходимо, чтобы обновления состояния отображались сразу, например, при взаимодействии с внешними библиотеками, требующими немедленного обновления DOM.

Используется крайне редко в исключительных случаях, так как может негативно влиять на производительность.

---

### Дополнительные материалы

- Дока [https://react.dev/reference/react-dom/flushSync](https://react.dev/reference/react-dom/flushSync)

- Статья ([https://www.dhiwise.com/post/understanding-react-flushsync-a-deep-dive-into-synchronous-rendering](https://www.dhiwise.com/post/understanding-react-flushsync-a-deep-dive-into-synchronous-rendering))

- YouTube ([https://www.youtube.com/watch?v=lj0JjbVJPz0](https://www.youtube.com/watch?v=lj0JjbVJPz0))

</details>

<details>
<summary><b>Для чего используется компонент Suspense в React?</b></summary>

`React.Suspense` — это компонент React, который позволяет отложить рендеринг части пользовательского интерфейса до тех пор, пока не будут загружены необходимые данные или ресурсы. Обычно используется вместе с `React.lazy` для ленивой загрузки компонентов.

---

### Дополнительные материалы

- Дока ([https://react.dev/reference/react/Suspense](https://react.dev/reference/react/Suspense))

- Статья ([https://www.freecodecamp.org/news/react-suspense/](https://www.freecodecamp.org/news/react-suspense/))

- YouTube ([https://www.youtube.com/watch?v=gxEb_QUZDE0](https://www.youtube.com/watch?v=gxEb_QUZDE0))

</details>

<details>
<summary><b>Какие существуют виды рендеринга на примере NextJS?</b></summary>

1. CSR (Client Side Rendering) — в браузер прилетает почти пустой HTML, JS запрашивает данные с бэкенда и рендерит их в браузере.

2. SSR (Server Side Rendering) — сервер по запросу клиента получает данные для отрисовки, рендерит полноценную HTML страницу и отправляет ее в браузер, который ее отображает.

3. SSG (Static Site Generation) — сервер во время сборки приложения (build) запрашивает необходимые данные для отрисовки, генерирует HTML-страницу и при запросе с клиента сразу отдает ее в виде HTML.

4. ISR (Incremental Static Regeneration) — сервер по запросу генерирует страницу и отдает клиенту. При следующих запросах страница будет возвращаться из кэша.

---

### Дополнительные материалы

- [https://developer-log.vercel.app/posts/render-pattern](https://developer-log.vercel.app/posts/render-pattern)

- [https://youtu.be/HXnRJZlLtwA?si=f9yglygEAfkZnEnR](https://youtu.be/HXnRJZlLtwA?si=f9yglygEAfkZnEnR)

- [https://blog.stackademic.com/ssr-vs-csr-vs-isr-vs-ssg-d74a2e093d64](https://blog.stackademic.com/ssr-vs-csr-vs-isr-vs-ssg-d74a2e093d64)

</details>

# Задачи

<details>
<summary><b>Что произойдет при клике по кнопке?</b></summary>

Full code: [https://codesandbox.io/p/sandbox/react-interviews-1-counter-forked-9qvyq3](https://codesandbox.io/p/sandbox/react-interviews-1-counter-forked-9qvyq3)

```React
/*
  Что произойдет при клике по кнопке?
*/
export default function Counter() {
  let count = 0;

  const changeCount = () => {
    count += 1;
  };

  return (
    <div>
      <div>Count = {count}</div>

      <button onClick={changeCount}>Кнопка</button>
    </div>
  );
}
```

### Ответ

Ничего. Счетчик на экране не обновится, так как ререндер не произойдет. Чтобы компонент ререндерился, необходимо использовать useState для счетчика.

</details>

<details>
<summary><b>Что выведет код? (useEffect + useState)</b></summary>

Full code: [https://stackblitz.com/edit/vitejs-vite-tm18dy?file=src%2FApp.jsx](https://stackblitz.com/edit/vitejs-vite-tm18dy?file=src%2FApp.jsx)

```React
import { useEffect, useState } from 'react';
import './App.css';

function App() {
  const [count, setCount] = useState(1);

  console.log('step 1');

  useEffect(() => {
    console.log('step 2');

    return () => console.log('step 3');
  }, [count]);

  useEffect(() => {
    console.log('step 4');
    setCount((count) => count + 1);
  }, []);

  return <Child count={count} />;
}

function Child({ count }) {
  useEffect(() => {
    console.log('step 5');

    return () => console.log('step 6');
  }, [count]);

  return null;
}

export default App;
```

### Ответ

```
step 1
step 5
step 2
step 4
step 1
step 6
step 3
step 5
step 2
```

</details>

<details>
<summary><b>Реализовать хук <code>useFetch</code> для запросов на сервер</b></summary>

```React
const useFetch = (url) => {
	// TODO
}
```

### Ответ

```React
const useFetch = (url) => {
	const [data, setData] = useState(null)
	const [error, setError] = useState(null)
	const [loading, setLoading] = useState(false)
	
	useEffect(() => {
		setLoading(true)
		
		fetch(url)
			.then(res => res.json())
			.then(data => setData(data))
			.catch(e => setError(e))
			.finally(() => setLoading(false))
		
		return () => {
			setData(null)
			setError(null)
		}	
	}, [url])
	
	return [data, loading, error]
}
```

</details>

<details>
<summary><b>Реализовать хук <code>useFirstRender</code></b></summary>

Реализовать хук `useFirstRender`, который возвращает `true`, если это первый рендер компонента и `false` при всех последующих

Full code: [https://stackblitz.com/edit/vitejs-vite-vyz8km?file=src%2Fhook.jsx](https://stackblitz.com/edit/vitejs-vite-vyz8km?file=src%2Fhook.jsx)

```React
/*
  Реализовать хук useFirstRender
*/
const useFirstRender = () => {};
```

### Ответ

```React
const useFirstRender = () => {
  const isFirstRender = useRef(true);

  if (isFirstRender.current) {
    isFirstRender.current = false;
    return true;
  }

  return false;
};
```

</details>

<details>
<summary><b>Реализовать компонент с таймером</b></summary>

```React
const Timer = () => {
	const [count, setCount] = useState(0)
	
	// count должен увеличиваться на единицу каждую секунду
	
	return (
		<div>count: {count}</div>
	)
}
```

### Ответ

Наивная реализация

```React
const Timer = () => {
	const [count, setCount] = useState(0)
	
	useEffect(() => {
          const interval = setInterval(() => {
            setCount((c) => c + 1)
          }, 1000);

          return () => clearInterval(interval)
        }, [start]);
	
	return (
		<div>count: {count}</div>
	)
}
```

Продвинутая реализация

```React
const useNow = (updateInterval = 1000) => {
  const [now, setNow] = useState(Date.now());

  useLayoutEffect(() => {
    setNow(Date.now());

    const interval = setInterval(() => {
      setNow(Date.now());
    }, updateInterval);

    return () => clearInterval(interval);
  }, [updateInterval]);

  return now;
};

const Timer = () => {
	const [startedAt, setStartedAt] = useState(() => Date.now());

    const now = useNow();
	
	return (
		<div>count: {Math.floor((now - startedAt) / 1000)}</div>
	)
}
```

### Дополнительные материалы

- [https://youtu.be/1QECyM6QhYw?si=Q0zt6F_3UXXOwBnT](https://youtu.be/1QECyM6QhYw?si=Q0zt6F_3UXXOwBnT)

</details>

<details>
<summary><b>Оптимизировать количество ререндеров компонентов при вводе символов в инпут</b></summary>

```React
import { useState } from 'react';
import './App.css';

const Button = ({ children, onClick }: any) => {
  console.log('Button rerender');

  return <button onClick={onClick}>{children}</button>;
};

const List = ({ items }: any) => {
  console.log('List rerender');

  return items.map((item: any, idx: number) => <div key={idx}>{item}</div>);
};

function App() {
  console.log('App rerender');

  const [names, setNames] = useState<string[]>([]);
  const [inputValue, setInputValue] = useState('');

  const handleChange = (e: React.FormEvent<HTMLInputElement>) => {
    setInputValue(e.currentTarget.value);
  };

  const handleAddName = () => {
    setNames((prev) => [...prev, inputValue]);
    setInputValue('');
  };

  return (
    <div>
      <input onChange={handleChange} type="text" value={inputValue} />
      <Button onClick={handleAddName}>Add name</Button>
      <List items={names} />
    </div>
  );
}

export default App;
```

### Ответ

```React
import { memo, useCallback, useState } from 'react';
import './App.css';

const Button = memo(({ children, onClick }: any) => {
  console.log('Button rerender');

  return <button onClick={onClick}>{children}</button>;
});

const List = memo(({ items }: any) => {
  console.log('List rerender');

  return items.map((item: any, idx: number) => <div key={idx}>{item}</div>);
});

function App() {
  console.log('App rerender');

  const [names, setNames] = useState<string[]>([]);
  const [inputValue, setInputValue] = useState('');

  const handleChange = (e: React.FormEvent<HTMLInputElement>) => {
    setInputValue(e.currentTarget.value);
  };

  const handleAddName = useCallback(() => {
    setNames((prev) => [...prev, inputValue]);
    setInputValue('');
  }, []);

  return (
    <div>
      <input onChange={handleChange} type="text" value={inputValue} />
      <Button onClick={handleAddName}>Add name</Button>
      <List items={names} />
    </div>
  );
}

export default App;
```

</details>

<details>
<summary><b>Провести рефакторинг компонента <code>&lt;Numbers/&gt;</code></b></summary>

Full code: [https://codesandbox.io/p/devbox/refactor-forked-4hmhpw?file=%2Fsrc%2FApp.tsx%3A7%2C18](https://codesandbox.io/p/devbox/refactor-forked-4hmhpw?file=%2Fsrc%2FApp.tsx%3A7%2C18)

```TypeScript
import React, { useState } from "react";
import "./styles.css";

interface Todo {
  id: number;
  value: number;
}

const defaultData: Todo[] = [
  {
    id: 1,
    value: 111,
  },
  {
    id: 2,
    value: 222,
  },
  {
    id: 3,
    value: 333,
  },
];

export const Numbers = () => {
  const [data, setData] = useState(() => defaultData);
  const [value, setValue] = useState("");

  return (
    <>
      <h2>Numbers</h2>
      <div className="list">
        {data.map(({ value }) => (
          <div>• number: {value}</div>
        ))}
      </div>
      <div className="create">
        Введите число:
        <input type="text" onChange={(e) => setValue(e.target.value)} />
        <div
          className="button"
          onClick={() => {
            setData([...data, { id: ++data.length, value: Number(value) }]);
            setValue("");
          }}
        >
          Создать
        </div>
      </div>
    </>
  );
};
```

### Ответ

Full code: [https://codesandbox.io/p/devbox/refactor-forked-pgh52x](https://codesandbox.io/p/devbox/refactor-forked-pgh52x)

```TypeScript
import React, { useState } from "react";
import "./styles.css";

interface Todo {
  id: number;
  value: number;
}

const defaultData: Todo[] = [
  {
    id: 1,
    value: 111,
  },
  {
    id: 2,
    value: 222,
  },
  {
    id: 3,
    value: 333,
  },
];

export const Numbers: React.FC = () => {
  const [data, setData] = useState(defaultData);
  const [value, setValue] = useState("");

  const onCreate = () => {
    if (value.length) {
      setData([...data, { id: data.length++, value: Number(value) }]);
      setValue("");
    }
  }

  return (
    <>
      <h2>Numbers</h2>
      <ul className="list">
        {data.map(({ value, id }) => (
          <li key={id}>number: {value}</li>
        ))}
      </ul>
      <div className="create">
        Введите число:
        <input type="number" onChange={(e) => setValue(e.target.value)} value={value}/>
        <div
          className="button"
          onClick={onCreate}
        >
          Создать
        </div>
      </div>
    </>
  );
};
```

</details>

<details>
<summary><b>Реализовать хук <code>usePrevious</code></b></summary>

Подробнее [https://usehooks.com/useprevious](https://usehooks.com/useprevious)

Подробное условие задачи: [https://bigfrontend.dev/react/usePrevious](https://bigfrontend.dev/react/usePrevious)

```TypeScript
export function usePrevious<T>(value: T): T | undefined {
  // todo
}
```

### Ответ

```TypeScript
export function usePrevious<T>(value: T): T | undefined {
  const data = useRef<T | undefined>(undefined)

  useEffect(() => {
    data.current = value
  }, [value])

  return data.current
}
```

</details>

<details>
<summary><b>Реализовать хук <code>useDebounce</code></b></summary>

Подробное условие задачи: [https://bigfrontend.dev/react/useDebounce](https://bigfrontend.dev/react/useDebounce)

```TypeScript
export function useDebounce<T>(value: T, delay: number): T {
 // todo
}
```

### Ответ

```TypeScript
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value)
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebounced(value)
    }, delay)
    
    return () => clearTimeout(timer)
  }, [value, delay])
  
  return debounced
}
```

</details>

# Еще больше задач

- [https://bigfrontend.dev/react](https://bigfrontend.dev/react)

- [https://bigfrontend.dev/react-quiz](https://bigfrontend.dev/react-quiz)

- [https://dev.to/allenarduino/live-coding-react-interview-questions-2ndh](https://dev.to/allenarduino/live-coding-react-interview-questions-2ndh)

- [https://coderpad.io/interview-questions/react-interview-questions/#best-interview-practices-for-react-roles](https://coderpad.io/interview-questions/react-interview-questions/#best-interview-practices-for-react-roles)

# Еще больше вопросов

- [https://github.com/sudheerj/reactjs-interview-questions](https://github.com/sudheerj/reactjs-interview-questions)

- [https://daily.dev/blog/react-senior-interview-questions-for-2024](https://daily.dev/blog/react-senior-interview-questions-for-2024#reactmemo-and-purecomponent)

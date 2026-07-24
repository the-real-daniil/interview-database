# Полифилы методов

<details>
<summary><b>Как реализовать Function.prototype.call и apply? (Циан)</b></summary>

Задача сводится к одному трюку: при вызове функции «через точку» её `this` становится объектом слева от точки. Значит, чтобы подменить контекст, достаточно временно повесить функцию на нужный объект как свойство.

```js
Function.prototype.myCall = function (ctx, ...args) {
  ctx = ctx ?? globalThis;
  ctx = Object(ctx);

  const key = Symbol('fn');
  ctx[key] = this;

  const result = ctx[key](...args);

  delete ctx[key];
  return result;
};

// apply отличается только тем, что аргументы приходят массивом
Function.prototype.myApply = function (ctx, args = []) {
  return this.myCall(ctx, ...args);
};
```

Детали, которые отличают хороший ответ от поверхностного:

- `ctx ?? globalThis` — в нестрогом режиме `null` и `undefined` заменяются на глобальный объект. В строгом режиме нативный `call` этого не делает и оставляет `this` как есть.

- `Object(ctx)` — примитивы боксируются в объекты-обёртки. Без этого `fn.call(5)` упадёт при попытке записать свойство.

- `Symbol('fn')` вместо строкового ключа — гарантия, что мы не затрём существующее свойство объекта. Это тот случай, когда символ решает реальную задачу, а не приводится как учебный пример.

- `delete` в конце убирает временное свойство. Если функция бросит исключение, свойство останется — в продакшен-версии нужен `try/finally`.

</details>

<details>
<summary><b>Как реализовать Function.prototype.bind? (Циан)</b></summary>

`bind` возвращает новую функцию с намертво привязанным контекстом и, опционально, с заранее зафиксированными аргументами.

```js
Function.prototype.myBind = function (ctx, ...boundArgs) {
  const fn = this;

  return function (...args) {
    return fn.apply(ctx, [...boundArgs, ...args]);
  };
};
```

Это рабочая базовая версия, но у неё есть отличие от нативной, о котором стоит сказать вслух: при вызове результата через `new` нативный `bind` игнорирует привязанный контекст, потому что `new` создаёт свой объект. Полная реализация это учитывает:

```js
Function.prototype.myBind = function (ctx, ...boundArgs) {
  const fn = this;

  function bound(...args) {
    const isNew = this instanceof bound;
    return fn.apply(isNew ? this : ctx, [...boundArgs, ...args]);
  }

  bound.prototype = Object.create(fn.prototype || null);
  return bound;
};
```

Ещё два отличия от нативного `bind`, которые полезно упомянуть: у настоящего результата `length` уменьшается на число зафиксированных аргументов, а `name` получает префикс `bound `.

Типичный follow-up: «а можно ли перепривязать уже привязанную функцию?» Нет — повторный `bind` контекст не поменяет, потому что внешняя обёртка всё равно вызовет внутреннюю с исходным `ctx`.

---

### Дополнительные материалы

- [https://learn.javascript.ru/bind](https://learn.javascript.ru/bind)

</details>

<details>
<summary><b>Как реализовать Array.prototype.flat? (Циан)</b></summary>

```js
function flat(arr, depth = 1) {
  return arr.reduce(
    (acc, item) =>
      Array.isArray(item) && depth > 0
        ? acc.concat(flat(item, depth - 1))
        : acc.concat([item]),
    []
  );
}

flat([1, [2, [3, [4]]]]);           // [1, 2, [3, [4]]]
flat([1, [2, [3, [4]]]], Infinity); // [1, 2, 3, 4]
```

Ключевая деталь — `depth` уменьшается только при спуске внутрь вложенного массива, поэтому соседние элементы одного уровня разворачиваются одинаково.

**Ловушка, на которой валится большинство реализаций:** в ветке `else` элемент нужно обернуть в массив — `acc.concat([item])`, а не `acc.concat(item)`. `concat` разворачивает переданный массив на один уровень, поэтому наивная версия развернёт вложенный массив даже при исчерпанном `depth` и даст лишний уровень уплощения. Проверяется одним вызовом: `flat([1, [2, [3]]])` должен вернуть `[1, 2, [3]]`, а не `[1, 2, 3]`.

Рекурсия здесь ограничена глубиной вложенности, а не длиной массива, так что переполнение стека маловероятно. Но если хочется итеративного варианта, разворачивают через стек:

```js
function flatIterative(arr) {
  const stack = [...arr];
  const result = [];

  while (stack.length) {
    const item = stack.pop();
    if (Array.isArray(item)) {
      stack.push(...item);
    } else {
      result.push(item);
    }
  }

  return result.reverse();
}
```

Нативный `flat` дополнительно пропускает дырки в разреженных массивах: `[1, , 2].flat()` даёт `[1, 2]`. Реализация на `reduce` ведёт себя так же, потому что `reduce` пропускает пустые слоты — это приятное совпадение, а не осознанная обработка.

</details>

<details>
<summary><b>Как реализовать Promise.allSettled, race и any? (Циан)</b></summary>

Все три строятся на одной идее: обернуть каждый элемент в `Promise.resolve`, чтобы корректно принимать не-промисы, и подписаться на результат.

**`allSettled`** ждёт завершения всех и никогда не отклоняется — проще всего выразить через `Promise.all` с перехватом ошибок:

```js
function allSettled(promises) {
  return Promise.all(
    promises.map((p) =>
      Promise.resolve(p)
        .then((value) => ({ status: 'fulfilled', value }))
        .catch((reason) => ({ status: 'rejected', reason }))
    )
  );
}
```

**`race`** отдаёт результат первого завершившегося, неважно, успех это или ошибка:

```js
function race(promises) {
  return new Promise((resolve, reject) => {
    promises.forEach((p) => Promise.resolve(p).then(resolve, reject));
  });
}
```

Реализация выглядит подозрительно короткой, но она корректна: после первого вызова `resolve` или `reject` промис зафиксирован, и последующие вызовы игнорируются движком.

**`any`** — зеркальное отражение `all`: отдаёт первый успех, а отклоняется только если провалились все:

```js
function any(promises) {
  return new Promise((resolve, reject) => {
    const errors = [];
    let failed = 0;

    promises.forEach((p, i) =>
      Promise.resolve(p).then(resolve, (e) => {
        errors[i] = e;
        if (++failed === promises.length) {
          reject(new AggregateError(errors, 'All promises were rejected'));
        }
      })
    );
  });
}
```

**Что спросят следом:**

- Поведение на пустом массиве. `all` и `allSettled` резолвятся немедленно, `race` зависает навсегда, `any` сразу отклоняется с `AggregateError`. Все четыре случая нужно обработать явно.

- Отменяются ли остальные промисы после того, как `race` завершился? Нет. Промис нельзя отменить, запросы продолжат выполняться — для реальной отмены нужен `AbortController`.

| Метод | Когда резолвится | Когда реджектится |
| --- | --- | --- |
| `all` | все успешны | первая же ошибка |
| `allSettled` | все завершились | никогда |
| `race` | первый завершившийся успешен | первый завершившийся отклонён |
| `any` | первый успех | все отклонены (`AggregateError`) |

---

### Дополнительные материалы

- [https://learn.javascript.ru/promise-api](https://learn.javascript.ru/promise-api)

</details>

# Утилиты и декораторы

<details>
<summary><b>Как реализовать throttle и чем он отличается от debounce? (Циан)</b></summary>

**Throttle** пропускает вызов не чаще одного раза в заданный интервал. **Debounce** откладывает вызов до тех пор, пока обращения не прекратятся на заданное время.

```js
function throttle(fn, ms) {
  let waiting = false;
  let savedArgs = null;
  let savedThis = null;

  return function wrapper(...args) {
    if (waiting) {
      savedArgs = args;
      savedThis = this;
      return;
    }

    fn.apply(this, args);
    waiting = true;

    setTimeout(() => {
      waiting = false;

      if (savedArgs) {
        wrapper.apply(savedThis, savedArgs);
        savedArgs = savedThis = null;
      }
    }, ms);
  };
}
```

Эта версия вызывает функцию сразу на первом обращении и дополнительно повторяет последний пропущенный вызов в конце интервала — так последнее состояние не теряется.

| | throttle | debounce |
| --- | --- | --- |
| Когда срабатывает | равномерно, раз в `ms` | один раз после паузы |
| Что происходит при непрерывном потоке | вызовы идут регулярно | не вызывается вообще |
| Типичный сценарий | `scroll`, `resize`, отрисовка позиции | поисковый инпут, автосохранение |

Формулировка, которая хорошо заходит на интервью: throttle гарантирует частоту, debounce гарантирует тишину перед вызовом.

Уточняющий вопрос, который задают почти всегда: нужны ли вызовы на границах интервала. У lodash это флаги `leading` и `trailing` — стоит сказать, что ваша реализация соответствует `leading: true, trailing: true`.

</details>

<details>
<summary><b>Как реализовать memoize? (Циан)</b></summary>

Мемоизация кэширует результат по аргументам, чтобы не считать одно и то же дважды.

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (!cache.has(key)) {
      cache.set(key, fn.apply(this, args));
    }

    return cache.get(key);
  };
}
```

Реализация на четыре строки, но интересна она именно ограничениями — их и будут выспрашивать:

- **`JSON.stringify` как ключ ненадёжен.** Порядок полей в объекте влияет на строку, `undefined` и функции теряются, циклические ссылки роняют сериализацию. Для объектных аргументов правильнее `WeakMap`, для одного примитивного аргумента — сам аргумент как ключ.

- **Кэш растёт неограниченно.** Обычная `Map` держит ссылки на ключи и значения, поэтому долгоживущий мемоизированный обработчик становится утечкой памяти. Лечится ограничением размера по LRU или переходом на `WeakMap`, где ключ-объект не удерживается от сборки мусора.

- **Мемоизировать можно только чистые функции.** Если результат зависит от времени, случайности или внешнего состояния, кэш начнёт врать.

- **Ошибки не кэшируются.** Если `fn` бросит исключение, в кэш ничего не попадёт и следующий вызов повторит попытку. Иногда это желаемое поведение, иногда — источник лавины запросов.

Проверяют понимание обычно на `fib`: наивная рекурсия за `O(2ⁿ)` после мемоизации превращается в `O(n)`.

</details>

<details>
<summary><b>Как реализовать глубокое клонирование объекта? (Циан)</b></summary>

```js
function deepClone(obj, map = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (map.has(obj)) return map.get(obj);

  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);

  const copy = Array.isArray(obj) ? [] : {};
  map.set(obj, copy);

  for (const key of Reflect.ownKeys(obj)) {
    copy[key] = deepClone(obj[key], map);
  }

  return copy;
}
```

Главная деталь — `WeakMap` для обработки **циклических ссылок**. Запись `map.set(obj, copy)` идёт до рекурсивного обхода, поэтому когда вложенное поле сошлётся на родителя, мы вернём уже созданную копию вместо бесконечного спуска. Это первое, что проверяют в решении.

`WeakMap`, а не `Map`, потому что она не удерживает ключи от сборки мусора — после клонирования временная карта не мешает освободить исходные объекты.

Почему нельзя обойтись `JSON.parse(JSON.stringify(obj))`: теряются `undefined`, функции и символьные ключи, `Date` превращается в строку, `Map`, `Set` и `RegExp` схлопываются в пустые объекты, а циклическая ссылка бросает `TypeError`.

Стоит упомянуть встроенный **`structuredClone`** — он появился в браузерах и Node 17+, умеет циклы, `Map`, `Set`, `Date`, типизированные массивы. Функции и прототипы он не переносит и на них бросает `DataCloneError`. Знание того, что задача уже решена платформой, обычно засчитывают в плюс — но написать руками всё равно попросят.

Полная реализация дополнительно обрабатывает `Map`, `Set` и сохраняет прототип через `Object.create(Object.getPrototypeOf(obj))` — про это достаточно сказать словами, если время ограничено.

</details>

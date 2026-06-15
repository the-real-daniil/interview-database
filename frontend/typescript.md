Роадмап по TypeScript: [https://www.figma.com/board/Opy3MZ4qtsGdQL4yZbV9AT/TypeScript-Roadmap?node-id=15-434&t=8Gj2P5LdT20v5J3k-1](https://www.figma.com/board/Opy3MZ4qtsGdQL4yZbV9AT/TypeScript-Roadmap?node-id=15-434&t=8Gj2P5LdT20v5J3k-1)

Ролик про родмап: [https://youtu.be/2hs8IHfq9Xo?si=6-OD8STUEBI1EEge](https://youtu.be/2hs8IHfq9Xo?si=6-OD8STUEBI1EEge)

# Теория

<details>
<summary><b>Что такое TypeScript и чем он отличается от JavaScript?</b></summary>

TypeScript - это язык программирования, который является надмножеством JavaScript. Он добавляет статическую типизацию и некоторые другие возможности, которых нет в JavaScript.

Основное отличие TypeScript от JavaScript заключается в следующем:

1. **Статическая типизация:** TypeScript позволяет указывать типы переменных, параметров функций и возвращаемых значений. В JavaScript типы определяются динамически во время выполнения.

2. **Поддержка классов и модулей:** TypeScript поддерживает классы, интерфейсы и модули, что делает код более структурированным и удобным для организации больших проектов. JavaScript имеет ограниченную поддержку классов и модулей.

3. **Дополнительные возможности:** TypeScript предоставляет дополнительные возможности, такие как *перечисления (enums*), *кортежи (tuples)*, *типы-алиасы (type aliases*) и другие. Это позволяет разработчикам писать более выразительный и понятный код.

4. **Компиляция:** TypeScript код компилируется в JavaScript, поэтому он может быть исполнен в любом совместимом с JavaScript окружении. Компилятор TypeScript проверяет синтаксис и типы на этапе компиляции, что помогает выявить ошибки до запуска программы.

</details>

<details>
<summary><b>Какие есть недостатки у TypeScript?</b></summary>

- **Повышенная сложность и кривая обучения**: TypeScript добавляет статическую типизацию, что требует времени на освоение. Для начинающего разработчика это может быть сложным, особенно если ты привык к гибкости JavaScript.

- **Дополнительные настройки и конфигурация**: Чтобы работать с TypeScript, нужно настроить конфигурационные файлы (`tsconfig.json`), что требует времени и знаний. Кроме того, интеграция TypeScript в существующий проект на JavaScript может потребовать существенных изменений в настройках сборки (например, Webpack).

- **Сложность при миграции старого кода**: Если у тебя уже есть большой проект на JavaScript, переход на TypeScript может быть довольно трудоемким процессом. Нужно будет добавить типы для всех переменных и функций, что может занять много времени и усилий.

- **Увеличение времени компиляции**: TypeScript требует компиляции в JavaScript, что добавляет еще один шаг в процессе разработки и может увеличить время сборки, особенно в крупных проектах.

- **Ощущение ложной безопасности**: TypeScript дает ощущение безопасности за счет типизации, но не гарантирует отсутствие ошибок в рантайме. Некорректно написанный тип или ошибка в логике могут привести к проблемам, которые TypeScript не обнаружит.

- **Увеличение объема кода**: Добавление типов часто увеличивает количество строк кода, что может затруднить чтение и сопровождение проекта, особенно если типы сложные или если их использование не всегда оправдано.

</details>

<details>
<summary><b><code>any</code>, <code>unknown</code>, <code>never</code> - в чем различия?</b></summary>

`any` — это универсальный тип данных. Переменной этого типа можно присвоить абсолютно любое значение. TypeScript не совершает никаких проверок для переменных типа `any`, поэтому использовать рекомендуется только в крайних случаях. Если использовать его постоянно, то смысла в использовании TypeScript нет.

`unknown` — тип данных, указывающий, что неизвестно, какие именно данные будут хранится в переменной этого типа. Поэтому при использовании переменных этого типа необходимо пользоваться проверкой типов в рантайме через оператор `typeof`. Ключевое отличие `unknown` от `any` в том, что при работе с `unknown` TypeScript не отключает свои проверки.

`never` — тип данных, означающий, что значение никогда не будет получено.

Несколько примеров использования `never`:

1. Функция, которая всегда бросает ошибку

    ```TypeScript
    function fail(message: string): never {
      throw new Error(message);
    }
    ```

2. Бесконечный цикл

    ```TypeScript
    function infiniteLoop(): never {
      while (true) {
        // выполняется бесконечно
      }
    }
    ```

3. Проверка корректности switch (exhaustive check)

    ```TypeScript
    type Shape = 
      | { kind: "circle"; radius: number }
      | { kind: "square"; side: number };
    
    function area(shape: Shape): number {
      switch (shape.kind) {
        case "circle":
          return Math.PI * shape.radius ** 2;
        case "square":
          return shape.side * shape.side;
        default:
          // сюда мы попасть НЕ должны
          const _exhaustive: never = shape; 
          return _exhaustive;
      }
    }
    ```

---

### Дополнительные материалы

- [https://dev.to/ponikar/typescript-any-unknown-never-1idc](https://dev.to/ponikar/typescript-any-unknown-never-1idc)

- [https://dev.to/babak/exhaustive-type-checking-with-typescript-4l3f](https://dev.to/babak/exhaustive-type-checking-with-typescript-4l3f) — подробно про exhaustive check

</details>

<details>
<summary><b>Какие типы данных есть в TypeScript?</b></summary>

1. Стандартные js-типа: string, number, boolean, object, bigint, null, undefined, symbol

2. Специальные типы: any, unknown, never, void, array, tuple и другие

3. Утилитарные типы: Omit, Record, Pick и другие

4. Кастомные типы: разработчик может объявлять свои типы данных через типы и интерфейсы

---

### Дополнительные материалы

- [https://www.typescriptlang.org/docs/handbook/2/everyday-types.html](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

- [https://www.typescriptlang.org/docs/handbook/2/objects.html](https://www.typescriptlang.org/docs/handbook/2/objects.html)

</details>

<details>
<summary><b>В чем разница между типами и интерфейсами?</b></summary>

1. Типы могут быть примитивами или функциями. Интерфейс же задает структуру объекта

2. Класс может имплементировать интерфейс

3. Интерфейсы может наследоваться от другого интерфейса

4. Типы можно объединять друг с другом

5. Declaration merging (объединение деклараций) — это механика интерфейсов в TypeScript, при которой компилятор автоматически объединяет несколько отдельных объявлений с одним и тем же именем в единое определение

---

### Дополнительные материалы

- [https://habr.com/ru/sandbox/186102/](https://habr.com/ru/sandbox/186102/)

</details>

<details>
<summary><b>Что такое Omit?</b></summary>

Тип `Omit` в Typescript предназначен для создания нового типа путём исключения заданных полей из существующего типа.

В качестве первого аргумента `Omit` ожидает тип данных, из которого будут исключены поля, переданные в качестве второго аргумента типа.

---

### Дополнительные материалы

- [https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys)

</details>

<details>
<summary><b>Что такое Pick?</b></summary>

`Pick` — это утилитарный тип, которая позволяет создавать новый тип, выбирая определённые свойства из существующего типа.

---

### Дополнительные материалы

- [https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys](https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys)

</details>

<details>
<summary><b>Что такое Partial?</b></summary>

`Partial` — утилитарный тип, который на основе переданного типа позволяет создать тип, у которого все поля будут опциональными.

---

### Дополнительные материалы

- [https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype)

</details>

<details>
<summary><b>Что такое type guards?</b></summary>

Type guards — это конструкции в TypeScript, которые позволяют определить более точный тип переменной в пределах условного блока кода.

---

### Дополнительные материалы

- [https://medium.com/@eqbits/что-такое-type-guards-в-typescript-24834d2b4f](https://medium.com/@eqbits/что-такое-type-guards-в-typescript-24834d2b4f)

- [https://claritydev.net/blog/understanding-implementing-type-guards-typescript](https://claritydev.net/blog/understanding-implementing-type-guards-typescript)

- [https://youtu.be/P2Ny05sAYoY?si=DBrqhJF8brxNZ9B0](https://youtu.be/P2Ny05sAYoY?si=DBrqhJF8brxNZ9B0)

</details>

<details>
<summary><b>Что такое дженерики?</b></summary>

Дженерики позволяют создавать функции, классы и интерфейсы, которые могут работать с различными типами, передаваемыми в качестве параметров. Это делает код более гибким и повторно используемым, поскольку ты можешь определить, как будет вести себя компонент или функция с различными типами данных, не привязываясь к конкретному типу.

Пример:

```TypeScript
interface ApiResponse<T> {
    data: T;
    error: string | null;
}

const response: ApiResponse<number> = {
    data: 200,
    error: null
};

const stringResponse: ApiResponse<string> = {
    data: 'Success',
    error: null
};
```

---

### Дополнительные материалы

- [https://youtu.be/k16Hgi753XQ?si=YbAzm-A3vkYzIxgL](https://youtu.be/k16Hgi753XQ?si=YbAzm-A3vkYzIxgL)

- [https://www.typescriptlang.org/docs/handbook/2/generics.html](https://www.typescriptlang.org/docs/handbook/2/generics.html)

</details>

<details>
<summary><b>Для чего используется оператор keyof?</b></summary>

Оператор `keyof` в TypeScript используется для получения объединения строковых литералов, представляющих имена свойств объекта. Он позволяет получить тип, который содержит все возможные имена свойств объекта.

Синтаксис использования оператора `keyof` выглядит следующим образом:

```TypeScript
typeKeys = keyofSomeType;

typePerson = {
  name: string;
  age: number;
  email: string;
};

typePersonKeys = keyof Person; // 'name' | 'age' | 'email'
```

</details>

<details>
<summary><b>Что такое <code>infer</code>?</b></summary>

Ключевое слово `infer` позволяет позволяет вывести тип данных из другого типа

Например:

```TypeScript
type MyReturnType<T> = T extends (...args: []any) => infer R ? R : never
```

---

### Дополнительные материалы

- [https://youtu.be/RKrkQxFYJ1c?si=iYycF8gyJMT4FIWJ&t=1574](https://youtu.be/RKrkQxFYJ1c?si=iYycF8gyJMT4FIWJ&t=1574)

- [https://purpleschool.ru/knowledge-base/article/infer](https://purpleschool.ru/knowledge-base/article/infer)

- [https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types)

</details>

<details>
<summary><b>Как реализуются принципы ООП в TypeScript?</b></summary>

1. Инкапсуляция: TypeScript поддерживает инкапсуляцию путем использования модификаторов доступа, таких как `public`, `private` и `protected`, для ограничения доступа к членам класса.

2. Наследование: TypeScript поддерживает наследование, позволяя классам наследовать свойства и методы от других классов с помощью ключевого слова `extends`.

3. Полиморфизм: TypeScript поддерживает полиморфизм, что означает, что объекты одного класса могут быть использованы вместо объектов другого класса, если они наследуются от одного и того же базового класса или реализуют один и тот же интерфейс.

4. Абстракция: TypeScript позволяет создавать абстрактные классы и методы с помощью ключевых слов `abstract`. Абстрактные классы не могут быть инстанциированы напрямую, а их методы должны быть реализованы в производных классах.

---

### Дополнительные материалы

- [https://www.typescriptlang.org/docs/handbook/2/everyday-types.html](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

- [https://www.typescriptlang.org/docs/handbook/2/objects.html](https://www.typescriptlang.org/docs/handbook/2/objects.html)

</details>

<details>
<summary><b>В чем разница между интерфейсами и абстрактными классами?</b></summary>

Интерфейс - это контракт, которому должен соответствовать имплементирующий его класс.

Абстрактный класс - это одновременно и контракт, и часть реализации класса

---

### Дополнительные материалы

- [https://stackoverflow.com/questions/50110844/what-is-the-difference-between-interface-and-abstract-class-in-typescript](https://stackoverflow.com/questions/50110844/what-is-the-difference-between-interface-and-abstract-class-in-typescript)

</details>

<details>
<summary><b>Для чего используются <code>.d.ts</code> файлы?</b></summary>

`.d.ts` файлы используются для описания типов отдельно от реализации. Например, в файле `index.js` код на javascript, а в файле `index.d.ts` описание типов для этого файла без самой реализации.

Такой подход удобен при написании библиотек, чтобы дать возможность потребителям использовать библиотеку, как в JavaScript проектах (в них `.d.ts` файлы можно не использовать и даже не скачивать), так и в TypeScript проектах (где типизация важна).

---

### Дополнительные материалы

- [https://stackoverflow.com/questions/50463990/what-are-d-ts-files-for](https://stackoverflow.com/questions/50463990/what-are-d-ts-files-for)

- [https://race-timo.gitbook.io/typescript/dts](https://race-timo.gitbook.io/typescript/dts)

</details>

<details>
<summary><b>Что такое геттеры и сеттеры?</b></summary>

В TypeScript геттеры (getters) и сеттеры (setters) - это специальные методы у классов, которые позволяют получать и устанавливать значения свойств класса соответственно. Они предоставляют контроль над доступом к свойствам и позволяют выполнять дополнительные действия при получении или установке значений.

Геттеры и сеттеры определяются с использованием ключевых слов `get` и `set` перед именем метода.

</details>

<details>
<summary><b>Что такое перегрузка функций?</b></summary>

Перегрузка функций — это возможность определить несколько версий одной функции, каждая из которых принимает свой набор параметров.

---

### Дополнительные материалы

- [https://ru.hexlet.io/courses/typescript-basics/lessons/function-overloads/theory_unit](https://ru.hexlet.io/courses/typescript-basics/lessons/function-overloads/theory_unit)

</details>

<details>
<summary><b>Поддерживает ли TypeScript перегрузку функций?</b></summary>

Да, TypeScript поддерживает перегрузку функций. Компилятор TypeScript будет использовать соответствующую версию функции в зависимости от переданных аргументов.

Пример:

```TypeScript
function combine(a:string, b:string):string;
function combine(a:number, b:number):number;
function combine(a:any, b:any):any {
return a + b;
}

let result1 = combine("Hello", "World");// result1 будет иметь тип string
let result2 = combine(5, 10);// result2 будет иметь тип number
```

</details>

<details>
<summary><b>В чем разница между типами объединения и пересечения?</b></summary>

В TypeScript есть два основных способа для комбинирования типов: тип объединения (union type) и тип пересечения (intersection type).

**Тип объединения (Union Type):**

- Обозначается символом `|`.

- Позволяет объединять несколько типов в один.

- Значение переменной с типом объединения может быть одним из указанных типов.

- Применяется, когда значение может быть одним из нескольких типов.

Пример использования типа объединения:

```TypeScript
let value: string | number;
value = "Hello"; // Допустимо
value = 10; // Допустимо
value = true; // Ошибка компиляции
```

**Тип пересечения (Intersection Type):**

- Обозначается символом `&`.

- Позволяет создавать новый тип, который объединяет свойства и методы из нескольких типов.

- Значение переменной с типом пересечения должно удовлетворять всем указанным типам.

- Применяется, когда значение должно удовлетворять нескольким типам одновременно.

Пример использования типа пересечения:

```TypeScript
type Person = {
  name: string;
  age: number;
};

type Employee = {
  company: string;
  position: string;
};

type EmployeePerson = Person & Employee;

let employee: EmployeePerson = {
  name: "John",
  age: 30,
  company: "ABC Corp",
  position: "Manager"
};
```

В целом, тип объединения используется, когда значение может быть одним из нескольких типов, а тип пересечения используется, когда значение должно удовлетворять нескольким типам одновременно и иметь их свойства и методы.

</details>

<details>
<summary><b>В чем разница между <code>extends</code> и <code>implements</code> в TypeScript?</b></summary>

В TypeScript ключевые слова `extends` и `implements` используются для определения отношений между классами и интерфейсами.

`extends`:

- Используется для создания наследования между классами и интерфейсами.

- Класс может расширять только один другой класс.

- Позволяет наследовать свойства и методы базового класса.

- Можно переопределить методы базового класса в производном классе.

- Можно добавить новые свойства и методы в производный класс.

`implements`:

- Используется для определения, что класс или объект должен реализовывать определенный интерфейс.

- Класс может реализовывать несколько интерфейсов.

- Обязывает класс реализовать все методы и свойства, указанные в интерфейсе.

- Не поддерживает наследование свойств и методов.

</details>

<details>
<summary><b>Что такое Type Assertions в TypeScript?</b></summary>

Type Assertions в TypeScript позволяют программисту явно указывать компилятору тип данных переменной, когда компилятор не может определить тип автоматически. Утверждения типа позволяют программисту переопределить тип переменной и сообщить компилятору, что программа знает, какой тип данных она ожидает.

С помощью угловых скобок (`<тип>`):

```TypeScript
let someValue:any = "Hello, world!";
let strLength:number = (<string>someValue).length;
```

С помощью ключевого слова `as`:

```TypeScript
let someValue:any = "Hello, world!";
let strLength:number = (someValue as string).length;
```

Важно отметить, что утверждения типа не изменяют фактический тип переменной во время выполнения. Они используются только на этапе компиляции для подсказки компилятору о типе данных переменной. Поэтому необходимо быть осторожным при использовании утверждений типа и убедиться, что тип, указанный в утверждении, соответствует фактическому типу данных переменной.

</details>

<details>
<summary><b>Какие модификаторы доступа к свойствам и методам класса поддерживает TypeScript?</b></summary>

- `public`: Это является модификатором доступа по умолчанию и означает, что член класса доступен из любого места в коде.

- `private`: Этот модификатор ограничивает доступ к члену класса только внутри самого класса. Члены, объявленные как `private`, не могут быть доступны извне класса.

- `protected`: Этот модификатор позволяет доступ к члену класса внутри самого класса и его подклассов. Члены, объявленные как `protected`, не могут быть доступны извне класса или его экземпляров.

</details>

<details>
<summary><b>Что такое декораторы?</b></summary>

Декораторы в TypeScript - это специальные функции, которые могут использоваться для изменения классов, методов, свойств и параметров функций во время их объявления. Они предоставляют возможность добавлять метаданные и расширять функциональность существующих элементов кода без изменения их исходного определения.

Декораторы обычно применяются с использованием символа @ перед объявлением элемента кода, к которому они применяются. Они могут быть применены к классам, методам, свойствам и параметрам функций.

Декоратор класса:

```TypeScript
function logClass(target: any) {
  console.log('Class decorator');
}

@logClass
class MyClass {
  // ...
}
```

Декоратор метода:

```TypeScript
function logMethod(target: any, key: string, descriptor: PropertyDescriptor) {
  console.log('Method decorator');
}

class MyClass {
  @logMethod
  myMethod() {
    // ...
  }
}
```

Декоратор свойства:

```TypeScript
function logProperty(target:any, key:string) {
  console.log('Property decorator');
}

class MyClass {
  @logProperty
  myProperty:string;
}
```

Декоратор параметра функции:

```TypeScript
function logParameter(target: any, key: string, index: number) {
  console.log('Parameter decorator');
}

class MyClass {
  myMethod(@logParameter param:string) {
    // ...
  }
}
```

---

### Дополнительные материалы

- [https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators)

</details>

# Задачи

Задачи по TypeScript встречаются редко, поэтому уделять им слишком много времени на старте может быть пустой тратой времени. Но если интересно разобраться в тонкостях работы TS и закрепить на практике, то welcome!

<details>
<summary><b>Напишите собственную реализацию <code>Omit</code> (IT-ONE)</b></summary>

### Ответ

```TypeScript
type myOmit<Source, Omitted> = {[key in keyof Source as key extends Omitted ? never : key]: Source[key]}
```

</details>

<details>
<summary><b>Чему будут равны <code>IsStrings</code>, <code>IsXY</code>, <code>IsXZ</code>? (Яндекс)</b></summary>

```TypeScript
type IsSameType<A,B> = A extends B
                       ? B extends A
                         ? 'yes'
                         : 'no'
                       : 'no'
 
type X = string | number;
type Y = object;
type Z = string | number;
 
type IsStrings = IsSameType<string, string>; // ?
 
type IsXY = IsSameType<X,Y>; // ?
 
type IsXZ = IsSameType<X,Z>; // ?
```

### Ответ

```TypeScript
type IsSameType<A,B> = A extends B
                       ? B extends A
                         ? 'yes'
                         : 'no'
                       : 'no'
 
type X = string | number;
type Y = object;
type Z = string | number;
 
type IsStrings = IsSameType<string, string>; // yes
 
type IsXY = IsSameType<X,Y>; // no
 
type IsXZ = IsSameType<X,Z>; // yes | no
```

</details>

<details>
<summary><b>Напишите собственный тип, который делает поля обязательными</b></summary>

```TypeScript
type A = {attr?: string}
type B = NonOptional<A> // {attr: string}


type NonOptional = ?
```

### Ответ

```TypeScript
type A = {attr?: string}
type B = NonOptional<A> // {attr: string}


type NonOptional<T> = {
  [K in keyof T]-?: T[K]
}
```

</details>

<details>
<summary><b>Типизировать функцию <code>debounce</code></b></summary>

```TypeScript
function debounce(fn, delay) {
  let timerId
  
  return (...args) => {
    if (timerId) {
      clearTimeout(timerId)
    }

    timerId = setTimeout(() => {
      fn(...args)
    }, delay)
  }
}

const f = (a) => console.log(a)

const debouncedF = debounce(f, 1000)

debouncedF(1)
debouncedF(2)
debouncedF(3) // выведется только 3
```

### Ответ

```TypeScript
function debounce<Args extends unknown[]>(
  fn: (...args: Args) => void,
  delay: number
): (...args: Args) => void {
  let timerId: number | undefined

  return (...args: Args) => {
    if (timerId) {
      clearTimeout(timerId)
    }

    timerId = setTimeout(() => {
      fn(...args)
    }, delay)
  }
}
```

</details>

# Еще больше задач

- [https://typescript-exercises.github.io/#exercise=1&file=%2Findex.ts](https://typescript-exercises.github.io/#exercise=1&file=%2Findex.ts)

- [https://typehero.dev/](https://typehero.dev/)

# Еще больше вопросов

- [https://www.turing.com/interview-questions/typescript](https://www.turing.com/interview-questions/typescript)

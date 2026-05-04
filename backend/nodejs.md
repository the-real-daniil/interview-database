<details>
<summary><b>Что такое event loop в Node.js?</b></summary>

### Зачем нужен Event Loop

Node.js — однопоточный, но при этом неблокирующий. Event loop — это механизм, который позволяет выполнять асинхронные операции (I/O, таймеры, сеть) без создания новых потоков для каждой задачи. Он делегирует тяжёлую работу ядру ОС или libuv thread pool, а сам ждёт колбэков.

---

### Архитектура: из чего состоит

```
Node.js Process
├── V8 Engine          — выполняет JS (call stack, heap)
├── libuv              — event loop + thread pool (4 потока по умолчанию)
│   ├── Event Loop     — 6 фаз + 2 очереди вне фаз
│   └── Thread Pool    — fs, crypto, dns.lookup, некоторые zlib
└── Node.js Bindings   — мост между JS и C++
```

---

### 6 фаз Event Loop

```
   ┌──────────────────────────┐
┌─>│  1. timers               │  setTimeout, setInterval (порог истёк)
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │  2. pending callbacks    │  I/O ошибки из прошлой итерации
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │  3. idle, prepare        │  внутреннее использование libuv
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │  4. poll                 │  ← основная фаза: получение новых I/O событий
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
│  │  5. check                │  setImmediate
│  └──────────┬───────────────┘
│  ┌──────────▼───────────────┐
└──│  6. close callbacks      │  socket.on('close'), etc.
   └──────────────────────────┘
```

**Детально по ключевым фазам:**

**1. timers** — выполняет колбэки `setTimeout`/`setInterval`, у которых порог уже истёк. Важно: это _минимальная_ задержка, а не гарантированная.

**2. pending callbacks** — I/O колбэки, отложенные на следующую итерацию (например, TCP-ошибки типа `ECONNREFUSED`).

**3. idle/prepare** — служебная фаза libuv, недоступна из JS.

**4. poll** — самая важная фаза:

- Если очередь poll не пуста — синхронно выполняет колбэки по очереди
- Если пуста — ждёт новых I/O событий (блокирует цикл, если нет `setImmediate` и таймеров)

**5. check** — `setImmediate`. Всегда выполняется после poll фазы, даже если poll заблокирован.

**6. close callbacks** — `'close'` события (например, `socket.destroy()`).

---

### Очереди вне фаз (выполняются между каждой фазой)

Это критично и часто путают:

```
После каждой фазы, перед переходом в следующую:
1. process.nextTick queue   — выполняется полностью
2. Promise microtask queue  — выполняется полностью
```

**Приоритет:** `nextTick` > `Promise microtasks` > следующая фаза event loop.

---

### Практический пример с порядком вывода

```js
setTimeout(() => console.log("1. setTimeout"), 0);
setImmediate(() => console.log("2. setImmediate"));
Promise.resolve().then(() => console.log("3. Promise"));
process.nextTick(() => console.log("4. nextTick"));
console.log("5. sync");
```

**Вывод:**

```
5. sync          ← call stack
4. nextTick      ← nextTick queue (между фазами)
3. Promise       ← microtask queue
1. setTimeout    ← timers фаза (почти всегда раньше setImmediate)
2. setImmediate  ← check фаза
```

> ⚠️ `setTimeout(fn, 0)` vs `setImmediate` — порядок **не детерминирован** в корне event loop. Но внутри I/O колбэка `setImmediate` **всегда** раньше `setTimeout`.

---

### Thread Pool (libuv)

Не весь async-код идёт через event loop напрямую. Тяжёлые операции уходят в thread pool:

| Через thread pool         | Через ядро ОС (async) |
| ------------------------- | --------------------- |
| `fs.*` (большинство)      | TCP/UDP сокеты        |
| `crypto` (pbkdf2, scrypt) | Pipes                 |
| `dns.lookup`              | `fs` на некоторых ОС  |
| `zlib` (частично)         | Child processes       |

По умолчанию 4 потока. Меняется через `UV_THREADPOOL_SIZE=8`.

---

### Частые ошибки и ловушки

1. **Блокировка event loop** — синхронный тяжёлый код (`JSON.parse` большого объекта, цикл по миллиону элементов) останавливает всё.
2. **Рекурсивный `nextTick`** — бесконечная рекурсия через `process.nextTick` заблокирует переход к I/O навсегда.
3. **Ожидание от `setTimeout(fn, 0)` точности** — минимальная задержка в браузере 4ms, в Node.js ~1ms, но реально зависит от нагрузки.

### Допоплнительные материалы

- [https://youtu.be/zDlg64fsQow?si=06Lfui_OgKZ_IPyP](https://youtu.be/zDlg64fsQow?si=06Lfui_OgKZ_IPyP)

- [https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick)

- [https://nodejs.org/api/timers.html](https://nodejs.org/api/timers.html)

</details>

<details>
<summary><b><code>setTimeout(fn, 0)</code> vs. <code>setImmediate</code></b></summary>

Если коротко:

> `setImmediate` — "выполни в этой итерации цикла, в check-фазе".  
> `setTimeout(fn, 0)` — "выполни в следующей итерации, если таймер истёк".

---

### Где они живут в цикле

```
[timers]  ← setTimeout попадает сюда
   ↓
[poll]    ← I/O колбэки
   ↓
[check]   ← setImmediate попадает сюда
   ↓
--- конец итерации ---
[timers]  ← новая итерация
```

`setImmediate` всегда в **текущей** итерации, в check-фазе.  
`setTimeout(fn, 0)` — в timers-фазе, которая либо в **текущей**, либо в **следующей** итерации — зависит от того, истёк ли 1ms.

---

### Почему setTimeout(fn, 0) — это не "0ms"

Минимальный порог в Node.js — **1ms**. Если передать 0 — Node.js сам преобразует его в 1.

Когда event loop доходит до timers-фазы, он смотрит на системные часы: прошла ли 1ms с момента вызова `setTimeout`? Если нет — пропускает. Если да — выполняет.

---

### Три случая

**1. Корень модуля — недетерминировано**

```js
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
```

Зависит от того, сколько заняла инициализация. Может быть любой порядок.

**2. Внутри I/O колбэка — всегда setImmediate первым**

```js
fs.readFile("file.txt", () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
});
```

```
immediate
timeout
```

Потому что I/O колбэк выполняется в poll-фазе. После неё — сразу check (setImmediate). До timers надо ждать следующей итерации.

**3. Внутри CPU-тяжёлого синхронного кода — скорее всего timeout первым**

```js
// долгая синхронная работа перед стартом event loop
heavySync(); // занимает 10ms
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
```

К моменту входа в timers-фазу уже прошло много времени → таймер истёк → timeout первым.

---

### Итог в одной таблице

|                         | `setTimeout(fn, 0)`                   | `setImmediate`                     |
| ----------------------- | ------------------------------------- | ---------------------------------- |
| Фаза                    | timers                                | check                              |
| Когда                   | следующая итерация (если 1ms истекла) | текущая итерация, после poll       |
| Гарантия порядка        | нет (в корне)                         | да (внутри I/O)                    |
| Практическое применение | "отложить чуть-чуть"                  | "после I/O, до следующих таймеров" |

**Правило:** если нужно "сразу после текущего I/O" — используй `setImmediate`. `setTimeout(fn, 0)` — не синоним "немедленно".

</details>

<details>
<summary><b>В чем разница между <code>process.nextTick()</code> и <code>setImmediate()</code>?</b></summary>

### Ключеваое отличие

`process.nextTick()` выполняет callback до перехода event loop к следующей фазе, а `setImmediate()` — в фазе check на следующей итерации цикла. `nextTick` имеет более высокий приоритет и при злоупотреблении может «задушить» I/O. `setImmediate` безопаснее, когда нужно отложить работу и не блокировать прогресс цикла. Очередь `nextTick` очищается раньше обычных фаз event loop. `setImmediate` запускается после poll в check-фазе. Избыточный `nextTick` задерживает таймеры и I/O-колбэки. Для отложенного выполнения чаще лучше `setImmediate`.

### Пример

```JavaScript
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
Promise.resolve().then(() => console.log('Promise'));

console.log('sync');

/*
Вывод:

sync
nextTick
Promise
setImmediate
*/
```

Важно избегать ошибки: рекурсивно вызывать `nextTick`.

```JavaScript
// ❌ ОПАСНО — Event Loop никогда не перейдёт к следующей фазе
function infinite() {
  process.nextTick(infinite);
}
infinite();

// Никакой I/O, никакие таймеры, никакие HTTP запросы не обработаются
```

`setImmediate` так не сломает — он выполняется **в рамках фазы**, и после неё Event Loop движется дальше.

```JavaScript
// ✅ Безопасно — другие события обрабатываются между итерациями
function infinite() {
  setImmediate(infinite);
}
```

### Дополнительные материалы

- [https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick](https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick)

- [https://nodejs.org/en/learn/asynchronous-work/understanding-setimmediate](https://nodejs.org/en/learn/asynchronous-work/understanding-setimmediate)

</details>

<details>
<summary><b>Когда использовать <code>worker_threads</code> и <code>cluster</code>?</b></summary>

### Корень проблемы

Event Loop однопоточный. Есть два разных типа задач, которые его убивают:

```JavaScript
Тип 1: Много одновременных запросов     → cluster
Тип 2: Тяжёлые вычисления в одном запросе → worker_threads
```

### Cluster — несколько процессов

```JavaScript
         ┌─────────────┐
         │   Master    │  ← слушает порт, раздаёт запросы
         └──────┬──────┘
    ┌───────────┼───────────┐
    ↓           ↓           ↓
┌───────┐  ┌───────┐  ┌───────┐
│Worker │  │Worker │  │Worker │  ← отдельные процессы Node.js
│ PID 1 │  │ PID 2 │  │ PID 3 │  ← каждый со своим Event Loop
└───────┘  └───────┘  └───────┘
  CPU 0      CPU 1      CPU 2
```

Каждый worker — **полноценный Node.js процесс** с отдельной памятью.javascript

```JavaScript
import cluster from 'cluster';
import os from 'os';
import http from 'http';

if (cluster.isPrimary) {
  const cpus = os.cpus().length; // например, 8

  for (let i = 0; i < cpus; i++) {
    cluster.fork(); // создаём 8 процессов
  }

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} умер — перезапускаем`);
    cluster.fork(); // автовосстановление
  });

} else {
  // Каждый worker слушает ОДИН порт — ОС сама распределяет запросы
  http.createServer((req, res) => {
    res.end(`Ответил worker ${process.pid}`);
  }).listen(3000);
}
```

### Worker Threads — несколько потоков

```JavaScript
┌──────────────────────────────────────┐
│          Один процесс Node.js        │
│                                      │
│  Main Thread                         │
│  ┌─────────────┐                     │
│  │ Event Loop  │──┬── Worker 1 🧵    │
│  └─────────────┘  ├── Worker 2 🧵    │
│                   └── Worker 3 🧵    │
│                                      │
│  Общая память через SharedArrayBuffer│
└──────────────────────────────────────┘
```

```JavaScript
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

if (isMainThread) {
  // Главный поток — раздаём задачи
  const worker = new Worker('./heavy-task.js', {
    workerData: { numbers: [1, 2, 3, ...Array(1_000_000)] }
  });

  worker.on('message', (result) => {
    console.log('Результат:', result); // главный поток не блокировался
  });

} else {
  // Worker поток — делаем тяжёлую работу
  const result = workerData.numbers.reduce((a, b) => a + b, 0);
  parentPort.postMessage(result);
}
```

---

### Главное отличие — изоляция памяти

|                | cluster                                   | worker_threads                             |
| -------------- | ----------------------------------------- | ------------------------------------------ |
| Память         | Изолирована (копия)                       | Может быть общей (SharedArrayBuffer)       |
| Коммуникация   | `process.send()` (медленно, сериализация) | `postMessage` + SharedArrayBuffer (быстро) |
| Падение одного | Не роняет остальных                       | Может уронить процесс                      |
| Overhead       | Высокий (новый процесс)                   | Низкий (новый поток)                       |

---

### Когда что выбирать — конкретно

#### Используй `cluster` когда:

- HTTP сервер под нагрузкой — много параллельных запросов

- Каждый запрос лёгкий (CRUD, проксирование)

- Нужна изоляция — падение worker не роняет других

- Stateless архитектура (сессии в Redis, не в памяти)

Пример: API gateway, REST API, BFF

#### Используй `worker_threads` когда:

Тяжёлые вычисления внутри одного запроса:

- парсинг огромных JSON/CSV

- шифрование / хэширование

- обработка изображений

- ML inference

- сложные математические расчёты

### Дополнительные материалы

- [https://tproger.ru/problems/what-is-the-difference-between-threads-and-processes](https://tproger.ru/problems/what-is-the-difference-between-threads-and-processes)

- [https://nodejs.org/api/worker_threads.html](https://nodejs.org/api/worker_threads.html)

- [https://nodejs.org/api/cluster.html](https://nodejs.org/api/cluster.html)

</details>
<details>
<summary><b>Что такое потоки (<code>streams</code>) в NodeJS? Для чего они нужны?</b></summary>

### Зачем они нужны — проблема без стримов

Представь, нужно прочитать файл 2GB и отправить его клиенту:

```js
// ❌ Без стримов
const data = fs.readFileSync("video.mp4"); // весь файл в памяти → 2GB RAM
res.end(data);
```

Проблема: весь файл загружается в память **сразу**. При 100 одновременных запросах — 200GB RAM.

```js
// ✅ Со стримами
fs.createReadStream("video.mp4").pipe(res); // в памяти ~64KB в любой момент
```

Стримы читают и передают данные **чанками** (по умолчанию 64KB), не загружая всё сразу.

---

### Что такое стрим

Стрим — это **абстракция над потоком данных**, который обрабатывается по частям. Все стримы — `EventEmitter`.

Node.js использует стримы внутри везде: `http.req`, `http.res`, `fs.createReadStream`, `process.stdin`, `crypto`, `zlib` — всё это стримы.

---

### 4 типа стримов

**1. Readable** — источник данных

```js
const readable = fs.createReadStream("file.txt");
readable.on("data", (chunk) => console.log(chunk));
readable.on("end", () => console.log("done"));
```

Примеры: `fs.createReadStream`, `http.IncomingMessage`, `process.stdin`

---

**2. Writable** — приёмник данных

```js
const writable = fs.createWriteStream("output.txt");
writable.write("hello");
writable.end();
```

Примеры: `fs.createWriteStream`, `http.ServerResponse`, `process.stdout`

---

**3. Duplex** — читает и пишет **независимо**

```js
// TCP сокет — можно и читать и писать одновременно
const net = require("net");
const socket = net.createConnection(3000);
socket.write("ping");
socket.on("data", (data) => console.log(data));
```

Примеры: `net.Socket`, `tls.TLSSocket`

---

**4. Transform** — читает, **преобразует**, пишет

```js
const { createGzip } = require("zlib");
fs.createReadStream("file.txt")
  .pipe(createGzip()) // Transform: сжимает на лету
  .pipe(fs.createWriteStream("file.txt.gz"));
```

Примеры: `zlib.createGzip()`, `crypto.createCipheriv()`, `csv-parser`

> Разница Duplex vs Transform: в Duplex read и write независимы. В Transform — output является **результатом обработки** input.

---

### pipe — главный инструмент композиции

```js
readable.pipe(transform1).pipe(transform2).pipe(writable);
```

`pipe` автоматически:

- передаёт чанки из одного стрима в другой
- управляет **backpressure** (см. ниже)
- останавливает readable, если writable не успевает

---

### Backpressure — ключевая концепция

Что если читаем быстро, а пишем медленно?

```
Readable (сеть, 1Gbps) → Writable (диск, 100Mbps)
```

Без контроля — данные накапливаются в памяти. Backpressure — механизм обратного давления:

```js
const readable = fs.createReadStream("big.file");
const writable = fs.createWriteStream("output.file");

readable.on("data", (chunk) => {
  const ok = writable.write(chunk);
  if (!ok) {
    readable.pause(); // ← стоп, writable переполнен
    writable.once("drain", () => {
      readable.resume(); // ← writable освободился, продолжаем
    });
  }
});
```

`pipe` делает это **автоматически**. Поэтому ручной код с `on('data')` без backpressure — антипаттерн.

---

### Два режима Readable стрима

|              | Flowing                                | Paused                           |
| ------------ | -------------------------------------- | -------------------------------- |
| Как работает | данные льются сами, через `on('data')` | данные тянешь сам через `read()` |
| Управление   | `resume()` / `pause()`                 | `read(n)`                        |
| Backpressure | ручное                                 | встроенное                       |

Современный способ — **async iteration**, он сам управляет paused режимом:

```js
async function process() {
  for await (const chunk of fs.createReadStream("file.txt")) {
    console.log(chunk);
  }
}
```

---

### Итог

| Задача                | Решение                             |
| --------------------- | ----------------------------------- |
| Читать большой файл   | `fs.createReadStream`               |
| Сжать на лету         | `.pipe(zlib.createGzip())`          |
| Стриминг видео        | `readStream.pipe(res)`              |
| Парсинг CSV построчно | `readStream.pipe(csvParser)`        |
| Шифрование файла      | `.pipe(crypto.createCipheriv(...))` |

**Главная ценность стримов:** обрабатывать данные любого размера с **константным потреблением памяти**.

### Дополнительные материалы

- https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/
- [Исходный код стримов](https://github.com/nodejs/node/tree/main/lib/internal/streams)
- https://habr.com/ru/companies/timeweb/articles/854330/
- https://nodejs.org/api/stream.html#stream

</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>
<details>
<summary><b>Что такое event loop в Node.js?</b></summary>
</details>

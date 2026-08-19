# SQL-задачи

Практических задач на SQL на собеседованиях обычно две категории: **написать запрос** по словесному описанию отчёта и **ускорить уже написанный запрос**. Ниже — типичные задачи уровня middle с разбором.

### Где тренироваться

Основной тренажёр, который стоит пройти целиком перед собеседованиями:

- [Интерактивный тренажёр по SQL (Stepik)](https://stepik.org/course/63054/promo) — бесплатный курс с практикой прямо в браузере: от `SELECT` до оконных функций и индексов. Это база, которой хватает на 90% вопросов по SQL на интервью.

Дополнительно полезно порешать [pgexercises.com](https://pgexercises.com/) и почитать вывод `EXPLAIN ANALYZE` на своих реальных таблицах — на собеседовании чаще спрашивают не синтаксис, а умение объяснить план запроса.

### Схема данных для задач ниже

```sql
CREATE TABLE users (
    id          bigserial PRIMARY KEY,
    email       text NOT NULL,
    name        text NOT NULL,
    created_at  timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE orders (
    id          bigserial PRIMARY KEY,
    user_id     bigint NOT NULL REFERENCES users (id),
    status      text NOT NULL, -- 'new' | 'paid' | 'shipped' | 'cancelled'
    total       numeric(12, 2) NOT NULL,
    created_at  timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
    id          bigserial PRIMARY KEY,
    order_id    bigint NOT NULL REFERENCES orders (id),
    product_id  bigint NOT NULL,
    quantity    int NOT NULL,
    price       numeric(12, 2) NOT NULL
);
```

---

# Написать SQL-запрос

<details>
<summary><b>Задача: вывести по каждому пользователю его 3 самых дорогих оплаченных заказа (id заказа, сумма, позиция в рейтинге). Пользователей без оплаченных заказов в выдаче быть не должно.</b></summary>

Задача на **оконные функции**: нужно ранжировать строки внутри группы, а не схлопывать группу в одну строку, поэтому `GROUP BY` не подходит.

```sql
SELECT user_id, id AS order_id, total, rn
FROM (
    SELECT
        o.user_id,
        o.id,
        o.total,
        ROW_NUMBER() OVER (PARTITION BY o.user_id ORDER BY o.total DESC, o.id) AS rn
    FROM orders o
    WHERE o.status = 'paid'
) t
WHERE rn <= 3
ORDER BY user_id, rn;
```

---

### Почему именно так

- `PARTITION BY user_id` — окно «сбрасывается» на каждом новом пользователе, нумерация идёт внутри пользователя.
- Фильтровать по `rn <= 3` в том же `SELECT` нельзя: оконные функции вычисляются **после** `WHERE`, поэтому нужен подзапрос или CTE.
- Пользователи без оплаченных заказов отсеиваются сами — у них просто нет строк в `orders` с `status = 'paid'`.
- `ORDER BY o.total DESC, o.id` — второй ключ нужен, чтобы порядок был детерминированным при одинаковых суммах.

### `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`

| Функция | Поведение при равных значениях | Результат для 100, 100, 90 |
|---|---|---|
| `ROW_NUMBER()` | всегда уникальные номера, порядок произвольный | 1, 2, 3 |
| `RANK()` | одинаковый ранг, следующий номер пропускается | 1, 1, 3 |
| `DENSE_RANK()` | одинаковый ранг, пропусков нет | 1, 1, 2 |

Если по условию «топ-3» означает «все заказы с тремя наибольшими суммами», нужен `DENSE_RANK()`, а не `ROW_NUMBER()`. Это ровно тот момент, который интервьюер обычно и проверяет — уточните требование вслух.

### Альтернатива без оконных функций

```sql
SELECT u.id, o.id, o.total
FROM users u
CROSS JOIN LATERAL (
    SELECT id, total
    FROM orders
    WHERE user_id = u.id AND status = 'paid'
    ORDER BY total DESC
    LIMIT 3
) o;
```

`LATERAL` часто оказывается быстрее на больших таблицах: при наличии индекса `(user_id, status, total DESC)` для каждого пользователя читаются ровно 3 строки индекса, а не вся партиция.

### Типичные уточняющие вопросы

- **Чем `LATERAL` отличается от обычного подзапроса?** В `LATERAL` можно ссылаться на колонки внешней таблицы (`u.id`), обычный подзапрос в `FROM` этого не умеет.
- **Что будет, если у пользователя меньше 3 заказов?** Вернутся все, что есть — фильтр `rn <= 3` не требует ровно трёх строк.
- **Как добавить пользователей без заказов?** Заменить `CROSS JOIN LATERAL` на `LEFT JOIN LATERAL ... ON true`.

</details>

<details>
<summary><b>Задача: найти всех пользователей, которые зарегистрировались более 30 дней назад и при этом не сделали ни одного заказа за последние 30 дней.</b></summary>

Это классический **anti-join** — «строки одной таблицы, для которых нет пары в другой».

```sql
SELECT u.id, u.email, u.created_at
FROM users u
WHERE u.created_at < now() - interval '30 days'
  AND NOT EXISTS (
      SELECT 1
      FROM orders o
      WHERE o.user_id = u.id
        AND o.created_at >= now() - interval '30 days'
  );
```

---

### Три способа написать anti-join и чем они отличаются

| Способ | Поведение с `NULL` | Производительность |
|---|---|---|
| `NOT EXISTS` | безопасен | планировщик строит `Anti Join` (hash/merge) — обычно лучший вариант |
| `LEFT JOIN ... WHERE o.id IS NULL` | безопасен | тоже `Anti Join`, но запрос менее читаем и легко ошибиться, поставив условие в `WHERE` вместо `ON` |
| `NOT IN (SELECT user_id ...)` | **опасен**: если в подзапросе есть хоть один `NULL`, результат всегда пустой | часто хуже, планировщику сложнее |

```sql
-- эквивалент через LEFT JOIN: условие по датам обязано быть в ON, а не в WHERE
SELECT u.id, u.email
FROM users u
LEFT JOIN orders o
       ON o.user_id = u.id
      AND o.created_at >= now() - interval '30 days'
WHERE u.created_at < now() - interval '30 days'
  AND o.id IS NULL;
```

Если перенести `o.created_at >= ...` в `WHERE`, `LEFT JOIN` вырождается в `INNER JOIN` и запрос вернёт пустой результат — это самая частая ошибка в этой задаче.

### Почему `NOT IN` ломается на `NULL`

`x NOT IN (1, 2, NULL)` — это `x <> 1 AND x <> 2 AND x <> NULL`. Последнее сравнение даёт `NULL`, а не `TRUE`, поэтому всё выражение никогда не становится истинным. `NOT EXISTS` работает по строкам и такой проблемы не имеет.

### Что спросят дальше

- **Какой индекс нужен?** `CREATE INDEX ON orders (user_id, created_at)` — по нему проверка существования заказа делается одним Index Only Scan.
- **Как посчитать «спящих» пользователей быстро на десятках миллионов строк?** Считать инкрементально: держать в `users` денормализованное поле `last_order_at`, обновляемое триггером или самим приложением.
- **Как найти обратное — активных пользователей?** Заменить `NOT EXISTS` на `EXISTS` (semi-join), а не на `JOIN`: `JOIN` продублирует пользователя по числу заказов.

</details>

<details>
<summary><b>Задача: построить отчёт по месяцам за текущий год: выручка за месяц, накопительный итог с начала года и рост в процентах к предыдущему месяцу. Месяцы без заказов должны присутствовать в отчёте с нулями.</b></summary>

Нужны три вещи: агрегация по периоду (`date_trunc`), окна (`SUM OVER`, `LAG`) и «календарь» (`generate_series`), чтобы не потерять пустые месяцы.

```sql
WITH months AS (
    SELECT generate_series(
        date_trunc('year', now()),
        date_trunc('month', now()),
        interval '1 month'
    ) AS month
),
revenue AS (
    SELECT date_trunc('month', o.created_at) AS month,
           SUM(o.total) AS amount
    FROM orders o
    WHERE o.status = 'paid'
      AND o.created_at >= date_trunc('year', now())
    GROUP BY 1
)
SELECT
    m.month::date,
    COALESCE(r.amount, 0) AS revenue,
    SUM(COALESCE(r.amount, 0)) OVER (ORDER BY m.month) AS running_total,
    ROUND(
        100.0 * (COALESCE(r.amount, 0) - LAG(COALESCE(r.amount, 0)) OVER (ORDER BY m.month))
        / NULLIF(LAG(COALESCE(r.amount, 0)) OVER (ORDER BY m.month), 0),
        2
    ) AS growth_pct
FROM months m
LEFT JOIN revenue r ON r.month = m.month
ORDER BY m.month;
```

---

### Разбор ключевых мест

- **`generate_series`** генерирует ряд месяцев; без него месяцы без единого заказа просто исчезнут из отчёта — на собеседовании это главный проверяемый момент.
- **`SUM(...) OVER (ORDER BY month)`** — накопительный итог. Указание `ORDER BY` внутри окна неявно задаёт рамку `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, то есть «сумма всего от начала до текущей строки».
- **`LAG(x) OVER (ORDER BY month)`** — значение предыдущей строки, из него считается рост.
- **`NULLIF(..., 0)`** защищает от деления на ноль: если в предыдущем месяце выручки не было, `growth_pct` станет `NULL`, а не упадёт с ошибкой.
- `date_trunc('month', created_at)` в `GROUP BY` — нормально, а вот в `WHERE` такую конструкцию писать нельзя, иначе индекс по `created_at` не сработает.

### Про часовые пояса

`date_trunc('month', created_at)` для `timestamptz` считается в таймзоне сессии. Для стабильного отчёта таймзону задают явно:

```sql
date_trunc('month', o.created_at AT TIME ZONE 'Europe/Moscow')
```

### Типичные уточняющие вопросы

- **Чем `RANGE` отличается от `ROWS` в рамке окна?** `ROWS` считает физические строки, `RANGE` — строки с одинаковым значением ключа сортировки. При дублях дат результаты будут разными.
- **Как посчитать скользящее среднее за 3 месяца?** `AVG(amount) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)`.
- **Почему в отчёте появились дробные копейки?** `numeric` не округляется сам — итог нужно приводить `ROUND(x, 2)`.

</details>

<details>
<summary><b>Задача: в таблице users накопились дубли по email. Нужно оставить по одному пользователю на email (самого раннего по created_at), остальные удалить, и после этого не дать дублям появиться снова.</b></summary>

Сначала находим лишние строки оконной функцией, потом удаляем их одним запросом, потом ставим уникальный индекс.

```sql
-- 1. посмотреть, что именно будет удалено
WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (
               PARTITION BY lower(email)
               ORDER BY created_at, id
           ) AS rn
    FROM users
)
SELECT * FROM users WHERE id IN (SELECT id FROM ranked WHERE rn > 1);

-- 2. удалить
WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (PARTITION BY lower(email) ORDER BY created_at, id) AS rn
    FROM users
)
DELETE FROM users
WHERE id IN (SELECT id FROM ranked WHERE rn > 1);

-- 3. запретить дубли на будущее
CREATE UNIQUE INDEX CONCURRENTLY idx_users_email_unique ON users (lower(email));
```

---

### На что смотрит интервьюер

- **`lower(email)`** — дубли почти всегда отличаются регистром. Уникальный индекс тоже должен строиться по `lower(email)`, иначе `Ivan@mail.ru` и `ivan@mail.ru` снова разойдутся.
- **Порядок в `ORDER BY`** определяет, какая строка выживет. «Самый ранний» — `created_at ASC`; для «самого свежего» — `created_at DESC`. Второй ключ `id` нужен для детерминизма.
- **Сначала `SELECT`, потом `DELETE`.** Показать, что вы проверяете выборку перед удалением — половина оценки за эту задачу.
- **`CONCURRENTLY`** строит индекс, не блокируя запись в таблицу. Важно на проде; работает только вне транзакции и требует проверки статуса индекса (`indisvalid`) после сбоя.

### Что делать с внешними ключами

Если на `users.id` ссылаются `orders`, простой `DELETE` упадёт по FK. Строки-дубли сначала «переклеивают» на выжившую запись:

```sql
WITH ranked AS (
    SELECT id, lower(email) AS key,
           first_value(id) OVER (PARTITION BY lower(email) ORDER BY created_at, id) AS keep_id
    FROM users
)
UPDATE orders o
SET user_id = r.keep_id
FROM ranked r
WHERE o.user_id = r.id AND r.id <> r.keep_id;
```

### Альтернатива для PostgreSQL без оконных функций

```sql
DELETE FROM users a
USING users b
WHERE lower(a.email) = lower(b.email)
  AND (a.created_at, a.id) > (b.created_at, b.id);
```

Работает, но на больших таблицах это `O(n²)`-подобный self-join — вариант с `ROW_NUMBER()` предпочтительнее.

### Типичные уточняющие вопросы

- **Как удалять миллионы дублей на живой базе?** Пакетами по 10–50 тыс. строк в отдельных транзакциях, иначе один `DELETE` держит блокировки и раздувает WAL.
- **Чем `UNIQUE CONSTRAINT` отличается от `UNIQUE INDEX`?** Constraint — логическое ограничение (виден в `information_schema`, на него можно сослаться из FK), под капотом создаётся тот же индекс; но constraint нельзя построить по выражению `lower(email)` — там нужен именно индекс.
- **Как избежать дублей при вставке?** `INSERT ... ON CONFLICT (lower(email)) DO NOTHING/DO UPDATE`.

</details>

<details>
<summary><b>Задача: посчитать, какая доля пользователей сделала повторный заказ в течение 30 дней после первого — в разрезе месяца первого заказа (когортный retention).</b></summary>

Задача на CTE и агрегацию с условием: сначала находим первый заказ каждого пользователя, потом проверяем наличие второго в окне 30 дней.

```sql
WITH first_orders AS (
    SELECT DISTINCT ON (user_id)
           user_id,
           id AS first_order_id,
           created_at AS first_order_at
    FROM orders
    WHERE status = 'paid'
    ORDER BY user_id, created_at, id
),
retained AS (
    SELECT f.user_id,
           date_trunc('month', f.first_order_at) AS cohort,
           EXISTS (
               SELECT 1
               FROM orders o
               WHERE o.user_id = f.user_id
                 AND o.status = 'paid'
                 AND o.id <> f.first_order_id
                 AND o.created_at BETWEEN f.first_order_at
                                      AND f.first_order_at + interval '30 days'
           ) AS is_retained
    FROM first_orders f
)
SELECT cohort::date,
       COUNT(*) AS users,
       COUNT(*) FILTER (WHERE is_retained) AS retained_users,
       ROUND(100.0 * COUNT(*) FILTER (WHERE is_retained) / COUNT(*), 2) AS retention_pct
FROM retained
GROUP BY cohort
ORDER BY cohort;
```

---

### Ключевые приёмы

- **`DISTINCT ON (user_id)`** — PostgreSQL-специфичный способ взять «первую строку в группе»: обязателен `ORDER BY`, начинающийся с тех же колонок. Портируемый аналог — `ROW_NUMBER() ... rn = 1`.
- **`COUNT(*) FILTER (WHERE cond)`** — стандартный SQL-способ условной агрегации, читается лучше, чем `SUM(CASE WHEN cond THEN 1 ELSE 0 END)` (результат тот же).
- **`o.id <> f.first_order_id`** — без этого условия первый заказ засчитается сам себе как повторный, и retention будет 100%. Это главная ловушка задачи.
- `100.0 * ...` — если умножать на целое `100`, в целочисленном делении получится 0; `100.0` переводит выражение в `numeric`.

### Как это масштабируется

На больших объёмах узкое место — `EXISTS` по каждому пользователю. Помогает индекс `(user_id, status, created_at)` и материализация когорт в отдельную таблицу-витрину, пересчитываемую по расписанию.

### Типичные уточняющие вопросы

- **Чем `HAVING` отличается от `WHERE`?** `WHERE` фильтрует строки до агрегации, `HAVING` — уже посчитанные группы.
- **Как получить полную когортную матрицу (retention на 30/60/90 дней)?** Добавить несколько выражений `COUNT(*) FILTER (...)` с разными интервалами или сделать `generate_series` по периодам и `JOIN`.
- **Почему нельзя просто `COUNT(orders) > 1`?** Так посчитаются пользователи с двумя заказами вообще, без ограничения по окну в 30 дней.

</details>

---

# Оптимизация запросов

<details>
<summary><b>Задача: запрос-отчёт по заказам за сутки работает 8 секунд на таблице в 50 млн строк. Нужно объяснить, почему он медленный, и ускорить его. Запрос: SELECT * FROM orders WHERE DATE(created_at) = '2024-05-01' AND status IN ('paid','shipped') ORDER BY total DESC LIMIT 100;</b></summary>

Запрос медленный по трём причинам: **функция над колонкой** убивает индекс по `created_at`, `SELECT *` тянет лишние данные, а `ORDER BY total DESC` заставляет сортировать весь суточный срез.

```sql
-- было
SELECT *
FROM orders
WHERE DATE(created_at) = '2024-05-01'
  AND status IN ('paid', 'shipped')
ORDER BY total DESC
LIMIT 100;

-- стало
SELECT id, user_id, status, total, created_at
FROM orders
WHERE created_at >= '2024-05-01'
  AND created_at <  '2024-05-02'
  AND status IN ('paid', 'shipped')
ORDER BY total DESC
LIMIT 100;

CREATE INDEX CONCURRENTLY idx_orders_status_total_created
    ON orders (status, total DESC, created_at);
```

---

### Что именно чинится

| Проблема | Почему это плохо | Исправление |
|---|---|---|
| `DATE(created_at) = ...` | предикат не sargable: значение колонки нужно вычислить для каждой строки → `Seq Scan` по 50 млн строк | заменить на полуинтервал `>= '2024-05-01' AND < '2024-05-02'` |
| `SELECT *` | ORM/клиент тянет лишние колонки и детоастит TOAST-поля (large object storage PostgreSQL для «толстых» значений), растёт объём I/O и трафик | перечислить только нужные колонки — сам по себе `Index Only Scan` это не даёт, для него нужно, чтобы все запрошенные колонки физически лежали в индексе (либо через `INCLUDE`) |
| `ORDER BY total DESC` | сортировка суточного среза (сотни тысяч строк) в памяти/на диске | поставить `total` в индексе перед диапазонным `created_at` — сортировка исчезает из плана |
| Нет индекса под фильтр | планировщику нечего выбрать | составной индекс `(status, total DESC, created_at)` |

### Порядок колонок в составном индексе

Правило **ESR (Equality, Sort, Range)**: сначала колонки под **равенство**, потом под **сортировку**, и только потом — под **диапазон**. `status` сравнивается через `IN` (набор равенств), `total` — сортировка, `created_at` — диапазон. Причина порядка Sort перед Range: у B-tree-индекса `(status, total DESC, created_at)` записи внутри группы `status` уже отсортированы по `total`, и сканирование в этом порядке позволяет полностью избежать узла `Sort` в плане — диапазон по `created_at` при этом проверяется как `Filter` по ходу сканирования, а не как отдельное индексное условие. Если поставить `created_at` перед `total` (то есть диапазон перед сортировкой), глобальная отсортированность по `total` внутри индекса потеряется — диапазон захватывает много разных значений `created_at`, и планировщик всё равно вставит узел `Sort`.

### Как это доказать на собеседовании

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

Что смотреть в плане:

- `Seq Scan` вместо `Index Scan` — предикат не sargable либо индекса нет;
- большой разрыв между `rows=` (оценка) и `actual rows=` — устарела статистика, нужен `ANALYZE`;
- узлы `Sort` с `Sort Method: external merge Disk` — сортировка не влезла в `work_mem`;
- `Buffers: shared read=...` — сколько страниц реально прочитано с диска.

### Альтернативы, если индекс не помогает

- **Частичный индекс**, если отчёт всегда по одним и тем же статусам: `CREATE INDEX ... ON orders (total DESC, created_at) WHERE status IN ('paid','shipped')` — `status` уже зафиксирован условием `WHERE` индекса, поэтому по ESR первой идёт колонка сортировки (`total`), а диапазон `created_at` — за ней; индекс при этом меньше и точнее.
- **Партиционирование по времени** (`PARTITION BY RANGE (created_at)`) — запрос за сутки читает одну секцию, старые секции легко отцеплять.
- **Витрина/материализованное представление** для тяжёлых агрегатов, обновляемая по расписанию.

### Типичные уточняющие вопросы

- **Что такое sargable-предикат?** Условие, которое СУБД может свести к поиску по диапазону индекса: колонка стоит «голая» слева от оператора.
- **Почему `created_at::date = ...` тоже плохо, а функциональный индекс это чинит?** `CREATE INDEX ON orders ((created_at::date))` сделает предикат индексируемым, но создаст второй индекс — обычно проще переписать условие.
- **Помогает ли `LIMIT 100` сам по себе?** Только если план умеет отдать первые строки сразу (`Index Scan` в нужном порядке). При наличии `Sort` СУБД всё равно обязана отсортировать весь набор.

</details>

<details>
<summary><b>Задача: список пользователей с количеством их заказов открывается за 6 секунд. Приложение делает 1 запрос за пользователями и по запросу на каждого за счётчиком заказов (N+1). Как переписать это на SQL и что ещё оптимизировать?</b></summary>

N+1 лечится одним агрегирующим запросом. Важно только не сделать при этом вторую классическую ошибку — не «размножить» строки джойном.

```sql
-- было: 1 + N запросов
-- SELECT * FROM users LIMIT 50;
-- SELECT count(*) FROM orders WHERE user_id = $1;  -- ×50

-- стало: один запрос
SELECT u.id,
       u.email,
       COALESCE(o.orders_count, 0) AS orders_count,
       COALESCE(o.total_sum, 0)    AS total_sum
FROM users u
LEFT JOIN (
    SELECT user_id,
           COUNT(*)   AS orders_count,
           SUM(total) AS total_sum
    FROM orders
    WHERE status = 'paid'
    GROUP BY user_id
) o ON o.user_id = u.id
ORDER BY u.id
LIMIT 50;
```

---

### Почему агрегат вынесен в подзапрос, а не сделан «в лоб»

Наивный вариант `LEFT JOIN orders ... GROUP BY u.id` тоже работает, но:

- при нескольких агрегируемых таблицах (например, заказы **и** отзывы) джойн даёт декартово произведение и `COUNT` завышается — это самая частая ошибка;
- `LIMIT 50` в таком запросе применяется после агрегации всей таблицы, то есть считаются все пользователи, а не 50.

Агрегация в отдельном подзапросе (или в `LATERAL`) обе проблемы снимает:

```sql
SELECT u.id, u.email, o.orders_count
FROM users u
LEFT JOIN LATERAL (
    SELECT COUNT(*) AS orders_count
    FROM orders
    WHERE user_id = u.id AND status = 'paid'
) o ON true
ORDER BY u.id
LIMIT 50;
```

Здесь агрегат считается **только для 50 отобранных пользователей** — на больших таблицах это разница в порядки.

### Что ещё стоит сделать

- **Индекс** `CREATE INDEX ON orders (user_id, status)` — счётчик берётся из индекса без похода в heap.
- **Пагинация по ключу** вместо `OFFSET`: `WHERE u.id > $last_id ORDER BY u.id LIMIT 50`. `OFFSET 100000` заставляет СУБД прочитать и выбросить 100 тыс. строк.
- **Денормализация**, если счётчик показывается на каждой странице: колонка `users.orders_count`, обновляемая триггером или в той же транзакции, что и создание заказа. Точный `COUNT(*)` в PostgreSQL всегда стоит полного прохода по строкам из-за MVCC.
- **Приблизительный счётчик** для «примерно N результатов»: `SELECT reltuples FROM pg_class WHERE relname = 'orders'`.

### Типичные уточняющие вопросы

- **Чем `COUNT(*)` отличается от `COUNT(column)`?** `COUNT(column)` не считает строки, где колонка `NULL`. `COUNT(*)` считает все строки.
- **Почему в PostgreSQL `COUNT(*)` медленный, а в MySQL/InnoDB иногда быстрый?** Из-за MVCC видимость каждой версии строки нужно проверять; `Index Only Scan` помогает лишь если visibility map актуальна (после `VACUUM`).
- **Как понять, что в приложении есть N+1?** По логам БД: сотни одинаковых запросов с разными параметрами на один HTTP-запрос. В ORM лечится eager loading (`include`/`relations`/`JOIN FETCH`).
- **Когда JOIN хуже двух запросов?** Когда джойн «разносит» много строк (много-ко-многим) и по сети едет дублированный набор — иногда дешевле забрать данные двумя запросами и склеить в приложении.

</details>

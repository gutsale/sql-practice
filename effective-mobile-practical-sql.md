# Effective Mobile Practical SQL Задачи
# Задача 1

Объяснить запрос

Есть 2 таблицы:

1) names:

| id | name         |
|----|--------------|
| 1  | John Doe     |
| 2  | Jane Doe     |
| 3  | Alice Jones  |
| 4  | Bobby Louis  |
| 5  | Lisa Romero  |

2) results:

| id | event            | winner_id |
|----|------------------|-----------|
| 1  | 100 meter dash   | 2         |
| 2  | 500 meter dash   | 3         |
| 3  | cross-country    | 2         |
| 4  | triathalon       | NULL      |

Что будет результатом следующего запроса

```sql
SELECT *
FROM names
WHERE id NOT IN (
    SELECT winner_id
    FROM results
);
```

## Разбор

Подзапрос:

```sql
SELECT winner_id
FROM results;
```

вернёт:

```text
2
3
2
NULL
```

Тогда условие превращается в:

```sql
WHERE id NOT IN (2, 3, NULL)
```

В SQL любое сравнение с `NULL` возвращает не `TRUE` и не `FALSE`, а `UNKNOWN`.

Поэтому выражение:

```sql
1 NOT IN (2, 3, NULL)
```

логически превращается в:

```sql
1 <> 2
AND 1 <> 3
AND 1 <> NULL
```

Последнее сравнение возвращает `UNKNOWN`.

Оператор `WHERE` возвращает только строки, где условие равно `TRUE`.

## Итог

Запрос не вернёт ни одной строки.

## Корректный вариант

```sql
SELECT *
FROM names n
WHERE NOT EXISTS (
    SELECT 1
    FROM results r
    WHERE r.winner_id = n.id
);
```
 Или если хотим оставить NOT IN:

```sql
 SELECT *
FROM names
WHERE id NOT IN (
    SELECT winner_id
    FROM results
    WHERE winner_id IS NOT NULL
);
```

---

# Задача 2

Напишите запрос SQL, используя Union All (не Union), который использует WHERE,, чтобы устранить дубликаты. Почему вам может это понадобиться?

## Решение

```sql
SELECT winner_id AS id
FROM results
WHERE winner_id IS NOT NULL

UNION ALL

SELECT id
FROM names
WHERE NOT EXISTS (
    SELECT 1
    FROM results
    WHERE results.winner_id = names.id
);
```

## Разбор

`UNION ALL` объединяет результаты запросов, но не удаляет дубликаты автоматически.

В первой части запроса выбираются все `winner_id` из таблицы `results`:

```sql
SELECT winner_id AS id
FROM results
WHERE winner_id IS NOT NULL
```

Во второй части через `NOT EXISTS` добавляются только те `id` из таблицы `names`, которых ещё нет среди победителей:

```sql
SELECT id
FROM names n
WHERE NOT EXISTS (
    SELECT 1
    FROM results r
    WHERE r.winner_id = n.id
);
```

Таким образом дубликаты устраняются вручную с помощью `WHERE NOT EXISTS`.

## Почему это может понадобиться

`UNION` автоматически удаляет дубликаты, но для этого выполняет для этого дополнительную сортировку, поэтому работает медленнее.

`UNION ALL` работает быстрее, потому что просто объединяет результаты без проверки повторов.

Если мы сами контролируем удаление дубликатов через `WHERE`, можно:
- повысить производительность;
- более гибко управлять логикой фильтрации;
- избежать лишней обработки данных.

---

# Задача 3

Написать SELECT запрос.

Напишите SQL-запрос, чтобы получить третью максимальную зарплату сотрудника из таблицы employees.

| worker_name | salary |
|---|---|
| A | 24000 |
| C | 34000 |
| D | 55000 |
| E | 75000 |
| F | 21000 |
| G | 40000 |
| H | 50000 |

## Решение

```sql
SELECT DISTINCT salary
FROM worker
ORDER BY salary DESC
LIMIT 1 OFFSET 2;
```

## Разбор

Сначала зарплаты сортируются по убыванию:

```text
75000
55000
50000
40000
34000
24000
21000
```

После сортировки:
- `OFFSET 2` пропускает первые две строки;
- `LIMIT 1` оставляет в результирующем наборе только одну следующую строку.

Таким образом получается третья максимальная зарплата.

## Почему используется DISTINCT

Если в таблице есть одинаковые зарплаты, без `DISTINCT` результат может быть некорректным.

Например:

```text
75000
55000
55000
50000
```

Без `DISTINCT` третьей строкой снова будет `55000`.

`DISTINCT` удаляет дубликаты зарплат перед сортировкой.

## Результат

```text
50000
```

---



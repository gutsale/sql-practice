# SQL-EX DML Задачи
1. Добавить в таблицу PC следующую модель:
- code: 20
- model: 2111
- speed: 950
- ram: 512
- hd: 60
- cd: 52x
- price: 1100

```sql
INSERT INTO pc (code, model, speed, ram, hd, cd, price)
VALUES (20, 2111, 950, 512, 60, '52x', 1100);
```

---

2. Добавить в таблицу Product следующие продукты производителя Z:
принтер модели 4003, ПК модели 4001 и блокнот модели 4002

```sql
INSERT INTO product (maker, model, type)
VALUES
    ('z', 4003, 'printer'),
    ('z', 4001, 'pc'),
    ('z', 4002, 'laptop');
```

---

3. Добавить в таблицу PC модель 4444 с кодом 22, имеющую скорость процессора 1200 и цену 1350.

Отсутствующие характеристики должны быть восполнены значениями по умолчанию, принятыми для соответствующих столбцов.

```sql
INSERT INTO pc (code, model, speed, price)
VALUES (22, 4444, 1200, 1350);
```

---

4. Для каждого из тех дней, когда на пункте с ежедневной отчетностью был расход, но не было прихода,
добавить в таблицу income_o за этот день для соответствующего пункта запись с inc, равным округленному до сотен значению out.

```sql
INSERT INTO Income_o (point, date, inc)
SELECT
    o.point,
    o.date,
    ROUND(o.out, -2) AS inc
FROM Outcome_o o
LEFT JOIN Income_o i
    ON o.point = i.point
   AND o.date = i.date
WHERE i.point IS NULL;
```

---

5. Удалить из таблицы PC компьютеры, имеющие минимальный объем диска или памяти.

```sql
DELETE FROM pc
WHERE hd = (
    SELECT MIN(hd)
    FROM pc
)
OR ram = (
    SELECT MIN(ram)
    FROM pc
);
```

---

6. Удалить все блокноты, выпускаемые производителями, которые не выпускают принтеры.

```sql
DELETE FROM laptop
WHERE model IN (
    SELECT model
    FROM product
    WHERE type = 'Laptop'
      AND maker IN (
          SELECT DISTINCT maker
          FROM product
          WHERE type = 'Laptop'

          EXCEPT

          SELECT DISTINCT maker
          FROM product
          WHERE type = 'Printer'
      )
);
```

---

7. Для кораблей, которые принимали участие всего в двух сражениях, поменять результаты (result) этих сражений.

Например, если в битве 1 результат был "ok", а в битве 2 - "sunk", то должно стать "ok" для битвы 2 и "sunk" - для битвы 1.

```sql
UPDATE o
SET result = CASE
    WHEN o.battle = t.battle1 THEN t.result2
    WHEN o.battle = t.battle2 THEN t.result1
END
FROM outcomes o
JOIN (
    SELECT
        o1.ship,
        o1.battle AS battle1,
        o1.result AS result1,
        o2.battle AS battle2,
        o2.result AS result2
    FROM outcomes o1
    JOIN outcomes o2
        ON o1.ship = o2.ship
       AND o1.battle < o2.battle
    WHERE o1.ship IN (
        SELECT ship
        FROM outcomes
        GROUP BY ship
        HAVING COUNT(*) = 2
    )
) t
    ON o.ship = t.ship
WHERE o.battle IN (t.battle1, t.battle2);
```

---

8. Удалите из таблицы Ships все корабли, потопленные в сражениях.

```sql
DELETE ships
FROM ships
LEFT JOIN outcomes
    ON ships.name = outcomes.ship
WHERE result = 'sunk';
```

---

9. Перенести все концевые пробелы, имеющиеся в названии каждого сражения в таблице Battles, в начало названия.

```sql
UPDATE battles
SET name = REPLICATE(' ', DATALENGTH(name) - DATALENGTH(RTRIM(name)))
           + RTRIM(name);
```

---

10. Добавить в таблицу PC те модели ПК из Product, которые отсутствуют в таблице PC.

При этом модели должны иметь следующие характеристики:

1. Код равен номеру модели плюс максимальный код, который был до вставки.

2. Скорость, объем памяти и диска, а также скорость CD должны иметь максимальные характеристики среди всех имеющихся в таблице PC.

3. Цена должна быть средней среди всех ПК, имевшихся в таблице PC до вставки.

```sql
INSERT INTO pc (code, model, speed, ram, hd, cd, price)
SELECT
    model + (SELECT MAX(code) FROM pc) AS new_code,
    model,
    (SELECT MAX(speed) FROM pc) AS max_speed,
    (SELECT MAX(ram) FROM pc) AS max_ram,
    (SELECT MAX(hd) FROM pc) AS max_hd,
    (
        SELECT CAST(MAX(CAST(REPLACE(cd, 'x', '') AS INT)) AS VARCHAR) + 'x'
        FROM pc
    ) AS max_cd,
    (SELECT AVG(price) FROM pc) AS avg_price
FROM product
WHERE type = 'PC'
  AND model NOT IN (
      SELECT model
      FROM pc
  );
```

---

11. Для каждой группы блокнотов с одинаковым номером модели добавить запись в таблицу PC со следующими характеристиками:
код: минимальный код блокнота в группе +20;
модель: номер модели блокнота +1000;
скорость: максимальная скорость блокнота в группе;
ram: максимальный объем ram блокнота в группе *2;
hd: максимальный объем hd блокнота в группе *2;
cd: cd c максимальной скоростью среди всех ПК;
цена: максимальная цена блокнота в группе, уменьшенная в 1,5 раза

```sql
INSERT INTO pc (code, model, speed, ram, hd, cd, price)
SELECT
    MIN(code) + 20 AS new_code,
    model + 1000 AS new_model,
    MAX(speed) AS max_speed,
    MAX(ram) * 2 AS new_ram,
    MAX(hd) * 2 AS new_hd,
    (
        SELECT TOP 1 cd
        FROM pc
        ORDER BY CAST(REPLACE(cd, 'x', '') AS INT) DESC
    ) AS max_cd,
    MAX(price) / 1.5 AS new_price
FROM laptop
GROUP BY model;
```

---

12. Добавить отсутствующие в таблице Ships головные корабли из Outcomes. Годом спуска на воду считать средний округленный до целого числа год по кораблям страны добавляемого корабля. Если средний год неизвестен, запись не вносить.

```sql
INSERT INTO ships (name, class, launched)
SELECT DISTINCT
    o.ship,
    o.ship,
    CAST((
        SELECT ROUND(AVG(CAST(s.launched AS FLOAT)), 0)
        FROM ships s
        JOIN classes c2
            ON s.class = c2.class
        WHERE c2.country = c.country
          AND s.launched IS NOT NULL
    ) AS INT)
FROM outcomes o
JOIN classes c
    ON o.ship = c.class
WHERE NOT EXISTS (
    SELECT 1
    FROM ships s
    WHERE s.name = o.ship
)
AND (
    SELECT AVG(CAST(s.launched AS FLOAT))
    FROM ships s
    JOIN classes c2
        ON s.class = c2.class
    WHERE c2.country = c.country
      AND s.launched IS NOT NULL
) IS NOT NULL;
```

---

13. Потопить в следующем сражении суда, которые в первой своей битве были повреждены и больше не участвовали ни в каких сражениях. Если следующего сражения для такого судна не существует в базе данных, не вносить его в таблицу Outcomes. Замечание: в базе данных нет двух сражений, которые состоялись бы в один день.

```sql
INSERT INTO Outcomes (ship, battle, result)
SELECT
    o.ship,
    next_b.name,
    'sunk'
FROM outcomes o
JOIN battles b
    ON o.battle = b.name
JOIN (
    SELECT
        o.ship,
        MIN(b.date) AS first_date
    FROM outcomes o
    JOIN battles b
        ON o.battle = b.name
    GROUP BY o.ship
) f
    ON o.ship = f.ship
   AND b.date = f.first_date
JOIN battles next_b
    ON next_b.date = (
        SELECT MIN(b2.date)
        FROM battles b2
        WHERE b2.date > b.date
    )
WHERE o.result = 'damaged'
  AND o.ship IN (
      SELECT ship
      FROM outcomes
      GROUP BY ship
      HAVING COUNT(*) = 1
  );
```

---

14. Удалите классы, имеющие в базе данных менее трех кораблей (учесть корабли из Outcomes).

```sql
DELETE FROM Classes
WHERE class IN (
    SELECT c.class
    FROM Classes c
    LEFT JOIN (
        SELECT name AS ship, class
        FROM Ships

        UNION

        SELECT o.ship, c.class
        FROM Outcomes o
        JOIN Classes c
            ON o.ship = c.class
    ) x
        ON c.class = x.class
    GROUP BY c.class
    HAVING COUNT(x.ship) < 3
);
```

---

15. Для каждого пассажира удалить из таблицы pass_in_trip все записи о его полетах, кроме первого и последнего.

```sql
DELETE FROM Pass_in_trip
WHERE NOT EXISTS (
    SELECT 1
    FROM (
        SELECT
            pit.ID_psg,
            MIN(CAST(pit.date AS datetime) + CAST(t.time_out AS datetime)) AS first_flight,
            MAX(CAST(pit.date AS datetime) + CAST(t.time_out AS datetime)) AS last_flight
        FROM Pass_in_trip pit
        JOIN Trip t
            ON pit.trip_no = t.trip_no
        GROUP BY pit.ID_psg
    ) x
    JOIN Trip t2
        ON Pass_in_trip.trip_no = t2.trip_no
    WHERE x.ID_psg = Pass_in_trip.ID_psg
      AND (
          CAST(Pass_in_trip.date AS datetime) + CAST(t2.time_out AS datetime) = x.first_flight
          OR
          CAST(Pass_in_trip.date AS datetime) + CAST(t2.time_out AS datetime) = x.last_flight
      )
);
```

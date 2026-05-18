# SQL-EX SELECT Задачи

1. Найдите номер модели, скорость процессора и размер жесткого диска для всех ПК стоимостью менее 500 долларов.

```sql
SELECT model, speed, hd
FROM pc
WHERE price < 500;
```

---

2. Найдите производителей принтеров.

```sql
SELECT DISTINCT maker
FROM product
WHERE type = 'printer';
```

---

3. Найдите номер модели, объем памяти и размеры экранов ноутбуков, цена которых превышает 1000 долларов.

```sql
SELECT model, ram, screen
FROM laptop
WHERE price > 1000;
```

---

4. Найдите все записи таблицы Printer для цветных принтеров.

```sql
SELECT *
FROM printer
WHERE color = 'y';
```

---

5. Найдите номер модели, скорость и размер жесткого диска ПК, имеющих 12x или 24x CD и цену менее 600 долларов.

```sql
SELECT model, speed, hd
FROM pc
WHERE (cd = '12x' OR cd = '24x')
  AND price < 600;
```

---

6. Для каждого производителя, выпускающего ноутбуки с объемом жесткого диска не менее 10 Гбайт, найти скорости таких ноутбуков.

```sql
SELECT DISTINCT p.maker, l.speed
FROM product p
JOIN laptop l
    ON p.model = l.model
WHERE l.hd >= 10;
```

---

7. Найдите номера моделей и цены всех имеющихся в продаже продуктов (любого типа) производителя B.

```sql
SELECT pc.model, pc.price
FROM pc
JOIN product
    ON pc.model = product.model
WHERE product.maker = 'B'

UNION

SELECT laptop.model, laptop.price
FROM laptop
JOIN product
    ON laptop.model = product.model
WHERE product.maker = 'B'

UNION

SELECT printer.model, printer.price
FROM printer
JOIN product
    ON printer.model = product.model
WHERE product.maker = 'B';
```

---

8. Найдите производителя, выпускающего ПК, но не ноутбуки.

```sql
SELECT maker
FROM product
WHERE type = 'PC'

EXCEPT

SELECT maker
FROM product
WHERE type = 'laptop';
```

---

9. Найдите производителей ПК с процессором не менее 450 Мгц.

```sql
SELECT DISTINCT maker
FROM product
JOIN pc
    ON product.model = pc.model
WHERE speed >= 450;
```

---

10. Найдите модели принтеров, имеющих самую высокую цену.

```sql
SELECT model, price
FROM printer
WHERE price = (
    SELECT MAX(price)
    FROM printer
);
```

---

11. Найдите среднюю скорость ПК.

```sql
SELECT AVG(speed)
FROM pc;
```

---

12. Найдите среднюю скорость ноутбуков, цена которых превышает 1000 долларов.

```sql
SELECT AVG(speed)
FROM laptop
WHERE price > 1000;
```

---

13. Найдите среднюю скорость ПК, выпущенных производителем A.

```sql
SELECT AVG(speed)
FROM pc p
JOIN product pr
    ON p.model = pr.model
WHERE pr.maker = 'A';
```

---

14. Найдите класс, имя и страну для кораблей из таблицы Ships, имеющих не менее 10 орудий.

```sql
SELECT s.class, s.name, c.country
FROM ships s
JOIN classes c
    ON s.class = c.class
WHERE numGuns >= 10;
```

---

15. Найдите размеры жестких дисков, совпадающих у двух и более PC.

```sql
SELECT hd
FROM pc
GROUP BY hd
HAVING COUNT(hd) > 1;
```

---

16. Найдите пары моделей PC, имеющих одинаковые скорость и RAM. В результате каждая пара должна выводиться только один раз.

```sql
SELECT DISTINCT pc2.model, pc1.model, pc1.speed, pc1.ram
FROM pc pc1
JOIN pc pc2
    ON pc1.model < pc2.model
   AND pc1.speed = pc2.speed
   AND pc1.ram = pc2.ram;
```

---

17. Найдите модели ноутбуков, скорость которых меньше скорости каждого из ПК.

```sql
SELECT DISTINCT pr.type, l.model, l.speed
FROM product pr
JOIN laptop l
    ON pr.model = l.model
WHERE l.speed < ALL (
    SELECT speed
    FROM pc
);
```

---

18. Найдите производителей самых дешевых цветных принтеров.

```sql
SELECT DISTINCT pro.maker, pri.price
FROM product pro
JOIN printer pri
    ON pro.model = pri.model
WHERE color = 'y'
  AND price = (
      SELECT MIN(price)
      FROM printer
      WHERE color = 'y'
  );
```

---

19. Для каждого производителя, имеющего модели в таблице Laptop, найдите средний размер экрана выпускаемых им ноутбуков.

```sql
SELECT maker, AVG(screen)
FROM product p
JOIN laptop l
    ON p.model = l.model
GROUP BY maker;
```

---

20. Найдите производителей, выпускающих по меньшей мере три различных модели ПК.

```sql
SELECT maker, COUNT(DISTINCT model)
FROM product
WHERE type = 'pc'
GROUP BY maker
HAVING COUNT(DISTINCT model) >= 3;
```

---

21. Найдите максимальную цену ПК, выпускаемых каждым производителем, у которого есть модели в таблице PC.

```sql
SELECT maker, MAX(price)
FROM product pr
JOIN pc
    ON pr.model = pc.model
GROUP BY maker;
```

---

22. Для каждого значения скорости ПК, превышающего 600 МГц, определите среднюю цену ПК с такой же скоростью.

```sql
SELECT speed, AVG(price)
FROM pc
WHERE speed > 600
GROUP BY speed;
```

---

23. Найдите производителей, которые производят как ПК со скоростью не менее 750 МГц, так и ноутбуки со скоростью не менее 750 МГц.

```sql
SELECT DISTINCT pr.maker
FROM product pr
JOIN pc
    ON pr.model = pc.model
WHERE pc.speed >= 750

INTERSECT

SELECT DISTINCT pr.maker
FROM product pr
JOIN laptop l
    ON pr.model = l.model
WHERE l.speed >= 750;
```

---

24. Перечислите номера моделей любых типов, имеющих самую высокую цену по всей имеющейся в базе данных продукции.

```sql
WITH united(model, price) AS (
    SELECT model, price
    FROM pc

    UNION

    SELECT model, price
    FROM laptop

    UNION

    SELECT model, price
    FROM printer
)

SELECT model
FROM united
WHERE price = (
    SELECT MAX(price)
    FROM united
);
```

---

25. Найдите производителей принтеров, которые производят ПК с наименьшим объемом RAM и с самым быстрым процессором среди всех ПК, имеющих наименьший объем RAM.

```sql
SELECT maker
FROM product
WHERE type = 'Printer'

INTERSECT

SELECT prod.maker
FROM product prod
JOIN pc
    ON prod.model = pc.model
WHERE pc.ram = (
    SELECT MIN(ram)
    FROM pc
)
AND pc.speed = (
    SELECT MAX(speed)
    FROM pc
    WHERE ram = (
        SELECT MIN(ram)
        FROM pc
    )
);
```

---

26. Найдите среднюю цену ПК и ноутбуков, выпущенных производителем A.

```sql
SELECT AVG(price)
FROM (
    SELECT price
    FROM pc
    JOIN product pr
        ON pc.model = pr.model
    WHERE pr.maker = 'A'

    UNION ALL

    SELECT price
    FROM laptop l
    JOIN product pr
        ON l.model = pr.model
    WHERE pr.maker = 'A'
) t;
```

---

27. Найдите средний размер диска ПК каждого из тех производителей, которые выпускают и принтеры.

```sql
SELECT pr.maker, AVG(hd)
FROM product pr
JOIN pc
    ON pr.model = pc.model
WHERE pr.maker IN (
    SELECT maker
    FROM product
    WHERE type = 'PC'

    INTERSECT

    SELECT maker
    FROM product
    WHERE type = 'printer'
)
GROUP BY pr.maker;
```

---

28. Используя таблицу Product, определить количество производителей, выпускающих по одной модели.

```sql
SELECT COUNT(*)
FROM (
    SELECT maker
    FROM product
    GROUP BY maker
    HAVING COUNT(model) = 1
) t;
```

---

29. В предположении, что приход и расход денег на каждом пункте приема фиксируется не чаще одного раза в день, вывести пункт, дату, приход и расход. Использовать таблицы Income_o и Outcome_o.

```sql
SELECT
    COALESCE(i.point, o.point) AS point,
    COALESCE(i.date, o.date) AS date,
    i.inc,
    o.out
FROM income_o i
FULL JOIN outcome_o o
    ON i.point = o.point
   AND i.date = o.date;
```

---

30. В предположении, что приход и расход денег на каждом пункте приема фиксируется произвольное число раз, получить таблицу, в которой каждому пункту за каждую дату соответствует одна строка с суммарным расходом и суммарным приходом за день.

```sql
SELECT
    COALESCE(i.point, o.point),
    COALESCE(i.date, o.date),
    o.out,
    i.inc
FROM (
    SELECT point, date, SUM(inc) AS inc
    FROM income
    GROUP BY point, date
) i

FULL JOIN (
    SELECT point, date, SUM(out) AS out
    FROM outcome
    GROUP BY point, date
) o
    ON i.point = o.point
   AND i.date = o.date;
```

---

31. Для классов кораблей, калибр орудий которых не менее 16 дюймов, укажите класс и страну.

```sql
SELECT class, country
FROM classes
WHERE bore >= 16;
```

---

32. Одной из характеристик корабля является половина куба калибра его главных орудий (mw). С точностью до 2 десятичных знаков определите среднее значение mw для кораблей каждой страны, у которой есть корабли в базе данных.

```sql
SELECT
    t.country,
    CAST(
        ROUND(AVG(POWER(t.bore, 3) / 2.0), 2)
        AS NUMERIC(10,2)
    ) AS mw
FROM (
    SELECT s.name AS ship, c.country, c.bore
    FROM Ships s
    JOIN Classes c
        ON s.class = c.class

    UNION

    SELECT o.ship AS ship, c.country, c.bore
    FROM Outcomes o
    JOIN Classes c
        ON o.ship = c.class
) t
GROUP BY t.country;
```

---

33. Укажите корабли, потопленные в сражениях в Северной Атлантике (North Atlantic).

```sql
SELECT ship
FROM outcomes
WHERE battle = 'North Atlantic'
  AND result = 'sunk';
```

---

34. По Вашингтонскому международному договору от начала 1922 г. запрещалось строить линейные корабли водоизмещением более 35 тыс. тонн. Укажите корабли, нарушившие этот договор.

```sql
SELECT s.name
FROM ships s
JOIN classes c
    ON s.class = c.class
WHERE s.launched >= 1922
  AND c.displacement > 35000
  AND c.type = 'bb';
```

---

35. В таблице Product найти модели, которые состоят только из цифр или только из латинских букв (A-Z, без учета регистра).

```sql
SELECT model, type
FROM product
WHERE model NOT LIKE '%[^0-9]%'
   OR model NOT LIKE '%[^a-z]%';
```

---

36. Перечислите названия головных кораблей, имеющихся в базе данных (учесть корабли в Outcomes).

```sql
SELECT c.class
FROM classes c
JOIN outcomes o
    ON c.class = o.ship

UNION

SELECT c.class
FROM classes c
JOIN ships s
    ON c.class = s.name;
```

---

37. Найдите классы, в которые входит только один корабль из базы данных (учесть также корабли в Outcomes).

```sql
SELECT class
FROM (
    SELECT name, class
    FROM Ships

    UNION

    SELECT o.ship AS name, c.class
    FROM Outcomes o
    JOIN Classes c
        ON o.ship = c.class
) t
GROUP BY class
HAVING COUNT(*) = 1;
```

---

38. Найдите страны, имевшие когда-либо классы обычных боевых кораблей ('bb') и классы крейсеров ('bc').

```sql
SELECT country
FROM classes
WHERE type = 'bb'

INTERSECT

SELECT country
FROM classes
WHERE type = 'bc';
```

---

39. Найдите корабли, сохранившиеся для будущих сражений: выведенные из строя в одной битве (`damaged`), но участвовавшие в другой, произошедшей позже.

```sql
SELECT DISTINCT o1.ship
FROM Outcomes o1
JOIN Battles b1
    ON o1.battle = b1.name
WHERE o1.result = 'damaged'
  AND EXISTS (
      SELECT 1
      FROM Outcomes o2
      JOIN Battles b2
          ON o2.battle = b2.name
      WHERE o2.ship = o1.ship
        AND b2.date > b1.date
  );
```

---

40. Найти производителей, которые выпускают более одной модели, при этом все выпускаемые производителем модели являются продуктами одного типа.

```sql
SELECT
    maker,
    MIN(type) AS type
FROM product
GROUP BY maker
HAVING COUNT(model) > 1
   AND COUNT(DISTINCT type) = 1;
```






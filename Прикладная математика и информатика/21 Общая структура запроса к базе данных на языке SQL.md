# Общая структура запроса к базе данных на языке SQL. Основные блоки оператора SELECT. Последовательность выполнения блоков в группировками данных.

## TL;DR
SQL-запрос — оператор `SELECT`, состоящий из блоков: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT/OFFSET`. **Логический порядок выполнения** (важен!): `FROM` → `JOIN`/`ON` → `WHERE` → `GROUP BY` → агрегаты → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`. Отсюда: алиасы из `SELECT` недоступны в `WHERE`, но доступны в `ORDER BY`. Фильтр строк до агрегации — `WHERE`; после — `HAVING`.

## Развёрнуто

### Полный синтаксис SELECT
```sql
SELECT [DISTINCT | ALL] выражения [AS алиас], ...
FROM таблица [JOIN ... ON ... | таблица2, ...]
[WHERE условие]
[GROUP BY выражения]
[HAVING условие_над_группами]
[ORDER BY выражения [ASC|DESC]]
[LIMIT n [OFFSET m]];
```

### Назначение блоков

**`SELECT`** — список выводимых столбцов и выражений. `*` — все, `DISTINCT` — устранение дубликатов. Алиасы: `expr AS name`.

**`FROM`** — источник данных: одна или несколько таблиц, представления, подзапросы, CTE; объединения соединениями (JOIN).

**`WHERE`** — фильтр строк до группировки. Логические операторы `AND, OR, NOT`, сравнения, `IN, BETWEEN, LIKE, IS NULL`, подзапросы (скалярные, `IN`, `EXISTS`).

**`GROUP BY`** — группировка строк по значениям выражений. Все элементы `SELECT` должны быть либо в `GROUP BY`, либо агрегатными функциями (`COUNT, SUM, AVG, MIN, MAX` и др.).

**`HAVING`** — фильтр **групп** (после агрегирования). Можно использовать агрегатные функции; для фильтрации до агрегирования — `WHERE`.

**`ORDER BY`** — сортировка результата. Можно по номеру столбца, по алиасу, по нескольким ключам.

**`LIMIT n OFFSET m`** — пагинация (PostgreSQL/MySQL/SQLite); в MS SQL Server `TOP n` или `OFFSET ... FETCH`.

### Логический порядок выполнения
Хотя пишется в порядке `SELECT...FROM...WHERE...`, СУБД логически исполняет в порядке:
1. `FROM` + `JOIN ON` — формирование исходного множества строк (декартово произведение + условия соединения).
2. `WHERE` — фильтр строк.
3. `GROUP BY` — формирование групп.
4. Вычисление агрегатных функций.
5. `HAVING` — фильтр групп.
6. `SELECT` — вычисление выражений списка вывода.
7. `DISTINCT` — устранение дубликатов.
8. `ORDER BY` — сортировка.
9. `LIMIT/OFFSET` — выборка нужного фрагмента.

**Следствия**:
- Алиас, заданный в `SELECT`, **не доступен** в `WHERE` и `GROUP BY` (строго по стандарту), но **доступен** в `ORDER BY` (а во многих СУБД и в `GROUP BY/HAVING`).
- Агрегатные функции нельзя использовать в `WHERE` — они вычисляются после.
- Колонка, не входящая в `GROUP BY` и не агрегируемая, не может стоять в `SELECT`.

### Группировка и агрегация
Агрегатные функции: `COUNT(*)`, `COUNT(col)`, `COUNT(DISTINCT col)`, `SUM`, `AVG`, `MIN`, `MAX`, `STDDEV`, `VARIANCE`, `STRING_AGG/GROUP_CONCAT`.

С `GROUP BY` агрегаты считаются по группам. Без `GROUP BY` — по всему набору.

Расширения: `GROUP BY ROLLUP/CUBE/GROUPING SETS` — для подытогов.

**Оконные функции** (`OVER (...)`) — агрегаты без коллапса в одну строку: `SUM(x) OVER (PARTITION BY g ORDER BY t)`. Применяются в фазе `SELECT`.

### Пример
```sql
SELECT department_id,
       AVG(salary) AS avg_sal,
       COUNT(*)    AS cnt
FROM   employees
WHERE  hire_date >= '2020-01-01'
GROUP BY department_id
HAVING COUNT(*) >= 5
ORDER BY avg_sal DESC
LIMIT 10;
```
Логика: фильтруем сотрудников, нанятых с 2020-01-01 (`WHERE`); группируем по отделу (`GROUP BY`); оставляем отделы с ≥5 нанятыми (`HAVING`); считаем `SELECT`; сортируем по убыванию средней зарплаты; берём топ-10.

### Категории SQL-операторов (контекст)
- **DDL** (Data Definition Language): `CREATE, ALTER, DROP, TRUNCATE`.
- **DML** (Data Manipulation Language): `SELECT, INSERT, UPDATE, DELETE, MERGE`.
- **DCL** (Data Control Language): `GRANT, REVOKE`.
- **TCL** (Transaction Control Language): `BEGIN, COMMIT, ROLLBACK, SAVEPOINT`.

Структура запроса, описанная выше, — про оператор `SELECT` (DML).

### Подводные камни
- В `WHERE` нельзя использовать агрегаты — нужно `HAVING`.
- `NULL` в сравнениях даёт `UNKNOWN`, а не `FALSE` — `WHERE col = NULL` всегда пусто; нужно `IS NULL`.
- При `LIMIT` без `ORDER BY` порядок строк не определён.
- `COUNT(*)` считает все строки, `COUNT(col)` пропускает `NULL`-ы.

См. `[[22 Запросы к базам данных с использованием нескольких таблиц]]`, `[[20 Нормальные формы реляционных отношений]]`.

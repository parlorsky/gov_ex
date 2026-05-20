# Общая структура запроса к базе данных на языке SQL. Основные блоки оператора SELECT. Последовательность выполнения блоков в группировками данных.

**Синтаксис SELECT:**

```sql
SELECT [DISTINCT] выражения [AS алиас]
FROM    таблицы / JOIN-ы
WHERE   условие
GROUP BY столбцы
HAVING  условие_над_группами
ORDER BY столбцы [ASC|DESC]
LIMIT   n [OFFSET m];
```

**Логический порядок выполнения (главное в билете):**

$$\text{FROM/JOIN} \to \text{WHERE} \to \text{GROUP BY} \to \text{агрегаты} \to \text{HAVING} \to \text{SELECT} \to \text{DISTINCT} \to \text{ORDER BY} \to \text{LIMIT}.$$

**Назначение блоков:**

- `SELECT` — какие столбцы и выражения возвращать. `DISTINCT` убирает дубли, `AS` — алиасы.
- `FROM` — источник данных (таблицы, JOIN-ы, подзапросы, CTE).
- `WHERE` — фильтр строк **до** группировки. Нельзя агрегаты и алиасы из SELECT.
- `GROUP BY` — группировка по значениям выражений.
- `HAVING` — фильтр **групп** после агрегирования. Можно агрегаты.
- `ORDER BY` — сортировка результата. Алиасы из SELECT доступны.
- `LIMIT n OFFSET m` — пагинация (PostgreSQL/MySQL); в MS SQL — `TOP` или `OFFSET ... FETCH`.

**Следствия из порядка:**
- Алиасы из `SELECT` недоступны в `WHERE` и `GROUP BY`, но доступны в `ORDER BY` (потому что ORDER BY логически после SELECT).
- Агрегаты нельзя в `WHERE` — они ещё не вычислены.
- Столбец, не входящий в `GROUP BY` и не агрегированный, не может стоять в `SELECT`.

**Агрегатные функции:** Схлопывают много значений в одно. `COUNT(*)`, `COUNT(col)`, `COUNT(DISTINCT col)`, `SUM`, `AVG`, `MIN`, `MAX`, `STDDEV`, `STRING_AGG`. Без `GROUP BY` — по всему набору, с ним — по группам.

**`COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`:** все строки / непустые значения / уникальные непустые.

**NULL — тернарная логика.** NULL = "неизвестное значение". Сравнение с NULL даёт **UNKNOWN** (не TRUE и не FALSE), строка не проходит фильтр. Проверка — `IS NULL` / `IS NOT NULL`, не `= NULL`.

**`LIMIT` без `ORDER BY`** — порядок строк не определён, результат непредсказуем. Почти всегда баг.

**Оконные функции** (`OVER (...)`) — агрегаты без коллапса в одну строку: `SUM(x) OVER (PARTITION BY g ORDER BY t)`. Вычисляются в фазе `SELECT`.

**Категории SQL-операторов:**
- **DDL** (Data Definition) — структура БД: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- **DML** (Data Manipulation) — данные: `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **DCL** (Data Control) — права: `GRANT`, `REVOKE`.
- **TCL** (Transaction Control) — транзакции: `BEGIN`, `COMMIT`, `ROLLBACK`.
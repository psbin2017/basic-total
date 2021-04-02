# 페이징 쿼리

## `HSQLDB` (Hyper SQL Database)

```sql
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME
FROM
    MEMBER M
ORDER BY
    M.NAME DESC
OFFSET ? LIMIT ?
```

## `MySQL` (`MariaDB`)

```sql
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME
FROM
    MEMBER M
ORDER BY
    M.NAME DESC
LIMIT ?, ?
```

## `PostgreSQL`

```sql
SELECT
    M.ID AS ID,
    M.AGE AS AGE,
    M.TEAM_ID AS TEAM_ID,
    M.NAME AS NAME
FROM
    MEMBER M
ORDER BY
    M.NAME DESC
LIMIT ? OFFSET ?
```

## `Oracle`

```sql
SELECT
    *
FROM (
    SELECT 
        ROW_.*, 
        ROWNUM ROWNUM_
    FROM (
        SELECT
            M.ID AS ID,
            M.AGE AS AGE,
            M.TEAM_ID AS TEAM_ID,
            M.NAME AS NAME
        FROM
            MEMBER M
        ORDER BY
            M.NAME
    )
    WHERE ROWNUM <= ?
)
WHERE ROWNUM_ > ?
```

## `SQLServer`

```sql
WITH query AS {
    SELECT
        inner_query.*,
        ROW_NUMBER() OVER (ORDER BY CURRENT_TIMESTAMP) as __hibernate_row_nr__
    FROM (
        select
            TOP(?) m.id as id,
            m.age as age,
            m.team_id as team_id,
            m.name as name
        from
            Member m
        order by
            m.name DESC
    ) inner_query
}
SELECT
    id,
    age,
    team_id,
    name
FROM query
WHERE
    __hibernate_row_nr__ >= ? AND __hibernate_row_nr__ < ?
```

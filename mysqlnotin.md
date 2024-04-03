针对您的需求，如果您想要查询在表a中，某些固定的id和name组合中不存在的数据，可以采取几种不同的方法。以下是一些可行的MySQL 5.7查询方式，每种方式都有其特点，您可以根据具体情况选择最合适的。

### 方法1: LEFT JOIN + IS NULL

这种方法通过左连接（LEFT JOIN）表a和一个包含所有固定id和name组合的虚拟表，然后选取那些在右表中没有匹配的行（即IS NULL条件）。

```sql
SELECT a.*
FROM 表a AS a
LEFT JOIN (
  SELECT '固定id1' AS id, '固定name1' AS name
  UNION ALL
  SELECT '固定id2', '固定name2'
  -- 根据需要添加更多固定组合
) AS b ON a.id = b.id AND a.name = b.name
WHERE b.id IS NULL;
```

### 方法2: NOT EXISTS

使用`NOT EXISTS`与子查询结合，检查不存在于某个固定id和name组合的数据。

```sql
SELECT *
FROM 表a AS a
WHERE NOT EXISTS (
  SELECT 1
  FROM (
    SELECT '固定id1' AS id, '固定name1' AS name
    UNION ALL
    SELECT '固定id2', '固定name2'
    -- 根据需要添加更多固定组合
  ) AS b
  WHERE a.id = b.id AND a.name = b.name
);
```

### 方法3: NOT IN

如果您确定id和name组合不会包含NULL值，可以使用`NOT IN`子句。这种方式在某些情况下可能不如前两种方式效率高，因为`NOT IN`对于大型数据集可能会较慢。

```sql
SELECT *
FROM 表a
WHERE (id, name) NOT IN (
  ('固定id1', '固定name1'),
  ('固定id2', '固定name2')
  -- 添加更多组合
);
```

### 性能考量

- **LEFT JOIN + IS NULL**: 通常效率较高，特别是在需要外连接查找不匹配行的情况下。
- **NOT EXISTS**: 对于某些数据库优化器来说，`NOT EXISTS`可以更有效地执行，因为一旦找到匹配的行就会停止搜索。
- **NOT IN**: 对于小型到中型数据集，这可能是一个简单直观的解决方案，但对于大型数据集或者存在NULL值的情况，性能可能下降。

选择哪种方法取决于您的具体数据和性能需求。建议对于您的具体数据集测试这些不同的查询方法，以确定哪种最适合您的需求。
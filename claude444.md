好的,如果你打算在存储过程内部直接使用`UNION ALL`进行查询,并根据条件动态拼接SQL语句,可以对存储过程进行如下修改:

```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),
    IN p_project_id VARCHAR(50)
)
BEGIN
    DECLARE v_feature_group_count INT;
    
    -- 获取特性分组的数量
    SELECT COUNT(DISTINCT feature_group) INTO v_feature_group_count FROM test_case_feature_group;
    
    SET @table_name = CONCAT('test_case_', p_project_id);
    
    SET @sql = CONCAT('
        SELECT c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time
        FROM ', @table_name, ' c
    ');
    
    IF p_version_list IS NOT NULL AND p_version_list != '' THEN
        SET @sql = CONCAT(@sql, '
            JOIN test_case_version cv ON c.id = cv.case_id
            WHERE FIND_IN_SET(cv.version, ?) > 0
        ');
    END IF;
    
    IF v_feature_group_count > 0 THEN
        SET @sql = CONCAT(@sql, '
            AND c.id IN (
        ');
        
        SET @feature_group_sql = '';
        
        SELECT CONCAT(
            CASE WHEN @feature_group_sql != '' THEN ' UNION ALL ' ELSE '' END,
            '
                SELECT c.id
                FROM ', @table_name, ' c
                JOIN test_case_feature cf ON c.id = cf.case_id
                JOIN test_case_feature_group fg ON cf.feature = fg.feature
                WHERE fg.feature_group = ''', feature_group, '''
            '
        ) INTO @feature_group_sql
        FROM test_case_feature_group
        GROUP BY feature_group;
        
        SET @sql = CONCAT(@sql, @feature_group_sql);
        
        SET @sql = CONCAT(@sql, '
            )
        ');
    END IF;
    
    SET @sql = CONCAT(@sql, '
        GROUP BY c.id
    ');
    
    PREPARE stmt FROM @sql;
    
    IF p_version_list IS NOT NULL AND p_version_list != '' THEN
        EXECUTE stmt USING p_version_list;
    ELSE
        EXECUTE stmt;
    END IF;
    
    DEALLOCATE PREPARE stmt;
END //

DELIMITER ;
```

在修改后的存储过程中:
- 我们首先获取特性分组的数量`v_feature_group_count`。
- 使用`@feature_group_sql`变量存储特性分组的查询语句,通过`SELECT INTO`语句动态拼接每个分组的查询语句。
- 在拼接特性分组查询语句时,使用`CASE WHEN`判断是否需要添加`UNION ALL`关键字。
- 将拼接好的特性分组查询语句使用`IN`子查询的方式嵌入到主查询中。
- 最后,根据版本列表是否为空,动态执行准备好的SQL语句。

通过以上修改,存储过程可以根据特性分组表动态拼接`UNION ALL`查询语句,并将结果与主查询进行关联。这样可以在存储过程内部直接使用`UNION ALL`进行查询,避免了使用临时表的开销。

MyBatis映射文件和Java代码保持不变,因为存储过程的修改对外部调用是透明的。

请注意,以上示例假设特性分组信息存储在`test_case_feature_group`表中,并与`test_case_feature`表进行关联。如果你的实际表结构有所不同,请根据实际情况进行相应的调整。
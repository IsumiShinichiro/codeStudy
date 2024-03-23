好的,我明白你的意思。你希望在存储过程中声明一个公共的SQL语句,然后使用类似Java中`String.format()`的方式,将参数动态传入到SQL语句中,以减少代码重复并提高可维护性。

下面是根据你的建议修改后的存储过程:

```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),
    IN p_project_id VARCHAR(50)
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_feature_group VARCHAR(50);
    DECLARE v_first_feature_group VARCHAR(50) DEFAULT '';
    
    -- 检查输入参数是否为空
    IF p_project_id IS NULL OR p_project_id = '' THEN
        SELECT NULL;
        RETURN;
    END IF;
    
    -- 声明游标,用于遍历特性分组
    DECLARE cur CURSOR FOR SELECT DISTINCT feature_group FROM test_case_feature_group;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    SET @table_name = CONCAT('test_case_', p_project_id);
    
    -- 声明公共的SQL语句
    SET @sql_template = '
        SELECT c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time
        FROM %s c
        JOIN test_case_feature cf ON c.id = cf.case_id
        JOIN test_case_feature_group fg ON cf.feature = fg.feature
        WHERE fg.feature_group = ''%s''
    ';
    
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
    
    -- 获取第一个特性分组
    OPEN cur;
    FETCH cur INTO v_first_feature_group;
    
    IF v_first_feature_group IS NOT NULL AND v_first_feature_group != '' THEN
        SET @sql = CONCAT(@sql, '
            AND c.id IN (', REPLACE(REPLACE(@sql_template, '%s', @table_name), '%s', v_first_feature_group), ')
        ');
    ELSE
        CLOSE cur;
        SELECT NULL;
        RETURN;
    END IF;
    
    -- 从第二个分组开始遍历特性分组
    read_loop: LOOP
        FETCH cur INTO v_feature_group;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SET @sql = CONCAT(@sql, '
            UNION ALL
            ', REPLACE(REPLACE(@sql_template, '%s', @table_name), '%s', v_feature_group), '
        ');
    END LOOP;
    
    -- 关闭游标
    CLOSE cur;
    
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
- 我们声明了一个名为`@sql_template`的变量,用于存储公共的SQL语句模板。
- 在SQL语句模板中,使用`%s`作为占位符,用于动态替换表名和特性分组的值。
- 当获取第一个特性分组时,使用`REPLACE`函数将`@sql_template`中的占位符替换为实际的表名和特性分组值,并将结果拼接到`@sql`变量中。
- 在遍历剩余特性分组时,同样使用`REPLACE`函数将`@sql_template`中的占位符替换为实际的表名和特性分组值,并使用`UNION ALL`关键字拼接到`@sql`变量中。
- 其他部分的逻辑保持不变,包括游标的使用、动态SQL的拼接以及执行准备好的SQL语句。

通过引入公共的SQL语句模板,我们可以将重复的SQL片段提取出来,并使用占位符动态替换表名和特性分组值。这样可以提高代码的可读性和可维护性,减少重复代码的编写。

如果你对这个修改有任何其他的建议或意见,欢迎随时提出。
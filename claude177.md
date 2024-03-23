你说得对,我们需要在循环之前取出第一个分组作为默认,然后从第二个分组开始判断和循环。下面是修改后的存储过程:

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
        LEAVE stored_procedure;
    END IF;
    
    -- 声明游标,用于遍历特性分组
    DECLARE cur CURSOR FOR SELECT DISTINCT feature_group FROM test_case_feature_group;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
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
    
    -- 获取第一个特性分组
    OPEN cur;
    FETCH cur INTO v_first_feature_group;
    
    IF v_first_feature_group IS NOT NULL AND v_first_feature_group != '' THEN
        SET @sql = CONCAT(@sql, '
            JOIN test_case_feature cf ON c.id = cf.case_id
            JOIN test_case_feature_group fg ON cf.feature = fg.feature
            WHERE fg.feature_group = ''', v_first_feature_group, '''
        ');
    ELSE
        CLOSE cur;
        SELECT NULL;
        LEAVE stored_procedure;
    END IF;
    
    -- 从第二个分组开始遍历特性分组
    read_loop: LOOP
        FETCH cur INTO v_feature_group;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SET @sql = CONCAT(@sql, '
            UNION ALL
            SELECT c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time
            FROM ', @table_name, ' c
            JOIN test_case_feature cf ON c.id = cf.case_id
            JOIN test_case_feature_group fg ON cf.feature = fg.feature
            WHERE fg.feature_group = ''', v_feature_group, '''
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
- 我们在游标循环之前,使用`FETCH`语句获取第一个特性分组,并将其赋值给`v_first_feature_group`变量。
- 如果`v_first_feature_group`不为空,则将其作为默认分组,拼接到初始查询语句中。
- 如果`v_first_feature_group`为空,表示没有特性分组,则关闭游标,返回`NULL`,并退出存储过程。
- 从第二个分组开始,使用游标遍历剩余的特性分组。
- 在遍历过程中,使用`UNION ALL`关键字拼接每个分组的查询语句。
- 遍历完所有分组后,拼接`GROUP BY`子句,并执行准备好的SQL语句。

通过以上修改,存储过程可以在循环之前取出第一个分组作为默认,然后从第二个分组开始判断和循环,动态拼接`UNION ALL`查询语句。

MyBatis映射文件和Java代码保持不变,因为存储过程的修改对外部调用是透明的。

请注意,以上示例假设特性分组信息存储在`test_case_feature_group`表中,并与`test_case_feature`表进行关联。如果你的实际表结构有所不同,请根据实际情况进行相应的调整。
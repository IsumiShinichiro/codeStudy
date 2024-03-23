好的,我明白了。你希望保持游标循环,根据特性分组的数量动态拼接`UNION ALL`查询语句,同时处理特性分组数量为0和输入参数为空的情况。下面是修改后的存储过程:

```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),
    IN p_project_id VARCHAR(50)
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_feature_group VARCHAR(50);
    DECLARE v_feature_group_count INT DEFAULT 0;
    
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
    
    -- 打开游标
    OPEN cur;
    
    -- 遍历特性分组
    read_loop: LOOP
        FETCH cur INTO v_feature_group;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SET v_feature_group_count = v_feature_group_count + 1;
        
        IF v_feature_group_count > 1 THEN
            SET @sql = CONCAT(@sql, '
                UNION ALL
            ');
        END IF;
        
        SET @sql = CONCAT(@sql, '
            SELECT c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time
            FROM ', @table_name, ' c
            JOIN test_case_feature cf ON c.id = cf.case_id
            JOIN test_case_feature_group fg ON cf.feature = fg.feature
            WHERE fg.feature_group = ''', v_feature_group, '''
        ');
    END LOOP;
    
    -- 关闭游标
    CLOSE cur;
    
    IF v_feature_group_count > 0 THEN
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
    ELSE
        SELECT NULL;
    END IF;
END //

DELIMITER ;
```

在修改后的存储过程中:
- 我们首先检查输入参数`p_project_id`是否为空,如果为空,则直接返回`NULL`并退出存储过程。
- 使用游标`cur`遍历特性分组表`test_case_feature_group`中的每个分组。
- 在遍历过程中,通过`v_feature_group_count`变量记录特性分组的数量。
- 如果特性分组数量大于1,则在拼接查询语句时添加`UNION ALL`关键字。
- 遍历完所有分组后,判断`v_feature_group_count`的值:
  - 如果大于0,表示有特性分组,则拼接`GROUP BY`子句,并执行准备好的SQL语句。
  - 如果等于0,表示没有特性分组,则直接返回`NULL`。

通过以上修改,存储过程可以根据特性分组的数量动态拼接`UNION ALL`查询语句,同时处理特性分组数量为0和输入参数为空的情况。

MyBatis映射文件和Java代码保持不变,因为存储过程的修改对外部调用是透明的。

请注意,以上示例假设特性分组信息存储在`test_case_feature_group`表中,并与`test_case_feature`表进行关联。如果你的实际表结构有所不同,请根据实际情况进行相应的调整。
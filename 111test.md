好的,我明白了。你希望在存储过程中判断传入的版本列表和特性列表是否为空,只有在不为空时才将它们作为条件加入到SQL查询中。下面是修改后的示例:

1. 修改MySQL存储过程:
```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),
    IN p_feature_list VARCHAR(1000),
    IN p_project_id VARCHAR(50)
)
BEGIN
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
    
    IF p_feature_list IS NOT NULL AND p_feature_list != '' THEN
        IF p_version_list IS NOT NULL AND p_version_list != '' THEN
            SET @sql = CONCAT(@sql, '
                AND FIND_IN_SET(cf.feature, ?) > 0
            ');
        ELSE
            SET @sql = CONCAT(@sql, '
                JOIN test_case_feature cf ON c.id = cf.case_id
                WHERE FIND_IN_SET(cf.feature, ?) > 0
            ');
        END IF;
    END IF;
    
    SET @sql = CONCAT(@sql, '
        GROUP BY c.id
    ');
    
    PREPARE stmt FROM @sql;
    
    IF p_version_list IS NOT NULL AND p_version_list != '' AND p_feature_list IS NOT NULL AND p_feature_list != '' THEN
        EXECUTE stmt USING p_version_list, p_feature_list;
    ELSEIF p_version_list IS NOT NULL AND p_version_list != '' THEN
        EXECUTE stmt USING p_version_list;
    ELSEIF p_feature_list IS NOT NULL AND p_feature_list != '' THEN
        EXECUTE stmt USING p_feature_list;
    ELSE
        EXECUTE stmt;
    END IF;
    
    DEALLOCATE PREPARE stmt;
END //

DELIMITER ;
```

在修改后的存储过程中:
- 我们使用`IF`语句判断`p_version_list`和`p_feature_list`是否为空。
- 如果`p_version_list`不为空,则将版本条件加入到SQL查询中。
- 如果`p_feature_list`不为空,则根据`p_version_list`是否为空,决定是否需要添加`JOIN`语句,并将特性条件加入到SQL查询中。
- 在`EXECUTE`语句之前,根据`p_version_list`和`p_feature_list`的组合情况,动态确定需要传递的参数。

2. MyBatis映射文件和Java代码保持不变。

通过以上修改,存储过程可以根据传入的版本列表和特性列表的情况,动态生成SQL查询语句。如果版本列表或特性列表为空,则不会将它们作为条件加入到查询中。

这样可以确保在版本列表或特性列表为空时,查询仍然可以正常执行,并返回相应的结果。

MyBatis映射文件和Java代码不需要进行修改,因为存储过程内部已经处理了参数的判断和动态SQL的生成。

请注意,以上示例假设版本和特性信息存储在单独的表中(`test_case_version`和`test_case_feature`),并与用例表进行关联。如果你的实际表结构有所不同,请根据实际情况进行相应的调整。
如果有一个特性分组表,需要对每个分组进行计算,然后使用`UNION ALL`将结果合并起来,可以对存储过程进行如下修改:

1. 修改MySQL存储过程:
```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),
    IN p_project_id VARCHAR(50)
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_feature_group VARCHAR(50);
    
    -- 创建临时表以存储结果
    CREATE TEMPORARY TABLE IF NOT EXISTS temp_case_details (
        id INT,
        title VARCHAR(255),
        creator VARCHAR(50),
        create_time DATETIME,
        last_modifier VARCHAR(50),
        last_modify_time DATETIME
    );
    
    -- 声明游标,用于遍历特性分组
    DECLARE cur CURSOR FOR SELECT DISTINCT feature_group FROM test_case_feature_group;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    -- 打开游标
    OPEN cur;
    
    -- 遍历特性分组
    read_loop: LOOP
        FETCH cur INTO v_feature_group;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SET @table_name = CONCAT('test_case_', p_project_id);
        
        SET @sql = CONCAT('
            INSERT INTO temp_case_details
            SELECT c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time
            FROM ', @table_name, ' c
        ');
        
        IF p_version_list IS NOT NULL AND p_version_list != '' THEN
            SET @sql = CONCAT(@sql, '
                JOIN test_case_version cv ON c.id = cv.case_id
                WHERE FIND_IN_SET(cv.version, ?) > 0
            ');
        END IF;
        
        SET @sql = CONCAT(@sql, '
            JOIN test_case_feature cf ON c.id = cf.case_id
            JOIN test_case_feature_group fg ON cf.feature = fg.feature
            WHERE fg.feature_group = ?
            GROUP BY c.id
        ');
        
        PREPARE stmt FROM @sql;
        
        IF p_version_list IS NOT NULL AND p_version_list != '' THEN
            EXECUTE stmt USING p_version_list, v_feature_group;
        ELSE
            EXECUTE stmt USING v_feature_group;
        END IF;
        
        DEALLOCATE PREPARE stmt;
    END LOOP;
    
    -- 关闭游标
    CLOSE cur;
    
    -- 查询临时表并返回结果
    SELECT * FROM temp_case_details;
    
    -- 删除临时表
    DROP TEMPORARY TABLE temp_case_details;
END //

DELIMITER ;
```

在修改后的存储过程中:
- 我们创建了一个临时表`temp_case_details`用于存储每个分组的计算结果。
- 使用游标`cur`遍历特性分组表`test_case_feature_group`中的每个分组。
- 对于每个分组,动态生成SQL查询语句,将版本条件和特性分组条件加入到查询中。
- 使用`INSERT INTO`语句将每个分组的查询结果插入到临时表中。
- 遍历完所有分组后,查询临时表并返回最终结果。
- 最后,删除临时表。

2. 修改MyBatis映射文件:
```xml
<select id="getCaseDetails" statementType="CALLABLE" parameterType="map" resultMap="caseDetailMap">
    {CALL get_case_details(
        #{versionList, mode=IN, jdbcType=VARCHAR},
        #{projectId, mode=IN, jdbcType=VARCHAR}
    )}
</select>
```

在MyBatis映射文件中,我们去掉了`featureList`参数,因为存储过程内部会遍历特性分组表,不需要外部传入特性列表。

3. 修改Java代码:
```java
Map<String, Object> params = new HashMap<>();
params.put("versionList", versionList);
params.put("projectId", projectId);

List<CaseDetail> caseDetails = testCaseDao.getCaseDetails(params);
```

在Java代码中,同样去掉了`featureList`参数。

通过以上修改,存储过程可以根据特性分组表进行计算,并使用`UNION ALL`将每个分组的结果合并起来。这样可以动态处理不同的特性分组,并返回最终的用例详情列表。

请注意,以上示例假设特性分组信息存储在`test_case_feature_group`表中,并与`test_case_feature`表进行关联。如果你的实际表结构有所不同,请根据实际情况进行相应的调整。
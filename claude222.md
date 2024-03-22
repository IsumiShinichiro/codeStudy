好的,我来为你编写一个MySQL 5.7的存储过程,接收版本列表、特性列表和项目ID,返回用例信息明细列表。同时,我会提供相应的MyBatis映射文件。

MySQL存储过程:
```sql
DELIMITER //

CREATE PROCEDURE get_case_details(
    IN p_version_list VARCHAR(1000),  -- 版本列表,以逗号分隔
    IN p_feature_list VARCHAR(1000),  -- 特性列表,以逗号分隔
    IN p_project_id INT,              -- 项目ID
    OUT p_case_details_list VARCHAR(4000)  -- 返回的用例信息明细列表
)
BEGIN
    SET p_case_details_list = '';
    
    SELECT GROUP_CONCAT(
        CONCAT_WS('|', c.id, c.title, c.creator, c.create_time, c.last_modifier, c.last_modify_time) 
        SEPARATOR ','
    ) INTO p_case_details_list
    FROM test_case c
    JOIN test_case_version cv ON c.id = cv.case_id
    JOIN test_case_feature cf ON c.id = cf.case_id
    WHERE c.project_id = p_project_id
      AND FIND_IN_SET(cv.version, p_version_list) > 0 
      AND FIND_IN_SET(cf.feature, p_feature_list) > 0
    GROUP BY c.id;
END //

DELIMITER ;
```

存储过程说明:
1. 定义输入参数`p_version_list`和`p_feature_list`,分别表示版本列表和特性列表,以逗号分隔。
2. 定义输入参数`p_project_id`,表示项目ID。
3. 定义输出参数`p_case_details_list`,用于返回用例信息明细列表。
4. 使用`JOIN`关联`test_case`、`test_case_version`和`test_case_feature`表,根据项目ID、版本列表和特性列表进行过滤。
5. 使用`GROUP_CONCAT`和`CONCAT_WS`函数将用例信息拼接成字符串,以`|`分隔字段,以`,`分隔记录。
6. 将拼接后的结果赋值给输出参数`p_case_details_list`。

MyBatis映射文件:
```xml
<mapper namespace="com.example.dao.TestCaseDao">
    <resultMap id="caseDetailMap" type="com.example.dto.CaseDetail">
        <result property="id" column="id"/>
        <result property="title" column="title"/>
        <result property="creator" column="creator"/>
        <result property="createTime" column="create_time"/>
        <result property="lastModifier" column="last_modifier"/>
        <result property="lastModifyTime" column="last_modify_time"/>
    </resultMap>

    <select id="getCaseDetails" statementType="CALLABLE" parameterType="map" resultType="list">
        {CALL get_case_details(
            #{versionList, mode=IN, jdbcType=VARCHAR},
            #{featureList, mode=IN, jdbcType=VARCHAR},
            #{projectId, mode=IN, jdbcType=INTEGER},
            #{caseDetailsList, mode=OUT, jdbcType=VARCHAR}
        )}
    </select>
</mapper>
```

在MyBatis映射文件中:
1. 定义`resultMap`,将数据库字段映射到`CaseDetail`对象属性。
2. 定义`getCaseDetails`语句,使用`CALLABLE`语句类型调用存储过程。
3. 使用`#{}`占位符传递输入参数和输出参数。
4. `resultType`设置为`list`,表示返回结果为列表类型。

最后,你可以在Java代码中调用MyBatis的`selectList`方法执行`getCaseDetails`语句,将版本列表、特性列表和项目ID作为参数传递,获取返回的用例信息明细列表。
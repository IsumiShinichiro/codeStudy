你说得对,我在之前的示例中将`caseDetailsList`作为输出参数传递给了`select`语句,这确实是错误的。输出参数应该在调用存储过程后通过`#{}`占位符获取,而不是作为参数传递。

让我来修正这个错误,并提供正确的示例:

1. 修改MyBatis映射文件:
```xml
<mapper namespace="com.example.dao.TestCaseDao">
    <resultMap id="caseDetailMap" type="com.example.dto.CaseDetail">
        <id property="id" column="id"/>
        <result property="title" column="title"/>
        <result property="creator" column="creator"/>
        <result property="createTime" column="create_time"/>
        <result property="lastModifier" column="last_modifier"/>
        <result property="lastModifyTime" column="last_modify_time"/>
    </resultMap>

    <select id="getCaseDetails" statementType="CALLABLE" parameterType="map">
        {CALL get_case_details(
            #{versionList, mode=IN, jdbcType=VARCHAR},
            #{featureList, mode=IN, jdbcType=VARCHAR},
            #{projectId, mode=IN, jdbcType=INTEGER},
            #{caseDetailsList, mode=OUT, jdbcType=VARCHAR, javaType=string}
        )}
    </select>
    
    <resultMap id="caseDetailsResultMap" type="hashmap">
        <result property="caseDetailsList" column="p_case_details_list" jdbcType="VARCHAR" javaType="string"/>
    </resultMap>
</mapper>
```

在修改后的示例中:
- 我们去掉了`select`语句的`resultMap`属性,因为存储过程的输出参数不能直接映射到`resultMap`。
- 在`select`语句中,我们将`caseDetailsList`参数的`mode`设置为`OUT`,表示它是一个输出参数。同时,我们指定了`jdbcType`和`javaType`属性。
- 我们添加了一个新的`resultMap`named `caseDetailsResultMap`,用于映射存储过程的输出参数`p_case_details_list`到`caseDetailsList`属性。

2. 修改Java代码:
```java
Map<String, Object> params = new HashMap<>();
params.put("versionList", versionList);
params.put("featureList", featureList);
params.put("projectId", projectId);

Map<String, Object> result = testCaseDao.getCaseDetails(params);
String caseDetailsListString = (String) result.get("caseDetailsList");

List<CaseDetail> caseDetails = parse(caseDetailsListString);
```

在Java代码中:
- 我们创建一个`Map`对象`params`,用于传递输入参数。
- 调用`testCaseDao.getCaseDetails()`方法,传递`params`作为参数,并接收返回的`Map`对象`result`。
- 从`result`中获取`caseDetailsList`属性,即存储过程的输出参数。
- 调用一个自定义的`parse()`方法,将`caseDetailsListString`解析为`CaseDetail`对象列表。

这样修改后,就可以正确地调用存储过程,获取输出参数,并将其转换为所需的`CaseDetail`对象列表了。

再次感谢你的提问,让我有机会改正之前的错误,并提供更准确的示例。
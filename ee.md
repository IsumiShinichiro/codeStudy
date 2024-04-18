在MySQL中，即使不使用事务，您也可以捕捉异常并处理。事务通常用于确保数据的一致性，如果数据操作之间的完整性不是您关注的重点，可以省略它。以下是一个不涉及事务的示例存储过程，依然包括异常处理机制：

```sql
DELIMITER $$

CREATE PROCEDURE try_catch_demo_no_transaction()
BEGIN
    -- 声明一个用于捕捉异常的处理器
    DECLARE exit_handler FOR SQLEXCEPTION
    BEGIN
        -- 这里是错误处理代码，比如记录错误信息到日志表
        INSERT INTO error_logs (error_message, error_time) VALUES ('An error occurred', NOW());
    END;

    -- 以下是您的业务逻辑
    -- 假设这里有可能抛出异常的操作
    INSERT INTO table1 (column1) VALUES ('value1');
    INSERT INTO table2 (column2) VALUES ('value2'); -- 假设这条语句可能会失败

END$$

DELIMITER ;
```

在这个版本的存储过程中，事务的部分被移除，直接进行数据插入操作。如果在插入过程中发生异常（如因数据违反完整性规则、外键约束、类型错误等），声明的异常处理器将会被触发，并执行定义的异常处理逻辑，例如将错误信息插入到`error_logs`表中。

这种方法可以用于简单的错误处理，当你不需要回滚之前的操作时非常合适。这样的处理方式可以保证即使部分操作失败，程序也能继续执行后续的语句。但请注意，由于没有事务控制，任何成功的操作（即使之后有操作失败）都将直接影响数据库，这可能会导致数据状态不一致。
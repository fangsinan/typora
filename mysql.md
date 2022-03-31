# Mysql



```mysql
 DELIMITER &      #修改结束符号为 &
```

## 查询视图



## 存储过程

```mysql
# INOUT   入参出参
# OUT  只出参
# IN 只传入
CREATE PROCEDURE name(INOUT param type)
BEGIN

END ;

#调用
CALL name()

#------ 举例  ------ 
  DELIMITER //
  CREATE PROCEDURE test_pro()
  BEGIN

    SET @NUM = 0;
    loop_a:LOOP
      SET @NUM = @NUM+1;
      IF @NUM > 10 THEN LEAVE loop_a; END IF;
      END LOOP loop_a;

    select @NUM;
  END //
  DELIMITER ;
```

## 存储函数

```mysql
CREATE FUNCTION 函数名(参数名  类型)
RETURNS 返回值类型
#  LANGUAGE SQL|NOT DETERMINISTIC|SQL SECURITY DEFINER|COMMENT '...'  {函数特性}  

BEGIN
	函数体 #return
END

#调用
SELECT 函数名()
```

### 存储函数特性

~~~mysql
{存储函数特性的说明}
1. LANGUAGE SQL:
这个子句是没有功能作用的,仅仅是说明下面过程体中所使用的是SQL语言编写.这也是系统默认的.默认不代表完全可省略,因为某些DBMS需要它,如果要考虑IBM的DB2的兼容性可以写上,此外,今后可能会出现SQL外的其他语言支持的存储过程.

2. NOT DETERMINISTIC:
传递给系统的信息,即不确定的.如果一个存储过程每次调用只要参数一样,输出也一样,即为确定的.本例中的主体语句为SELECT语句,则返回值肯定是未知的,因此,此处为 NOT DETERMINISTIC.MySQL内置的优化程序不会注意这些,至少现在不注意.

3. SQL SECURITY DEFINER:
指定权限控制,指定在调用时如何认定调用方的权限:DEFINER表示不管调用方是谁,都拥有定义者的权限,INVOKER表示仅按调用方本身的权限调用.

4. COMMENT '...':

存储过程的注释说明文本.默认值为空('').
~~~



### 存储过程和函数的查看与修改

```mysql
#1、查看
SHOW CREATE PROCEDURE  存储过程名字;
SHOW CREATE FUNCTION   存储函数名字;


#2、查看状态信息
SHOW PROCEDURE STATUS; # LIKE  "名称" 可以模糊查询

#3、也可以从 information_schema.Routines 查看
SELECT * FROM information_schema.Routines WHERE  ROUTINE_NAME = '名字';



#修改存储过程或者函数。 不能修改其功能   只是修改相关的特性
ALTER PROCEDURE 名字 
SQL SECURITY INVOKER
COMMENT '修改后的描述';


#删除存储过程或者函数
DROP FUNCTION IF EXISTS 存储函数名字;
DROP PROCEDURE IF EXISTS  存储过程名字;
```



## mysql系统变量

### 系统变量

```mysql
# 两个 @@ 来标示系统变量    一个@是用户自定义变量

###########     查看系统变量
SHOW GLOBAL VARIABLES;
#SHOW GLOBAL VARIABLES LIKE 'admin_%';

#SHOW GLOBAL VARIABLES LIKE 'admin_%';

查看会话变量
SHOW SESSION VARIABLES;

SHOW VARIABLES; #默认查询会话变量


#查看系统指定一个系统变量  用两个 @@ 来标示系统变量    一个@是用户自定义变量
SELECT @@global.max_connections;
#查看 指定的一个会话变量
SELECT @@session.变量名;
 #先查询会话系统变量。再次查询全局系统变量
SELECT @@max_connections;


###########      修改全局系统变量

两种修改方式
SET @@global.max_connections = 161;
SET GLOBAL  max_connections = 171;


修改以上变量后 针对当前数据库实例有效。重启后失效


###########      修改会话系统变量

SET @@session.character_set_client = "gbk";
SET SESSION character_set_client = "utf8mb4";

修改以上变量后 针对当前数据库连接有效。


```

### 用户变量

```mysql
###########      用户会话变量
# 赋值 = 或者 :=
方式1:
SET @用户变量 = 变量值;
使用
SELECT @变量名;

方式2:
SELECT @count := COUNT(*) FROM tableName;
SELECT @count;
#其他用法
SELECT count(*) into @c FROM tableName;
SELECT @c;

以上变量当前连接有效


###########      用户局部变量
### {必须在存储过程或者存储函数中}
#定义
DECLARE 变量名 类型  DEFAULT 0 #如果没有DEFAULT。则默认为null
#赋值
SET 变量名 = 值;
SET 变量名 := 值;
#使用
SELECT 变量名


```

## 错误处理

**存储过程或者存储函数中使用**

### 错误名称定义（自定义错误码）

```mysql
#语法格式
DECLARE 错误名称 CONDITION FOR 错误码(或错误条件)

#定义  相当于为该错误码1048。自定义名称
 方式1: 
DECLARE error_field CONDITION FOR 1048;
 方式2:
DECLARE error_field CONDITION FOR SQLSTATE '42000';


```

### 定义错误捕捉程序

``` mysql
#语法 
DECLARE 处理方式 HANDLER FOR 错误类型 处理语句

#处理方式。 有三个值 CONTINUE EXIT UNDO
	 CONTINUE：表示遇到错误不处理。继续执行
   EXIT  	 ：遇到错误退出
   UNDO    ：遇到错误撤回之前的操作 mysql 暂时不支持
#错误类型
		SQLSTATE  '错误码' :表示长度为5的sqlstate_value类型的错误代码
		MYSQL_error_code : 匹配数值类型的错误代码
		错误名称 ： 表示自定义的错误名称{DECLARE 错误名称 CONDITION FOR 错误码(或错误条件)}
		SQLWARNING : 匹配所有01开头的sqlstate 错误代码
		NOT FOUND： 匹配所有的02开头的sqlstate 错误代码
		SQLEXCEPTION： 匹配所有没有被SQLWARNING和NOT FOUND捕捉到的错误码

#处理语句
	如果出现以上条件之一。则执行指定的错误语句。可以是简单的SET 变量=值。也可以是使用 BEGIN ... END 编写的复合语句

```

## 流程控制

### 1、分支结构 

#### 	1.1、if else

```mysql
#语法 ：
IF 分支条件  THEN 分支执行结果
ELSEIF ...  THEN 分支执行结果
ELSE ...  THEN 分支执行结果
END IF
```

#### 	1.2、CASE

```mysql
#语法：
CASE var
	WHEN 1 THEN ...
	WHEN 2 THEN ...
	ELSE ...
END CASE
```



### 2、循环结构

- **LOOP  : 一般用于简单的死循环**
- **WHILE : 先判断后执行**
- **REPEAT: 先执行一次 在进行判断**
- 循环名称(自定义) ：循环中的自定义名称可以省略。但是如果添加了 循环控制语句（LEAVE 或者 ITERATE） 则必须添加循环名称

#### 	2.1、LOOP

```mysql
# 需要在存储过程中使用
#loop_label 
循环名称(自定义):LOOP
	循环体
END LOOP 循环名称(自定义)

#退出循环 LEAVE loop_label

#---- 举例-- ----

DELIMITER //
CREATE PROCEDURE test_pro()
BEGIN

	SET @NUM = 0;
  loop_label:LOOP
  	SET @NUM = @NUM+1;
  	IF @NUM > 10 THEN LEAVE loop_label; END IF;
  	END LOOP loop_label;
  
	select @NUM;
END //
DELIMITER ;
```



#### 	2.2 WHILE

```mysql
循环名称(自定义 可忽略):WHILE 循环条件 DO
	循环体
END WHILE 循环名称(自定义 可忽略)


-- 举例

  DELIMITER //
  CREATE PROCEDURE test_pro()
  BEGIN
    SET @NUM = 0;
    WHILE @num < 10 DO
      SET @num = @num + 1;
      END WHILE;
    select @NUM;
  END //
  DELIMITER ;
  
  CALL test_pro()
```



#### 	2.3 REPEAT（执行一次循环体后在判断）

```mysql
# 语法
循环名称(自定义) REPEAT
	循环体
	UNTIL 结束循环的条件
	END REPEAT 循环名称(自定义)
	
#-------- 举例 --------
  DELIMITER //
  CREATE PROCEDURE test_pro()
  BEGIN
    SET @NUM = 0;

    REPEAT 
      SET @NUM = @NUM + 1;
      UNTIL  @NUM > 10
    END REPEAT;


    select @NUM;
  END //
  DELIMITER ;
```



#### 2.4 LEAVE和ITERATE的使用

- LEAVE   : 结束当前循环
- ITERATE : 

```mysql
```


































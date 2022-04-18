# Mysql基础



```mysql
 DELIMITER &      #修改结束符号为 &
```

### 创建临时表

```mysql
CREATE TEMPORARY  TABLE a_table
select sum(`view_num`) as sum_view from my_test_c;
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

#调用存储过程
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

#调用存储函数
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

​		关键字： UNTIL: 退出 REPEAT的循环

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



### 3、LEAVE和ITERATE的使用

- LEAVE   : 结束当前流程 （不一定非在循环里使用。也可以结束存储过程体、IF等）
- ITERATE : 结束本次循环 继续下一次（只能在循环里使用）



#### 	3、1 LEAVE-结束当前流程

~~~ mysql
# 举例  存储过程
DELIMITER //
CREATE PROCEDURE test_pro()
begin_lable: BEGIN
	DECLARE num int(10)  DEFAULT 0;
	IF num = 0 THEN LEAVE begin_lable;  #退出整个存储过程
		ELSE select 1;
	END IF;
END //
DELIMITER ;

# 举例  循环
DELIMITER //
CREATE PROCEDURE test_pro()
BEGIN
	DECLARE num int(10)  DEFAULT 0;

	while_lable : WHILE TRUE DO 
		IF num =10 THEN
			LEAVE while_lable; #退出 自定义为while_lable的循环 
		END IF;
		SET num = num + 1;
	END WHILE while_lable;
	select num;
END //
DELIMITER ;
~~~



#### 	3、2 ITERATE-退出本次循环 

~~~mysql
# 举例  循环 ITERATE
DELIMITER //
CREATE PROCEDURE test_pro()
BEGIN
	DECLARE num int(10)  DEFAULT 0;
	DECLARE count int(10)  DEFAULT 0;
	while_lable : WHILE TRUE DO 
		SET count = count + 1;
		SET num = num + 1;
		if num = 4 THEN 
			ITERATE while_lable;
		ELSEIF num = 10 THEN
			LEAVE while_lable; 
		END IF;
	END WHILE while_lable;
	select count;
END //
DELIMITER ;
~~~

## 关于游标

**注意**：MySQL游标只能用于存储过程和函数。 

游标(cursor)，是一个存储在MySQL服务器上的数据库查询，游标不是一条 SELECT语句，而是被该语句检索出来的结果集；可以看做是指向查询结果集的指针；通过cursor，就可以一次一行的从结果集中把行拿出来处理。



优势： 在存储程序中使用 逐条读取结果集中的数据，效率高 程序简洁

劣势： 性能问题 使用游标的过程 会对数据加锁，并发大的时候会影响业务之间的效率，因为是内存中处理，还会消耗系统资源，造成内存不足。



### 	1、游标的处理过程

 - **1、游标声明**  没有检索数据，只是定义要使用的select语句

 - ```mysql
   DECLARE cursor_name CURSOR FOR select_statement
   #声明一个游标cursor_name，让其指向查询select_statement的结果集。
   
   #注意： 游标声明必须出现在变量和条件声明的后面，但是在异常处理声明的前面
   #　   一个过程中可以有多个游标声明
   ```

   

- **2、打开游标** 打开游标以供使用，用上一步定义的select语句把数据实际检索出来

- ~~~mysql
  OPEN cursor_name;
  #cursor_name是声明中定义的名字；打开游标时才执行相应的select_statement。
  ~~~

  

- **3、检索使用游标**  对于填有数据的游标，根据需要取出(检索)各行

  ~~~mysql 
  FETCH cursor_name INTO var_name [, var_name] ...
  #这条语句是指使用cursor_name这个游标来读取当前行，将数据保存到var_name这个变量中，游标指针知道下一行，如果游标读取数据行有多个列，则在into 关键字后赋值给多个变量名即可
  比如： FETCH cur_emp into emp1,emp1,emp1
  
  #一次只拿一行，拿完后，自动移动指针到下一行；
  #如果没有拿到行，会抛出异常，其SQLSTATE代码值为‘02000’，此时要检测到该情况，需要声明异常处理程序 (针对条件NOT FOUND也可以)，通常需要在一个循环中来执行fetch语句，通过检测以上异常来结束循环。
  ~~~

​		

- **4、关闭游标** 在结束游标使用时，必须关闭游标 收回游标占用的内存

- ```mysql
  CLOSE cursor_name;
  ```





### 2、游标的使用实例

```mysql
#游标使用实例
DELIMITER //
CREATE PROCEDURE test_pro(OUT out_id int,OUT out_num varchar(255))
BEGIN
	DECLARE num int(10)  DEFAULT 0;
	DECLARE count int(10)  DEFAULT 10;
	DECLARE l_id int(10)  DEFAULT 0;
	DECLARE l_title varchar(255)  DEFAULT '';
	DECLARE sum_cursor  CURSOR FOR SELECT id,title FROM nlsg_live limit count;
	OPEN sum_cursor;
	while_lable : WHILE TRUE DO 
		IF num = count THEN
			LEAVE while_lable;
		END IF;
		FETCH sum_cursor INTO l_id,l_title;
		#update nlsg_live set reason = 0 where `id` = l_id;
		SET num = num + 1;
	END WHILE while_lable;
	SET out_id = l_id;
	SET out_num = l_title;
	#SELECT id,title;
	CLOSE sum_cursor;
	END //
DELIMITER ;
```



## 触发器

#### 1、1 触发器定义 

​	触发器是由事件来触发某个操作， 事件包括 insert update delete事件，

**优势：**

1. 保证数据完整性。
2. 触发器可以帮助记录操作日志
3. 触发器可以用在操作数据前，对数据进行合法性检测

**劣势：**

1. 可读性差。由于触发器是由事件驱动，不受应用层控制。对系统维护不友好
2. 相关数据表结构的变更，有可能会导致触发器错误。



#### 1、2 触发器使用

- 创建触发器

  ```mysql
  # 触发器创建语法
  CREATE TRIGGER name
  {BEFORE|AFTER} {INSERT|UPDATE|DELETE} ON 表名
  FOR EACH ROW
  触发执行的语句块
  
   #错误程序 固定格式
  SIGNAL SQLSTATE "错误码" SET MESSAGE_TEXT = '错误信息'
  
  
  # 举例
  DELIMITER //
  CREATE TRIGGER trigger_insert_live
  AFTER INSERT ON nlsg_live
  FOR EACH ROW
  BEGIN
  	# 自定义错误流程判断
  	IF NEW.id > 10 
  		THEN SIGNAL SQLSTATE "HY001" SET MESSAGE_TEXT = 'error id > 10'; #自定义校验
  	END IF;
  	INSERT INTO nlsg_live_info(live_pid) VALUE(NEW.id);  #NEW 为当前insert 数据对象
  END //
  DELIMITER ;
  ```

  说明：

  	+ BEFORE|AFTER  表示事件触发时间。BEFORE在事件之前触发 AFTER在事件之后触发。
  	+ 表名 ：表示触发器监控对象。
  	+ {INSERT|UPDATE|DELETE} ： 表示触发事件。
   + 语句块 ： 可以是单条 也可以是由BEGIN...END 结构组合的复合语句。
     + 自定义错误程序：  SIGNAL SQLSTATE "错误码" SET MESSAGE_TEXT = '错误信息';
     + NEW : 关键字new为触发事件（insert|update）数据的对象。
     + OLD : 关键字 为delete事件数据的对象  OLD值因为是删除操作，所以全部是只读的。

- 查看触发器

  ```mysql
  #方式一  查看当前数据库触发器信息
  SHOW TRIGGERS;
  #方式二  查看当前数据库某个触发器信息
  SHOW CREATE TRIGGER trigger_name;
  #方式三   查看系统库所有的触发器信息
  select * from information_schema.TRIGGERS;
  ```

  

- 删除触发器

  ~~~mysql
   #方式一
  DROP TRIGGER trigger_name;
   #方式二。 检索后删除   推荐使用
  DROP TRIGGER IF EXISTS trigger_name;
  ~~~

  





## Mysql8.0 的新特性



### 1、窗口函数（部分）

​	类似于group by排列，但是不会对数据进行聚合，依旧保持原有数据行数，对数据列进行规则排序

​	

- 语法结构

```mysql
#语法格式
 mysql函数 OVER ([PARTITION BY 字段名 ORDER BY 字段名 ASC｜DESC]) 
```

- 序号类型函数

  -在结果集中多现实一列 作为分组之后排序的列

  

  **ROW_NUMBNER() 函数   **

  ```mysql
  # ROW_NUMBER() 函数  
  #  - 在结果集中多现实一列 作为分组之后排序的列   条件值相同时进行不重复排序
  select * from (
  SELECT ROW_NUMBER() OVER(PARTITION BY type ORDER BY collection_num DESC) as row_num 
  ,id,type,name,collection_num,view_num,real_view_num from my_test_c
  )  t where 
  row_num  <= 3
  
  ```

  ![image-20220406101801289](assets/image-20220406101801289.png)

  **RANK() 函数 **

  ```mysql
  # RANK() 函数  
  # - 在结果集中多现实一列 作为分组之后排序的列  不同的是条件值相同时 进行重复值排序
  SELECT RANK() OVER(PARTITION BY type ORDER BY collection_num DESC) as row_num 
  ,id,type,name,collection_num,view_num,real_view_num from my_test_c
  
  ```

  ![image-20220406102410492](assets/image-20220406102410492.png)

​	**DENSE_RANK() 函数**

```mysql
# RANK() 函数  
# - 在结果集中多现实一列 作为分组之后排序的列  不同的是条件值相同时 进行重复排序，下一个会继续排序
select * from (
SELECT DENSE_RANK() OVER(PARTITION BY type ORDER BY collection_num DESC) as row_num 
,id,type,name,collection_num,view_num,real_view_num from my_test_c
)  t where 
row_num  <= 3
```

![image-20220406102640878](assets/image-20220406102640878.png)









# 高性能mysql





## MySql用户与权限控制



##### -- 刷新权限命令

```mysql
#	-- 刷新mysql权限命令
flush privileges;
```

### 用户管理

###### 	1、查看用户

```mysql
#查看用户
USE mysql;
SELECT host,user FROM user;
```

###### 	2、创建用户

```mysql
#创建用户
CREATE USER 'test1' identified by '000000';  #创建用户   默认host是%
CREATE USER 'test1'@'localhost' identified by '000000'   #创建本地host连接
```



###### 	3、修改用户

```mysql
# 3.1、修改用户
UPDATE user SET user ='test2' WHERE user ='test1' AND host = '%'


```

###### 	4、删除用户

```mysql
# 4、删除用户
DROP user 'test2';  #建议使用
DELETE FROM user WHERE user = 'test2' AND host = '%'; #此方式可能会有残留权限数据 删除不干净
```

###### 	5、修改密码

```mysql
#	3.2 修改当前链接用户的密码 
alter  user user() identified by 'new_password'; #写法一
SET PASSWORD= 'new_password';#写法二

#	3.3 修改其他用户的密码  root登陆后
alter user 'test1'@'%' identified by 'new_password'; #写法一
SET PASSWORD FOR 'test1'@'%' = '000000'; #写法二
```

###### 	6、密码过期策略

```mysql
#设置 test1 用户的密码立刻过期
alter user 'test1'@'%' password expire 

#设置密码90天过期
create user "new_user"@"localhost" PASSWORD EXPIRE INTERVAL 90 DAY;  #新建用户
alter user "new_user"@"localhost" PASSWORD EXPIRE INTERVAL 90 DAY;  #修改用户

#设置密码永不过期
create user "new_user"@"%" PASSWORD EXPIRE NEVER;  #新建用户
alter user "new_user"@"%" PASSWORD EXPIRE NEVER;  #修改用户
```

### 权限管理

#### 	1、查看权限

```mysql
SHOW GRANTS 
```

	#### 	2、赋予权限

```mysql
# 赋予用户 所有权限   
grant ALL PRIVILEGES ON *.* TO 'new_user'@'%';


# 赋予用户 对 test库的查询和修改权限 
grant select,update  on test.* to 'new_user'@'%';
# 叠加赋予用户
grant delete  on test.* to 'new_user'@'%';
```


































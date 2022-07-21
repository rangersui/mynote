[toc]
# 创建和管理schema
## 背景信息
schema又称作模式。通过管理schema，允许多个用户使用同一数据库而不相互干扰，可以将数据库对象组织成易于管理的逻辑组，同时便于将第三方应用添加到相应的schema下而不引起冲突。 

管理schema包括：创建schema、使用schema、删除schema、设置schema的搜索路径以及schema的权限控制。

## 注意事项

- 数据库集群包含一个或多个已命名数据库。用户和用户组在整个集群范围内是共享的，但是其数据并不共享。任何与服务器连接的用户都只能访问连接请求里声明的那个数据库。
- 一个数据库可以包含一个或多个已命名的schema，schema又包含表及其他数据库对象，包括数据类型、函数、操作符等。同一对象名可以在不同的schema中使用而不会引起冲突。例如，schema1和schema2都可以包含一个名为mytable的表。
- 和数据库不同，schema不是严格分离的。用户根据其对schema的权限，可以访问所连接数据库的schema中的对象。进行schema权限管理首先需要对数据库的权限控制进行了解。
- 不能创建以PG_为前缀的schema名，该类schema为数据库系统预留的。
- 在初始数据库gaussdb中创建用户时，系统会为新用户创建一个同名Schema。在其他数据库中，若需要同名Schema，则需要用户手动创建。
- 通过未修饰的表名（名字中只含有表名，没有“schema名”）引用表时，系统会通过search_path（搜索路径）来判断该表是哪个schema下的表。pg_temp和pg_catalog始终会作为搜索路径顺序中的前两位，无论二者是否出现在search_path中，或者出现在search_path中的任何位置。search_path（搜索路径）是一个schema名列表，在其中找到的第一个表就是目标表，如果没有找到则报错。（某个表即使存在，如果它的schema不在search_path中，依然会查找失败）在搜索路径中的第一个schema叫做"当前schema"。它是搜索时查询的第一个schema，同时在没有声明schema名时，新创建的数据库对象会默认存放在该schema下。
- 每个数据库都包含一个pg_catalog schema，它包含系统表和所有内置数据类型、函数、操作符。pg_catalog是搜索路径中的一部分，始终在临时表所属的模式后面，并在search_path中所有模式的前面，即具有第二搜索优先级。这样确保可以搜索到数据库内置对象。如果用户需要使用和系统内置对象重名的自定义对象时，可以在操作自定义对象时带上自己的模式。

## 创建schema
```sql
CREATE SCHEMA myschema;
```
指定owner
```sql
CREATE SCHEMA myschema AUTHORIZATION dbadmin;
```
在myschema下创建mytable
```sql
CREATE TABLE myschema.mytable(id int,name varchar(20));
```
设置schema的搜索路径，可以设置search_path配置参数指定寻找对象可用schema的顺序。在搜索路径列出的第一个schema会变成默认的schema。

如果在创建对象时不指定schema，则会创建在默认的schema中。执行如下命令将搜索路径设置为myschema、public，首先搜索myschema。
```sql 
SET SEARCH_PATH TO myschema, public;
```

## 权限控制
默认情况下，用户只能访问属于自己的schema中的数据库对象。如果需要访问其他schema的对象，则该schema的所有者应该赋予他对该schema的usage权限。

通过将模式的CREATE权限授予某用户，被授权用户就可以在此模式中创建对象。

查看现有的schema:
```sql
SELECT current_schema();
```

设置现有schema:
```sql
set current_schema='retail_obs_data';
```

执行如下命令创建用户jack，将myschema的usage权限赋予jack
```sql
CREATE USER jack IDENTIFIED BY 'Bigdata@123';
CREATE USER
GRANT USAGE ON schema myschema TO jack;
GRANT
```
收回jack的权限
```sql
REVOKE USAGE ON schema myschema FROM jack;
```

## 删除schema

当schema为空，可以使用DROP SCHEMA命令删除
```sql
DROP SCHEMA IF EXISTS nulschema;
```
当schema非空，如果要删除一个schema以及其所有对象，使用CASCADE关键词。
```sql
DROP SCHEMA myschema CASCADE;
```

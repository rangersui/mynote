[toc]

本文翻译自：learn sql the hard way，有删节，主要为了自己看着方便。

# 基本概念

- 数据库 database
- 表 table
- 模式 schema 
  - 关于数据库和表的布局以及特性的信息
- 列 column 
  - 表中的一个字段
- 数据类型 data type
- 行 row 
  - 表中的一个记录
- 主键 primary key
  - 任意两行都不具有相同的主键值
  - 每一行都必须有一个主键值（主键列不允许NULL值）
  - 主键列中的值不允许修改或更新
  - 主键值不能重用，如果某行从表中删除，它的主键值分配给新行
- 外键 foreign key 
  - 值必须列在另一表的主键中，外键是保证引用完整性的重要部分。
- 关键字 key word 
  - 作为SQL组成部分的保留字，不能作为表/列的名字
- 子句 clause
  -  SQL语句由子句构成，一个子句通常由一个关键字加上所提供的数据组成，有的子句是必须的，有的可选
- 操作符 operator
  - 用来联结或改变WHERE 子句中的子句的关键字，也称为逻辑操作符(logical operactor)
- 通配符 wildcard
  - 用来匹配值的一部分的特殊字符
- 搜索模式 search pattern
  - 由字面值、通配符或两者组合构成的搜索条件

# 创建表

```sql
CREATE TABLE person(
    id INTEGER PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    age INTEGER
);
```

## 练习

1. 把上述语句全改成小写，再次建立db

2. 添加两个和person有关的INTEGER和TEXT两个字段

   ```sql
   CREATE TABLE person(
       id INTEGER PRIMARY KEY,
       first_name TEXT,
       last_name TEXT,
       age INTEGER,
       weight INTEGER,
       address TEXT
   );
   ```

## 注解

SQLLite3的类型基本和其他数据库一致，但需要注意SQL数据库供应商会融合和扩展某些数据类型，其中最大的重灾区是日期和时间类型。

# 创建多表数据库

```sql
CREATE TABLE person (
	id INTEGER PRIMARY KEY,
    first_name TEXT,
    last_name TEXT,
    age INTEGER
);

CREATE TABLE pet(
	id INTEGER PRIMARY KEY,
    name TEXT,
    breed TEXT,
    age INTEGER,
    dead INTEGER
);

CREATE TABLE person_pet(
	person_id INTEGER,
    pet_id INTEGER
);
```

在这个文件里，你构建了有两种数据类型的多个表，并用第三个表把它们连接在一起。人们称这些“连接”为“关系”，有些人会把所有的表都称为“关系”。在本文中，有数据的表被叫做表，哪些连接其他表在一起的表称为“关系”

在person_pet表中你生成了两个列：person_id和pet_id。把这两个表连接在一起，只需要简单的插入一行person_id和pet_id到person_pet即可。

在后面的练习我们会接触到如何插入数据。

## 练习

1. 如何把关系表person_pet的信息直接放到person表中？这种变化意味着什么？

2. 如果你能把一行加入到person_pet，你能加入更多吗？你如何记录一个拥有50只猫的猫咪狂热爱好者？*创建另一个表来记录人们可能有的猫，然后创建相关的关系表。
3. 在网上搜索sqlite3的数据类型，并阅读"Datatypes in SQLite Version 3"文档。记录下你能用到的类型并且看看还有什么重要的内容。

## 注解

数据库有很多选项来指明这些关系中的键，单目前我们简化这些内容。

# 插入数据

```sql
INSERT INTO person (id, first_name, last_name, age)
VALUES(0, 'Zed', 'Shaw', 37);

INSERT INTO pet (id, name ,breed, age, dead)
VALUES(0, 'Fluffy', 'Unicorn', 1000, 0);

INSERT INTO pet
VALUES(1, 'Gigantor', 'Robot', 1, 1);
```

在这个文件中我使用了两种INSERT指令，第一个形式更加明确，你也应该使用这种形式，它指定了要被插入的列，跟随VALUES包含被插入的数据。这些列（列名和值）在括号内以逗号分隔。

第二种缩写版没有指明列而是依赖于表中的隐含顺序，这种形式是危险在你不知道你究竟在访问哪一列，有些数据库没有可靠的的列顺序，最好别用这种形式。

## 练习

1. 插入你自己和你的宠物信息（或者想象中的宠物）
2. 如果在上一个练习中你改变了数据库，那么创建一个新的含有该schema的数据库并插入同样信息

## 注解

正如在上一个练习中提到的，数据库厂商倾向于通过扩展或改变所使用的数据类型来增加对平台的依赖。比如稍微改变TEXT列或把DATETIME列改名为TIMESTAMP然后增加不同格式。使用不同数据库时请注意这点。

# 插入关系数据

在上一个练习，你填写了人和宠物的数据。唯一缺失的就是谁拥有哪些宠物，这些数据应该被填入person_pet表：

```sql
INSERT INTO person_pet (person_id, pet_id) VALUES (0, 0);
INSERT INTO person_pet VALUES (0, 1);
```

当你建立一个这样的表时，你正在创建一个“关系”。技术上来讲每一个表都是一个“关系”，但是我认为这让人迷惑，因为只有某些表真正用于建立表之间的关系。person_pet表使得你关联person中的数据到pet中的数据。稍后你将会学到用等价表达式来进行联表查询。在本书中，本书只会覆盖基础版本的joins（联表）, relations和foreign keys因为这些是高级主题，在不同的数据库之间用法也不够统一。

## 练习

1. 给你和你的宠物添加关系。
2. 使用这张person_pet表，一个宠物能被多个人所拥有吗？这在逻辑上可行吗？一个家庭的宠物狗怎么办？家里的每个人都能在技术上拥有家庭宠物狗吗？
3. 基于以上问题，假设你有一个替代设计来把pet_id放进person table，哪一种设计应用在以上场景更好？

# 选择数据

```sql
SELECT * FROM person;

SELECT name, age FROM pet;

SELECT name, age FROM pet WHERE dead = 0;

SELECT * FROM person WHERE first_name != 'Zed';
```

第一条SQL：代表从person表选择所有列表并返回所有行（记录）。其中WHERE子句是可选的，'*'字符代表选中所有列

第二条SQL：此处代表只从pet表请求两列name和age，并返回所有行。

第三条SQL：和第二条目的一致，但添加了限定WHERE dead=0。这将返回所有没有死亡的宠物。

第四条SQL：从person表返回所有列但first_name不等于Zed，WHERE子句决定了某行是否返回。

## 练习

1. 编写查询，找到所有大于10岁的宠物。
2. 编写查询，找到所有比你年轻的人，再查询所有比你年长的人。

3. 编写查询，在WHERE子句中使用多于一条的测试，使用AND连接。例如：`WHERE first_name = 'Zed' AND age > 30;`
4. 编写查询，使用AND和OR操作符，用三列搜索行。

## 注解

有些数据库有额外的操作符和布尔逻辑，但暂时你只要关心大多数编程语言中存在的即可。

# 联表

希望你对从表中选取数据有所概念，永远记住：SQL关心表！SQL独爱表！SQL返回表！表！表！表！你应该从这疯狂强调中认识到，你从编程中得来的知识在这里没有帮助。在编程中，你处理图，在SQL中你处理表。他们是关联的概念，但是心智模式是不同的。

这里有一个例子来解释有何不同：比如你想要知道Zed拥有哪些宠物，你需要写SELECT来查找person表然后通过一些方式来找到Zed的宠物。为了做到这点，你需要查询person_pet表来获取你需要的id列。为了做到这点，你需要用等价表达式连接这三张表，或者你需要第二次SELECT来返回正确的pet.id编号

```sql
SELECT pet.id, pet.name, pet.age, pet.dead 
#我只想得到pet中特定的列，所以不用*而是指定特定列形如talbe.column
	FROM pet, person_pet, person 
	#为了连接宠物到个人，我需要经过person_pet关系表，在SQL中代表着我需要在FROM后列出这三张表
	WHERE
	#开始WHERE子句
	pet.id = person_pet.pet_id AND
	#通过关联pet.id和person_pet.id连接pet到person_pet
	person_pet.person_id = person.id AND
	#AND 通过同样的方式连接person到person_pet，此时数据库只能搜索到id列匹配的已连接数据
	person.first_name = 'Zed';
	#AND 通过添加person.first_name检测查询Zed的宠物
	
SELECT pet.id, pet.name, pet.age, pet.dead
	FROM pet
	#只需要pet表在“主选择”中因为我们使用了IN关键词来启动一个子选择来获得我们需要的pet.id
	WHERE pet.id IN
	#在小括号中开始子选择
	(
        SELECT pet_id FROM person_pet, person
        #对person和person_pet用一个更简单的连接获得正确的pet_id
        WHERE person_pet.person_id = person.id
        #需要子句来建立需要的等价式，但只需要关心person.id和person_pet.person_id相匹配即可
        AND person.first_name = 'Zed'
        #用限定条件'Zed'来获得Zed的宠物
    );
```

## 练习

1. 花些时间来使用类和对象来构建相同的关系模型然后建立相同的数据（map it to this setup）
2. 查询你目前为止增加的宠物
3. 改变查询用你自己的person.id来代替上面的person.name

## 注解

其实有其他方法来完成上面称为联表的工作，我避免使用这些概念，因为他们极其容易让人混淆。现在只要坚持这样的联表查询方式，不要理会那些告诉你这样慢和低级的人。

# 删除数据

这个练习相当简单，但是希望你能在写下这些代码之前三思。如果你已知`SELECT * FROM`来选择，`INSERT INTO	`来插入数据，那么你在删除的时候应该怎么写？你可能会继续往下看，但要猜想它将是怎么写的，然后再看。

```sql
#确保有死掉的宠物
SELECT name, age FROM pet WHERE dead = 1;

#噢，可怜的robot
DELETE FROM pet WHERE dead = 1;

#确保robot已经被删除
SELECT * FROM pet;

#让我们复活robot
INSERT INTO pet VALUES (1, 'Gigantor', 'Robot', 1, 0);

#再检查一下
SELECT * FROM pet;
```

我单纯的实现了一个非常复杂的更新，通过删除robot然后把记录重新插入到表中但是dead置为0。在之后的练习中我会向你展示如何使用UPDATE关键字来做到这件事，所以不要用这种方法来更新数据。

大多数的脚本对你来讲已经很熟悉了，除了第二条。可以把`DELETE FROM table WHERE tests`当作一种删除记录的SELECT，在WHERE子句中可行的所有条件在这里都可行。

# 通过其他表来删除数据

记住我所说的，DELETE就是一个会删除记录的SELECT，但是你一次只能从一个表中删除记录。这意味着删除所有宠物你需要做一些额外的查询然后在这基础上进行删除。

其中一种方式就是使用子查询来选择你想要删除的id。还有其他方法也能做到这点，但是下面的方法基于你已经知道的知识：

```sql
DELETE FROM pet WHERE id IN(
	SELECT pet.id
    FROM pet, person, person
    WHERE
    person.id = person_pet.person_id AND
    pet.id = person_pet.pet_id AND
    person.first_name = 'Zed'
);

SELECT * FROM pet
SELECT * FROM person_pet;

DELETE FROM person_pet
	WHERE pet_id NOT IN(
    	SELECT id FROM pet
    );
    
SELECT * FROM person_pet;
```

## 注解

取决于数据库，子查询可能会比较慢。

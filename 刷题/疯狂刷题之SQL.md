1. 表1: `Person`

   ```
   +-------------+---------+
   | 列名         | 类型     |
   +-------------+---------+
   | PersonId    | int     |
   | FirstName   | varchar |
   | LastName    | varchar |
   +-------------+---------+
   PersonId 是上表主键
   ```

   表2: `Address`

   ```
   +-------------+---------+
   | 列名         | 类型    |
   +-------------+---------+
   | AddressId   | int     |
   | PersonId    | int     |
   | City        | varchar |
   | State       | varchar |
   +-------------+---------+
   AddressId 是上表主键
   ```

    

   编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

   ```sql
   FirstName, LastName, City, State
   ```

   解答：

   ```mysql
   select `FirstName`,`LastName`,`City`,`State`
   from Person as p left outer join Address as a
   on p.`PersonId` = a.`PersonId`
   ```

   分析：考题内外连接，默认情况下使用 `join ... on` 表示的是内连接，外连接需要使用 `outer join ...on` ，外连接又分为左外连接和右外链接，区别在于加左表还是加右表。



2. 编写一个 SQL 查询，获取 `Employee` 表中第二高的薪水（Salary） 。

   ```
   +----+--------+
   | Id | Salary |
   +----+--------+
   | 1  | 100    |
   | 2  | 200    |
   | 3  | 300    |
   +----+--------+
   ```

   例如上述 `Employee` 表，SQL查询应该返回 `200` 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 `null`。

   ```
   +---------------------+
   | SecondHighestSalary |
   +---------------------+
   | 200                 |
   +---------------------+
   ```

   解答：

   ```mysql
   select ifnull(
       (select distinct(Salary)
       from Employee
       order by `Salary` desc
       limit 1,1),
       null
   ) as `SecondHighestSalary`
   ```

   分析：按照工资降序查询员工，然后通过 `limit` 查询第二条记录，如果非空返回，如果为空则返回空。



3. `Employee` 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

   ```
   +----+-------+--------+-----------+
   | Id | Name  | Salary | ManagerId |
   +----+-------+--------+-----------+
   | 1  | Joe   | 70000  | 3         |
   | 2  | Henry | 80000  | 4         |
   | 3  | Sam   | 60000  | NULL      |
   | 4  | Max   | 90000  | NULL      |
   +----+-------+--------+-----------+
   ```

   给定 `Employee` 表，编写一个 SQL 查询，该查询可以获取收入超过他们经理的员工的姓名。在上面的表格中，Joe 是唯一一个收入超过他的经理的员工。

   ```
   +----------+
   | Employee |
   +----------+
   | Joe      |
   +----------+
   ```

   解答：

   ```mysql
   select e.`Name` as `Employee`
   from Employee as e join Employee as m
   on e.`ManagerId` = m.`Id`
   where e.`Salary` > m.`Salary`
   ```

   分析：对于这种能使用自连接解决的问题最好别用子查询，子查询的效率会比自连接的低，而且可读性也大大降低。



4. 编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

   **示例：**

   ```
   +----+---------+
   | Id | Email   |
   +----+---------+
   | 1  | a@b.com |
   | 2  | c@d.com |
   | 3  | a@b.com |
   +----+---------+
   ```

   根据以上输入，你的查询应返回以下结果：

   ```
   +---------+
   | Email   |
   +---------+
   | a@b.com |
   +---------+
   ```

   **说明：**所有电子邮箱都是小写字母。

   解答：

   ```mysql
   select Email
   from Person
   group by Email
   having count(Email) > 1
   ```

   分析：查重复数据的思路大致都是这样，先使用 `group by` 对这个数据所对应的字段进行分组，然后通过 `having` 添加分组条件，数量大量1的就是出现重复的。



5. 某网站包含两个表，`Customers` 表和 `Orders` 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

   `Customers` 表：

   ```
   +----+-------+
   | Id | Name  |
   +----+-------+
   | 1  | Joe   |
   | 2  | Henry |
   | 3  | Sam   |
   | 4  | Max   |
   +----+-------+
   ```

   `Orders` 表：

   ```
   +----+------------+
   | Id | CustomerId |
   +----+------------+
   | 1  | 3          |
   | 2  | 1          |
   +----+------------+
   ```

   例如给定上述表格，你的查询应返回：

   ```
   +-----------+
   | Customers |
   +-----------+
   | Henry     |
   | Max       |
   +-----------+
   ```

   解答：

   ```mysql
   select c.`Name` as Customers
   from Customers as c
   where c.`Id` not in(
       select c.`Id`
       from Customers as c join Orders as o
       on c.`Id` = o.`CustomerId`
   )
   ```

   分析：子查询，先查询有订单记录的，再查不在这个查询结果里面的就是没有订单记录的。
##  JPA

---

### 一、JPA 概述

`JPA` 是 `Java Persistence API` 的缩写，用于对象持久化的 `API` ，是 `Java EE 5.0` 平台标准的 `ORM` 规范。它只是对 `ORM` 的抽象，并没有提供具体的实现类（就好像 `JDBC` 一样，具体的实现还是由具体的数据库厂商来制定，然后在项目中引入对应的数据库驱动就可以使用具体的实现）。其中，最为著名的实现框架就是 `Hibernate` 了。因为最近的项目需要使用到 `Spring Data JPA` ，所以就打算回顾一下原生的 `JPA` 以及 `Hibernate` ，这样才能更好的使用并理解 `Spring Data JPA` 。

---

### 二、从一个 HelloWorld 开始

#### 步骤一. 引入jar包

引入需要用到 `jar` 包，包括 `Hibernate`，`MySQL` 驱动

![](C:\Users\jonas\Desktop\JavaCore\JPA\img\jpa-20190501154658.png)

#### 步骤二. 配置数据库

如果使用 `Spring` 官方提供的 `STS` ，直接创建 `JPA` 工程里面就会自动生成一个 `JPA` 专属的配置文件：`persistence.xml` ：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.0"
	xmlns="http://java.sun.com/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
	<!-- 
		name属性：必需的属性，用于标识一个持久化单元，默认为项目名；
		transaction-type属性：可选属性，用于指定JPA的事务处理策略。
			默认值为RESOURCE_LOCAL，只能针对一种数据库，不支持分布式事务
			如果需要支持分布式事务，使用 JTA：transaction-type="JTA"
		-->
	<persistence-unit name="JPA-Test" transaction-type="RESOURCE_LOCAL">
		<!-- 指定ORM框架的实现类，若项目只有一个实现则可以省略 -->
		<provider>org.hibernate.ejb.HibernatePersistence</provider>
		<properties>
			<!-- 连接数据库的基本配置 -->
			<property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
			<property name="javax.persistence.jdbc.url" value="jdbc:mysql:///jpa"/>
			<property name="javax.persistence.jdbc.user" value="root"/>
			<property name="javax.persistence.jdbc.password" value="123456"/>
			<!-- Hibernate的配置 -->
			<property name="hibernate.show_sql" value="true"/>
			<property name="hibernate.format_sql" value="true"/>
			<property name="hibernate.hbm2ddl.auto" value="update"/>
		</properties>
	</persistence-unit>
</persistence>
```

#### 步骤三. 编写实体类

```java
package com.jonas.jpa.pojo;
//注意别导错包
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

//注解@Entity用于标注一个持久化类
@Entity
//注解@Table用于将这个持久化类与数据库的某个表建立映射关系，其中name属性表示数据库中的表。默认情况下会映射为持久化类的类名小写的表
@Table(name="customer")
public class Customer {
	private Integer id;
	private String lastName;
	private String email;
	private int age;
	
    //注解@GeneratedValue用于标识主键的生成策略，默认为GenerationType.AUTO
	@GeneratedValue(strategy=GenerationType.AUTO)
    //注解@Id用于标识主键
	@Id
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
    //@Column用于建立数据表字段与持久化类属性的映射关系。在默认情况下，如果数据表的字段名跟持久化类的属性名一致的时候则会自动建立映射关系，在不一致的情况可以使用@Column进行配置
    @Column(name="last_name")
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
}
```

然后在配置文件中添加一条配置：

```xml
<!--持久化类的全类名-->
<class>com.jonas.jpa.pojo.Customer</class>
```

#### 步骤四. 编写测试程序

```java
public class TestJPA {
	public static void main(String[] args) {
		String persistenceUnitName = "JPA-Test";
		//1.创建EntityManagerFactory对象
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory(persistenceUnitName);
		//2.创建EntityManager对象
		EntityManager entityManager = entityManagerFactory.createEntityManager();
		//3.开启事务
		EntityTransaction transaction = entityManager.getTransaction();
		transaction.begin();
		//4.执行持久化操作
		Customer customer = new Customer();
		customer.setAge(18);
		customer.setEmail("jonas_von@163.com");
		customer.setLastName("jonas");
		entityManager.persist(customer);
		//5.提交事务
		transaction.commit();
		//6.关闭资源
		entityManager.close();
		entityManagerFactory.close();
	}
}
```

运行程序后，发现数据库中新建了一个 `customer` 表，并且有一条 `jonas` 的记录。这就是一个简单的 `HelloWorld` 。简单一句话回顾上面的过程：通过 `JPA` 与 `Hibernate` 建立了持久化类与数据表的映射，通过操作持久化对象从而操作数据库，这大概就是 `ORM` 的思想吧，接下来就好好研究一下 `JPA` 吧。

---

### 三、JPA的注解

#### 1. @Entity

注解 `@Entity` 标注在类上以标识这个类为持久化类。

#### 2. @Table

注解 `@Table` 标注在类上以建立持久化类与数据表的映射。在默认情况下，会将持久化类与持久化类名一致（首字母小写）的表建立映射关系。如果我们需要自定义持久化类与表的映射就可以通过注解 `@Table(name="tableName")`来显示指定映射关系。 

#### 3. @Id

注解 `@Id` 是必须的，用于表示一个持久化类的主键，可以标注在主键对应属性或者对应的 `get` 方法上。

#### 4. @GeneratedValue

注解 `@GeneratedValue` 标注在主键对应的属性或对应的 `get` 方法上，用于指定主键的生成策略。

```java
//注解@GeneratedValue用于标识主键的生成策略，默认为GenerationType.AUTO
@GeneratedValue(strategy=GenerationType.AUTO)
@Id
public Integer getId() {
    return id;
}
```

`strategy` 属性的可选值：

- `GenerationType.IDENTITY`，采用数据库自增，`Oracle` 不支持这种方式
- `GenerationType.AUTO`，自动选择，默认值。
- `GenerationType.SEQUENCE`，通过序列产生主键，通过 `@SequenceGenerator` 注解指定序列名，`MySQL` 不支持这种方式。
- `GenerationType.TABLE`，通过一张表产生主键。

#### 5. @Basic

注解 `@Basic` 表示一个简单的属性到数据表字段的映射，对于没有任何标注的`get`方法，默认标注为 `@Basic` 。

#### 6. @Column

注解 `@Column` 用于建立数据表字段与持久化类属性的映射关系。默认情况下，如果数据表中的字段与持久化类中的属性完全一致，则可以省略该注解。如果需要自定义映射规则，那么就使用该注解，比如：持久化类上的某个属性 `lastName` 对应着表中的字段 `last_name` ，那么就可以在其对应的 `get`方法上进行配置：

```java
@Column(name="last_name")
public String getLastName() {
    return lastName;
}
```

除了 `name` 属性以外，还可以使用 `length` 属性指定字段的长度；`unique` 属性指定字段唯一；`nullable`属性指定字段是否可以为`null`。

#### 7. @Transient

有时候一个持久化类的某些属性不需要跟数据表的字段进行映射，该属性可能只在类中作用工具存在，所以，这样的属性就不必跟数据库建立起任何联系，这时候就可以使用注解 `@Transient` 来标识该属性。

#### 8. @Temporal

专门用于处理数据库中的时间字段，用法如下：

```java
@Temporal(TemporalType.TIMESTAMP)//精确到时分秒
public Date getCreatedTime() {
    return createdTime;
}

@Temporal(TemporalType.DATE)//精确到某年某月
public Date getBirth() {
    return birth;
}
```

---

### 四、JPA 相关的接口或类

在前面 `HelloWorld` 的例子里面，其实已经使用到非常多的接口，这里就来详细看看：

#### 1. Persistence

```java
//创建EntityManagerFactory对象
		EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory(persistenceUnitName);
```

在前面例子的第一步就是通过这个类来创建一个 `EntityManagerFactory` 。该类只有几个方法，其中最重要的就是这个。

#### 2. EntityManagerFactory

`EntityManagerFactory` 跟 `Hibernate` 中的 `SessionFactory` 是类似的，都是负责初始化工作和建立一个可以操作数据库的实例。

重要方法：

`createEntityManager()` ，用于创建 `EntityManager` 

`close()`，关闭资源。

#### 3. EntityManager

在 `JPA` 规范中，`EntityManager` 是完成持久化操作的核心对象。

重要方法：

`find` ，返回指定的 `OID` 对应的实体对象。与 `Hibernate` 中的 `get` 方法类似。

`getReference` ，懒查询，与 `Hibernate` 中的 `load()` 方法类似，通过代理对象去查询。

`persist` ，用于将新创建的 `Entity` 转为持久化对象。

`remove` ，删除实例。

`flush`，同步持久化上下文环境，将所有未保存的状态信息保存到数据库中。

`getTransaction` ，返回资源层的事务对象。

`close`，关闭实体管理器。


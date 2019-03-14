## JPA 框架

### 使用指南

1. #### maven 引入

   1. jpa
   2. lombok 自动构建getter , setter

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-data-jpa</artifactId>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.16.10</version>
               <scope>provided</scope>
           </dependency>
   ```

2. #### 实体类与仓库类配置

   ```java
   @Entity
   @Table(name = "user")
   @Setter   //lombok
   @Getter   //lombok
   public class User implements Serializable {
       @Id   //主键
       @Column(name = "user_id", nullable = false, length = 30)
       private String userId;
       @Column(name = "password")
       private String password;
   
       public User(String userId) {
           this.userId = userId;
       }
   
       public User() {
       }
   }	
   ```

[][]

> [实体类的注解][https://blog.csdn.net/Mr_DouDo/article/details/79380642]
>
> [关联映射][https://blog.csdn.net/johnf_nash/article/details/80642581]

这里详细介绍一下关联映射的内容

- @OneToOne  一对一

```java
---外键方式
people 表（id，name，sex，birthday，address_id）

address 表（id，phone，zipcode，address）

@Entity
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;//id
    @Column(name = "name", nullable = true, length = 20)
    private String name;//姓名
    @Column(name = "sex", nullable = true, length = 1)
    private String sex;//性别
    @Column(name = "birthday", nullable = true)
    private Timestamp birthday;//出生日期
    @OneToOne(cascade=CascadeType.ALL)//People是关系的维护端，当删除 people，会级联删除 address
    @JoinColumn(name = "address_id", referencedColumnName = "id")//people中的address_id字段参考address表中的id字段
    private Address address;//地址
}

@Entity
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;//id
    @Column(name = "phone", nullable = true, length = 11)
    private String phone;//手机
    @Column(name = "zipcode", nullable = true, length = 6)
    private String zipcode;//邮政编码
    @Column(name = "address", nullable = true, length = 100)
    private String address;//地址
    //如果不需要根据Address级联查询People，可以注释掉
//    @OneToOne(mappedBy = "address", cascade = {CascadeType.MERGE, CascadeType.REFRESH}, optional = false)
//    private People people;
}

---关联表/映射表
people 表（id，name，sex，birthday）

address 表 (id，phone，zipcode，address）

people_address (people_id，address_id)
           
@Entity
public class People {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;//id
    @Column(name = "name", nullable = true, length = 20)
    private String name;//姓名
    @Column(name = "sex", nullable = true, length = 1)
    private String sex;//性别
    @Column(name = "birthday", nullable = true)
    private Timestamp birthday;//出生日期
    @OneToOne(cascade=CascadeType.ALL)//People是关系的维护端
    @JoinTable(name = "people_address",
            joinColumns = @JoinColumn(name="people_id"),
            inverseJoinColumns = @JoinColumn(name = "address_id"))//通过关联表保存一对一的关系
    private Address address;//地址
}
```

可以看到，通过外键& @JoinColumn 可以进行

通过中间表& @JoinTable

- 一对多 、多对一

被维护端（一）用 @OneToMany(mappedBy = "author",cascade=CascadeType.ALL,fetch=FetchType.LAZY)

维护端（多）用@ManyToOne 

并且用 @JoinColumn 在[多]中配置外键

- 多对多

需要一个关联表来维护关系

​	关联表

```java
@ManyToMany
    @JoinTable(name = "user_authority",joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "authority_id"))
  private List<Authority> authorityList;
```

- #### 小结

  ###### 一对一

```java
// 1. OneToOne
class project{
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "CONTRACT_ID", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
    //@JSONField(serialize = false)
    private Contract contract;
    
    //---------------------------------
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "TESTREPORT_ID", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
    //@JSONField(serialize = false)
    private TestReport testReport;
...
}

class Contrast{
    
    @OneToOne(mappedBy = "contract")
    @JSONField(serialize = false)
    private Project project;
}

class TestReport{
    @OneToOne(mappedBy = "testReport")
    @JoinColumn(name = "TESTREPORT_ID",foreignKey = @ForeignKey(name = "none",value = ConstraintMode.NO_CONSTRAINT))
    @JSONField(serialize = false)
    private Project project;
}
```

###### 	多对一

```java
class User{
    @OneToMany(fetch = FetchType.LAZY)
	@JoinColumn(name = "USER_ID",foreignKey = @ForeignKey(value = ConstraintMode.CONSTRAINT))
	@JSONField(serialize = false)
	private List<Contract> contracts;
}

class Contrast{
    /**
     * 用户信息
     */
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "USER_ID", foreignKey = @ForeignKey(value = ConstraintMode.CONSTRAINT))
    @JSONField(serialize = false)
    private User user;
}
```

###### 	多对多 (含有中间表)

```java
class User{
    @ManyToMany(fetch = FetchType.EAGER)
	@JoinTable(name = "TBL_SYS_ROLE_USERS", joinColumns =
	{
		@JoinColumn(name = "USER_ID", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
	}, inverseJoinColumns = 
	{
		@JoinColumn(name = "ROLE_ID", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
	})

	/**
	 * 用户所对应的角色组
	 */
	private List<Role> roles;
}

class Role{
    
}
```


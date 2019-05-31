## POJO

### 前言

动手写这个，主要是因为自己 `JPA` 感觉掌握的不是很扎实，对于一些高级的级联操作没有什么接触，借这个机会好好整理一下

### 表间映射

首先需要了解一个 `维护与被维护` 的这个关系

- 维护者：能够拥有更大的权利，是一对多中的一，也是多对多中的占有主导权的那一方

  维护者的删除，将会导致所有被维护者全部被 **级联删除** ；

- 被维护者：只能够简单拥有一个维护者的引用；

  作为一对多中的那个 '一' ，对于被维护者的自己增删改查，绝不应该影响到维护者

其次，就是级联操作的影响问题；



#### 多对多映射关系

我们举一个老师 & 学生的例子

老师作为维护者，学生作为被维护者

```java
维护者：


	@ManyToMany(cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
    @JoinTable(name = "Teacher_Student",
            joinColumns = {@JoinColumn(name = "teacher_id", referencedColumnName = "id")},
            inverseJoinColumns = {@JoinColumn(name = "student_id", referencedColumnName = "id")})
    public Set<Student> getStudents() {
        return students;
    }


被维护者：

	@ManyToMany(mappedBy = "students")
    public Set<Teacher> getTeachers() {
        return teachers;
    }
```

通过建立一个 中间表 `Teacher_Student` 来构建多对多的映射关系。其中，`joinColumn` 作为了中间表的字段，而 `referencedColumnName` 作为了连接到目标表的字段。

还有一个要注意的是，中间表的字段名称遵循 `目标表名称 + 目标表主键id` 来进行命名

- 此时，我们修改 `teacher` 实例中的 `students` 列表并且保存时，将会修改到中间表内容；而对 `student`进行修改则不发生变化 (其实此时，`student` 保留那个 `teachers` 列表与否，只是为了查询方便而已，没有任何删除、添加、修改的权利)

#### 一对多 & 多对一

我们来举一个 

作者&文章

​		的例子

一个作者可以拥有多个文章，作为一个 `一对多` 的关系

```java
维护端：作者

	@OneToMany(mappedBy = "author",cascade=CascadeType.ALL,fetch=FetchType.LAZY)
    //级联保存、更新、删除、刷新;延迟加载。当删除用户，会级联删除该用户的所有文章
    //拥有mappedBy注解的实体类为关系被维护端
     //mappedBy="author"中的author是Article中的author属性
    private List<Article> articleList;//文章列表
    
被维护端：文章

	@ManyToOne(cascade={CascadeType.MERGE,CascadeType.REFRESH},optional=false)//可选属性optional=false,表示author不能为空。删除文章，不影响用户
    @JoinColumn(name="author_id")//设置在article表中的关联字段(外键)
    private Author author;//所属作者
```

这里相对就简单一些了，作者自身维护一个文章列表，拥有权限直接删除文章表里面的内容；

而对于文章来说，它建立了一个 `@JoinColumn(name = "author_id")` ， 作为自己表里面的外建。这里的名称格式，和多对多的一致  `目标表名称 + 目标表主键id`

#### 更多的思考

这时候就会问了：如果不是简单的维护者与被维护者，而是两者平等的双向绑定应该如何去做呢？
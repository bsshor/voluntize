# 华北电力大学志愿活动系统
## 软件概要设计
本软件使用前后端分离的设计模式。不使用jsp，使用html网页中的js脚本进行前后端通信。

本软件使用maven管理依赖，tomcat作为web服务器。

软件后台架构为 spring boot + spring data jpa + spring mvc。

后台采用经典的分层结构，依次分为视图控制器controller，控制器操作的对象vo，
持久实体类entity，数据库访问层repository，业务逻辑层service。

## 软件详细设计
### 实体类设计

**志愿活动**

志愿活动生命周期：
<ul>
<li>CONFIRMING 部门已发送等待审核</li>
<li>SEND 审核并修改</li>
<li>APPLY 报名</li>
<li>STARTED 录用并开始活动</li>
<li>FINISHED 结束并评价</li>
</ul>
共5个阶段。

志愿活动的报名十分复杂，结合具体案例分析，例如，在2019级迎新活动中，有多个
岗位，同一个岗位也可以分为不同地点，与时段。最终学生按照单一时间段报名。

因此，设计Activity类中包含多个ActivityStation，即一场志愿活动中有多个岗位；
设计ActivityStation中有多个ActivityPeriod，即一个岗位中分多个时间段，最终按
时间段报名。

**学生与志愿活动的关系**

多个学生参加多个志愿活动，这是多对多关系，而且这个多对多关系是多层的，蕴含了
例如志愿报名结果，志愿成绩等信息。学生参加志愿活动分为如下流程：

<ul>
     <li>APPLIED 已报名</li>
     <li>PASSED 已审核</li>
     <li>EVALUATED 已评价</li>
</ul>

因此新建一个关联表Record即志愿记录类来解决这个问题。

**多级评论**

允许添加评论的评论，因此评论是一个树状结构，只用如下方法处理树状结构的储存问题。

```
    /**
     * 父级评论，没有父级则为空
     */
    @ManyToOne(targetEntity = Comment.class)
    @JoinColumn(name = "parent_comment", referencedColumnName = "id")
    private Comment parentComment;

    @OneToMany(targetEntity = Comment.class, mappedBy = "parentComment")
    private List<Comment> sonComment;
```

**图片实体类的多用性** 

Image实体类用于储存图片，可以是Student或Department的头像，也可以是评论与
活动介绍所带内容。该类与多个实体类保持多对一关系，减小了多个图片表造成的冗余。

**数据表结构**

![img](./readmeSrc/dbo.png)

### 业务逻辑设计
**登录** 

设计UserVo类，用于接收用户填写信息。在业务逻辑中自动判断用户身份，跳转向不同主页。

**账户信息修改**

由于密码修改的特殊逻辑，这里不修改密码，只修改其他信息。
 
**密码修改**

在点击*找回密码邮件*中的超链接之后，或者在*修改密码页面*输入原来密码之后，
就可以直接修改密码。

找回密码邮件中的超链接应包含加密的用户信息，在跳转之后进行解码判断是什么用户， 
从而修改密码。

### web作用域限定
**session作用域**

UserId 学生或部门的id

### 前后端通信设计



## 软件调试

### jpa uuid主键生成策略
### 数据库长文本存储，mysql引擎
### spring service层分开实现的方法
类适配器模式与对象适配器模式

抽象类的注入与实现 https://blog.csdn.net/evilcry2012/article/details/78927479

一个接口或抽象类是难以分开成多个方法分别实现的，将其分成多个接口
并继承同一个接口也不行。


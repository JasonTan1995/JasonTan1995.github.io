---
title: SpringData Jpa
date: 2017-09-09 13:46:21
tags: Jpa
categories: Spring
---

1. ##### SpringData Jpa的环境搭建

   - 如果是Maven项目的话,添加以下依赖

   ```
   <dependency>
   			<groupId>org.springframework.data</groupId>
   			<artifactId>spring-data-jpa</artifactId>
   			<version>1.9.0.RELEASE</version>
   			<!-- <version>1.10.5.RELEASE</version> -->
   		</dependency>
   ```

   - 由于SpringData Jpa使用了动态代理(JDK&Cglib),因此我们在DAO中只需提供接口即可实现与数据库进行交互.

     <!--more-->

   - Spring与Jpa 的整合(配置文件)

     ```xml-dtd
     <!--因为Spring与Jpa的整合封装了entityManager.我们需要配置工厂,从工厂获取entityManager-->
     <bean id="entityManagerFactory"
     		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
     <!--注入连接池-->
     		<property name="dataSource" ref="dataSource" />
     <!--对持久化对象进行扫描.读取持久化对象中的主角-->
     		<property name="packagesToScan" value="com.jpa.entity" />
     		<property name="persistenceProvider">
     			<bean class="org.hibernate.jpa.HibernatePersistenceProvider" />
     		</property>
     		<!--JPA的供应商适配器-->
     		<property name="jpaVendorAdapter">
     			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
     				<property name="generateDdl" value="false" />
     				<property name="database" value="ORACLE" />
     				<property name="databasePlatform" value="org.hibernate.dialect.Oracle10gDialect" />
     				<property name="showSql" value="true" />
     			</bean>
     		</property>
     <!--配置方言-->
     		<property name="jpaDialect">
     			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect" />
     		</property>
     		<property name="jpaPropertyMap">
     			<map>
     				<entry key="hibernate.query.substitutions" value="true 1, false 0" />
     				<entry key="hibernate.default_batch_fetch_size" value="16" />
     				<entry key="hibernate.max_fetch_depth" value="2" />
     				<entry key="hibernate.enable_lazy_load_no_trans" value="true"></entry>
     				<entry key="hibernate.generate_statistics" value="true" />
     				<entry key="hibernate.bytecode.use_reflection_optimizer"
     					value="true" />
     				<entry key="hibernate.cache.use_second_level_cache" value="false" />
     				<entry key="hibernate.cache.use_query_cache" value="false" />
     			</map>
     		</property>
     	</bean>
     ```

     - 准备持久化对象(实体类)

       ​

2. ##### SpringData Jpa入门(简单实用)

   - 2.1  首先创建Dao的接口

     ==这里的Dao接口不需要填写注解@Repository的原因是因为JpaRepository接口的实现类SimpleJpaRepository已经帮我们填写了==

     ```java
     public interface DeptDao extends JpaRepository<Dept,String>{

     }
     ```

     - 2.2  根据ID查询

       ```java
       @RunWith(SpringJUnit4ClassRunner.class)
       @ContextConfiguration("classpath:applicationContext.xml")
       public class DeptTest {

           @Autowired
           private DeptDao deptDao;

           @Test
           public void fun (){
               System.out.println(deptDao);
               System.out.println(deptDao.getClass());
               Dept dept = deptDao.findOne("100");
               System.out.println(dept);
           }
       ```

       - 2.3 查询所有-findAll

         ```java
           @Test
             public void findAll(){
                 List<Dept> deptList =  deptDao.findAll();
                 for (Dept dept : deptList) {
                     System.out.println(dept);
                 }
             }
         ```

         - 2.4 添加 - save

           ```java
            @Test
               public void  saveDept(){
                   Dept dept = new Dept();
                   dept.setDeptName("研发部");
                   Dept parent = new Dept();
                   parent.setDeptId("101");
                   dept.setDept(parent);
                   dept.setState(1);
                   dept.setDeptId("297e595a61da8d3e0161da8d43ba0000");
                   deptDao.save(dept);
               }
           ```

           - 2.5 删除 - delete

             ```java
              @Test
                 public void delete(){
                     Dept dept = new Dept();
                     dept.setDeptId("402829816256c9ad016256c9b3480000");
                     deptDao.delete(dept);
                 }
             ```

##### **==注意:更新与保存使用的是同一个方法save(),区分更新与保存(添加用户)的依据是,传入的对象是否有ID属性值并且该ID在数据库是否存在.如果有ID,数据库也存在该ID,该操作为更新;反之,则为保存(添加)==**

1. ##### SpringData Jpa(源码及执行流程分析)

   ![SpringDataJpa 接口继承关系](C:\Users\User\Pictures\SpringDataJpa 接口继承关系.png)

   - 因为在SpringData Jpa 里面使用了动态代理的技术,所以我们只需要编写接口继承JpaRespository便可以使用基本的增删改差的操作.

   - 3.1 在测试类中的findOne方法打上断点.

   - 3.2 当断点通过findOne方法后,进入到JdkDynamicAopProxy(代理工厂中)为我们生成代理对象.

   - 3.3 代理概念:关于代理,是会在程序运行的期间动态生成类的字节码;

     ​      代理作用:在原有的目标对象方法的基础上,添加代码;对原有的方法进行加强.

     - Jave的两种代理方法:该两种技术的不同之处就是JDK中需要用到接口来生成代理对象,而Cglib则是通过继承来生成代理对象.因此使用了Cglib技术的被代理对象的类不能使用final;
       - JDK(静态代理&动态代理)
       - Cglibe代理

2. ##### SpringData Jpa (进阶)

   - **4.1 分页的使用findAll(Pageable pageable)**

     ==这里需要注意的是:分页中的起始页数在JPA中规定是从0开始的,但页面中通常是会传1过来.因此在这里要处理一下==

     ```java
       @Test
         public void page (){
             //分页查询,定义从第几页开始查询,显示多少条记录
             int start = 1;
             int size = 5;
             Pageable pageable = new PageRequest(start-1,size);
             
             //页面要显示的信息
             Page<Dept> deptPage = deptDao.findAll(pageable);

             //页面的总记录数
             long totalElements = deptPage.getTotalElements();
             //获取页面显示记录数的总页数
             int totalPages = deptPage.getTotalPages();
             
             System.out.println(totalElements);
             System.out.println(totalPages);
         }
     ```

   - **4.2 dao接口自定义方法**

     - 以下为官文文档中对自定义接口方法的说明
     - 我们定义的方法需要按照指定的格式:==findBy+持久化对象中的属性并且为驼峰格式.==

- Dao接口

  ==这里建议使用List集合来接收数据.如果查询所得的数据有多个,而接收数据的类型却不是List类型会报错.因此为了稳定,这里建议使用List集合类型来接收数据==

  ​

```java
public interface DeptDao extends JpaRepository<Dept,String>{
//1. 按照部门名称查询
    List<Dept> findByDeptName(String deptName);
    //2. 按照部门名称和状态查询
    List<Dept> findByDeptNameAndState(String deptName,Integer state);
    //3. 按照部门名称模糊查询
    List<Dept> findByDeptNameLike(String deptName);
```

  }

```java
  - Dao 接口测试类

    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("classpath:applicationContext.xml")
    public class DeptTest2 {

        @Autowired
        private DeptDao deptDao;

        //按照部门名称查询
        @Test
        public void findByDeptName(){
            List<Dept> deptList = deptDao.findByDeptName("网络部");
            for (Dept dept : deptList) {
                System.out.println(dept);
            }
        }

        //按照部门名称和状态查询
        @Test
        public void findByDeptNameAndState(){
            List<Dept> deptList = deptDao.findByDeptNameAndState("市场部", 1);
            System.out.println(deptList);
        }

        //按照部门名称模糊查询
        @Test
        public void findByDeptNameLike(){
            List<Dept> like = deptDao.findByDeptNameLike("%总%");
            for (Dept dept : like) {
                System.out.println(dept);
            }
        }
    }
```

- **4.3 JPQL(Java Persist Query Language) Java 持久化查询语言--跟HQL差不多**

```
  ```java
  @Query(value="from Dept where state =?2 and state =?2 deptName=?1",nativeQuery =false)
  List<Dept> findByCondition(String deptName,Integer state);
```

  ==注意:这里占位符的索引从1开始,并且要与参数列表中的顺序一一对应.==

  - **4.4 原生SQL语句**

    - 开启对原生SQL语句的支持

      ```java
        @Query(value="select * from DEPT_P where DEPT_NAME=? and STATE=?",nativeQuery = true)
            List<Dept> findByCondition2(String deptName,Integer state);
      ```

      ==注意:这里的占位符与参数列表必须一一对应.这里没有索引的定义了==

      - **4.5JPQL语句之更新**

        - 执行更新操作前的准备

          >1)JPQL更新,必须在接口方法上添加注解@Modifying,否则会报错:
          >
          >Not supported for DML operations 
          >
          >2) JPQL更新,更新需要事务,需要在测试方法或接口方法上添加注解:@Transactional,否则会报错:
          >
          >TransactionRequiredException:Executing an update/delete query
          >
          >3)如果操作要反映到数据库,需要提交事务,需要在测试方法或接口方法上添加注解:
          >
          >@Rollback(false) 代表在测试时提交事务.
          >
          >@Rollback(false)     Committed transaction for test
          >
          >  @Rollback(true)       Rolled backtransaction for test
          >
          >​

          ​

          ```java
            @Query(value="update Dept set deptName = ?1 where deptId= ?2",nativeQuery = false)
                @Modifying
                @Rollback(false)
                @Transactional
                void update(String deptName,String deptId);
          ```

          ​
```

##### 5. SpringData Jpa(条件查询)

- 5.1 JpaSpecificationExecutor动态条件查询

  > - 问题:如果是使用dao接口自定义方法来进行条件查询的话,在某些情况下会两种情况,一种是该条件有值,另一种是该条件没有值为空.所以在为了应对这两种情况就需要在接口中定义两个方法了.如果条件越来越多,就意味着越来越麻烦了.
  >
  >   ​
  >
  > - 解决方法:使用JpaSpecificationExecutor动态条件查询接口.
  >
  > #####  

  - 5.2 多条件测试

    在这里需要用到一个接口,并且这个接口由我们自己来实现.

    > Specification<T>;

    在这个接口需要重写一个方法

    ```java
    public Predicate toPredicate(Root<Dept>root,CriteriaQuery<?>query,CriteriaBuilder cb){
    }
```

- 5.3 参数的含义

| Param |        mean         |
| :---: | :-----------------: |
| root  |  通过root可以获取查询的属性对象  |
| query | query对Criteria查询的支持 |
|  cb   |    cb条件构造器,构造条件     |


```java


- 5.4 实现方法,并进行查询

  ```java
  	   @Test
      public void fun() {
          //从页面传递数据过来
          String deptName = "市场部";
          Integer state = 1;

          //实现接口specification
          Specification<Dept> spec = new Specification<Dept>() {

              //实现specification接口中的方法
              @Override
              public Predicate toPredicate(Root<Dept> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                  //定义一个Predicate集合
                  List<Predicate> predicateList = new ArrayList<Predicate>();

                  //通过root获取查询的对象属性
                  if (StringUtils.isNoneBlank(deptName)) {
                      Predicate p1 = criteriaBuilder.equal(root.get("deptName").as(String.class), deptName);
                      predicateList.add(p1);
                  }

                  if (state != null) {
                      Predicate p2 = criteriaBuilder.equal(root.get("state").as(Integer.class), state);
                      predicateList.add(p2);
                  }
                  //新建一个Predicate数组
                  Predicate[] predicate = new Predicate[predicateList.size()];
                  //将集合拼接成数组,因为criteriaBuilder.and()方法里面可以传递可变参数.也就是说数组也可以
                  return criteriaBuilder.and(predicateList.toArray(predicate));
              }
          };
                  List<Dept> list = deptDao.findAll(spec);

          for (Dept dept : list) {
              System.out.println(dept);
          }
      }
  }	
```


---
title: SpringData Jpa(多表关联查询)
date: 2017-09-10 13:59:56
tags: Jpa
categories: Spring
---

#### SpringData JPA的多表关联查询,可以使用JPQL,Specification,原生SQL.这里说一下为什么要用Specification

因为如果是使用JPQL或者SQL查询的话,要另外在Dao接口中的方法定义,而如果使用Specification的话,我们只需要封装好条件,再把条件放进Dao中的findAll方法即可.

下面通过一个栗子来学习一下吧!

> 例如:我们要在货物表中根据合同的ID查询货物.但显示的数据中包括合同表中的数据.这时我们就需要同时查询合同表和货物表两张表.
>
> 原生的SQL语句:select * from contract_product_c where contract_id='';
>
> JPA:需要两表连接查询:from ContractProduct cp where cp.contract.id = ?;

<!-- more--> 

```
//使用Specification查询:两表关联查询
//1. 构造条件表达式
	
		// 构造条件表达式（Specification两表关联查询）
		Specification<ContractProduct> spec = new Specification<ContractProduct>() {
			public Predicate toPredicate(Root<ContractProduct> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				// 查询条件：ContractProduct对象的购销合同id(contract.id)
				
				//root.get(是指ContractProduct对象的属性)
				//root.get(contract.id 这种写法不支持)
				
				// 关联Contract对象
				// 参数1： 关联ContractProduct对象的哪个属性，或者说关联哪个表
				// 参数2： 两表关联的类型：inner join、left join 、 right join
				// 相当于: sql=select * from contract_product_c cp inner join contract_c c on cp.contract_id=c.contract_id
				//泛型中的类型,为关联的两张表对应的实体类类型.
				Join<ContractProduct, Contract> join = root.join("contract",JoinType.INNER);
				// 构造查询的条件 . 条件表达式
				Expression<String> exp = join.get("id").as(String.class);
				// 设置条件值返回Predicate条件表达式对象.   sql+=" where c.contract_id=?"
				return cb.equal(exp, contractId);
			}
		};
		// 调用service查询货物
		List<ContractProduct> list = contractProductService.findAll(spec);

```

#### 关于in 查询

```
// 注入dao
	@Resource
	private DeptDao deptDao;

	@Test
	public void testSimple(){
		
		// 根据id查询，如：
		// select * from dept_p where dept_id in ('100','101')
		final String[] ids = {"100","101"};
		
		Specification<Dept> spec = new Specification<Dept>() {
			public Predicate toPredicate(Root<Dept> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				// 根据Dept对象的id属性查询
				Expression<String> as = root.get("id").as(String.class);
				// in 查询
				In<String> in = cb.in(as);
				for (String id : ids) {
					// 设置in 查询的值，如： in(100,101)
					in.value(id);
				}
				// 添加in 查询条件,如：where dept_id in(100,101)
				return cb.and(in);
			}
		};
		List<Dept> list = deptDao.findAll(spec);
		System.out.println(list);
		
	}
	
```

#### 关于exists

```
// 原始sql：
	// select * from dept_p d where exists (select p.dept_id from dept_p p)
	@Test
	public void testSimple2(){
		Specification<Dept> spec = new Specification<Dept>() {
			public Predicate toPredicate(Root<Dept> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				//=======================================
				// 构造子查询的条件
				Subquery<Dept> subquery = query.subquery(Dept.class);  
				Root<Dept> subRoot = subquery.from(Dept.class);  
				subquery.select(subRoot);  
				//=================================
				Predicate p = cb.equal(subRoot.get("id"), root);  
				//subquery.where(p);
				return cb.exists(subquery);
			}
		};
		List<Dept> list = deptDao.findAll(spec);
		System.out.println(list);
	}
```


#### in 与exists 一起使用

```
// 原始sql：
	// select * from dept_p d where exists 
	//	(select p.dept_id from dept_p p where p.dept_id in ('100','101') )
	@Test	
	public void testSimple3(){
		// 查询条件
		final String[] ids = {"100","1011"};
		Specification<Dept> spec = new Specification<Dept>() {
			public Predicate toPredicate(Root<Dept> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
				// Subquery 表示子查询
				Subquery<Dept> subquery = query.subquery(Dept.class);  
				// subRoot 从这个对象中可以获取子查询的对象的属性，如p.dept_id
	            Root<Dept> subRoot = subquery.from(Dept.class);  
	            subquery.select(subRoot);  
	            
	            // 构造子查询的查询条件，注意这里用的是subRoot
	            In<String> in = cb.in(subRoot.get("id").as(String.class));
	            for (String id : ids) {
					in.value(id);
				}
	            // 添加子查询查询条件
	            subquery.where(in); 				
				return cb.exists(subquery);
			}
		};
		List<Dept> list = deptDao.findAll(spec);
		System.out.println(list);
	}
```
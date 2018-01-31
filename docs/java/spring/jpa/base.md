### jpa 动态条件查询
- 其实类似hibernate中使用Restrictions构建查询条件的Criterion对象
- 可以使用Specification来构建查询条件
    - 让Repository继承JpaSpecificationExecutor<T>接口,
    - 则可以继承一部分支持Specification的实例作为参数的方法
    - Specification是一个接口,所以需要根据不同的Entity进行实现,内部只要实现一个toPredicate方法即可
    - toPredicate有3个参数：
        - Root对应着Entity获取字段的
        - CriteriaQuery,应该是标准查询语句的对象,暂时没有很具体的了解
        - CriteriaBuilder,重点是构建查询条件的构建工具
- 例子
```
public static Specification getSpecificationForList(Accounts accounts) {
    return (Specification<Accounts>) (root, criteriaQuery, criteriaBuilder) -> {
        List<Predicate> predicates = new ArrayList<>();
        predicates.add(criteriaBuilder.like(root.get("accountCode"), "%"+accounts.getAccountCode()+"%"));
        predicates.add(criteriaBuilder.like(root.get("accountName"), "%"+accounts.getAccountName()+"%"));
        Predicate or = criteriaBuilder.or(predicates.toArray(new Predicate[predicates.size()]));
        Predicate and = criteriaBuilder.and(
                criteriaBuilder.equal(root.get("sysDelete"),"0"),
                criteriaBuilder.equal(root.get("sysCompanyId"),accounts.getSysCompanyId()));
        return criteriaBuilder.and(or,and);
    };
}
```
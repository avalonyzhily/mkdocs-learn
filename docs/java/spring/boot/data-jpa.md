### jpa原生sql查询的使用和分页
```
public NativeJpaQuery(JpaQueryMethod method, EntityManager em, String queryString, EvaluationContextProvider evaluationContextProvider, SpelExpressionParser parser) {
   super(method, em, queryString, evaluationContextProvider, parser);
   JpaParameters parameters = method.getParameters();
   boolean hasPagingOrSortingParameter = parameters.hasPageableParameter() || parameters.hasSortParameter();
   boolean containsPageableOrSortInQueryExpression = queryString.contains("#pageable") || queryString.contains("#sort");
   if(hasPagingOrSortingParameter && !containsPageableOrSortInQueryExpression) {
       throw new InvalidJpaQueryMethodException("Cannot use native queries with dynamic sorting and/or pagination in method " + method);
   }
}
```
jpa中可以使用@Query注解来写原生的sql,但是需要设置nativeQueury为true;
如果需要分页的话,是可以与Pageable适配的,如下:

如上面源代码所示,需要将pageable作为辅助参数写入查询sql,在最后手动添加pageable参数(譬如 #{#pageable}  \n#pageable\n --#pageable\n  /*#pageable*/等,可能有数据库差异);

同时还需要添加一个countQuery的sql,用于获取查询结果;

附上该问题的一个解决方案链接
https://stackoverflow.com/questions/38349930/spring-data-and-native-query-with-pagination
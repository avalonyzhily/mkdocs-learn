### jackson序列化的问题

- 对于long类型数字超过18位会无法正常转化,最多只能17位,之后全是0
- jackson序列化转换的注解,可以在转换时自定义目标数据的类型,例 @JsonSerialize(using = ToStringSerializer.class)
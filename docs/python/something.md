### 描述符

- 描述符类是特殊性类型,实现了以下至少一个魔法方法
    - __get__(self,instance,owner)
    - __set__(self,instance,value)
    - __delte__(self,instance)
- 将描述符类的实例赋给另一个类的属性,就是描述符
- 描述符必须定义在类层次的属性,如果定义在实例的属性上,则会变成普通的类实例
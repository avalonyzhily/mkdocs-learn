### React生命周期

- 首次渲染:```getDefaultProps,getInitialState,componentWillMount,render,componentDidMount```;
- 首次卸载:```componentWillUnmount```;
- 第二次渲染:```getInitialState,componentWillMount,render,componentDidMount```;
- 属性改变:```componentWillReceiveProps,shouldComponentUpdate,componentWillUpdate,render,componentDidUpdate```;
- 状态改变:```shouldComponentUpdate,componentWillUpdate,render,componentDidUpdate```;
- 父组件的```componentWillMount/componentWillUpdate```先于子组件执行,父组件的```componentDidMount/componentDidUpdate```后于子组件执行(递归效果);
- ```componentWillMount/componentWillReceiveProps```方法中调用```setState```不会重新渲染,而是```state```合并;```componentWillUnmount```方法中也不会重新渲染,因为所有的```state```都会置为```null```;
- ```shouldComponentUpdate/componentWillUpdate```方法中不能使用```setState```,否则会有循环调用```setState```的风险,导致内存崩溃;

### React表单组件

#### HTML的组件在react的jsx中都有对应的实现,但是用法上有区别。
- 以下组件都有onChange事件。
 - input(text)/textarea：文本框,value用来表示表单的值(HTML中的textarea的值用children表示)。
 - input(radio/checkbox)：单选/复选框,value用来表示值,checked用来表示勾选状态,同时通过state中的一个属性来获取选中项的值。
 - select：通过一个multiple=(true/false)的属性来控制单选和多选，value代表值,注意option/options是一个对象,键值对对应选项。(HTML通过selected属性来控制已选,react的jsx则通过value的值来控制已选,多选时是一个数组)


#### React受控组件和非受控组件
- ```受控组件``` ——变化的状态都会记录到state中,通过onChage事件改变state来渲染组件,这样完成了数据的双向绑定(state的状态通过props传递下去,通过onChange回写回来),这样使得表单组件的状态更可靠,并且在渲染之前可以拦截事件进行自定义处理;

- ```非受控组件``` ——没有使用value/checked等props时,组件就叫非受控组件,此时无法使用state和props来控制组件状态,但是可以通过添加ref来访问底层的DOM元素来操作,拥有默认值(defaultValue、defaultChecked等,只会渲染一次)。

#### React的CSS modules
- msTransition中ms是唯一小写的浏览器前缀,其他都需要驼峰转换(对应-符号,比如WebkitTransition对一个-webkit-transition)

- classnames库用于动态设置类名,例如
```
const btnClass = classNames({
<br/>
'btn': true,
<br/>
'btn-pressed': this.state.isPressed,
<br/>
'btn-over': !this.state.isPressed && this.state.isHovered,
<br/>
})
```

- composes:用于样式组合 但是非CSS语法,会影响预编译处理器的使用
```
/* components/Button.css */
<br/>
.base { /* 所有通用的样式 */ }
<br/>
.normal {
<br/>
composes: base;
<br/>
/* normal 其他样式 */
<br/>
}
```

- 命名规范(也可以不遵守)
```
{block名(模块)}__{element名(节点)}--{modifier名(状态)}
```

- 使用属性选择器覆盖全局样式
```
// dialog.js
return (
<div className={styles.root} data-role="dialog-root">
<a className={styles.disabledConfirm} data-role="dialog-confirm-btn">Confirm</a>
...
</div>
);
// dialog.css
[data-role="dialog-root"] {
// override style
}
```

#### React中的mixin和高阶组件
mixin是混合的意思,意义是在原有的类设计基础上混入其他同性质类的属性和方法,达到组合复用的效果。
高阶组件是React用于替换mixin的方案。

> 高阶组件还没有弄懂！

#### compose函数的作用
组合函数的，将函数串联起来执行，将多个函数组合起来，一个函数的输出结果是另一个函数的输入参数，一旦第一个函数开始执行，就会像多米诺骨牌一样推导执行了

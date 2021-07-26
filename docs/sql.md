# sub sql

原sql查询:
```sql
select * from employees e 
where e.salary > (
  select avg(d.salary)
  from employees d
  where d.office_id = e.office_id
);
```

无论什么代码, sql, xml, js还是java, 都会被设计成可读性良好的.

where 后面需要是一个运算结果是布尔值的表达式
最简单的表达式就是常量 `true`

`select * from employees where true` 就是最简单的sql,
当然, 这时这条可以简化为 `select * from employees`

当然, where子句可以写得更复杂, 只要最后它返回一个运算结果是布尔值的表达式就行, 
如: `where 2 > 1` , 当然这里 2 > 1 永远是true, 这里也就等同于`where true` 了.

但上述运算符的两边是常量, 其实表达式的一边也可以是变量,
变量可以是表的某一列, 也可以是一个子查询
如: `where date > "2015-06-11"`

那两边可不可以是变量? 也都可以.

注意上面的表中, 外面的表别名是`e`, 里面的表别名是`d`(虽然是同一个, 这里使用一个简写以作连接时的区分)

这句sql的意思是, 查询 各个办公室中工资排行前50%的雇员

AVG是一个求平均值的内置函数, 

`select * from employees e ` 是拿着e表的行, 一行一行地遍历, 看看满足不满足条件, 满足就筛选出来.

条件就由where子句里的表达式来定义.

当遍历到某一行时, 把这一行的雇员的工资就是 e.salary

然后我们的条件是工资大于办公室的平均工资, 那么我们要查雇员所在办公室的平均工资.

然后, 我们通过 `e.office_id` 得到这个雇员的办公室, 进而查询该雇员办公室的平均工资

```sql
select avg(d.salary)
from employees d
where d.office_id = e.office_id
```

在执行上述的子查询中, e.office_id 是确定的一个值了, 假如它是 10 (当外层的查询 查询到下一行时, 可能又变动了, 但是, 在执行子查询时, e.* 的值都是确定的), 
那实际子查询执行时是类似这样:

`select avg(salary) from employees where office_id = 10`


不过这是一个嵌套查询, e.office_id 虽然在某次子查询中是固定的, 但在不同的子查询间可能是变动的, 所以, 子查询中仍然应该用表达式 `e.office_id` 来引用当前雇员所在的办公室.

再把10替换成 `e.office_id` , 完整的where表达式应该是下面这样:
```
e.salary > (
select avg(d.salary)
from employees d
where d.office_id = e.office_id
)
```

也就是, 一开始时原sql的模样.


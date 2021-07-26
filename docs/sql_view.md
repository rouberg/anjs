# SQL VIEW

假如有一个雇员表叫 employees:

它有 姓名(name), 工号(employee_id), 性别(sex), 出生年月(birthday), 领导(leader_id)

现在, 有一个基于表employees的查询 `select * from employees`

一切都是好好的, 但是, 有一天, 假设我们有了如下的这些改动, 这将变得比较有心无力:

1. 表结构要重构(出于规范化, 可读性, 实现要求等)
如: employee_id 我觉得可读性不好, 要变成 staff_id;
    sex之前是用字符串'F' | 'M' 表示男和女, 觉得性能不够优, 打算换成 0 和 1 来存储性别, 以节省存储空间
   如 employees表里多了一些敏感信息如salary, 希望这些信息能被保护, 不想是个人都能把工资查出去.
   
那么直接基于表 employees的查询, 会比较头疼.

select * from employees 查询出来的工号是用 staff_id 表示了(不再是employee_id), 写java的人表示不想改sql
   (就算是自己写的java, 那sql查询比较多, 改起来麻烦, 尤其是, java中很多代码中直接用了employee_id)
   我们想改表结构, 但又不想改sql, 如果是直接从表上查询的, 那就没办法.
   
那么, 后果就是, 我把employees表的employee_id调整成了 staff_id 时, 所有的sql中(和可能所有的java代码中)的 employee_id 都要改了.

但是, 如果我的查询是基于视图的(如 `select * from view_employees`), 那么我完全可以调整表结构时, 不用调整对应sql, 只调整视图和表的映射关系.

假如我以前的视图是

```sql
create view view_employees 
(employee_id, name, sex, birthday, leader_id)
as select
employee_id, name, sex, birthday, leader_id
from
employees
```

employees表中的列 employee_id 调整成 staff_id后, 不需要调整任何sql, 只需要调整 视图 view_employees 成如下:

```sql
create view view_employees 
(employee_id, name, sex, birthday, leader_id)
as select
staff_id, name, sex, birthday, leader_id
from
employees
```

假如 sex列 之前是 char(2) 类型, 值是 "F" 和 "M", 那么把sex的类型调整成 TINYINT(1),

如果是基于视图查询的, 那么也容易屏蔽实现细节, 不用改点啥都要改sql, 只用改视图和表的映射关系成如下即可保持sql查询到的数据结构保持不变.

具体视图我就不写了啦, 还要去找点资料.具体的函数我不熟悉.

假如 employees 表中多了一个列 salary, 之前的select * from employees也很容易暴露敏感的salary信息.


但是, 如果查询是基于试图的, 如 select * from view_employees, 
那么, 只要我不在view_employees中添加salary列, 那别人无论怎么从视图中select * , 都不可能拉出salary信息.

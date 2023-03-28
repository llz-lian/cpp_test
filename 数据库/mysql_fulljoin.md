# leetcode [1965] (https://leetcode.cn/problems/employees-with-missing-information)

## full join 实现
```
select e.employee_id
from
Employees as e left join Salaries as s on e.employee_id = s.employee_id
UNION 
select s.employee_id
from
Employees as e right join Salaries as s on e.employee_id = s.employee_id
```


# leetcode [1484] (https://leetcode.cn/problems/group-sold-products-by-the-date)
group_concat([distinct] 要连接的字段 [order by 排序字段] [separator '分隔符'])
将组内字段按照排好的顺序以分隔符链接

select sell_date,count(distinct product) as num_sold,group_concat(distinct product order by product ASC separator ',') as products
from Activities
group by sell_date
order by sell_date ASC
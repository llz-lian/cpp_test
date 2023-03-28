
# leetcode [1667] (https://leetcode.cn/problems/fix-names-in-a-table)

upper():转大写
lower():转小写

left(str1,从左边开始获取字符个数)
substring(str1,获取字符的开始位置,获取的个数)
right(str1,从右边开始获取的字符个数)

select user_id ,concat(upper(left(name,1)),lower(substring(name,2))) as name 
from Users
order by user_id ASC
# leetcode [1393](https://leetcode.cn/problems/capital-gainloss/)
```
#### C1:
CASE WHEN condition1 THEN result1
     WHEN condition2 THEN result2
     ...
     ELSE resultN
END

#### C2：
CASE column_name
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    ELSE resultN
END
```
```
select stock_name, sum(
    case operation 
        when 'Buy' 
        then -price 
        else price 
    end) capital_gain_loss 
from stocks
group by stock_name;
```

 - where---过滤过滤指定的行
- having--过滤分组，与group by连用
* where条件语句后面不能加聚合函数（分组函数）
* having不能单独使用，必须和group by 联合使用
 
eg :
\ [返回订单总和不小于100的所有订单的订单号](https://www.nowcoder.com/practice/ff77e82b59544a15987324e19488aafd?tpId=298&tags=&title=&difficulty=0&judgeStatus=0&rp=0&sourceUrl=%2Fexam%2Foj)

##### 代码
 ```mysql
select order_num

from OrderItems

group by order_num

having sum(quantity)>=100

order by order_num;
```


 
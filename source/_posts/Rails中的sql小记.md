---
title: Rails中的sql小记
date: 2017-08-07 13:47:03
tags:
  - rails
  - sql
---

## Rails中的Sql小记

### 关于`join`

#### `sql`中的`join`

- `left join`
```sql
select count(distinct store_name) from scores left join stores on scores.scoreable_id = stores.id and scores.scoreable_type = 'Store' where store.status = 0 and scores.user_type=0;
# 这个是查询 所有带有评分的门店,并且门店是营业状态(0)和评分的类型是用户评分(0)
```

- `right join`
和`left join`差不多.不过这个是以右表全表为基础进行处理.

- `inner join`

```sql
SELECT `companies`.* FROM `companies` INNER JOIN `departments` ON `departments`.`company_id` = `companies`.`id`
```

#### `rails`中的`join`

- `joins`
```ruby
Company.joins(:departments).to_sql
# => SELECT `companies`.* FROM `companies` INNER JOIN `departments` ON `departments`.`company_id` = `companies`.`id`
```

可以发现,`rails`中的`joins`对应`sql`中的`inner join`.

- `includes`

```ruby
Company.includes(:departments).where(departments: {department_name: '#'}).first
# => SELECT  DISTINCT `companies`.`id`, `companies`.`id` AS alias_0 FROM `companies` LEFT OUTER JOIN `departments` ON `departments`.`company_id` = `companies`.`id` WHERE `departments`.`department_name` = '#'  ORDER BY `companies`.`id` ASC LIMIT 1
```

可以发现,有一个`LEFT OUTER JOIN`.是不是和`left join`很像.

#### 小结

##### 对应的关系
1. `join`就是`inner join`, `inner`是可选的.
2. `left outer join`就是`left join`, `outer`是可选的.
3. 具体的可[查看此链接](https://stackoverflow.com/questions/406294/left-join-vs-left-outer-join-in-sql-server)


### 其他方法

- `sum`

```sql
SUM(CASE WHEN num > 0 THEN 1 else 0 END) AS available_times
```

- `round`
```sql
ROUND('123.654',2)
# 123.654 取小数后两位
```

- `GROUP_CONCAT/CONCAT`
返回拼接的字符串. GROUP_CONCAT与group by配合使用,效果更佳.


[原文链接](https://github.com/xiaohesong/TIL/issues/6)

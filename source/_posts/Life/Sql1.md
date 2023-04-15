---
abbrlink: f6d88674
---
---
title: 分享一个sql查询重复记录
date: 2020-03-31 23:04
tags:
categories: MyLife
--- 

### ***postgres的查询重复sql记录***
```javascript
select bill_code,count(1) from t_fee_agencyfee_bill GROUP BY bill_code HAVING count(*) > 1 
```     
group by 根据前面查询的字段来分组的。     
![avator](http://img.yangjiapo.cn/sql1.png)     
工作中会用到的。
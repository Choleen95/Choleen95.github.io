## ElasticSearch7.10配置Search-Guard之配置用户

![](http://img.yangjiapo.cn/ElasticSearcch/角色配置.png)

### 配置sg_internal_user.yml

密码是：elastic

```bash
jode:

  hash: $2y$12$nUzkcjdnufzvI1HlmN7xSuND3skGhmwV5le5IINejz.asMFpLYNRy

  backend_roles:

  - "hr_department"



psmith:

  hash: $2y$12$nUzkcjdnufzvI1HlmN7xSuND3skGhmwV5le5IINejz.asMFpLYNRy

  backend_roles:

  - "hr_department"



cmaddock:

  hash: $2y$12$nUzkcjdnufzvI1HlmN7xSuND3skGhmwV5le5IINejz.asMFpLYNRy

  backend_roles:

  - "devops"
```



### sg_roles.yml

```bash
sg_human_resources:

  cluster_permissions:

​     - "SGS_CLUSTER_COMPOSITE_OPS"

  index_permissions:

​     - index_patterns:

​       - "humanresources"

​       allowed_actions:

​         - "SGS_READ"





sg_devops:

  cluster_permissions:

​     - "SGS_CLUSTER_COMPOSITE_OPS"

  index_permissions:

​     - index_patterns:

​       - "infrastructure"

​       allowed_actions:

​         - "SGS_READ"

​         - "SGS_WRITE"

​     - index_patterns:

​        - "logs-*"

​        allowed_actions:

​          - "SGS_READ"
```





### sg_roles_mapping.yml

```yml
sg_human_resources:

  backend_roles:

​     - "hr_department"



sg_devops:

  backend_roles:

​     - "devops"
```





### sgadmin命令

```bash
./sgadmin.sh -cd ../sgconfig/ 

-cert /home/software/es_ssl/es/config/certs/kirk.crtfull.pem

-key /home/software/es_ssl/es/config/certs/kirk.key.pem 

-cacert /home/software/es_ssl/es/config/certs/root-ca.pem 

-keypass changeit 

-nhnv 

-icl
```

### 创建索引

（1）用jode

- 去创建索引、删除索引

- 去查询除了**humanresources**索引之外的索引



（2）用cmaddock

- 去创建索引、删除索引

- 去查询*infrastructure*索引、写入数据

- 去查询*log*开头的索引、写入

- 查询、写入其他索引



#### jode操作

##### 创建索引

![](http://img.yangjiapo.cn/ElasticSearcch/jode创建索引.png)



返回来的异常信息，为admin才能创建索引，失败了。





##### 创建文档

```json
{

​    "username":"张三",

​    "age":18,

​    "gender":"male"

}
```





![](http://img.yangjiapo.cn/ElasticSearcch/jode创建文档.png)



以经用admin创建了*<font color=red>humanresources</font>*索引，现在用jode用户去写入数据，明显无权限，由于*jode*属于*hr_department*角色，只能有查询**humanresources**的权限



##### 查询索引

查询*humanresources*索引，能查询出数据。

![](http://img.yangjiapo.cn/ElasticSearcch/jode查询humanresources.png)



我现在用**jode**用户，查询*infrastructure*索引，显示无权限。



![](http://img.yangjiapo.cn/ElasticSearcch/jode查询其他索引-无权限.png)





### cmaddock操作

*cmaddock*是**devops**角色的用户，有两个索引，其中*infrasturcture*可以读和写的权限，*logs-**只可以读

#### 创建索引

![](http://img.yangjiapo.cn/ElasticSearcch/devops创建索引.png)



显然是无权限，正确。



#### 创建文档

![](http://img.yangjiapo.cn/ElasticSearcch/infrastructure创建文档.png)



显示成功，正确



对于logs-*索引，无写权限，如下：

![](http://img.yangjiapo.cn/ElasticSearcch/devops无写权限.png)



#### 查询索引

##### infrastructure索引

![](http://img.yangjiapo.cn/ElasticSearcch/infrastructure查询索引.png)



显示数据，成功。



##### logs-*索引

我创建了**logs-2023**、**log-2023**两个索引。



查询**log-2023**

![](http://img.yangjiapo.cn/ElasticSearcch/devops查询log索引.png)



显示无权限，正确，由于对于**logs-***正则表达式的索引，才能查询。



查询**logs-2023**

![](http://img.yangjiapo.cn/ElasticSearcch/devops查询logs日志.png)



显示数据，成功
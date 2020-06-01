#### 安装elasticsearch的问题 ####

* 1.cannot assign requested address:bind

  安装elasticsearch5.5.2,配置了端口9200，让后启动。发生了这个错误；百度查询是绑定host出了问题。

  ###### 解决问题 ######

  [~#]vim elasticseach/config/elasticsearch.yml

  增加两行

  [~#]network.bind_host:127.0.0.1

  [~#]network.publish_host:127.0.0.1

  [~#]wq

  启动成功
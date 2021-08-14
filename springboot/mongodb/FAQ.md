# @Indexed没有自动生成索引

> 原因
  在MongoDB 3.x开始，自动建立索引功能默认关闭，可以通过后面描述的方式进行设置。到这里的话，可算是找到问题出现的根源。
  
> 解决方法
  spring.properties文件中新增一行配置即可
  `spring.data.mongodb.auto-index-creation=true`
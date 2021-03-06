@[TOC](目录)

# Kong网关中重要的几个概念介绍

![Kong重要概念关系图](https://img-blog.csdnimg.cn/3f4577a8b024492a8985094ee2ec22a7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5aSq56m655y8552b,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center "Kong重要概念关系图")

上图就是kong网关的调用过程，一共分为五部分：用户客户端、routes、services、upstream、target服务端。

routes路由匹配客户端的请求规则。匹配成功后分配到service层。一个路由指向一个service，一个service可以被多个不通规则的路由routes指向。

访问地址是kong网关地址+代理端口（默认http:8000,https:8443）

service服务是一个抽象服务层，可以用于指向具体物理服务（target），也可以指向upstream用于实现物理服务的负载效果。一个service对于upstream、target都是一对一的关系。

upstream主要用于实现kong的负载功能，一个service匹配到一个upstream后，upstream可以指向多个target服务以此来实现负载的效果。

target服务端指的就是最后的物理服务，当然也可以是其他的虚拟服务。


# 展示所有routes

`curl -ik "http://127.0.0.1:8001/routes"`

# 查看指定service的路由

`curl -ik "http://127.0.0.1:8001/services/test-b2b-ota/routes"`

# 给指定的service 添加路由

`curl -ik -XPOST "http://127.0.0.1:8001/services/test-b2b-ota/routes" --data "name=test-b2b-ota" --data "paths[]=/getUserInfo" --data "methods=POST"`

PS: 目前部署的Konga在Service中新增route时，前端页面有bug，无法添加成功，提交请求时参数没有随着form提交，因此这一步需要用命令行操作，也可以关注konga版本的更新。

# 访问api

`curl -ik -XPOST "http://127.0.0.1:8000/getUserInfo" -H "x-access-token:eyJhbGciOiJSUzI1NiIsImtpZCI6ImVjYzlmNmUzLTJmMGQtNDQ0Yi1hZjlkLWYwMTU4YjJjZDI5YiJ9.eyJhY2NvdW50SWQiOiI5MjliNzZjNTMyYjNkZDQ3ZWEwNzhkYzlkZTBiODg3NSIsImFjY291bnRUeXBlIjoiMSIsInN1YiI6ImZyb250X2FkbWluIiwiYXVkIjoibXNtcCIsImV4cCI6MTY0Mzk0MTQ4NiwiaW5mbyI6IntcImFjY291bnRJZFwiOlwiOTI5Yjc2YzUzMmIzZGQ0N2VhMDc4ZGM5ZGUwYjg4NzVcIixcImZyb250VXNlcklkXCI6XCI2NjJhZTBiNzE2MzdlNjdiZDI3ODNjYTFjNDU1NjYwNVwiLFwiYmFja2VuZFVzZXJJZFwiOlwiXCIsXCJhY2NvdW50VHlwZVwiOjEsXCJ1c2VybmFtZVwiOlwiZnJvbnRfYWRtaW5cIixcIm9mZmljZVwiOlwiRFVPQjJCXCIsXCJkZWZhdWx0RGVwYXJ0dXJlQ2l0eVwiOlwiS1dFXCIsXCJzYWxlc0RlcGFydG1lbnRDb2RlXCI6XCJET0NcIixcInJlYWxOYW1lXCI6XCLkvZXlqafmmI7nnYZcIixcInVzZXJJZGVudGl0eVwiOjUsXCJzYWxlc0RlcGFydG1lbnROYW1lXCI6XCLlpJrlvanoiKrnqbpcIixcImNvbnRhY3RQZXJzb25cIjpcIuS9leWpp-aYjuedhlwiLFwiY29udGFjdFBob25lXCI6XCIxMzk4NDAxNTA2MFwiLFwiYmFja2VuZERlcGFydG1lbnRJZFwiOm51bGwsXCJkZXBhcnRtZW50SWRcIjpcIjZhOTM3N2EzZTMxMTE0MzkyZThjNThjNDc1ODcxZmVlXCIsXCJvcmdhbml6YXRpb25JZFwiOm51bGwsXCJhZ2VudE5hbWVcIjpudWxsLFwiaWF0YU5vXCI6bnVsbCxcImduXCI6bnVsbH0iLCJpYXQiOjE2NDEzNDk0ODZ9.Dqn2OlcQjlhQcKsxMnrJpaBg8z8kpSTogxQq4xhfXKNaZCIYoarEi3Av3WvKYtFvuR_h6RZ7UfQ-BXyNHoriNYARaqJ-2SP3nzEWJVlD2cgnpGiUsAUMmxU3UkihudQcx5UwsF8y5E_sb1VVWNqTNjoG6QHcbFJU39pFgB5hcSGbxO8ySf2HGCMvL8dlgsMAzI3GJxLwdJ38exy-a9YpHGlOsEMNkFTU99RNacs5YUCHX6iCDJBzuDRYHJbN_vxNoOL8eIDIWy_JKSQjgIwISYzMBTwaw9chYVmzatsKGvu1L1A-Af9Kcd5To2DeeV9PMLjrUabcCgNxMfm_Q0UQ9g" -H "authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ0ZXN0LWIyYi1vdGEifQ.eLvsUnhiM1N-XfkPh1rIXtI23Hnk_8IbxmkBHZ369FI"`
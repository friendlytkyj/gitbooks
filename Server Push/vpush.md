# VIVO PUSH

## 服务端API接口文档

`https://dev.vivo.com.cn/documentCenter/doc/362`

## 服务端SDK文档

`https://dev.vivo.com.cn/documentCenter/doc/363`

## 错误码参考

`https://dev.vivo.com.cn/documentCenter/doc/368`

# vivo推送常见问题汇总

`https://dev.vivo.com.cn/documentCenter/doc/156`

## 推送系统性能问题

- 应用每天可以发送的消息数量是多少？
- 应用推送速度配置策略是什么？
- 用户每天可以接收单应用的消息条数是多少？
- 未接入“消息分类功能”会有什么影响？
- vivo推送系统最高并发能达多少？
- 推送消息的安全性如何保障？

## 推送系统服务问题

- 当应用不存活状态下，用户是否能收到消息？
- 当用户只安装从未使用过应用时，是否可以收到消息推送？
- vivo推送支持哪些消息类型？
- vivo推送支持哪些消息发送方式？
- vivo推送支持哪些机型和系统版本？
- 推送是否支持自定义icon图标？是否支持带图消息？ 
- 推送是否支持“提示音”设置？
- 手机晚上无法收到推送？在限制时间之外发送推送，是会延迟推送还是被直接抛弃？
- 如何判断系统是否支持vpush？
- vpush目前支持deeplink吗？
- 如何创建应用并申请接入vivo消息推送服务？
- 广播推送是否有去重操作？

## 客户端接入问题

- 设备唯一id的token类型的是不是在继承 OpenClientPushMessageReceiver 类的 onReceiveRegId 获取 regId？
- RegId的获取是使用 onReceiveRegIdonReceiveRegId(Context context, String regId) 还是在PushClient 初始化后通过 PushClient.getRegId() 获取？
- OpenClientPushMessageReceiver - onReceiveRegId(Context context, String token) 这个回调，什么场景下会被调用？
- regId通过接收器能收到和通过init初始化回调获取有区别吗？
- regId会变化吗？
- PushClient.getInstance(context).getRegId();  这个方法什么时机调用可以确保有值？
- 一个alias允许绑定多少设备,alias长度限制是多少？一个regId只能绑定一个alias吗？
- 初始化时需要调用 PushClient.checkManifest 吗？

## API接入问题

## 推送统计数据问题

## 异常问题处理




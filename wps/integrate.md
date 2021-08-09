# 集成WPS Office在线编辑

# WPS回调接口

[WPS开放平台回调API](https://wwo.wps.cn/docs/server/callback-api-standard/callback-notification/)  
获取文件元数据: /v1/3rd/file/info  
获取用户信息: /v1/3rd/user/info  
通知此文件目前有哪些人正在协作: /v1/3rd/file/online  
上传文件新版本: /v1/3rd/file/save  
获取特定版本的文件信息: /v1/3rd/file/version/:version  
文件重命名: /v1/3rd/file/rename  
获取所有历史版本文件信息: /v1/3rd/file/history  
新建文件: /v1/3rd/file/new  
回调通知: /v1/3rd/onnotify  

# WPS对接过程总结
云盘:
1. 上传接口只能根据fullpath上传，不能根据fileId上传内容，就会导致当文件名变化的时候，保存了文件内容，会出现2个文件  
2. 云盘“打开”、“锁定”、“预览”等功能，可以被WPS取代，当开启了在线编辑功能，云盘的部分功能可以屏蔽。  

wps集成:
1. 如果利用WPS错误码，响应必须不能是200，这是WPS官网暂时无任何说明的细节，是在不断调试中试出来的，并且要结合我们已有的微服务框架在不同的入口进行处理，改变响应的内容  
2. 多人协同打开文件时，一个人修改了文件名，不会实时同步给其他协作者，其他人不知道文件名已经发生变化  
3. 打开在线编辑后，如果修改了文件名，之后保存文件内容时，WPS回调的/file/save接口，传过来的文件名是修改前的名字，要注意这个细节。  
4. WPS在线编辑的页面，有些菜单和功能是可以通过前端js进行屏蔽，有些不支持屏幕，需要屏蔽不支持的功能可以向WPS提需求。  
5. WPS建议最佳实践是，一个文件只通过WPS一个入口进行修改，如果存在多个版本变化的入口，“最近改动”相关功能则无法正常显示。  
6. iphoneX兼容问题。手机端重命名时，输入框的键盘会把顶部菜单顶消失。WPS已确认此bug，新的JSSDK会修复此问题。
7. 手机端导出PDF兼容性问题。LINK终端请求任何地址都会在请求头Authorization放入access_token，导出PDF等外部地址请求时会出现请求失败的问题。



# FAQ
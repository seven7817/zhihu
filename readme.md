> 基于springboot 的前后端分离架构的在线问答社区
作者信息 @deng @chen @sun
## TODO LLST
1. 邮件验证 -done 
2. https -done 需要openssl生成一个IP方式访问的签名证书
3. 服务器端推送 WebSocket 甚至可以做私信功能 WS+SSL = WSS 管理员可以全站广播之类的
    - 异常处理
    - 基于消息发布订阅 暂时看来不可行
    - 通过stomp可以设置header
4. 使用rabbitMQ作为消息代理(broker)
5. docker
6. ngix 动静分离
7. 头像 云
8. redis持久化
>>...

**TOKEN**
1. token有两个时间属性,一个过期时间(短),一个有效刷新时间(长)
2. 后端每次对token校验时 
    - 若token过期 但还在有效期内 则在响应头加入access_token字段
    - 若token不在有效刷新期内 返回403或其他类型
2. 前端通过拦截器 检查收到的请求 若响应头带有 access_token字段 则将其更新local_storage
5. 当改密码或其它需要将所有已登录的客户端重登时 更新redis 白名单 用jti字段唯一标识一个用户对应的token (白名单都用上了 越来越像session)

- **私信**
1. 客户端发出connect请求的时候 将 用户id及对应的websocket session  id 存入redis缓存
2. 客户端发出disconnect请求时 清空缓存
3. 客户端向use发私信
4. 服务器端首先尝试根据user id 获取对应的redis缓存中的websocket session id
    - 当命中时 根据该ws session id 发送数据
    - 当未命中时 存入 缓存 messsageVo(senderId,timestamp,content)
        - 客户端登录时检查缓存 若有未读信息 则服务器端将其发出
1. 聊天用senderId+receiverId存储为列表 定长100条
    - 一个api 一个vo对象用于redis存储聊天记录
> 为什么不使用queue模式 直接让client上线时订阅queue接收未被消费的模式
>> 这样需要为每一个用户提供一个持久化队列 rabbitmq服务器开销大
> 如何降低发消息时的对缓存的请求的压力? 暂时无解
- **关注**
用bitmap类型实现 

- **点赞**
用zset实现 支持排序

- 回复提醒

### 重构项目 减少数据库/缓存请求 增加图片支持
编写swagger API doc
##### Email应用构成

1. 构成组件
   +  邮件客户端（User Agent）
   + 邮件服务器
   + SMTP协议（Simple Mail Transfer Protocol）
2. 邮件客户端
   + 读写email消息
   + 与服务器交互，收发email消息
3. 邮件服务器
   + 邮箱：存储发给该用户的Email
   + 消息队列（Message Queue）：存储等待发送的Email
4. SMTP协议
   + 邮件服务器之间传递消息所使用的协议
   + 客户端：发送消息的服务器
   + 服务器：接收消息的服务器
   + 使用TCP进行email消息的可靠传输
   + 端口25
   + 命令/响应模式
     + 命令：ASSIC文本
     + 响应：状态代码和语句
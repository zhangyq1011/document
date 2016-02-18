# skywalking-web-plugin 使用说明 
使用Filter处理请求、有以下步骤进行处理：
-  创建 RPCBuriedPointReceiver 对象
- 调用 RPCBuriedPointReceiver#beforeReceived() 方法进行预处理 参考 1
- 交给下一个过滤器链进行处理
- 过滤器 处理完毕后 向Response 写入 SW-TraceId=traceId 请求头信息
- 如果出现异常调用 RPCBuriedPointReceiver#handleException()
- 最后 调用 RPCBuriedPointReceiver#afterReceived()结束


## 1.收集信息之前
  先获得 tracingName Head内容 
  
 
  - 包装 com.ai.cloud.skywalking.model.Identification 对象 
    viewPoint = request.getRequestURL().toString();
    businessKey = NULL
    spanType = W
    callType = 'S'
  - 判断 构造 com.ai.cloud.skywalking.model.ContextData对象
  - 根据上述 步骤生成 com.ai.cloud.skywalking.protocol.Span 放入 com.ai.cloud.skywalking.context.Context 栈中
  
## 2.处理异常信息
  从  com.ai.cloud.skywalking.context.Context 栈中 获得 com.ai.cloud.skywalking.protocol.Span并 调用其 handleException()方法
  设置  exceptionStack = 错误队列值 、statusCode = 1
## 3.收集信息之后
 获取上下文的栈顶中的元素 Span spanData = Context.removeLastSpan(); 设置  节点调用花费时间cost。

存放到本地发送进程中 ContextBuffer.save(spanData);

## 4.核心代码类com.ai.cloud.skywalking.buffer.BufferGroup
初始化时初始化了 com.ai.cloud.skywalking.conf.Config.Buffer.POOL_SIZE 对象、该类有内部线程类 com.ai.cloud.skywalking.buffer.BufferGroup.ConsumerWorker(建议该类的构造函数也添加可以设置线程名称)负责将数据发送出去、 那么发送者呢？

## 5. com.ai.cloud.skywalking.sender.DataSenderFactoryWithBalance 核心处理类
该类有一个数据发送检查者线程类com.ai.cloud.skywalking.sender.DataSenderFactoryWithBalance.DataSenderChecker(建议该类的构造函数也添加可以设置线程名称)
### 5.1 com.ai.cloud.skywalking.sender.DataSender 是DataSenderChecker创建并维护的。
该类作用就是：发送内容





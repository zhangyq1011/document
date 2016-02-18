# skywalking-server代码review说明
 请看如下代码:com.ai.cloud.skywalking.reciever.CollectionServer程序调用步骤如下

## 1.初始化配置文件 config.properties

## 2.com.ai.cloud.skywalking.reciever.persistance.PersistenceThreadLauncher#doLaunch()
- RegisterPersistenceThread().start() 参见 2.1 
- 启动多个持久化线程PersistenceThread 参见 2.2
     
      for (int i = 0; i < Config.Server.MAX_DEAL_DATA_THREAD_NUMBER; i++) {
            new PersistenceThread().start();
        }
### 2.1 启动线程将内存中的数据读取到文件中
    线程数据来源：Collection<FileRegisterEntry> fileRegisterEntries = MemoryRegister.instance().getEntries()；内存注册器参见2.1.1
  准备REGISTER_FILE_NAME文件 并通过    MemoryRegister 类获得要写入的(文件-偏移量-状态)值写入到文件中、最后追加 "EOF\n" 内容
    

写入到以下文件内容中
  - 文件夹:  com.ai.cloud.skywalking.reciever.conf.Config.RegisterPersistence.REGISTER_FILE_PARENT_DIRECTORY
  - 文件名称：com.ai.cloud.skywalking.reciever.conf.Config.RegisterPersistence.REGISTER_FILE_NAME
  - 备份文件名称：com.ai.cloud.skywalking.reciever.conf.Config.RegisterPersistence.REGISTER_BAK_FILE_NAME

#### 2.1.1 内存注册器类功能 MemoryRegister
	提供初始化写入文件内容准备数据、提供工具类供 PersistenceThread 线程类(参见  2.2 )中调用

### 2.2 PersistenceThread要做的任务有以下
- 从缓存目录com.ai.cloud.skywalking.reciever.conf.Config.Buffer.DATA_BUFFER_FILE_PARENT_DIRECTORY 中获得文件如果没有被占用就注册到内存注册器 MemoryRegister 中参见2.1.1
- 开始一行行读取文件内容然后处理 使用 StorageChainController 进行处理 参见 2.2.1

#### 2.2.1 StorageChainController 存储链控制器进行处理埋点数据
将埋点数据根据不同Status值分别放入redis或者hbase数据中

## 3.数据缓冲线程容器初始化
com.ai.cloud.skywalking.reciever.buffer.DataBufferThreadContainer#init() 参见3.1 类定义
private static List<DataBufferThread> buffers = new ArrayList<DataBufferThread>()
### 3.1 DataBufferThreadContainer#init()方法做的操作有
- 从缓存目录com.ai.cloud.skywalking.reciever.conf.Config.Buffer.DATA_BUFFER_FILE_PARENT_DIRECTORY读取所有缓存文件
- 如果缓冲文件不为空 启动 com.ai.cloud.skywalking.reciever.buffer.AppendEOFFlagThread(参见3.1.2)线程类
- 创建MAX_DEAL_DATA_THREAD_NUMBER 个 DataBufferThread(参考3.1.1)线程 start 并添加 到 buffers 容器列表中


#### 3.1.1 DataBufferThread
   在缓存目录创建缓存的文件 文件名称算法为:System.currentTimeMillis() + "-" + UUID.randomUUID().toString().replaceAll("-", "")
   该类是一个线程类如果变量   private byte[][] data = new byte[PER_THREAD_MAX_BUFFER_NUMBER][];有值就写入到文件中。
 每次写入的内容是通过 4 收集的内容 转换为 byte 然后再追加 "\n".getBytes()写入到文件中。如果文件内容 大于 DATA_FILE_MAX_LENGTH 就重新生成文件写。
   
#### 3.1.2 AppendEOFFlagThread
向所有缓冲区文件追加 EOF\n 

## 4.启动Netty服务器开始服务
    com.ai.cloud.skywalking.reciever.handler.CollectionServerDataHandler做的操作是：设置当前线程名称为ServerReceiver;当接收到的消息不为空并且消息大小不超过 MAX_DATA_PACKAGE 时 DataBufferThreadContainer.getDataBufferThread().saveTemporarily(msg) 使用随机DataBufferThread线程类保存到缓存 文件中。





## 5.疑问、问题
DataBufferThreadContainer#init()
	AppendEOFFlagThread   线程创建是否需要传入线程名称的构造函数
	DataBufferThread	     线程创建是否需要传入线程名称的构造函数
	PersistenceThread	     线程创建是否需要传入线程名称的构造函数
	














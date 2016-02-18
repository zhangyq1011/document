# skywalking-alarm整理


## 1.  应用程序启动类
	com.ai.cloud.skywalking.alarm.AlarmProcessServer
		private static List<AlarmMessageProcessThread> processThreads = new ArrayList<AlarmMessageProcessThread>();
	做的动作：
		1.在zookeeper 中创建 如果不存在  /skywalking/alarm-server/register-servers
		2.启动 com.ai.cloud.skywalking.alarm.UserInfoCoordinator 线程类
		3.配置并启动 com.ai.cloud.skywalking.alarm.AlarmMessageProcessThread 线程类  (可能存在多个)
		4.启动 com.ai.cloud.skywalking.alarm.UserInfoInspectThread 线程类
	
## 2.三个核心类

###	2.1 com.ai.cloud.skywalking.alarm.UserInfoCoordinator
		a.不是协调者要变更为协调者
		b.重新分配 标记为  true
		c.获取当前所有的注册的(/skywalking/alarm-server/register-servers上面)处理线程(AlarmMessageProcessThread提供线程类)
		d.修改状态 (开始重新分配状态）
		e.检查所有的服务是否都处于空闲状态（只要有一个不是空闲的就返回  fasle）
	《逻辑比较简单，这是个协调器，在多个进程启动时，负责通知协调。这是发现需要重新分配用户。通知各个工作线程，等待返回确认》
		
		a.查询当前有多少用户
		b.将用户重新分配给服务
		c.修改状态(分配完成)
		d.检查所有的服务是否都处于忙碌状态(如果忙碌继续等待)
	
###	2.2 com.ai.cloud.skywalking.alarm.AlarmMessageProcessThread
		private String threadId;//线程唯一标识
	    private ProcessThreadStatus status;//处理线程状态4个状态中的一个
	    private List<String> processUserIds;//所有的用户ids
	    private CoordinatorStatusWatcher watcher = new CoordinatorStatusWatcher();//协调状态观察者
	    private Map<UserInfo, List<AlarmRule>> cacheRules = new HashMap<UserInfo, List<AlarmRule>>();//缓存的所有报警规则
	    private static AlarmMessageProcessor processor = new AlarmMessageProcessor();//警报消息处理器
	    
	    
		a.注册服务(默认为空闲状态)
		
		b.根据不同状态进行对应的操作
		
			自身状态是否为忙碌状态
				重新加载用户配置：cacheRules填充值
				开始报警：processor.process(entry.getKey(), rule);
				
			自身状态是否分配线程的状态(重新分配状态)
				修改自身状态：(空闲状态) ProcessThreadStatus.FREE(zookeeper对应线程key变更为 FREE)
				清空缓存数据:(cacheRules 清空)
				获取待处理的用户:processUserIds填充值
							cacheRules填充值
				修改自身状态 ：(忙碌状态)ProcessThreadStatus.FREE(zookeeper对应线程key变更为 BUSY)
			自身状态分配线程的状态(分配完成状态)
				获取待处理的用户：		从zookeeper中获得
				缓存数据：				cacheRules
				修改自身状态 ：			(忙碌状态)ProcessThreadStatus.BUSY
				修改zookeeper中状态:	ProcessThreadStatus.BUSY
			自身状态为空闲状态 ：
	            线程休眠 com.ai.cloud.skywalking.alarm.conf.Config.ProcessThread.THREAD_WAIT_INTERVAL 时间
				
####	2.2.1 com.ai.cloud.skywalking.alarm.procesor.AlarmMessageProcessor
该类从  redis中获取 key 和field 然后读取  缓存的exception 然后发送邮件

####   2.2.2 com.ai.cloud.skywalking.alarm.AlarmMessageProcessThread.CoordinatorStatusWatcher
    该类非常重要：监听处理ProcessThreadStatus状态值的变化(变化的来源在com.ai.cloud.skywalking.alarm.UserInfoCoordinator类中)
			
		
###	2.3 com.ai.cloud.skywalking.alarm.UserInfoInspectThread
			如果用户数目发生变化->UserInfoCoordinator.activateRedistribute()
			redistributing = true(重新分配)
			
###	2.4 alarm key的生成算法为：
	private String generateAlarmKey(Span span) {
		return span.getUserId() + "-" + span.getApplicationId() + "-"
				+ (System.currentTimeMillis() / (10000 * 6));
	}
			
			
			
	alarm_rule
		rule_id
		uid
		config_args
		todo_type
		is_global
	application_info
		app_id
		uid
		app_code 
		
		
		
		
		
		
		
		
		
		
		
		
		
		 
		
	
		
		

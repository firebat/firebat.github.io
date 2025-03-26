+++
date = '2022-04-19T00:00:00+08:00'
title = '异步任务队列'
tags = ["调度"]
+++
在我们介绍更复杂的调度管理问题之前，先了解一种基本的异步任务执行方案。

## 场景
业务系统构建过程中，为了实现逻辑复用，面临大量的计算组装问题。如:
- 数仓、复杂计算任务执行调度
- 风控、营销类的策略管理
- 数据核查规则管理
- 多仿真验证

这些场景在执行时，按调用方式可以分为:
- 同步组装，可以支持事务，包括节点、上下游、输入输出、常量变量定义等（择日细讲）
- 异步组装，适用支持复杂流程管理，包括任务队列、优先级、回调事件等

## 结构定义
通过app区分，使服务可以为多个应用使用

### 任务队列
- app 应用标识
- queueId 队列标识
- queueType 队列类型, 并行=parallel, 串行=serial
- priority 优先级, 最小=0
- status 队列状态, 非激活=inactive, 激活中=active

### 任务消息
- app 应用标识
- queueId 所属队列
- messageId 消息标识
- status 任务状态, 待执行=wait, 处理中=processing, 处理完成=completed
- taskClassName 任务类名
- parameter 任务参数
- stopQueueOnError 执行出错时，是否停止队列
- sendTime 客户发起时间
- receiveTime 服务接收时间
- acceptTime 分派时间
- startTime 开始执行时间
- nodeId 执行节点

## 调度目标
找到最优先需要执行的任务
- 队列状态=激活中
- 任务状态=待执行
- 若为串型队列，只允许其中一个任务执行
- 队列优先级最高
- 创建时间最早

### 单并行队列
每个app拥有一个并行队列(`parallel_task_queue`)和多个串型队列(自定义)
```sql
select
    t.message_id,
    t.queue_id
from task_info t
join queue_info q on q.queue_id = t.queue_id and q.status = 'active'
where t.status = 'wait'
and not exists (
    select 1 from task_info i
    where i.status = 'processing'
      and i.queue_id = t.queue_id
      and i.queue_id <> 'parallel_task_queue'
)
order by q.priority desc, t.receive_time, t.id;
```

### 多并行队列
这里给出多并行队列的筛选方法，实际上单个并行队列足够满足大多数场景。
```sql
select
    t.message_id,
    t.queue_id
from task_info t
join (
    select
        q.queue_id,
        q.queue_type,
        q.status,
        q.priority,
        count(t.message_id),
        case q.queue_type when 'parallel' then count(t.message_id) = 0 else true end as available
    from queue_info q
    left join task_info t on t.queue_id = q.queue_id and t.status = 'processing'
    group by q.queue_id
) q on t.queue_id = q.queue_id
where t.status = 'wait'
  and q.available
order by q.priority desc, t.receive_time, t.id limit 1;
```

## 服务结构
- 定义Task逻辑
- 客户端通过TaskManager提交任务到队列
- 服务端通过上述SQL选取待执行的任务
- 执行端通过TaskReceiver接收任务
- 执行端通过TaskRunner管理线程执行

### 任务定义
系统（客户端）通过实现Task接口，描述待执行的业务逻辑。可在回调方法中继续发起其他任务，实现逻辑编排。
```java
interface Task {
    setParameter(Map)
    taskAccepted(TaskEvent)
    taskStarted(TaskEvent)
    taskCompleted(TaskEvent)
    taskRejected(TaskEvent)
}
```

### 任务管理
客户端通过TaskManager实现队列和任务管理。
- active 是否激活
- reentry 任务释放后，是否重入队列
- stop 任务释放后，是否暂停队列执行

```java
interface TaskManager {
    setParallelizedTaskQueueActive(active)
    addParallelizedTask(taskClassName, parameter)
    releaseRunningParallelizedTask(messageId, reentry, stop)
    removeParallelizedTask(messageId)

    addSerializedTaskQueue(queueId, active)
    removeSerializedTaskQueue(queueId)
    setSerializedTaskQueueActive(active)
    addSerializedTask(queueId, taskClassName, parameter, stopOnError)
    releaseRunningSerializedTask(messageId, reentry, stop)
    removeSerializedTask(messageId)
}
```

### 任务接收
执行端通过TaskReceiver接收服务中心的调度结果，获取最优先需要执行的任务
```java
interface TaskReceiver {

    decideRunTask()

    acceptProcessingParallelizedTask(messageId, acceptedTime)
    startProcessingParallelizedTask(messageId, startTime)
    reentryParallelizedTask(messageId)
    finishParallelizedTask(messageId)

    acceptProcessingSerializedTask(messageId, acceptedTime)
    startProcessingSerializedTask(messageId, startTime)
    reentrySerializedTask(messageId)
    finishSerializedTask(messageId)
    reentrySerializedTask(messageId)

    attachNodeToTask(messageId, node)
}
```

### 任务执行
执行端通过TaskRunner管理执行线程
```java
interface TaskRunner extends Runnable {

    startRunner()

	stopRunner()

    isRunning()

    releaseTask(messageId)
	
    release()
}
```

执行端处理流程
- 服务是否运行
- 线程是否空闲
- 获取节点信息
- 是否有待执行任务
- 创建任务
- 绑定节点
- 启动任务

## 参考
- [IntraMart Accel Platform 异步规范](https://document.intra-mart.jp/library/iap/public/im_asynchronous/im_asynchronous_specification/texts/overview/index.html)

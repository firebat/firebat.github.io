+++
date = '2023-03-05T00:00:00+08:00'
title = '嵌入式系统设计模式'
tags = ["rtos"]
+++
近期读到两本书
- 王绍伟等编著的《嵌入式微系统》讲解了msOS的设计原理
- Michael J Pont编写的《Patterns for Time-Triggered Embedded Systems》提到的SCH51系统的实现

两者主要面向51系列，与之前了解到的面向STM32通过PendSV和SysTick中断实现RTOS思路略有不同，准确地说是一种简化实现，以适应资源有限场景下的任务调度，其中的设计模式值得借鉴。

# 超级大循环
系统初始化后，循环读取传感器数据，分析计算后执行驱动控制，一直循环直到掉电。
```c
void main()
{
  Init();
  
  while(1)
  {
	Scan();
	Analyse();
	Control();
  }
}
```

- 结构简单，易于编写、调试、维护
- 硬件资源依赖少，不使用定时器，只需少量字空间
- 非常容易移植，安全可靠
- 系统一直“满负荷”运行，耗能高
- 无法提供精准的定时处理

# 中断触发型
通过触发中断驱动相关设备，大循环不做任何事情，往往处于休眠省电状态。
```c
#define INTERRUPT_Timer_1_Overflow 3

void main()
{
  Timer_1_Init();
  EA = 1;
  while(1)
  {
    PCON |= 0x01;  // 空闲模式
  }
}

void Timer_1_Init()
{
  // 初始化定时器
}

void Timer_1_Update() interrupt INTERRUPT_Timer_1_Overflow
{
  // 定时中断处理
}
```
- 系统初始化后处于休眠模式，省电
- 使用中断服务程序(ISR)处理事件，中断退出后继续休眠
- 由于硬件定时器数量有限，适用于对定时需求较少的场景

# 节拍调度型
本质是对上述两种类型的综合折衷
- 使用定时器作为节拍触发任务分时切换
- 使用中断服务实时响应事件
- 通过消息将中断信息传递给大循环处理
```c
typedef data struct
{
  void (code * pTask)();  // 任务函数指针
  tWord Delay;            // 执行延迟
  tWord Period;           // 调度间隔
  tByte RunMe;            // 需执行时加1
} sTask;

// 任务列表
sTask SCH_tasks_G[SCH_MAX_TASKS]; 

void main()
{
  SCH_Init_T2();
  SCH_AddTask(LED_Update, 0, 1000)
  SCH_Start();
  
  while(1)
  {
    SCH_Dispatch_Tasks();
  }
}

void SCH_Init_T2()
{
  // 初始化、启动定时器
}

void SCH_Add_Task(void (code * pFunction)(), const tWord Delay, const tWord Peroid)
{
  // 添加任务到列表
}

void SCH_Start()
{
  EA = 1;
}

void SCH_Update() interrupt INTERRUPT_Timer_2_Overflow
{
  for (Index = 0; Index < SCH_MAX_TASKS; Index++)
  {
    // 如果Delay==0, 则RunMe加1, 重设Period给Delay
	// 否则Delay减1
  }
}

void SCH_Dispatch_Tasks()
{
  for (Index = 0; Index < SCH_MAX_TASKS; Index++)
  {
    // 如果RunMe>0, 执行任务, RunMe减1
  }
  
  SCH_Go_To_Sleep();
}

void SCH_Go_To_Sleep()
{
  PCON |= 0x01;
}
```
- 通过定时器2节拍触发，更新任务的Delay和RunMe标记
- 通过大循环判定，并执行任务逻辑，执行完毕后休眠

# 消息驱动型
触发中断，将传感器数据放入消息队列，交由大循环处理，是典型的前后台处理模式。
```c
void main()
{
  while(1)
  {
    message = PendMessageQueue();
	type = GetMessageType(message);
	value = GetMessageData(message);
	switch(type)
	{
	  case MessageKey:
	    KeyProcess(value);
		break;
      case MessageUsart:
	    // ...
	  default:
	    // ...
	}
  }
}

void SysTickHandler(void) interrupt 5
{
  KeyRoutine();
  RtcRoutine();
  TimerRoutine();
}
```
- 通过消息机制连接中断、节拍、大循环三要素，解决了高速响应与低速执行的矛盾
- 消息可携带更多的数据，根据需要可灵活控制


# 参考
- [Patterns for Time-Triggered Embedded Systems](https://www.safetty.net/download/pont_pttes_2001.pdf)

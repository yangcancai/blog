---
title: NFActorPlugin
date: 2020-07-10 11:02:24
categories:
- c++
- NoahGameFrame
tags: 
- Actor
- Thread

---
NFActorPlugin主要进行Actor间的消息通信模型，有点像erlnag的进程间通信。下面主要来分析Actor在多线程下是如何运作的。

## 插件主要的模块
- NFActorPlugin
- NFActorModule
- NFActor
- NFIComponent

具体业务都在NFActor和NFActorModule这两个类中,NFActorModule类中主要的成员有
```c++
    //  Kernel模块下次再分析
    NFIKernelModule* m_pKernelModule;
	//这个是线程池模块在之前模块有稍微提及过
    NFIThreadPoolModule* m_pThreadPoolModule;
	// 存放所有生成的Actor
	std::map<NFGUID, NF_SHARE_PTR<NFIActor>> mxActorMap;
	// 存放执行结果的消息队列
	NFQueue<NFActorMessage> mxResultQueue;
	// 封装了std::map,提供一些通用的函数, 存储异步执行后获取的结果进行函数回调
	NFMapEx<int, ACTOR_PROCESS_FUNCTOR> mxEndFunctor;
```
NFActor类的主要成员有
```c++
// 当前Actor的唯一id
	NFGUID id;
// 管理所有Actor的ActorModule指针
	NFIActorModule* m_pActorModule;
// 消息队列
    NFQueue<NFActorMessage> mMessageQueue;
// Actor下的所有组件
	NFMapEx<std::string, NFIComponent> mComponent;
// 消息对应的处理函数
	NFMapEx<int, ACTOR_PROCESS_FUNCTOR> mxProcessFunctor;
```
来看一下插件注册的函数注册了哪些模块
```c++
void NFActorPlugin::Install()
{
	REGISTER_MODULE(pPluginManager, NFIActorModule, NFActorModule)
}
```
几个模块的依赖关系应该是，NFActorPlugin->NFActorModule->NFActor->NFIComponent
NFActorModule对应多个NFActor
NFActor对应多个NFIComponent(代码里面Component还没有些逻辑只是做初始化动作)
### 分析执行过程
#### 主线程NFActorModule:Execute
```c++
bool NFActorModule::Execute()
{
	ExecuteEvent();
	ExecuteResultEvent();
    return true;
}
// 循环所有的Actor异步执行Actor->Execute
bool NFActorModule::ExecuteEvent()
{
	for (auto it : mxActorMap)
	{
		NF_SHARE_PTR<NFIActor> pActor = it.second;
		if (pActor)
		{
			if (test)
			{
				pActor->Execute();
			}
			else
			{
				m_pThreadPoolModule->DoAsyncTask(pActor->ID(), "",
					[pActor](NFThreadTask& threadTask) -> void
					{
                        // 在非主线程执行
						pActor->Execute();
					});
			}
		}
	}
	
	return true;
}
// 主线程执行函数结束回调
bool NFActorModule::ExecuteResultEvent()
{
	NFActorMessage actorMessage;
	while (mxResultQueue.try_dequeue(actorMessage))
	{
		ACTOR_PROCESS_FUNCTOR_PTR functorPtr_end = mxEndFunctor.GetElement(actorMessage.msgID);
		if (functorPtr_end)
		{
			functorPtr_end->operator()(actorMessage);
		}
	}
	
	return true;
}
```
#### Actor->Execute
```c++

bool NFActor::Execute()
{
	//bulk
	NFActorMessage messageObject;
	while (mMessageQueue.TryPop(messageObject))
	{
		//must make sure that only one thread running this function at the same time
		//mxProcessFunctor is not thread-safe
       // 这个lock在这里没有效果
		NFLock lock;
		lock.lock();
		ACTOR_PROCESS_FUNCTOR_PTR xBeginFunctor = mxProcessFunctor.GetElement(messageObject.msgID);
		lock.unlock();

		if (xBeginFunctor != nullptr)
		{
			//std::cout << ID().ToString() << " received message " << messageObject.msgID << " and msg index is " << messageObject.index << " totaly msg count: " << mMessageQueue.size_approx() << std::endl;

			xBeginFunctor->operator()(messageObject);

			//return the result to the main thread
            // 使用了Concurrent所以是线程安全的
			m_pActorModule->AddResult(messageObject);
		}
	}

	return true;
}
//  注册回调是从这里添加的 
bool NFActor::AddMessageHandler(const int nSubMsgID, ACTOR_PROCESS_FUNCTOR_PTR xBeginFunctor)
{
	return mxProcessFunctor.AddElement(nSubMsgID, xBeginFunctor);
}
// HelloWorld4Module.cpp的测试例子是从Component添加的
class NFHttpComponent : public NFIComponent
{
public:
	NFHttpComponent() : NFIComponent(GET_CLASS_NAME(NFHttpComponent))
	{
	}
	virtual ~NFHttpComponent()
	{

	}
	virtual bool Init()
	{
		AddMsgHandler(0, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(1, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(2, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(3, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(4, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(5, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(6, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(7, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(8, this, &NFHttpComponent::OnMsgEvent);
		AddMsgHandler(9, this, &NFHttpComponent::OnMsgEvent);
		return true;
	}
	//it's very important to note here:
	//sometimes, the function below not finished but other thread call it at the same time
	//the reason is the main thread post a new message to this actor and the schedulel assigned another thread to take the execution right
	//so, you wouldnot use the data which not thread-safe in this function
	virtual int OnMsgEvent(NFActorMessage& arg)
	{
		std::cout << "Thread: " << std::this_thread::get_id() << " MsgID: " << arg.msgID << " Data:" << arg.data << std::endl;
		return 0;
	}
};
class NFIComponent{
......
	template<typename BaseType>
	bool AddMsgHandler(const int nSubMessage, BaseType* pBase, int (BaseType::*handler)(NFActorMessage&))
	{
		ACTOR_PROCESS_FUNCTOR functor = std::bind(handler, pBase, std::placeholders::_1);
		ACTOR_PROCESS_FUNCTOR_PTR functorPtr(new ACTOR_PROCESS_FUNCTOR(functor));
	     // mSelf就是Actor	
		return mSelf->AddMessageHandler(nSubMessage, functorPtr);
	}
...
}
```
#### 使用四部曲
1. 创建Actor对象
2. 创建Component注册回调函数和处理回调函数(可以直接调用Actor->AddMessageHandler注册回调)
3. 注册结束回调函数
4. 发送消息给Actor

#### 画个UML方便记忆推演
![类图](NFActorPluginClass.png)
![时序图](NFActorPluginSeq.png)

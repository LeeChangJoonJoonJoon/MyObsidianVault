

2024-09-11

----


## 개요
현재 회사에서 스레드 풀을 사용하여 멀티스레딩 작업을 관리하고자 한다.
여기서 `FAsyncTaskBase`를 상속 받은 `FAsyncTask`를 사용하고자 한다. 

## 내용
코드는 다음과 같다. 
```cpp
class FAsyncTaskBase  
    : private UE::FInheritedContextBase  
    , private IQueuedWork  
{  
    /** Thread safe counter that indicates WORK completion, no necessarily finalization of the job */  
    FThreadSafeCounter WorkNotFinishedCounter;  
    /** If we aren't doing the work synchronously, this will hold the completion event */  
    FEvent*             DoneEvent = nullptr;  
    /** Pool we are queued into, maintained by the calling thread */  
    FQueuedThreadPool* QueuedPool = nullptr;  
    /** Current priority */  
    EQueuedWorkPriority Priority = EQueuedWorkPriority::Normal;  
    /** Current flags */  
    EQueuedWorkFlags Flags = EQueuedWorkFlags::None;  
    /** Approximation of the peak memory (in bytes) this task could require during it's execution. */  
    int64 RequiredMemory = -1;  
    /** Text to identify the Task; used for debug/log purposes only. */  
    const TCHAR * DebugName = nullptr;  
    /** StatId used for FScopeCycleCounter */  
    TStatId StatId;  
  
    /* Internal function to destroy the completion event  
    **/    void DestroyEvent()  
    {       FPlatformProcess::ReturnSynchEventToPool(DoneEvent);  
       DoneEvent = nullptr;  
    }  
    EQueuedWorkFlags GetQueuedWorkFlags() const final  
    {  
       return Flags;  
    }  
    /* Generic start function, not called directly  
       * @param bForceSynchronous if true, this job will be started synchronously, now, on this thread    **/    void Start(bool bForceSynchronous, FQueuedThreadPool* InQueuedPool, EQueuedWorkPriority InQueuedWorkPriority, EQueuedWorkFlags InQueuedWorkFlags, int64 InRequiredMemory, const TCHAR * InDebugName)  
    {       CaptureInheritedContext();  
  
       FScopeCycleCounter Scope(StatId, true);  
       DECLARE_SCOPE_CYCLE_COUNTER( TEXT( "FAsyncTask::Start" ), STAT_FAsyncTask_Start, STATGROUP_ThreadPoolAsyncTasks );  
       // default arg has InRequiredMemory == -1  
       RequiredMemory = InRequiredMemory;  
       DebugName = InDebugName;  
  
       FPlatformMisc::MemoryBarrier();  
       CheckIdle();  // can't start a job twice without it being completed first  
       WorkNotFinishedCounter.Increment();  
       QueuedPool = InQueuedPool;  
       Priority = InQueuedWorkPriority;  
       Flags = InQueuedWorkFlags;  
       if (bForceSynchronous)  
       {          QueuedPool = 0;  
       }       if (QueuedPool)  
       {          if (!DoneEvent)  
          {             DoneEvent = FPlatformProcess::GetSynchEventFromPool(true);  
          }          DoneEvent->Reset();  
          QueuedPool->AddQueuedWork(this, InQueuedWorkPriority);  
       }       else   
{  
          // we aren't doing async stuff  
          DestroyEvent();  
          DoWork();  
       }    }  
    /**   
    * Tells the user job to do the work, sometimes called synchronously, sometimes from the thread pool. Calls the event tracker.  
    **/    void DoWork()  
    {       UE::FInheritedContextScope InheritedContextScope = RestoreInheritedContext();  
       FScopeCycleCounter Scope(StatId, true);  
  
       DoTaskWork();  
       check(WorkNotFinishedCounter.GetValue() == 1);  
       WorkNotFinishedCounter.Decrement();  
    }  
    /**   
    * Triggers the work completion event, only called from a pool thread  
    **/    void FinishThreadedWork()  
    {       check(QueuedPool);  
       if (DoneEvent)  
       {          FScopeCycleCounter Scope(StatId, true);  
          DECLARE_SCOPE_CYCLE_COUNTER( TEXT( "FAsyncTask::FinishThreadedWork" ), STAT_FAsyncTask_FinishThreadedWork, STATGROUP_ThreadPoolAsyncTasks );        
DoneEvent->Trigger();  
       }    }  
    /**   
    * Performs the work, this is only called from a pool thread.  
    **/    void DoThreadedWork() final  
    {  
       DoWork();  
       FinishThreadedWork();  
    }  
    /**  
     * Always called from the thread pool. Called if the task is removed from queue before it has started which might happen at exit.     * If the user job can abandon, we do that, otherwise we force the work to be done now (doing nothing would not be safe).     */    void Abandon() final  
    {  
       if (TryAbandonTask())  
       {          check(WorkNotFinishedCounter.GetValue() == 1);  
          WorkNotFinishedCounter.Decrement();  
       }       else  
       {  
          DoWork();  
       }       FinishThreadedWork();  
    }  
    /**  
    * Internal call to synchronize completion between threads, never called from a pool thread    * @param bIsLatencySensitive specifies if waiting for the task should return as soon as possible even if this delays other tasks    **/    void SyncCompletion(bool bIsLatencySensitive)  
    {       TRACE_CPUPROFILER_EVENT_SCOPE(FAsyncTask::SyncCompletion);  
  
       FPlatformMisc::MemoryBarrier();  
       if (QueuedPool)  
       {          FScopeCycleCounter Scope(StatId);  
          DECLARE_SCOPE_CYCLE_COUNTER(TEXT("FAsyncTask::SyncCompletion"), STAT_FAsyncTask_SyncCompletion, STATGROUP_ThreadPoolAsyncTasks);  
  
          if (LowLevelTasks::FScheduler::Get().IsWorkerThread() && !bIsLatencySensitive)  
          {             LowLevelTasks::BusyWaitUntil([this]() { return IsWorkDone(); });  
          }  
          check(DoneEvent); // if it is not done yet, we must have an event  
          DoneEvent->Wait();  
          QueuedPool = 0;  
       }       CheckIdle();  
    }  
protected:  
    void Init(TStatId InStatId)  
    {       StatId = InStatId;  
    }  
    /**   
    * Internal call to assert that we are idle  
    **/    void CheckIdle() const  
    {  
       check(WorkNotFinishedCounter.GetValue() == 0);  
       check(!QueuedPool);  
    }  
    /** Perform task's work */  
    virtual void DoTaskWork() = 0;  
  
    /**   
    * Abandon task if possible, returns true on success, false otherwise.  
    **/    virtual bool TryAbandonTask() = 0;  
  
public:  
    /** Destructor, not legal when a task is in process */  
    virtual ~FAsyncTaskBase()  
    {       // destroying an unfinished task is a bug  
       CheckIdle();  
       DestroyEvent();  
    }  
    /**  
     * Returns an approximation of the peak memory (in bytes) this task could require during it's execution.     **/    int64 GetRequiredMemory() const final  
    {  
       return RequiredMemory;  
    }    const TCHAR * GetDebugName() const final  
    {  
       return DebugName;  
    }  
    /**   
    * Run this task on this thread  
    * @param bDoNow if true then do the job now instead of at EnsureCompletion    **/    void StartSynchronousTask(EQueuedWorkPriority InQueuedWorkPriority = EQueuedWorkPriority::Normal, EQueuedWorkFlags InQueuedWorkFlags = EQueuedWorkFlags::None, int64 InRequiredMemory = -1, const TCHAR * InDebugName = nullptr)  
    {       Start(true, GThreadPool, InQueuedWorkPriority, InQueuedWorkFlags, InRequiredMemory, InDebugName);  
    }  
    /**   
    * Queue this task for processing by the background thread pool  
    **/    void StartBackgroundTask(FQueuedThreadPool* InQueuedPool = GThreadPool, EQueuedWorkPriority InQueuedWorkPriority = EQueuedWorkPriority::Normal, EQueuedWorkFlags InQueuedWorkFlags = EQueuedWorkFlags::None, int64 InRequiredMemory = -1, const TCHAR * InDebugName = nullptr)  
    {       Start(false, InQueuedPool, InQueuedWorkPriority, InQueuedWorkFlags, InRequiredMemory, InDebugName);  
    }  
    /**   
    * Wait until the job is complete  
    * @param bDoWorkOnThisThreadIfNotStarted if true and the work has not been started, retract the async task and do it now on this thread    * @param specifies if waiting for the task should return as soon as possible even if this delays other tasks    **/    void EnsureCompletion(bool bDoWorkOnThisThreadIfNotStarted = true, bool bIsLatencySensitive = false)  
    {       bool DoSyncCompletion = true;  
       if (bDoWorkOnThisThreadIfNotStarted)  
       {          if (QueuedPool)  
          {             if (QueuedPool->RetractQueuedWork(this))  
             {                // we got the job back, so do the work now and no need to synchronize  
                DoSyncCompletion = false;  
                DoWork();   
FinishThreadedWork();  
                QueuedPool = 0;  
             }          }          else if (WorkNotFinishedCounter.GetValue())  // in the synchronous case, if we haven't done it yet, do it now  
          {  
             DoWork();   
          }  
       }       if (DoSyncCompletion)  
       {          SyncCompletion(bIsLatencySensitive);  
       }       CheckIdle(); // Must have had bDoWorkOnThisThreadIfNotStarted == false and needed it to be true for a synchronous job  
    }  
    /**  
    * If not already being processed, will be rescheduled on given thread pool and priority.    * @return true if the reschedule was successful, false if was already being processed.    **/    bool Reschedule(FQueuedThreadPool* InQueuedPool = GThreadPool, EQueuedWorkPriority InQueuedWorkPriority = EQueuedWorkPriority::Normal)  
    {       if (QueuedPool)  
       {          if (QueuedPool->RetractQueuedWork(this))  
          {             QueuedPool = InQueuedPool;  
             Priority = InQueuedWorkPriority;  
             QueuedPool->AddQueuedWork(this, InQueuedWorkPriority);  
             return true;  
          }       }       return false;  
    }  
    /**  
    * Cancel the task, if possible.    * Note that this is different than abandoning (which is called by the thread pool at shutdown).    * @return true if the task was canceled and is safe to delete. If it wasn't canceled, it may be done, but that isn't checked here.  
    **/    bool Cancel()  
    {       if (QueuedPool)  
       {          if (QueuedPool->RetractQueuedWork(this))  
          {             check(WorkNotFinishedCounter.GetValue() == 1);  
             WorkNotFinishedCounter.Decrement();  
             FinishThreadedWork();  
             QueuedPool = 0;  
             return true;  
          }       }       return false;  
    }  
    /**  
    * Wait until the job is complete, up to a time limit    * @param TimeLimitSeconds Must be positive, otherwise polls -- same as calling IsDone()    * @return true if the task is completed    **/    bool WaitCompletionWithTimeout(float TimeLimitSeconds)  
    {       if (TimeLimitSeconds <= 0.0f)  
       {          return IsDone();  
       }  
       FPlatformMisc::MemoryBarrier();  
       if (QueuedPool)  
       {          FScopeCycleCounter Scope(StatId);  
          DECLARE_SCOPE_CYCLE_COUNTER(TEXT("FAsyncTask::SyncCompletion"), STAT_FAsyncTask_SyncCompletion, STATGROUP_ThreadPoolAsyncTasks);  
  
          uint32 Ms = uint32(TimeLimitSeconds * 1000.0f) + 1;  
          check(Ms);  
  
          check(DoneEvent); // if it is not done yet, we must have an event  
          if (DoneEvent->Wait(Ms))  
          {             QueuedPool = 0;  
             CheckIdle();  
             return true;  
          }          return false;  
       }       CheckIdle();  
       return true;  
    }  
    /** Returns true if the work and TASK has completed, false while it's still in progress.   
     * prior to returning true, it synchronizes so the task can be destroyed or reused  
     */    bool IsDone()  
    {       if (!IsWorkDone())  
       {          return false;  
       }       SyncCompletion(/*bIsLatencySensitive = */false);  
       return true;  
    }  
    /** Returns true if the work has completed, false while it's still in progress.   
     * This does not block and if true, you can use the results.  
     * But you can't destroy or reuse the task without IsDone() being true or EnsureCompletion()    */    bool IsWorkDone() const  
    {  
       if (WorkNotFinishedCounter.GetValue())  
       {          return false;  
       }       return true;  
    }  
    /** Returns true if the work has not been started or has been completed.   
     * NOT to be used for synchronization, but great for check()'s   
     */  
    bool IsIdle() const  
    {  
       return WorkNotFinishedCounter.GetValue() == 0 && QueuedPool == 0;  
    }  
    bool SetPriority(EQueuedWorkPriority QueuedWorkPriority)  
    {       return Reschedule(QueuedPool, QueuedWorkPriority);  
    }  
    EQueuedWorkPriority GetPriority() const  
    {  
       return Priority;  
    }};  
  
template<typename TTask>  
class FAsyncTask  
    : public FAsyncTaskBase  
{  
    /** User job embedded in this task */   
TTask Task;  
  
public:  
    FAsyncTask()  
       : Task()  
    {       // Cache the StatId to remain backward compatible with TTask that declare GetStatId as non-const.  
       Init(Task.GetStatId());  
    }  
    /** Forwarding constructor. */  
    template <typename Arg0Type, typename... ArgTypes>  
    FAsyncTask(Arg0Type&& Arg0, ArgTypes&&... Args)  
       : Task(Forward<Arg0Type>(Arg0), Forward<ArgTypes>(Args)...)  
    {       // Cache the StatId to remain backward compatible with TTask that declare GetStatId as non-const.  
       Init(Task.GetStatId());  
    }  
    /* Retrieve embedded user job, not legal to call while a job is in process  
    * @return reference to embedded user job    **/    TTask& GetTask()  
    {       CheckIdle();  // can't modify a job without it being completed first  
       return Task;  
    }  
    /* Retrieve embedded user job, not legal to call while a job is in process  
    * @return reference to embedded user job    **/    const TTask& GetTask() const  
    {  
       CheckIdle();  // could be safe, but I won't allow it anyway because the data could be changed while it is being read  
       return Task;  
    }  
    bool TryAbandonTask() final  
    {  
       if (Task.CanAbandon())  
       {          Task.Abandon();  
          return true;  
       }  
       return false;  
    }  
    void DoTaskWork() final  
    {  
       Task.DoWork();  
    }};
```

`IQueuedWork`를 살펴 보니, 다음과 같은 주석이 있었음.
```cpp
/**  
 * Interface for queued work objects. * * This interface is a type of runnable object that requires no per thread * initialization. It is meant to be used with pools of threads in an * abstract way that prevents the pool from needing to know any details * about the object being run. This allows queuing of disparate tasks and * servicing those tasks with a generic thread pool. */
```

즉, 스레드 풀이 먼저 있는 것이고, 이 스레드들에 부여할 작업을 정의하는 추상클래스가 바로 `IQueuedWork`인 것.
이를 다중상속 받아서 구현하고 있는 `FAsyncTaskBase`에서 내가 작업을 정의해 놓은 부분을 찾을 수 있었다. 
```cpp
    /** Perform task's work */  
    virtual void DoTaskWork() = 0;  
  
    /**   
    * Abandon task if possible, returns true on success, false otherwise.  
    **/    virtual bool TryAbandonTask() = 0;  
```

이론적으로, 스레드 풀을 구현할 때 필요한 재료들을 먼저 생각해 보자. 
이걸 먼저 리스트업하고 거기에 해당하는 코드를 찾아가 보자.
- 어떤 일을 할지 정의된 `Runnable`, 혹은 `work object`. 아마 함수로 구현해야겠지?
- 일을 하기 위해 필요한 변수들. 우리의 경우 현재 부여되는 `Runnable`이 처리하고 있는 .csv 파일의 행 인덱스 `int64`와 파싱한 애들을 담을 `TArray<FVector4>&`
- 몇개의 스레드를 생성하여 관리할지에 대한 정수현 인수 `int`... 아니면 내가 조정히지 않아도 알아서 갯수를 조절하나?

그리고 클래스 내에 당연히 구현돼 있어야 하는 부분들...
- 스레드를 담는큐 자료형
- `Runnable`이 부여되면 가용 스레드를 찾아 이를 실행시키는 부분
- `Runnable`이 끝나면 다시 큐로 돌려놓는 부분
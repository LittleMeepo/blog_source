---
title: '[6.824] MapReduce Lab'
date: 2020-12-6 16:53
tags:
- 6.824
- Labs
categories:
- 分布式
---

## MapRudece Lab

### Master 节点

MapReduce是一个分布式计算模型，这是MapReduce的go语言版本的一个简单实现。

- mrmaster.go

```go
func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "Usage: mrmaster inputfiles...\n")
		os.Exit(1)
	}

	m := mr.MakeMaster(os.Args[1:], 10) //创建一个master
	for m.Done() == false { //主线程等待结束
		time.Sleep(time.Second)
	}

	time.Sleep(time.Second)
}
```

- master.go

```go
//
// create a Master.
//
func MakeMaster(files []string, nReduce int) *Master {
	m := &Master{}
	m.mu = sync.Mutex{}
	m.nReduce = nReduce //输入的参数nReduce（输入的文件会被划分成几个task来处理）
	m.files = files //文件名数组
	if nReduce > len(files) { //确定分发task的channel的缓冲区大小
		m.taskCh = make(chan Task, nReduce)
	} else {
		m.taskCh = make(chan Task, len(m.files)) //文件数量多于分成的task数量
	}

	m.initMapTask()
	go m.tickSchedule()
	m.server()
	DPrintf("Master init")
	return m
}
```

Master结构的定义：

```go
type Master struct {
	files      []string   //需要处理的files
	nReduce    int        //输入的参数nReduce（输入的文件会被划分成几个task来处理）
	taskPhase  TaskPhase  //taskPhase（map阶段还是reduce阶段）
	taskStats  []TaskStat //taskStats（各个task的状态）
	taskNum    int        //task数量
	mu         sync.Mutex //mu（全局锁）
	done       bool       //done（任务是否已完成）
	workerSeq  int        //workerSeq（有几个worker）
	taskCh     chan Task  //taskCh（用来分发task的channel）
	finishTask int32      //statCh（用来接受完成task数量）
}
```

Master包含的本地method有：initMapTask()  taskSchedule()  getTask()  tickSchedule()  initReduceTask()

```go
func (m *Master) initMapTask() {
	m.mu.Lock()
	defer m.mu.Unlock()

	DPrintf("Init Map Task")
	m.taskPhase = MapPhase //设置阶段
	m.taskStats = make([]TaskStat, len(m.files)) //创建task状态数组
	m.taskNum = len(m.files) //task数量
	m.finishTask = 0
	for index := range m.taskStats {
		go m.taskSchedule(index) //对每一个task，创建一个状态机循环
	}
}
```

单个task的调度，用一个Loop来实现一个状态机。

1. 如果处在TaskStatusReady状态，则getTask()并放进channel
2. 如果处在TaskStatusQueue，什么都不做
3. 如果处在TaskStatusRunning，如果任务超时，则重新getTask()并放进channel
4. 如果处在TaskStatusFinish，则finishTask自加1
5. 如果处在TaskStatusErr，则getTask()并放进channel

```go
func (m *Master) taskSchedule(taskSeq int) {
	for {
		if m.Done() { //如果Done则结束
			return
		}
		m.mu.Lock()
		DPrintf("Schedule begin, task:%v, Status: %v", taskSeq, m.taskStats[taskSeq].Status)
		switch m.taskStats[taskSeq].Status { //根据处在的不同状态，完成不同的操作
		case TaskStatusReady:
			m.taskCh <- m.getTask(taskSeq)
			m.taskStats[taskSeq].Status = TaskStatusQueue
		case TaskStatusQueue:
		case TaskStatusRunning:
			if time.Since(m.taskStats[taskSeq].StartTime) > MaxTaskRunTime {
				m.taskStats[taskSeq].Status = TaskStatusQueue
				m.taskCh <- m.getTask(taskSeq)
			}
		case TaskStatusFinish:
			m.finishTask += 1
			m.mu.Unlock()
			return //单个task完成了就结束
		case TaskStatusErr:
			m.taskStats[taskSeq].Status = TaskStatusQueue
			m.taskCh <- m.getTask(taskSeq)
		default:
			m.mu.Unlock()
			panic("Task status err")
		}
		DPrintf("Schedule end, task:%v, Status: %v", taskSeq, m.taskStats[taskSeq].Status)
		m.mu.Unlock()
		time.Sleep(ScheduleInterval) //睡眠一个间隙
	}
}
```

接下来是getTask()的实现

```go
func (m *Master) getTask(taskSeq int) Task {
	task := Task{
		FileName: "",
		NReduce:  m.nReduce,
		NMaps:    len(m.files),
		Seq:      taskSeq, //序号
		Phase:    m.taskPhase,
		Alive:    true,
	}
	DPrintf("Get task, taskseq:%d, len files:%d, len tasks:%d", m, taskSeq, len(m.files), len(m.taskStats))
	if task.Phase == MapPhase {
		task.FileName = m.files[taskSeq]
	}
	return task
}
```

task的结构体

```go
type Task struct {
	FileName string
	NReduce  int
	NMaps    int
	Seq      int
	Phase    TaskPhase
	Alive    bool // worker should exit when alive is false
}
```

当map阶段完成，则开始进入reduce阶段。该函数由独立线程执行，以检测当前map状态

```go
func (m *Master) tickSchedule() {
	for !m.Done() {
		m.mu.Lock()
		DPrintf("Global schedule, finTask:%v, taskNum:%v\n", m.finishTask, m.taskNum)
		if m.finishTask == int32(m.taskNum) {
			if m.taskPhase == MapPhase {
				m.mu.Unlock()
				m.initReduceTask()
				time.Sleep(ScheduleInterval)
				continue
			} else {
				m.done = true
			}
		}
		m.mu.Unlock()
		time.Sleep(ScheduleInterval)
	}
}
```

initReduceTask()的实现：

```go
func (m *Master) initReduceTask() {
	m.mu.Lock()
	defer m.mu.Unlock()

	DPrintf("Init Reduce Task")
	m.taskPhase = ReducePhase
	m.taskStats = make([]TaskStat, m.nReduce)
	m.taskNum = m.nReduce
	m.finishTask = 0
	for index := range m.taskStats {
		go m.taskSchedule(index) //和map阶段类似，对每一个task，创建一个状态机循环
	}
}
```

接下来是server()函数，用来监听Worker的RPC调用

```go
func (m *Master) server() {
	rpc.Register(m) //注册RPC
	rpc.HandleHTTP()
	sockname := masterSock()
	os.Remove(sockname)
	l, e := net.Listen("unix", sockname)
	if e != nil {
		log.Fatal("listen error:", e)
	}
	go http.Serve(l, nil) //开始监听
}
```

Master其余method均为给Worker调用，在本地RPC框架中注册，包括regTask()  GetOneTask()  ReportTask()  RegWorker()

```go
//结构体
type TaskArgs struct {
	WorkerId int
}

type TaskReply struct {
	Task *Task
}
```

注册task，其实就是在Master端记录当前task的状态

```go
func (m *Master) regTask(args *TaskArgs, task *Task) {
	m.mu.Lock()
	defer m.mu.Unlock()

	if task.Phase != m.taskPhase {
		panic("Task phase doesn't match")
	}

	m.taskStats[task.Seq].Status = TaskStatusRunning
	m.taskStats[task.Seq].WorkerId = args.WorkerId
	m.taskStats[task.Seq].StartTime = time.Now()
}
```

从channel中获得一个task，并返回给worker

```go
func (m *Master) GetOneTask(args *TaskArgs, reply *TaskReply) error {
	task := <-m.taskCh
	reply.Task = &task

	if task.Alive {
		m.regTask(args, &task)
	}
	DPrintf("Get one Task, args:%+v, reply:%+v", args, reply)
	return nil
}
```

报告task的函数，如果完成了，就将task在状态数组中的状态改为Finish，否则改为Error

```go
type ReportTaskArgs struct {
	Done     bool
	Seq      int
	Phase    TaskPhase
	WorkerId int
}

type ReportTaskReply struct {
}
```

```go
func (m *Master) ReportTask(args *ReportTaskArgs, reply *ReportTaskReply) error {
	m.mu.Lock()
	defer m.mu.Unlock()

	DPrintf("Get report task: %+v, taskPhase: %+v", args, m.taskPhase)

	if m.taskPhase != args.Phase || args.WorkerId != m.taskStats[args.Seq].WorkerId {
		return nil
	}

	if args.Done {
		m.taskStats[args.Seq].Status = TaskStatusFinish
	} else {
		m.taskStats[args.Seq].Status = TaskStatusErr
	}

	return nil
}
```

注册Worker的函数，Master给各个Worker分配一个序号

```go
type RegisterArgs struct {
}

type RegisterReply struct {
	WorkerId int
}
```

```go
func (m *Master) RegWorker(args *RegisterArgs, reply *RegisterReply) error {
	m.mu.Lock()
	defer m.mu.Unlock()

	m.workerSeq++
	reply.WorkerId = m.workerSeq
	return nil
}
```

### Worker 节点

- mrworker.go

mapf和reducef分别为从外部库导入到自定义map函数和reduce函数

```go
func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: mrworker xxx.so\n")
		os.Exit(1)
    }
	mapf, reducef := loadPlugin(os.Args[1])
	mr.Worker(mapf, reducef)
}
//加载自定义的map函数和reduce函数
func loadPlugin(filename string) (func(string, string) []mr.KeyValue, func(string, []string) string) {
	p, err := plugin.Open(filename)
	if err != nil {
		log.Fatalf("cannot load plugin %v", filename)
	}
	xmapf, err := p.Lookup("Map")
	if err != nil {
		log.Fatalf("cannot find Map in %v", filename)
	}
	mapf := xmapf.(func(string, string) []mr.KeyValue)
	xreducef, err := p.Lookup("Reduce")
	if err != nil {
		log.Fatalf("cannot find Reduce in %v", filename)
	}
	reducef := xreducef.(func(string, []string) string)

	return mapf, reducef
}
```

worker函数，函数参数为map和reduce函数指针

```
func Worker(mapf func(string, string) []KeyValue,
	reducef func(string, []string) string) {

	w := worker{}
	w.mapf = mapf
	w.reducef = reducef
	w.register()
	w.run()
}
```

worker结构体

```go
type worker struct {
	id      int
	mapf    func(string, string) []KeyValue
	reducef func(string, []string) string
}
```

将mapf和reducef函数指针进行赋值后，向Master注册自己，其实就是获得一个Master分配的递增id

```go
func (w *worker) register() {
	args := &RegisterArgs{}
	reply := &RegisterReply{}
	if ok := call("Master.RegWorker", args, reply); !ok {
		log.Fatal("reg fail")
	}
	w.id = reply.WorkerId
}
```

然后执行run函数

```go
func (w *worker) run() {
	// if reqTask conn fail, worker exit
	for {
		t := w.reqTask()
		if !t.Alive {
			DPrintf("worker %v get task not alive, exit", w.id)
			return
		}
		DPrintf("worker %v get task alive", w.id)
		w.doTask(t)
	}
}
```

在run函数中，循环请求（reqTask）和处理（doTask）任务

```go
func (w *worker) reqTask() Task {
	args := TaskArgs{}
	args.WorkerId = w.id
	reply := TaskReply{}

	if ok := call("Master.GetOneTask", &args, &reply); !ok {
		DPrintf("worker get task fail,exit")
		os.Exit(1)
	}
	DPrintf("worker get task:%+v", reply.Task)
	return *reply.Task //请求一个任务并返回
}
```

根据不用阶段，分别doMapTask和doReduceTask，其中分别调用了mapf和reducef，完成或者出错，则调用reportTask向Master进行报告

```go
func (w *worker) doTask(t Task) {
	DPrintf("Worker do Task")

	switch t.Phase {
	case MapPhase:
		w.doMapTask(t)
	case ReducePhase:
		w.doReduceTask(t)
	default:
		panic(fmt.Sprintf("task phase err: %v", t.Phase))
	}
}
```

最后是RPC调用过程中call函数的实现，对系统提供的RPC接口进行了封装

```go
//
// send an RPC request to the master, wait for the response.
// usually returns true.
// returns false if something goes wrong.
//
func call(rpcname string, args interface{}, reply interface{}) bool {
	// c, err := rpc.DialHTTP("tcp", "127.0.0.1"+":1234")
	sockname := masterSock()
	c, err := rpc.DialHTTP("unix", sockname)
	if err != nil {
		DPrintf("dialing:", err)
		return false
	}
	defer c.Close()

	err = c.Call(rpcname, args, reply)
	if err == nil {
		return true
	}

	DPrintf("%+v", err)
	return false
}
```


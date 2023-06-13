#! https://zhuanlan.zhihu.com/p/635629729
# MIT6.5840 raft 2A 思考
raft中文翻译版本：[寻找一种易于理解的一致性算法（扩展版）](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)。
（之前对着英文版看了半天才发现有中文。。。。）

依据我一小勺的raft理解，如果真要实现的话，可能把每个节点抽象成状态机更好一点。
也就是每个节点只对接受的网络请求有反应，根据这个请求来更改自身的状态（比如存储，还有自己的角色转换，或者什么都不做只返回一个response）。

如果真要自己写出来，我个人更倾向于先捋清楚这个状态的转换。
所以在这里做一个可能不完全对的思考记录。

---
# A. 领导者选举
首先看一下领导者选举。

当前节点处于follower状态下有如下几种可能：
1. 自己超时了，无缝切换到candidate状态。
2. 收到投票请求，分几种情况：
    - 请求中 *term* 比自己小，过期消息，回复拒绝。
    - 请求中 *term* 大于等于自己的，需要看日志。
        - 如果请求中日志比自己长，说明对面更新，尝试投票。
        - 如果请求中日志长度跟自己一样，比较任期，任期大于等于就尝试投票。
        - 比自己短，拒绝投票。
3. 收到投票请求回复，忽略，自己不是candidate。
4. 收到追加请求：
    - 请求中 *term* 比自己更**小**，这条过期了，回复拒绝。
    - 请求中 *term* **跟自己一样**或者更**大**，联系到leader了，更新一下计时器，回复leader表示自己已阅。
5. 收到追加请求回复，忽略，自己不是leader。

当节点处于leader状态下有如下几种可能：
1. 收到追加请求，上一个leader遗留下来的，或者自己断联：
    - 检查term，term比自己**大**那转为follower。
    - 其他不管
2. 收到追加请求的回复，肯定是发给自己的。
3. 收到投票请求：
    - 请求中 *term* 比自己**小**或者**跟自己一样**，说明是过期的东西，忽略。
    - 请求中 *term* 比自己**大**，说明当前状态下，有节点收不到自己的消息，但是自己收到。同样分情况，转到follower，继续处理请求。
4. 收到投票请求回复，自己在上N轮的投票请求的回复，忽略。

当节点处于candidate状态下有如下几种可能：
1. 定时器超时或者转到此状态，*term* 自增然后重置定时器。
1. 收到追加请求：
    - 请求的任期比自己的大或者相等，变为follower
    - 请求的任期比自己的小，拒绝
2. 收到追加请求回复，大概率是自己前一刻为leader发送请求宕机了，恢复后超时，可以忽略了，自己知道也没用。
3. 收到投票请求，尝试投票（但是票没了，所以一定是拒绝）。
4. 收到投票请求回复，统计一下收到的票数，半数以上可以转到leader。

## 真是 *not understandable* ，来实现。
先冥想一下，实现起来是什么样的。
如果让我实现，我可能用一个多进程，每个进程表示一个节点，进程之间找一个RPC框架进行通信。

每个进程内存在一个线程，执行大的循环，根据当前进程的状态来确定当前该怎么办，可能统一侦听一个端口，想要实现计时器，可以弄成非阻塞的io（异步io不确定好不好，毕竟要确定超时）。

然后是状态的表示，我倾向于用一个static int来表示状态，可能0表示follower，1表示leader，2表示candidate。

```c++
static int state = 0;
static int time_out_cnt = 100;//不同状态可能有所不同
static int term = 0;
static int vote = -1;
static int total_nodes = 5;

using handle = function<>;
map<int,handle> handles;
auto getHandle()
{
    return handles[state];
}
void handleLeader()
{
    auto buf = recv();//一次非阻塞调用
    auto ret = parse_recv();
    switch(ret.rpc_type)
    {
        case....// maybe updateState()
    }
}
void handleFollower(){/*like handleLeader*/}
void handleCandidate(){/*like handleLeader*/}
void updateState(int new_state){state = new_state;// reset time_out}
void raft()
{
    while(true)
    {
        getHandle()();
        update_timer();
    }
}
int main()
{
    raft();
}
```
## 6.5840中go的代码。
我看了半天才看明白他具体让我做的什么，是说有一个 `struct` 叫 `Raft`，其中包含节点的各种信息，然后在*raft.go*中存在一个函数叫 `Make`，它返回结构题的指针用于测试当前状态对不对，然后节点实际运行在一个或者多个**goroutine**中，这个**goroutine**是在`Make`函数中开始运行的。

所以任务总结一下就是：
1. 定义一下`Raft struct`。
2. 实现**goroutine**调用的函数。

比较烦的一点是，RPC调用是纯纯异步的，整个RPC的执行过程是异步的，所以需要将这个异步变成同步的过程，我考虑用channel实现。
然后还有外部是有另一个线程需要读取`Raft struct`中的信息的，所以这个过程是读写冲突，一定要上锁了，在内部也给了一个大锁。

那么我的思考是，整体输入分为**RPC请求和reply**，RPC的执行过程中需要进行阻塞来进行同步，reply为自己发送的RPC请求的回复，同时定时器也抽象成一个reply，那么在整个循环中可以组合成检查RPC请求，接受并处理reply和执行自己的状态转移。
我就把回复组合在一起了，用一个int进行区分。
```go
const (
	APPEND_ENTRIES_RPC = iota
	REQUEST_VOTE       = iota
	TIMER              = iota
)
type RPC_Reply struct {
	rpc_type int
	term     int32
	good     bool
}
```
那么`Raft`结构内部就可以加入几个channel用于阻塞，再加入用于无日志选举的数据：
```go
type Raft struct {
	mu        sync.Mutex          // Lock to protect shared access to this peer's state
	peers     []*labrpc.ClientEnd // RPC end points of all peers
	persister *Persister          // Object to hold this peer's persisted state
	me        int                 // this peer's index into peers[]
	dead      int32               // set by Kill()

	// Your data here (2A, 2B, 2C).
	// Look at the paper's Figure 2 for a description of what
	// state a Raft server must maintain.
	term      int32
	vote_to   int32
	state     int32
	time_out  time.Duration
	voted_cnt int32

	// channel for loop
	RPC_arrive_channel chan int
	RPC_block_channel  chan int
	RPC_done_channel   chan int

	Reply_channel chan RPC_Reply
}
```
整体的一个大循环，每sleep多少秒就唤醒去更新自己的状态，比如定时器，处理RPC，发心跳。
这里reply应该是有一个优先级的，不然会随机报一个warning。
注意整个检查RPC请求和reply的过程要非阻塞的，RPC请求过程是阻塞的，少写default就出错。
```go
func (rf *Raft) loopHandle() {
	select {
	case <-rf.RPC_arrive_channel: // multiple rpc may arrive at the same time, just pick one
		// do rpc
		rf.RPC_block_channel <- 1 //执行RPC请求
		// wait rpc complete
		<-rf.RPC_done_channel //等待RPC请求处理完毕
	default://不要阻塞
	}
	// handle may call rpcs
	// handle these rpcs's reply
    // 回复集中处理
	has_reply := true
	for has_reply {
		select {
		case reply := <-rf.Reply_channel:
			rf.handleReply(&reply)
		default:
			has_reply = false
		}
	}
	// do handle
	switch rf.state {
	case FOLLOWER:
		rf.HandleFollower()
	case LEADER:
		rf.HandleLeader()
	case CANDIDATE:
		rf.HandleCandidate()
	}
}
```
每个**收到的**RPC请求，需要进行一个阻塞，并在最后取消loop线程的阻塞
```go
func (rf *Raft) RequestVote(args *RequestVoteArgs, reply *RequestVoteReply) {
	// Your code here (2A, 2B).
	rf.RPC_arrive_channel <- 1 //告知loop有RPC请求需要处理
	<-rf.RPC_block_channel//等待loop处理，在此处阻塞
    // do something
	rf.RPC_done_channel <- 1
}
```
每个loop handle自己**发出的**RPC请求，需要将结果进行保留，最好不要在发起请求的线程中进行处理
```go
func (rf *Raft) HandleCandidate() {
    // 其他逻辑.....
    // 并发发送投票请求
	for i := 0; i < len(rf.peers); i++ {
		if i != rf.me {
			go func(i int, args *RequestVoteArgs) {
				reply := &RPC_Reply{}
				rf.sendRequestVote(i, args, reply) // do not send twice whether the node is connected
				// reply will send to channel
                // 不要在此处处理投票
			}(i, args)
		}
	}
    // 其他逻辑.....
}
func (rf *Raft) sendRequestVote(server int, args *RequestVoteArgs, reply *RPC_Reply) bool {
	ok := rf.peers[server].Call("Raft.RequestVote", args, reply)
	if ok {
		rf.Reply_channel <- *reply
	}
	return ok
}
```
那么这么处理加锁就可以加一个大的，在loop里直接加:
```go
func (rf *Raft) loop() {
	for rf.killed() == false {
		// Your code here (2A)
		// Check if a leader election should be started.
		rf.mu.Lock()
		rf.loopHandle()
		rf.mu.Unlock()
		time.Sleep(time.Duration(25) * time.Millisecond)
	}
}
```
之后就是实现一下状态转移的细枝末节，然后进行测试。
Lab 2A中有3个测试
1. 初始选举测试（3个节点）
2. 掉线测试（3个节点，leader掉线）
3. 多重掉线测试（7个节点，10次测试）
每个测试都会有race的检查，有并发冲突就不行。
其中掉线的表现是网线拔了，但是程序还在走，所以要处理leader掉线后重连怎么转移状态。

## B. 日志
更新一下之前的所有可能，日志复制要考虑如何比较日志状态。

当前节点处于follower状态下有如下几种可能：
1. 自己超时了，无缝切换到candidate状态。
2. 收到投票请求，分几种情况：
    - 请求中 *term* 小与等于自己，过期消息，回复拒绝。
    - 请求中 *term* 大于自己的，需要看日志。
        - 如果请求中日志比自己长，说明对面更新，尝试投票。
        - 如果请求中日志长度跟自己一样，比较任期，任期大于等于就尝试投票。
        - 比自己短，拒绝投票。
3. 收到投票请求回复，查看任期号，比自己**高**立刻切换为follower并更新任期，否则忽略。
4. 收到追加请求：
    - 请求中 *term* 比自己更**小**，这条过期了，回复拒绝。
    - 请求中 *term* **跟自己一样**或者更**大**，联系到leader了，更新一下计时器，回复leader表示自己已阅，追加请求中要包含leader的上一个日志的 *index* 和*term*，所以根据这两个信息有如下几种情况。
		- 如果*index*和对应的*term*一致，追加这条请求（如果有日志的话），注意追加是根据prev追加的要截断一下log[]。
		- 如果没有这条记录，拒绝这次追加（别出空洞，索引从1开始，0索引认为一直有东西并且任期一直匹配，leader会一直往前找直到0）。
		- 如果*index*对应条目中含有的*term*与接受到的不一致，说明新leader不含有此条记录，进行一致性检查,删除不匹配之后的一切记录,拒绝这次追加。
5. 收到追加请求回复，忽略，自己不是leader。

leader节点要维护一下每个节点RPC请求的参数结构体，以及回复的状态。初始情况下，默认所有节点都一致。

将定时器拆分，每个其他节点一个单独定时器。

当节点处于leader状态下有如下几种可能：
1. 收到追加请求，上一个leader遗留下来的，或者自己断联：
    - 检查term，term比自己**大**那转为follower。
    - 其他不管，或者告诉返回拒绝。（自己的任期号一定大的，不用心跳传达能快一丢丢）
2. 收到追加请求的回复，填充下一次请求：
	- 如果当前节点与自己一致，填充同样内容。
	- 如果当前节点与自己不一致，查看回复：
		- 如果回复true，向下填充日志内容，*index*和*term*。
		- 如果回复false，向前移动
3. 收到投票请求：
    - 请求中 *term* 比自己**小**或者**跟自己一样**，说明是过期的东西，忽略。
    - 请求中 *term* 比自己**大**，说明当前状态下，有节点收不到自己的消息，但是自己收到。同样分情况，转到follower，继续处理请求。
4. 收到投票请求回复，自己在上N轮的投票请求的回复，忽略。
5. 定时器超时，给对应节点发送追加请求。
5. 收到客户端的请求时，立刻追加日志，如果当前节点与自己一致，填充请求，并置对应定时器为0。


当节点处于candidate状态下有如下几种可能：
1. 定时器超时或者转到此状态，*term* 自增然后重置定时器。
2. 收到追加请求：
    - 请求的任期比自己的大或者相等，变为follower再处理。
    - 请求的任期比自己的小，拒绝，也可以忽略。
3. 收到追加请求回复，大概率是自己前一刻为leader发送请求宕机了，恢复后超时，可以忽略了，自己知道也没用。
4. 收到投票请求，尝试投票（但是票没了，所以一定是拒绝）。
5. 收到投票请求回复，统计一下收到的票数，半数以上可以转到leader。

## go的代码
实现上，我看要求好像是要完成一个`Start`函数，注意这个函数是非阻塞的。

那么定时器逻辑需要进一步修改，整个结构体也要添加东西，定义日志结构。

定时器修改的话，加入**n-1**个*goroutine*用于计时，在不是leader状态时这些定时器阻塞，成为leader时重置定时器，并开始计时，每个节点均有**n-1**个这种定时器。
同时老定时器也要加入一个逻辑，如果当前节点是leader的话进行阻塞，不是leader时重置定时器，并开始计时。

同时确定*index*为可用的index



# MIT6.5840 raft 思考
raft中文翻译版本：[寻找一种易于理解的一致性算法（扩展版）](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)。
（之前对着英文版看了半天才发现有中文。。。。）

依据我一小勺的raft理解，如果真要实现的话，可能把每个节点抽象成状态机更好一点。
也就是每个节点只对接受的网络请求有反应，根据这个请求来更改自身的状态（比如存储，还有自己的角色转换，或者什么都不做只返回一个response）。

如果真要自己写出来，我个人更倾向于先捋清楚这个状态的转换。
所以在这里作一个可能不完全对的思考记录。

---
## A. 领导者选举
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
1. 收到追加请求，上一个leader遗留下来的，过期不管。
2. 收到追加请求的回复，肯定是发给自己的。
3. 收到投票请求：
    - 请求中 *term* 比自己**小**或者**跟自己一样**，说明是过期的东西，忽略。
    - 请求中 *term* 比自己**大**，说明当前状态下，有节点收不到自己的消息，但是自己收到。同样分情况，转到follower，继续处理请求。
4. 收到投票请求回复，自己在上N轮的投票请求的回复，忽略。

当节点处于candidate状态下有如下几种可能：
1. 定时器超时或者转到此状态，*term* 自增然后重置定时器。
1. 收到追加请求，大概率是自己超时后马上有请求过来，回复拒绝。
2. 收到追加请求回复，大概率是自己前一刻为leader发送请求宕机了，恢复后超时，可以忽略了，自己知道也没用。
3. 收到投票请求，尝试投票（但是票没了，所以一定是拒绝）。
4. 收到投票请求回复，统计一下收到的票数，半数以上可以转到leader。

### 真是 *not understandable* ，来实现。
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










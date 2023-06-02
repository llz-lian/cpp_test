follower:
1. 定时器超时==》进入candidate状态。
2. 收到`心跳append`。
    - 可以检查能提交哪个日志。
    - 可以触发一致性检查。
3. 收到`日志append`。
    - 
    - 
4. 收到`投票request`

leader:
1. 定时器==》发送`心跳append`。
2. 发送`日志append`。
3. 接受`append response`。
4. 收到`投票request`


candidate：
1. 收到更新leader的心跳时==》进入follower状态。
    - 每个follower一个定时器。
    - 一致性检查时暂定心跳发送与定时器更新。
2. 定时器超时，任期+1。
3. 进入candidate状态时立即给自己投票。
4. 收到`投票request`，返回`投票response`。

append RPC
```
struct append
{
    term int;
    prevIndex int;
    prevTerm int;
    entries;
    commitIndex;
};
struct append_response
{

};
```


无论什么情况，开始状态均为follower。
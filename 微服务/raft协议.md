- 数据一致的需求
  - 复制状态机：只要操作相同，操作的执行顺序相同，那么最终的结果也会相同。
    - 客户端所有的请求都送到leader，同过一个leader向其他follower发送操作日志，follower进行同步。
      - 出现的问题：leader挂了怎么办？——选举机制。
        - 如何参加选举？一段时间内没有收到leader或者candidate发送的包，说明此时没有leader也不在竞选态，因此可以自己成为candidate参与选举。
        - 一个选举周期为term，是任意长度的时间片，可以是leader挂了重新选举，也可以是超过一定时间重新选举，也可以是选举没有结果后重新选举。
        - 那么如何确定是那场选举的呢？如果选举混乱了怎么办？给term加一个序号。任一个节点收到了比当前term序号小的RPC，那么就忽略，因为已经过时了。
        - 每个节点每个term只有一票。
        - 每次选举，term的序号就加1。
        - 作为leader，刚选上和没过一段时间都要发送心跳包，确认自己是leader。
        - 任意节点收到leader 的心跳包且term 不小于自己当前的term，就自动变为follower（因为已经选举出leader了，candidate也不需要选举了）。
        - 如果收到的心跳包是小于当前的term，就忽略。
        - leader收到了一个比当前term序号高的包，那么就自动变为follower。
        - 如果一次选举没法获得大部分选票怎么办？——重选。
        - 重选又有冲突怎么办？——随机timeout，确保不同的candidate不会在短时间内同时发起竞选（类似于网络的数据链路层二进制避让策略）。
      - 同步机制——强制复制
        - 当大部分节点都完成复制了，leader进行提交。如果大部门节点没有完成复制，可能出现了网络中断的故障，如果这个时候强行提交，而另一个网络中也选出了一个新leader并也进行了提交，那么两个提交就会冲突。
        - 每次leader发送心跳包都会附带最新的提交序号（操作日志也有序）。
        - follower根据心跳包进行提交。
        - 有几个问题
          - leader崩溃了，但follower还没复制完，可能会造成几种情况：follower少了操作，follower多了操作，follower操作不一致，怎么办？——leader强制follower进行同步。将少的补上，错的全部重写。
          - follower崩溃了，没提交上，但是下回选成了leader，这样就少比其他follower少了操作日志。如果强制同步其他的所有follower都要回滚，怎么办？——在竞选的时候加以限制——如果candidate发送的拉票的包的最后一次提交比follower最后一次提交要旧，那么follower就不理这个包。这样可以保证选出的leader提交的操作要比大部分follower多。
      - 日志压缩——日志太多影响性能怎么办？
        - snapshotting——系统的全部状态会写入snapshot保存起来，然后丢弃截止到snapshot节点之气那的所有日志。
        - 每个server有自己的snapshot，但是当follower严重落后于leader时，leader将自己的snapshot发送给follower加快同步。







raft协议：将客户端的请求都作为日志写在一台同步服务器上，由同步服务器将日志分发给每个服务器，只要每个服务器都按顺序执行了日志的内容，那么他们就会达到一致性状态

有一个leader负责接受客户端的请求。将请求转为日志并复制给其他服务器
leader故障的时候必须选择一个新的leader
其他节点的日志应该与leader保持一致。
必须有措施使所有服务器都按相通的顺序执行日志

客户端的所有请求都会被重定向到leader
其他都是follower，会有candidate在leader故障的时候参与选举leader

raft协议将时间分成了任意长度的时间片，称为term，是有序号的。一个leader选举成功后在term剩下的时间都是leader，如果在规定时间内没有选举出新的leader 那么很快就会开启下一个term

编号用于同步。如果收到的请求的term序号小于当前的term序号，就忽略，如果一个leader收到比当前的term序号高的请求，那么就自动变为follower

leader一定时间就会发送心跳包，确认自己的leader地位。
follower收到leader或者candidate的选举请求，就保持自己为follower避免竞争
如果follower长时间没有收到leader或candidate的选举请求，就变为candidate发起选举。（term加1）

没法赢得大部分选票：设置一个随机timeout。candidate在timeout内没有成为leader也没有收到其他leader的心跳包，就会发起一轮新的选举。由于有随机timeout的存在，第二次也没有candidate获取大部分选票（选票冲突）的情况就会变少，跟网络数据链路层中二进制避让的原理差不多

当大部分节点都已经成功复制节点了，那么leader发出命令，follower都会将之前的日志执行。日志还有一个标志位，表明leader是否已经执行了这个命令

leader崩溃了，日志还没有复制完
leader强制follower复制自己的日志，有冲突要重写
如果follower收到的最后提交的日志序号与自身日志不匹配，会拒绝执行，并要求leader提供更早一次的提交。直到同步的位置。
如果follower在提交的时候宕机了，少了几条日志，然后又成了leader，会导致已经提交的命令被删除。raft在选举的时候做了一个限制，candidate发送的拉票请求包含的最新提交的日志要等于或新于follower的最新提交的日志，follower才会响应投票。这样确保了缺少了提交的candidate不成为leader

日志太多了影响性能
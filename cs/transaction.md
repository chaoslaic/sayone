# 简单说一下分布式事务

## 两阶段提交（2PC）

两阶段提交（Two-phase Commit，2PC），通过引入协调者（Coordinator）来协调参与者的行为，并最终决定这些参与者是否要真正执行事务。

- 准备阶段：协调者询问参与者事务是否执行成功，参与者发回事务执行结果。
- 提交阶段：如果事务在每个参与者上都执行成功，事务协调者发送通知让参与者提交事务；否则，协调者发送通知让参与者回滚事务。

存在的问题：

- 同步阻塞：所有事务参与者在等待其它参与者响应的时候都处于同步阻塞状态，无法进行其它操作。
- 单点问题：协调者在2PC中起到非常大的作用，发生故障将会造成很大影响。特别是在阶段二发生故障，所有参与者会一直等待状态，无法完成其它操作。
- 数据不一致：在阶段二，如果协调者只发送了部分Commit消息，此时网络发生异常，那么只有部分参与者接收到Commit消息，也就是说只有部分参与者提交了事务，使得系统数据不一致。
- 太过保守：任意一个节点失败就会导致整个事务失败，没有完善的容错机制。

## 补偿事务（TCC）

针对每个操作，都要注册一个与其对应的确认和补偿（撤销）操作。

- Try：对业务系统做检测及资源预留。
- Confirm：对业务系统做确认提交，Try阶段执行成功并开始执行Confirm阶段时，默认Confirm阶段是不会出错的。
- Cancel：在业务执行错误，需要回滚的状态下执行的业务取消，预留资源释放。

举个例子，假如Bob要向Smith转账：首先在Try阶段，要先调用远程接口把Bob和Smith的钱给冻结起来。在Confirm阶段，执行远程调用的转账的操作，转账成功进行解冻。如果第二步执行成功，那么转账成功，如果第二步执行失败，则调用远程冻结接口对应的解冻方法（Cancel）。

## 最终一致性（RocketMQ事务消息）

- 应用模块遇到要发送事务消息的场景时，先发送prepare消息给MQ。
- prepare消息发送成功后，回调应用模块执行数据库事务（本地事务）。
- 根据数据库事务执行的结果，再返回Commit或Rollback给MQ。
- 如果是Commit，MQ把消息下发给Consumer端，如果是Rollback，直接删掉prepare消息。
- 第3步的执行结果如果没响应，或是超时的，启动定时任务回查事务状态（最多重试15次，超过了默认丢弃此消息），处理结果同第4步。
- MQ消费的成功机制由MQ自己保证。
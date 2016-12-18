#### PBFT 算法

Practical Byzantine Fault Tolerance

##### 1. synchronous环境==》what？

  asynchronous 环境（真实的互联网环境）：消息丢失、消息延迟、消息重复传播、消息乱序

a). 使用 Byzantine 错误模型，即恶意节点可以表现出任意行为，与此同时，要受到以下限制：独立的节点错误，为了实现这一点，我们可以在每一个节点上运行该服务代码的不同实现版本，操作系统应该有不同的 root 用户密码，和不同的管理员

b). 使用消息加密的方式来防止消息的仿造、消息的重放、检测恶意的消息，我们的消息包括：公钥签名，消息鉴证码，消息摘要（使用抵抗冲突的哈希函数生成）。D(m) 代表消息 m 的摘要，<m> 代表被签过名的消息 m，我们对消息 m 的数字摘要进行签名，将签名追加在消息 m 的后面，而不是对整个消息进行签名，所有的节点都应该知道其它节点的公钥，以便来验证签名
我们允许非常强的对手，对手可以联合其它恶意节点，延迟传播善意节点的消息来对该服务造成巨大的损害，我们假定该对手无法无限期的延迟善意节点消息。我们还假定，该对手的计算能力是受限的，这样可以保证它不能颠覆以上提到的加密技术，例如，对手不能产生一个属于善意节点的有效的签名，不能从消息摘要中计算出消息，即不能找到拥有相同数字签名的两个不同消息

##### 2. 算法简单工作如下：
a). 一个客户端发送一个请求来唤醒对 primary 节点的服务操作（service operations）

b). primary 节点多播此请求到 backups 节点

c). 节点（primary 和 backup）执行此请求，发送回复给客户端

d). 客户端等待至来自不同节点的 f+1 条回复（回复带有相同的结果），结果是该请求操作的结果。

我们在节点上强制有两个必要条件：1. 它们必须从同一个状态开始 2. 它们必须是可决定的（即在给定的状态，给定的参数，同一个操作的执行结果总是产生相同的结果），有了这两个给定的必要条件，此算法通过确保所有的非恶意节点在请求的执行上协调在一个最终的顺序来保证了安全性

##### 3. 算法执行细节

**3.1 客户端**

client c 向 primary 服务节点发送 <REQUEST，o，t，c>，o 代表客户端要执行的操作命令，t 代表 Timestamp，保证服务端节点只执行一次该请求，可以用客户端当前的时间戳来赋值，保证后一个请求的 t大于上一个请求的 t，c 标识该客户端。

服务端节点 i 发送回复消息 <REPLY，v，t，c，i，r>，v 代表当前 view number，t 代表对应客户端请求中的 t，c 代表客户端，i 代表该服务节点，r 代表客户端请求中 o 的执行结果。

客户端等待至收到来自不同服务节点的合法签名的 f+1个 回复，这 f+1 个回复需有相同的 t 和 r，然后客户端才接受该结果 r，这就保证了结果是合法有效的，因为我们假定至多有 f 个服务端节点是恶意的

如果客户端在一段时间内没有收到足够的回复，它就会广播该请求到所有的服务节点，在服务节点这边，如果收到的请求已经被处理过了，那么该服务节点简单的再次发送回复即可，每个服务节点都记录了它们回复的最后一个 reply message。否则，如果该节点不是 primary，它就把该请求传送给 primary 节点。如果 primary 节点没有多播该请求到组，它将会被怀疑是恶意的，如果被足够多的节点怀疑是恶意的，那么会导致 view-change。

**3.2 正常情况下操作**

每个服务节点的 state 包括服务的 state，一个消息日志（记录了已经收到的各种消息），一个整数（代表当前 view 的编号），我们接下来会介绍如何舍弃日志中的内容。

当一个 primary 节点 p 收到一个客户端请求 m，它就开始三段协议的执行过程，首先，多播该请求到所有的其它服务节点。当 primary 节点正在执行协议过程中的消息数量大于一个给定的数值时，primary 节点再接收到新的请求时，不会立马开始新的执行过程，它会把新收到的消息缓冲起来，稍后，缓冲区内的消息会作为一个 group 多播出去，有效的减小了 CPU 的负荷和消息的拥堵。

三段提交协议：pre-prepare，prepare，commit。pre-prepare和 prepare 阶段被用来给在同一个 view 内发送的请求排序，即使提出请求顺序的 primary 阶段是恶意的；prepare 和 commit 阶段被用来保证跨 view 提交的请求的最终顺序。

在 pre-prepare 阶段，primary 给请求分配一个序列号 n，多播一个携带有 m 的 pre-prepare message 给所有的 backup 节点，然后将该消息插入到日志中，该消息有以下组成方式 <<PRE_PREPARE，v，n，d>（加密），m>，m 代表客户端的请求消息，d 是 m 的消息摘要。

backup 节点接收 pre-prepare 消息代表：

a). pre-prepare 消息的签名是正确的，m 消息的签名也是正确的，d 是 m 的消息摘要；

b). 该节点位于的 view 编号是 v；

c). 对于 view v 来说，它还没有接受过 pre-prepare 消息（具有序列号为 n 却有着不同摘要）；

d). pre-prepare 消息中的序列号 n 在 h 和 H 之间。

如果 backup 节点 i 接收了 <<PRE_PREPARE，v，n，d>（加密），m> 消息，它就会进入到 prepare 阶段，通过多播 <PREPARE，v，n，d，i>（加密）消息给其它的服务节点，然后将 pre-prepare 和 prepare 消息追加到日志中，如果不接收 <<PRE_PREPARE，v，n，d>（加密），m> 消息，则不做任何事情

服务节点（包括 primary ）接收 prepare 消息，然后将该消息追加到日志。接收 prepare 消息的前提是它们的签名是正确的，它们的 view 编号跟该服务节点当前的 view 编号是一致的，而且序列号位于 h 和 H 之间

我们定义 **prepared(m，v，n，i)** 为真，当且仅当节点 i 把以下内容插进了日志中：请求 m，在 view 编号为 v 中的对于 m 的 pre-prepare 消息（序列号为 n），2f 个匹配 pre-prepare 消息的来自不同 backup 节点的 prepare 消息，节点检查 prepare 消息是否匹配 pre-prepare 消息的方法是确认它们是否有相同的 view 编号，序列号，数字摘要。

pre-prepare 和 prepare 阶段保证了善意节点在同一个 view 中对请求的一个最终顺序达成一致。更详细来说，它们保证以下不变量：如果 **prepared（m，v，n，i）**为真，那么，**prepared（m’，v，n，j）**是错误的，对于任何善意节点 j 来说（包括 i==j）和任何 m’（D（m‘） ！= D（m））。原因如下：由 prepared（m，v，n，i）为真和 |R| = 3f+1 暗示着至少 f+1 个善意的节点对于消息 m在 view 编号 V 中已经发送了一个序列号为 n 的 pre-prepare 或者是 prepare 消息（如果 primary 节点是善意的，善意的 backup 节点根本不会发出冲突的 prepare 消息，因此不可能同时成立；假设 primary 节点是恶意的，那么意味着在 backup 节点中至多有 f-1 个恶意的节点，prepared（m，v，n，i）为真，则证明有 f+1 个善意节点达成了一致，prepared（m’，v，n，j）为真 ，意味着另外 f+1 个善意节点达成了一致，因为系统中只有 2f+1 个善意节点，因此最少有一个善意节点发送了两个冲突的 prepare 消息，是不可能的）。
当 prepared（m，v，n，i）为真，服务节点 i 广播 <COMMIT, v, n, D(m), i>（加密）给其它的服务节点，服务节点会接收 commit 消息，如果该 commit 消息中的 v 等于该节点当前位于的 view，且 n 位于 h 与 H 之间，符合签名的验证。然后将 commit 消息写入到本地 log 中。

我们定义了 **commited 和 commited-local**：commited（m，v，n）为真当且仅当对于某个 f+1 个的善意节点集合中的每一个节点 i，prepared（m，v，n，i）为真；commited-local（m，v，n，i）为真当且仅当 prepared（m，v，n，i）为真，并且已经接收到 2f+1 个匹配对于 m 来说的 pre-prepare 消息的 commits 消息(可能包含自己)。

commit 阶段保证以下不变量：如果 committed-local（m，v，n，i）为真，i 为善意节点，那么 committed（m，v，n）就为真。这个不变量和 view-change 协议保证了善意的节点们对本地提交的（即使是在不同的 view 里提交的）请求的序列编号达成一致。还有，它确保了一个善意节点本地提交的任何请求最终会在至少 f+1 个善意节点那里提交。

当 committed-local（m，v，n，i）为真，每一个节点 i 就会执行请求 m 要求的操作，节点 i 的状态反映了具有低于序列号 n 的所有请求的序列化执行结果，这就确保了所有善意的节点以相同的顺序执行请求来提供了 safety。执行完请求所要求的操作后，节点会发送 reply 给客户端，节点会丢弃时间戳比它最后一次对客户端的回复中请求的时间戳小的请求来确保恰好执行一次的语义。

我们不依赖于有序的消息传送，因此对于节点来说，commit request 乱序是有可能发生的，但是这个没关系，既然节点在相关的请求被执行之前，保持了 pre-prepare、prepare、commit 消息在本地日志中。

**3.3 Garbage Collection**

该节讨论了从日志中删除消息的机制。为了保证安全性，消息会一直被存在本地日志中，直到该节点关心的该请求已经被至少 f+1 个善意的节点执行，并且在 view-change 中能向其它节点证明这一点。此外，如果，一些节点错过了被其它善意节点丢失的消息，它需要通过传输所有的或一部分节点状态来更新。因此，节点需要一些证据来证明某一点处的状态是正确的。

如果在每次请求执行完成之后就生成证据，那么代价是及其昂贵的，因此，这些证据是阶段性的生成的，当一个请求的序列号被一些常数整除的时候（例如100），该请求执行完即可以生成证据。我们把这些请求执行完后的节点状态称为 **checkpoint**，把具有证据的 checkpoint 称为 **stable checkpoint**。

一个节点包含多个服务状态的逻辑性拷贝：最后一个 stable checkpoint，0 或多个不稳定的 checkpoint，还有当前的节点状态。写时拷贝技术可以被用来减轻存储这些额外状态拷贝带来空间上的负载。

checkpoint 正确性的证据按以下步骤生成：当节点 i 产生一个 checkpoint 的时候，它会多播消息 <CHECKPOINT，n，d，i>（加密）给其它所有的节点，n 是最后一个被执行的请求的序列号（该请求指的是生成 checkpoint 时最后一个请求），d 是该状态的一个数字摘要。每一个节点接收 checkpoint 消息放入它的本地日志中，直到接收到 2f+1 个具有相同数字摘要和序列号的来自不同的节点的 checkpoint 消息。这 2f+1 个消息即是该 checkpoint 准确性的证据。

具有证据的 checkpoint 变成 stable 状态，然后节点从日志中丢弃所有序列号小于等于 n 的 pre-prepare，prepare，commit 消息，同样也丢弃所有的更早的 checkpoint 和 checkpoint message。

计算这些证据的效率是很高的，因为数字签名可以使用 incremental cryptography，并且这些证据很少被生成。

checkpoint 协议被用来提高low和high water marks（限制了被接受的消息）。low-water mark h 等于最后一个稳定的 checkpoint 的序列号，high-water mark H = h+k，k 是足够大的以至于节点等待一个 checkpoint 变为稳定的期间不会阻塞住，例如，如果 checkpoint 每 100 个请求被生成，那么 k 可能为 200。

**3.4 View Changes**

**view-change** 协议通过允许系统当 primary 节点失败的时候继续保持运转来提供 liveness。View Changes 被定时器的超时所触发，定时器防止 backup 节点无限期的等待请求的执行。backup 正在等待一个请求如果它接收到一个有效的请求并且还没有执行它。当 backup 节点接收到一个请求并且定时器没有正在运行的时候，它开始一个定时器，当它不再等待请求的执行时，停止该定时器；但是会重启定时器，如果在那个时候（猜测上一个正在等到执行的请求执行完成之后），该节点正在等待执行其它的请求。

如果在 view v，backup 节点 i 的定时器超时，那么该 backup 开始一个 view change 来使系统移向下一个 view v+1。他停止接收消息（除了 checkpoint，view-change，new-view 消息之外），并且多播 <VIEW_CHANGE，v+1，n，C，P，i>（加密）消息给所有的节点，n 是 backup i 节点知道的最后一个稳定的 checkpoint s 的序列号，C 是 2f+1个 有效的证明 s 的正确性的 checkpoint message 的集合；P 也是一个集合，该集合中的每一个元素也是一个集合 Pm，Pm 对应在 backup i 节点处已经 prepared 的序列号大于 n的请求。Pm 包括一个有效的 pre-prepare 消息（不带有相应的客户端消息）和 2f 个来自不同 backup 签名的具有相同 view，相同序列号，相同数字摘要的匹配的有效的 prepare 消息。

当属于 view v+1 的 primary 节点收到 2f 个有效的 view 为 v+1 的 view-change 消息的时候，它就会多播一个 <NEW-VIEW，v+1，V，O>（加密）给所有其他的节点，V 代表一个集合，它包括 primary 节点接收到的有效的 view-change 的消息，加上 primary 节点发送的对于 view v+1 的 view-change 消息。O 是 pre-prepare 消息（无请求附加）的集合，O 按以下方式产生：

- primary 节点决定 V 中的 stable checkpoint 的序列号 min-s，和在 V 中的 prepare 消息中的最大序列号 max-s；
- primary 节点在 view v+1 中对于位于 min-s 和 max-s 之间的任何一个 n 产生一个新的 pre-prepare 消息。现在有以下两种情况：（1）在 P 中至少存在一个集合 Pm，它的序列号是 n；（2）不存在这样的集合。在第一种情况中，primary 节点产生一个新的消息 <PRE-PREPARE，v+1，n，d>（加密），d 是 V 里具有序列号为 n 的 pre-prepare 消息中的 request 摘要。第二种情况中，它产生一个新的 pre-prepare 消息 <PRE-PREPARE，v+1，n，dnull>（加密），dnull 是一种特殊的 null request 消息的摘要，该消息像普通消息一样会进行各个阶段的协议执行过程，但是它的执行结果是一个 no-op 操作。

接下来，primary 节点将 O 里面的消息追加到它的日志中，如果 min-s 比它的最后一个 stable checkpoint 的序列号大，那么 primary 节点也会将 min-s 的 checkpoint 的证据插入到它的日志中，然后从日志中丢弃消息，然后它进入 view v+1：在那个时候，它就可以接收来自 v+1 的消息了。

backup 节点接收具有 view v+1 的 new-view 的消息，如果该消息被合适签名。如果它包含的 view-change 消息对于 view v+1 是有效的，并且如果集合 O 是正确的，它确认集合 O 的正确性，通过上述产生 O 的过程。然后，它按照上述 primary 的方式将新信息存入到它的日志中，然后对于 O 里面的每一条消息，发送一条相应的 prepare 消息给其它所有的节点，并且将这些 prepares 加入到本地 log 中，进入 view v+1。

此后，协议继续进行。节点针对位于 min-s 和 max-s 之间的消息进行协议执行过程，但是它们避免再次执行客户端请求，通过它们自己存储的最后一条回复给客户端的消息。

一个节点可能会丢失一些请求消息 m 或者是一个稳定的 checkpoint（既然在 new-view 消息中这些没被发送），它可以从其它的节点处获得丢失的信息。例如，节点i能够获得一个丢失的 checkpoint 状态 s 从某个节点处（它的 checkpoint 消息的正确性在 V 中得到验证）。既然这些节点中 f+1 个是正确的，节点 i 将总能够获得 s 或者一个随后的经过验证的稳定 checkpoint。我们可以避免发送整个的 checkpoint，通过将状态分离成块，把每一块的状态与改变它的请求中的最后一个的序列号映射。为了将一个节点更新到最新状态，只需发送它落后处之后的分区消息，而不是整个 checkpoint。

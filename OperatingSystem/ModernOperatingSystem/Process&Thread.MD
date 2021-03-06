# 操作系统

## 操作系统概念

### 进程

进程本质上是**正在的执行的一个程序**，与每个进程相关的是**地址空间**，这是从某个最小值的**存储位置**通常是0到某个最大值**存储位置**的**列表**。在这个地址空间中进程可以读写，该地址空间存放有**可执行程序、程序的数据以及程序的堆栈**。与每个进程相关的还有资源集，通常包括寄存器、打开的文件清单、突出的报警、有关进程清单等。**进程基本上是容纳一个程序所需要所有信息的容器**。

用户启动一个视频编辑程序，指示它按照某个格式转换一个小时的视频，然后离开去浏览网页。同时一个被周期性唤醒、用来检查进来的电子邮件的后台程序开始运行。这样就有了三个活动进程：视频编辑器、检查电子邮件的后台进程、web浏览器。**操作系统周期性的挂起一个进程然后启动运行另一个进程，这可能是由于在过去的一两秒钟内，第一个进程已经使用完分配给它的时间片。**

一个进程暂时被挂起后，在随后的某个时刻里，该进程再次启动时的状态必须与先前暂停时完全相同，这就意味着在挂起时该进程的所有信息都要保存下来。例如与打开文件相关的指向当前位置的指针。**许多操作系统中，与一个进程相关的所有信息，除了该进程自身地址空间的内容以外，均放在操作系统的一张表中，称为进程表，进程表时数组或链表结构，当前存在的每个进程都要占用其中一项。**

一个挂起的进程包括：进程的地址空间、对应的进程表项（包括**寄存器**以及稍后重启动该进程所需要的许多**其他信息**）

与进程管理有关的最关键的系统调用是那些进行**进程创建**和**进程终止**的系统调用。考虑这种情况，一个命令解释器活shell从终端上读命令。用户输入一个命令要求编译一个程序，shell必须先创建一个新进程来执行编译程序。当执行编译的进程结束后，它执行一个系统调用终止自己。

如果进程可以创建一个或多个进程（子进程），这些进程又可以创建子进程，则可以得到进程树。

其它可用的进程系统调用包括：申请更多的内存、释放内存、**等待一个子进程结束**、用另一个程序覆盖程序

想象tcp的重传机制在两个进程上实现，为了保证一条消息的应答不会丢失，发送者要求它所在的操作系统在指定的若干秒后给一个通知，如果尚未收到确认消息就可以重发。

经过一定的时间，操作系统向该进程发送一个**警告信号(alarm signl)**，此信号让该进程暂时挂起，将寄存器的值保存到堆栈，并开始重新发送丢失的信息

系统管理器授权每个进程使用一个给定的**UID（User IDentification）**。**子进程拥有父进程一样的UID**。用户可以是每个组的成员，每个组也有一个GID

### 地址空间

较复杂的操作系统中允许在内存中同时运行多道程序。为了避免他们互相干扰，需要有某种保护机制。虽然这种机制必然是硬件形式的，但是由操作系统掌控。

虚拟内存技术。**操作系统可以把部分地址空间装入主存，部分留在磁盘上，并且在需要时来回交换它们**。本质上操作系统创建了一个**地址空间的抽象**，作为进程可以引用地址的集合。改地址空间与机器的物理内存解耦，可能大于也可能小于该物理空间。

### 文件

在读写文件前，首先要打开文件，检查其访问权限。若权限许可，系统将返回一个小整数，称作**文件描述符**，供后续操作使用。

在UNIX中另一个重要概念是**特殊文件**，提供特殊文件是为了使I/O设备看起来像是文件一般，I/O设备一个可通过系统调用进行读写。有两类特殊文件：**块特殊文件和字符特殊文件**。块特殊文件指那些可由随机存取的块组成的设备如磁盘等。字符特殊文件用于打印机、调制解调器和其他接受或输出字符流的设备。

**管道**是一种虚文件，它可连接两个进程，如果进程A和B希望通过管道对话，他们必须提前设置该管道。

### 系统调用

记住下列事项是有益的。任何单CPU计算机一次只能执行一条命令。如果一个进程正在用户态与运行一个用户程序，兵器需要一个系统调用，比如从文件读数据，那么它必须执行一个系统调用将控制转移到操作系统，操作系统根据参数查找所需要的调用进程。然后执行系统调用，并把控制返回给在系统调用后面跟随的指令，在某种意义上系统调用就像进行一个特殊的过程调用，但是只有系统调用可以进入内核，而过程调用则不能。

read系统调用：

```c
count = read(fd, buffer, nbytes)
```

count中实际返回读出的字节数。这个值和nbytes相同或更小。

![image-20200705115819705](image-20200705115819705.png)

- 1-3：在准备调用这个实际用来进行read系统调用的库过程中，调用程序先把参数压入堆栈。第一个和第三个参数是值调用，第二个参数通过引用传递实际传递的是地址。

- 4：对库过程的调用

- 5：在可能是由汇编语言写成的库过程中，一般吧系统调用的编号放在操作系统所期望的地方比如寄存器中

- 6：执行一个TRAP指令把用户态切换到内核态，并在内核中的一个固定地址执行

  >TRAP指令实际上与过程调用非常类似，他们后面都跟随一个来自远处位置的指令，已经共以后使用的一个保存在栈中的返回地址，即调用前的用户空间的地址。并且TRAP指令不能跳转到任意地址上，根据机器的体系结构，或者跳转到一个单固定地址上。 

- 7：跟随在TRAP指令后的内核代码开始检查系统调用编号，然后分派给正确的系统处理器，这通常通过一张由系统调用编号所引用的、指向系统调用处理器的指针表来完成

- 8：系统调用处理器运行

- 9：一旦系统调用处理器完成其工作，控制可能会跟随TRAP指令后面的指令中返回给用户空间进程，系统调用者可能堵塞调用者，避免它继续执行。例如如果试图读键盘，但是并没有任何键入，那么调用者就必须被堵塞。这种情况下，操作系统会查看是否有其他可以与运行的进程。稍后需要的输入出现时，进程会提醒系统注意，然后步骤9-11会接着进行

- 10：这个过程接着以通常的过程调用返回的方式，返回到用户程序

- 11：为了完成整个工作，用户程序还必须清除堆栈，如同它在进行任何过程调用后一样

![image-20200705151056364](image-20200705151056364.png)

![image-20200705151113854](image-20200705151113854.png)

#### 用于进程管理的系统调用

##### FORK

**fork**是唯一可以在POSIX中创建进程的途径，它创建一个原有进程的精确副本，包括所有的文件描述符、寄存器等内容。**fork之后原有的进程及其副本就分开了**。fork调用返回一个值，在子进程中该值为0，在父进程中等于子进程的**进程标识符**。

考虑shell的情形，从终端读取命令创建一个子进程，等待子进程执行命令，为了等待子进程结束父进程执行**waitpid**系统调用，等待直到子进程终结。在键入一条命令后，shell调用fork创建一个新的进程，通过使用**execve**系统调用可以实现这一点，这个系统调用会**引起整个核心映像被一个文件替代**，该文件由第一个参数给定。

![image-20200711154626610](image-20200711154626610.png)

waitpid第一个参数-1代表等待任何一个老的子进程，第二个参数为statloc所指向的地址是子进程的退出状态

在UNIX中进程将其存储空间划分为三段：正文段(如程序代码)、数据段(如变量)以及堆栈。数据向上增长而堆栈向下增长，夹在中间的是未使用的地址段：

![image-20200711155123056](image-20200711155123056.png)

### 思考问题

1、内核态和用户态有哪些区别？解释在设计操作系统时存在两种不同模式有什么帮助

2、考虑一个有两个CPU的系统，并且每一个CPU有两个线程。假设有三个程序P0、P1、P2，分别以运行5ms、10ms、20ms开始。运行这些程序需要多少时间？假设这三个程序都是100%限于CPU，在运行时无阻塞，并且一旦设定就不改变CPU。

3、在用户进程进行一个系统调用，以读写磁盘文件时，该程序提供指示说明了所需要的文件、要给指向数据缓冲区的指针以及计数。然后控制权交给操作系统，它调用相关的驱动程序。假设驱动程序启动磁盘并且直到中断发生才终止，在磁盘读的情况下，很明显，调用者会被阻塞，因为文件中没用数据。宰相磁盘写时会发生什么情况

4、什么是陷阱指令？在操作系统中解释它的用途

5、在分时系统中为什么需要进程表？

6、对于以下系统调用给出引起失败的条件：fork、exec以及unlink

7、一台计算机有一个四级流水线，每一级都花费相同的时间执行其工作即1ns。这台机器每秒可执行多少指令

8、假设一个计算机系统有高速缓存、内存（RAM）以及磁盘，操作系统用虚拟内存。读取缓存中的一个词需要1ns，RMA需要10us，磁盘需要10ms。如果缓存的命中率为95%，内存是99%（缓存失效时），读取一个词的平均时间是多少？

9、块特殊文件和字符特殊文件的差异是什么？

10、现代操作系统将进程的地址空间从机器物理内存中分离出来。列举这样设计的两个好处

## 进程与线程

进程时操作系统提供的最古老的也是最重要的抽象概念之一。即使使用的CPU只有一个也有支持伪并发的能力，它们将一个单独的CPU变换成多个虚拟的CPU。没有进程的抽象现代计算将不复存在

### 进程

#### 进程模型

在进程模型中计算机中所有可运行的软件，通常也包括操作系统被组织成若干**顺序进程**，简称进程。一个进程就是一个正在执行程序的实例，包括程序计数器、寄存器和变量的当前值。从概念上说，每个进程拥有它自己的虚拟CPU，实际的CPU在进程之间来回切换，这种快速的切换称作**多道程序设计**。

![image-20200711161510679](image-20200711161510679.png)

进程和程序之间的区别是很微妙的，但非常重要，用一个比喻可以理解这一点：

> 想象一位计算机科学家为他的女儿制作生日蛋糕，他有做蛋糕的食谱，厨房里有所需要的原料。在这个比喻中，食谱就是程序，计算机科学家是CPU，而原料就是输入数据。进程就是厨师阅读食谱，取来各种原料以及烘焙蛋糕的一系列动作的总和。现在假设计算机科学家的儿子被蜜蜂蛰了，计算机科学家记录下他照着食谱做到哪了然后拿出一本急救手册按照其中的指示处理蛰伤，处理完成后他又回来从离开时的那一步开始做蛋糕。

这里最关键的思想是：**一个进程是某种类型的一个活动，它有程序、输入、输出以及状态。单个处理器可以被若干进程共享，它使用某种调度算法决定何时停止一个进程的工作，转而为另一个进程服务**

值得注意的是如果一个程序运行了两边，则算作两个进程

#### 进程的创建

4种事件会导致进程的创建：

1、系统初始化

2、正在运行的程序执行了创建进程的系统调用

3、用户请求创建一个新进程

4、一个批处理作业的初始化

停留在后台处理诸如电子邮件、Web页面、新闻、打印之类的活动的进程称为**守护进程**

一个正在运行的进程经常发出系统调用，以便创建一个或多个进程协助其工作。在所要从事的工作可以容易的划分成若干**相关的但是没有相互作用的进程时**，创建新的进程就特别有效果。

fork是唯一一个可以创建新进程的系统调用,这个系统调用会创建一个**与调用进程相同的副本**。在调用了fork之后，这两个进程拥有相同的**内存映像、同样的环境字符串和同样的打开文件**。通常子进程接着执行execve或一个类似的系统调用以修改其内存映像并运行一个新的程序。

进程创建之后，父进程和子进程有各自**不同的地址空间**。如果其中某个进程在其地址空间中修改了一个字，这个修改对其它进程而言是不可见的。

#### 进程的终止

进程种终止的事件

1、正常退出，自愿的

2、出错退出，自愿的

3、严重错误，非自愿的

4、被其他进程杀死，非自愿的

第三个原因主要是一些执行非法指令、引用不存在的内存等错误

#### 进程的层次结构

在UNIX中，进程和它的所有子进程及其后裔共同组成了一个**进程组**。当用户从键盘发出一个信号时，该信号被送给当前与键盘相关的进程组中的所有成员，每个进程可以分别捕获该信号，忽略该信号或者采用默认动作，即该信号被杀死。

#### 进程的状态

进程的三种状态：

1、运行态，该时刻进程实际占用CPU

2、就绪态，可运行，但因为其他进程正在运行而暂时停止

3、阻塞态，除非某种外部事件发生，否则进程不能运行

![image-20200713230121363](image-20200713230121363.png)

转换操作系统发现进程不能运行下去的时候发生转换1，某些系统中进程可以执行一个系统调用进入阻塞态，当一个进程从管道或设备文件读取数据时，如果没有有效的输入，进程会被自动阻塞

转换2和转换3是由进程调度程序引起的，进程调度程序是操作系统的一部分，它的主要工作就是决定应当运行哪个进程、何时运行以及它应该运行多长时间，这是很重要的一点

使用进程模型是我们易于想象系统内部的操作状况

![image-20200713232101730](image-20200713232101730.png)

关于中断处理、启动进程、停止进程的具体细节都隐藏在调度程序中。

#### 进程的实现

为了实现进程模型，操作系统维护着一张表格，即进程表，每个进程占用一个表项，该表项包含了进程状态的重要信息，包括程序计数器、堆栈指针、内存分配状况、所打开的文件的状态、账号和调度信息以及其它在进程由运行态转换到就绪态或阻塞态时必须保存的信息，从而保证该进程随后能再次启动，就像从未被中断过一样。

![image-20200713232501784](image-20200713232501784.png)

单个cpu如何维持多个顺序进程的，与每一类I/O类关联的是要给称作中断向量的位置，它包含中断服务程序入口地址。假设当一个磁盘中断发生时，用户进程3正在运行，中断硬件将程序计数器、程序状态字、有时还有一个或多个寄存器压入堆栈，计算机随即跳转到中断向量所指示的地址然后就是软件，中断服务例程接管剩下的工作了。

![image-20200713232845940](image-20200713232845940.png)

#### 多道程序计算模型

如果进程用于计算的平均时间是进程在内存中停留时间的20%，即计算的概率为1/5，且内存中同时有5个进程，则CPU一直满负载运行，这个模型的前提时5个进程不会同时等待I/O

从概率的角度来看CPU利用率，假设一个进程等待I/O操作的时间与其停留在内存中的时间的比为P，当内存中有n个进程且都在等待I/O的概率即CPU空转的概率为pn。cpu利用率为1-pn。

![image-20200713233416462](image-20200713233416462.png)

更精确的模型应该用排队论构建，但上述的模型依然是有效的（当进程就绪时，给进程分配CPU，否则让CPU空转）。

假设计算机有8GB内存，操作系统及相关表格占2GB，每个用户程序占2GB，这些内存孕育3个用户程序同时驻留在内存中，若80%的时间用于I/O等待，则CPU的利用率为1-0.8^3，即49%。增加8GB的内存后，从三道程序设计提高到7道程序设计，cpu利用率提高到79%，换言之第二个8GB提高了30%的吞吐。主要是增加了n的大小
### 线程

在传统操作系统中，每个进程有一个地址空间和一个控制线程。

#### 线程的使用

线程存在的意义是什么？为什么一个进程中还要再有一类进程（线程），线程存在的必要性是什么。

通过将应用程序分解成可以准并行的多个顺序线程，程序设计模型会变得更简单。

- 有了进程模型的抽象，我们可以不必考虑中断，定时器，和上下文切换，而只需考察并行进程。类似的只有在有了多线程概念之后我们才加入了一种新的元素：并行实体拥有了共享同一个地址空间和所有可用数据的能力，而多进程无法共享一个地址空间和数据，这正是多进程模型不具备的。

- 另一个原因是因为线程比进程更轻量级，他比进程更容易创建也更容易撤销。

- 第三个原因涉及性能方面的讨论。若多个线程都是CPU密集型的，那么并不能获得性能上的增强，但是如果存在大量的计算（大量计算的任务单线程和多线程有区别吗？如果多个线程可以运行在不同核上是有区别的）和大量I/O处理拥有多个线程允许这些活动彼此重叠进行，从而加快应用程序的执行速度，在多CPU系统中多线程是有益的

举个例子，一个字处理软件，一个线程与用户交互，另一个在后台进行格式处理，还有一个进行持久化存储，一旦有文字变化交互线程就通知格式化线程对整本书进行处理，同时交互线程继续监控键盘和鼠标。

如果程序是单线程的，那么在备份时来自键盘和鼠标的命令就会被忽略，直到备份工作完成，另一个方法是引入中断驱动模型设计。但是如果使用三个线程，程序设计模型就会变得很简单，很显然三个进程无法做到这件事情，因为三个线程都需要对同一个文件进行操作，多个线程可以共享内存，所以他们可以访问同一个正在编辑的软件。

![image-20200718101740226](image-20200718101740226.png)

##### 多线程web服务器

在多数web站点上，某些页面较其它页面相比有更多的访问。利用这一事实web服务器可以把获得大量访问的页面集合保存在内存中，避免到磁盘去调用这些页面进而改变性能，这样一个页面集合称为高速缓存。一种组织web服务器的方式如图，一个称为分派程序的线程从网络中读入工作请求，分派线程挑选一个空转的工作线程提交该进程，通常是在每个线程所配有的某个专门字中写入一个消息指针。接着分派线程唤醒睡眠的工作线程，将它从阻塞态转为就绪态。

这种模型允许把服务器编写为一个顺序线程的集合。在**分派线程**的程序中包含一个无限循环，该循环用来获得工作请求并且把工作请求派发给**工作线程**。每个工作线程的代码包含一个从分派线程接受请求并且检查Web高速缓存中是否存在所需页面的无限循环。如果存在就将该页面返回客户机，接着该工作线程阻塞，等待一个新的请求，如果没有就从磁盘调入该页面，将该页面返回给客户机，然后该工作线程阻塞等待一个新的请求。

考虑如果没有多线程如何编写web服务器，一种可能的方式，使其像一个线程一样运行。web服务器的主循环获取请求，检查请求，并且在取下一个请求之前完成整个工作，在**等待磁盘操作时服务器就空转**，并且不处理任何到来的请求，结果导致**每秒只有很少的请求被处理**。

第三种情况，如果有可以使用的read系统调用的非阻塞版本，还存在第三种可能的设计。在请求到来时，这个唯一的线程对请求进行考察，如果该请求不能在高速缓存中得到满足那么启动一个非阻塞的磁盘操作。服务器在表格中记录当前请求的状态，然后去处理下一个事件。下一个事件可能是一个新的工作请求或者是磁盘对先前操作的应答。如果是新的工作请求就开始工作，**如果是磁盘的应答就从表格中取出对应信息并处理**，对于非阻塞I/O这种回答多数会以**信号或中断**的方式出现。

每次服务器从为某个请求工作的状态切换到另一个状态时都必须**显式的保存或重新装入相应的计算状态**，事实上我们以一种困难的方式模拟了**线程及其堆栈**。这里每个计算都有一个被保存的状态，存在一个会发生且使得相关状态发生改变的事件集合，我们把这类设计称为**有限状态机**。

多线程使得**顺序进程**的思想得以保留下来，这种顺序进程**阻塞了系统调用，但仍旧实现了并行性。**

![image-20200718104608429](image-20200718104608429.png)

#### 经典的线程模型

进程模型基于两种独立的概念：**资源分组处理与执行**。有时将这两种概念分开会更好，这就引入了线程的概念。

**理解进程的一个角度是，用某种方法把相关的资源集中在一起**。进程有存放程序正文和数据以及其他资源的地址空间。这些资源包括打开的文件，子进程，即将发生的定时器，信号处理程序，账号信息等。把他们放到进程中更容易管理。

另一个概念是进程拥有一个执行的线程，简写为线程，线程有程序计数器，记录接下来要执行哪条命令。线程拥有寄存器，保存线程当前的工作变量。线程还有堆**栈记录执行历史，其中每一帧保存了一个已调用但还没有返回的过程**。**进程用于把资源集中到一起，而线程则是在CPU上被调度执行的实体。**

在同一个进程中并行运行多个线程是对同一台计算机上并行运行多个进程的模拟。前一种情况下多个线程共享同一个地址空间和其它资源，后一种情况下多个进程共享物理内存，磁盘打印机和其它资源

![image-20200718105845677](image-20200718105845677.png)

![image-20200718105929567](image-20200718105929567.png)

线程概念试图实现的是，**共享一组资源的多个线程的执行能力，以便这些线程可以为完成某一任务而共同工作。**

线程的状态：运行、阻塞、就绪、终止

正在运行的线程拥有CPU并且是活跃的。被阻塞的线程正在等待某个释放它的事件。就绪的线程可以被调度运行。线程间的状态转换和进程是一样的。

每个线程的堆栈有一帧供各个被调用但是还没有从中返回的过程使用。在该栈帧中存放了相应过程的局部变量以及过程调用完成后使用的返回地址，例如如果过程X调用了Y，Y又调用了Z，供X、Y、Z使用的栈帧都会保存在堆栈中。

在某些系统中通过调用一个过程，如thread_join，一个线程可以等待一个线程退出。这个过程阻塞**调用线程**直到那个线程退出。另一个常见的线程调用是thread_yield，**它允许线程自动放弃CPU从而让另一个线程运行，这个很重要，因为线程库无法像进程一样利用始终中断强制让出CPU。**

线程在某种程度上也引入了一些复杂性。考虑下fork调用。如果父进程有多个线程，子进程也需要有吗？如果不是则子进程可能无法工作，因为该子进程中的线程都是必须的。然而，如果子进程拥有了与父进程一样的多个进程，如果父进程在read调用上阻塞了会发生什么情况？两个线程被阻塞在键盘上？在键入输入两个线程都应该得到输入的副本吗？还是仅有父进程得到该输入的副本？或者仅有子进程得到？类似的问题在进行网络连接时也会出现。

#### POSIX线程

所有的pthread线程都有某些特性，每个都含有一个标识符、一组寄存器、一组存储在结构中的属性。这些属性包括堆栈大小、调度参数以及其他线程需要的项目

![image-20200801113615172](image-20200801113615172.png)

phread_exit调用可以终止该线程**并释放它的栈**。phread_attr_init建立关联一个线程的属性结构并初始化成默认值。这些值可以通过修改属性结构中的阈值来改变。**phread_attr_destroy删除一个线程的属性结构，释放它占用的内存。**它不会影响调用它的线程，这些线程会继续存在。

#### 在用户空间实现线程

有两种方法实现线程包：**在用户空间和内核中**

在用户空间实现线程：

- 用户级线程可以在不支持线程的操作系统实现，从内核角度考虑，就按正常的方式管理，即单线程进程。现在基本没啥操作系统不支持线程了把。

- 在用户空间管理线程时，每个线程需要专用的线程表，用来跟踪该进程中的线程，它记录线程的各个属性如程序计数器、堆栈指针、寄存器和状态，该线程表由运行时系统管理

  等待进程中另一个线程完成某项工作，他调用一个一个运行时系统的过程，这个过程检查该线程是否必须进入阻塞状态，如果是，它在线程表中保存该线程的寄存器，查看表中可运行的就绪线程，把新的线程保存值重新装入机器的寄存器中。只要堆栈指针和程序计数器一被切换新的线程就又自动投入运行。如果机器有一台保存所有寄存器的指令和另一条装入全部寄存器的指令，那么整个线程的切换可以在几条指令内完成。**进行类似于这样的线程切换至少比陷入内核要快一个数量级，这是使用用户级线程包的极大的优点。**

  **保存线程状态的过程和调度程序都只是本地过程，所以启动他们比进行内核调用效率更高，另一方面不需要陷入内核，不需要进行上下文切换，也不需要对内存高速缓存进行刷新，这使得线程调度非常快捷。**

- 用户级线程允许每个进程有自己定制的调度算法，用户级线程有较好的扩展性，因为在内核空间需要固定一些表格空间和堆栈空间，如果内核线程的数量非常大就会出现一些问题。

- 在出现缺页中断时由于内核不知道用户级线程的存在，会把整个进程阻塞，尽管其它线程还可以运行

- 用户级线程还有一个问题，如果一个线程开始运行了，那么进程中的其它线程就不能运行，除非第一个线程主动放弃cpu，一个单独的进程内部没有时钟中断，所以不可能有轮转调度的方式调度进程。

#### 在内核中实现线程

这时假设没有运行时系统，进程中也没有线程表了。在内核中有记录所有线程的线程表了。当某个线程希望创建一个新线程或撤销一个已有线程它进行一个系统调用，这个系统调用通过对线程表的更新完成线程创建或撤销动作。

- 当一个线程阻塞时，内核根据其选择可以运行当前进程中的另一个线程也可以选择运行另一个进程中的线程，而在用户级线程中运行时系统只能始终运行自己的线程直到内核剥夺它的cpu。
- 由于在内核中创建或撤销线程的开销代价比较大，某些系统采取了环保的处理方式回收线程。当某个线程被撤销时标记为不可用但内核数据结构不受影响，稍后在必须创建一个新线程时就重启某个旧线程从而节省了一些开销。
- 如果某个进程中的线程引起了页面故障，内核可以很方面的检查该进程是否有其他可运行的线程，如果有就选择一个可运行的线程运行，这样做的缺点是系统调用开销比较大

#### 混合实现

采用这种方法内核只能识别内核级线程，并对其进行调度。**其中一些内核级线程会被多个用户级线程复用**，如同在没有多线程能力的操作系统中某个进程的用户级线程一样，可以创建，撤销，调度这些线程。

![image-20200801122110786](image-20200801122110786.png)

#### 调度程序激活机制

**调度程序激活机制的目标是模拟内核线程的功能，但是为线程包提供在用户空间才能实现的更好的性能和更大的灵活性**。如果线程阻塞在某个系统调用或页面故障上，只要在同一个进程中有任意一个就虚的线程，就应该有可能运行的其它线程。

使用调度程序激活机制时，内核给每个进程安排一定数量的虚拟处理器，并且让运行时系统将线程分配到处理器上。

该机制的工作思路是，当内核了解到一个线程被阻塞后，内核通知该进程的运行时系统。并且在堆栈中以参数的形式传递有问题的线程编号和所发生事件的一个描述，内核通过在一个已知的起始地址启动运行时系统从而发出了通知，这是对unix中信号的一个粗略模拟。这个机制称为**上行调用**。

一旦如此激活，运行时系统就重启调度线程，调度过程如下：**把当前线程标记为阻塞并从就绪表中去除另一个线程设置寄存器，然后启动它，当原来的进程可运行时内核又发起一次上行调用，通知运行时系统。**

#### 弹出式线程

当服务请求到来时传统的方法是将进程或线程阻塞在一个receive调用上，另一种策略是为该消息创建一个处理该消息的线程，这种线程称为**弹出式线程**。这种线程的好处是由于线程相当新，没有历史----没有必须存储的寄存器、堆栈等内容，每个线程从全新开始每个线程之间彼此都一样，这样就能快速创建线程，使用弹出式线程的结果**就是消息到达与处理开始的事件会非常短。**

![image-20200801123547380](image-20200801123547380.png)

在内核空间中运行弹出式线程通常比在用户空间中快并且容易，而且内核空间中的弹出式线程很容易访问所有表格和I/O设备。

#### 使单线程代码多线程化

对线程来说是全局变量，并不是对于整个程序来说。多个线程使用同一个全局变量可能会引发一些问题。

![image-20200801124013108](image-20200801124013108.png)

解决方案是为每个线程赋予私有的全局变量。

- 试图将单一线程程序转化为多线程程序的另一个问题是，有许多库过程并不是可重入的，即对于任何给定的过程，当前面的调用没有结束之前，不可以进行第二次调用。

  一种解决方案是为每个过程提供一个包装器，该包装器设置一个二进制位，从而标志某个库处于使用中，在先前的系统调用还没完成前，任何使用该库的线程都应该被阻塞。**这样能解决问题但是极大程度上降低了系统的并行性。**

- 接着考虑信号，当线程在用户空间实现时，内核根本不知道线程存在，因此很难将信号发送给线程。

- 由多线程引入的最后一个问题的堆栈的管理。当一个进程的堆栈溢出时，内核只是为该进程提供更多的堆栈，当一个进程由多个线程时就必须提供多个堆栈，如果内核不了解所有堆栈，就不能使他们自动增长直到堆栈出错。事实上内核可能还没意识到内存错误是和某个线程栈的增长是有关系的。

### 进程间通信（IPC）

进程间通信主要讨论三个问题：

- 一个进程如何把信息传递给另一个进程
- 确保两个或更多的进程在关键活动中不会出现交叉，例如在飞机订票系统中的两个进程为不同的客户试图争夺飞机上的最后一个座位。
- 第三个问题与正确的顺序有关，比如，如果进程A产生数据而进程B打印数据，那么B在打印前必须等待，直到A产生了一些数据。

#### 竞争条件

**两个或多个进程读写某些共享数据，而最后的结果取决于进程运行的精确时序，称为竞争条件。**

![image-20200802150525173](image-20200802150525173.png)

如上图，进程A发现7槽位为空，将7存在一个局部变量中，此时突然发生一次时钟中断，CPU认为进程A已经运行了足够长的时间，决定切换到进程B。进程B此时发现7槽位也为空，于是也将7存入局部变量中，这个时刻两个进程都认为下一个可用槽位都为7。然后进程B继续运行将文件存在7槽位，然后将下一可用槽位设置为8，这时CPU又切换到了进程A，这时它认为可用槽位为7，于是将文件存入7中将B的文件覆盖掉了。

#### 临界区

**互斥**，即以某种手段确保当一个进程在使用一个共享变量或者文件时，其他进程不能做同样的操作。

我们把对共享内存进行访问的**程序片段**称为**临界区域**或**临界区**。

避免竞争条件并且使共享数据的并发进程能够高效和正确协作的条件：

- 任何两个进程不能同时处于临界区
- 不应对CPU的速度和数量做任何假设，这句话没理解？
- 临界区外运行的进程不得阻塞其他进程
- 不得使进程无限期等待进入临界区

![image-20200802152019687](image-20200802152019687.png)

#### 忙等待的互斥

##### 1、屏蔽中断

在单处理器系统中，最简单的方法是使每个进程刚刚进入cpu的时候立即屏蔽所有中断，并在离开前打开中断。这个方案不好，因为把屏蔽中断的权力交给用户是不明智的，设想如果一个程序屏蔽完中断不再打开的情况，整个操作系统可能会因此终止。

并且在一个多核系统中，屏蔽一个CPU中断不会阻止其它CPU干预第一个CPU所做的操作，结果是人们需要更复杂的计划。

##### 2、锁变量

设想有一种锁变量，其初始值为0。当一个进程要进入临界区时，测试锁如果锁为0，将锁置1并进入临界区，如果锁值为1，则等待直到锁变成0。

**这种方法和竞争条件中的例子存在一样的问题，当一个进程发现琐变量为0，将它置1之前另一个进程被调度运行，将该锁变量置1，当第一个进程再次运行时，它同样也会将该锁置为1，则有两个进程进入了**

如果在改变锁值之前再检查一遍它的值会不会解决问题？答案是不会，如果第二个进程刚好在第一个进程完成第二次检查修改锁值之前运行并改变了锁的值还是会有两个进程进入临界区。

##### 3、严格轮换法

连续测试一个变量知道某个值出现为止，称为**忙等待**。由于这种方式浪费CPU时间，所以应该避免。只有在有理由认为等待时间是非常短的情形下，才使用忙等待。用于忙等待的锁称为**自旋锁**。自己在旋转，这个名起挺好，给个赞。b(￣▽￣)d

![image-20200802194624026](image-20200802194624026.png)

整型变量turn，初始值为0，用来记录哪个进程进入临界区，并检查或更新共享内存。开始时，进程0检查turn发现其值为0，进入临界区。进程1也发现其值为0，所以在一个等待循环中不停的测试turn，看其值何时变为1。

**当一个进程比一个进程慢了很多的情况下，轮流进入临界区不是一个好办法**。当进程0很快执行完，turn变为1，进程1此时如果在临界区外卡住，则进程0需要等进程1执行完才能执行。该方案要求两个进程严格地轮流进入它们的临界区，但违反前面叙述条件3：进程被一个临界区外的进程阻塞。

##### 4、Peterson解法

![image-20200802200755722](image-20200802200755722.png)

当进程0进入临界区的时候，进程1在进入时会被卡在while循环那里，直到进程0执行完成调用leave_region离开临界区。

##### 5、TSL指令

**测试并加锁**（test and set lock），它将一个内存字读到寄存器RX中，然后在该内存地址上存一个非零值。读字和写字操作保证时不可分割的，**即该指令结束之前其他处理器均不允许访问该内存字**。执行TSL指令的CPU将锁住内存总线，禁止其他CPU在本指令结束前访问内存。

![image-20200802201722222](image-20200802201722222.png)

锁住总线不同于屏蔽中断。屏蔽中断只能屏蔽当前处理器的不能阻止第二个CPU通过总线访问该内存，所以唯一的办法是锁住总线。

进程必须在正确的时间调用enter_region和leave_region，如果有一个进程有**欺诈行为**，则互斥会失败，换言之，只有进程合作，临界区才能工作。

#### 睡眠与唤醒

上述忙等待的解法都有缺点，这些解法的本质上都是这样的：**当一个进程想进入临界区时，先检查是否允许进入，若不允许，则该进程在原地等待，直到允许为止。**

这种方法不仅会**浪费CPU时间**，还会引起一些问题。考虑两个进程H、L，H优先级高，L优先级低。调度规则规定，只要H处于就绪态它就可以运行。某一时刻，L处于临界区，H变到就绪态，准备运行。现在H开始忙等待，由于H就绪L不会被调度，无法离开临界区，H会永远忙等待下去，这种情况被称为**优先级反转**。还有这种操作？调度只会调度H不会调度L，调度H不就把H换到临界区？调度H和调度L是分离的，调度H不是把L换出来再把H放到临界区？好难理解为啥要这样设计

考察几条进程间通信原语，他们在无法进入临界区时会被阻塞，而不是忙等待。最简单的时sleep和wakeup。**sleep是一个将引起调用进程阻塞的系统调用，即被挂起，直到另一个进程被唤醒**。**wakeup调用有一个参数，即要被唤醒的进程。**

##### 生产者-消费者问题

生产者-消费者问题，也称作**有界缓冲区**问题。问题出现在缓冲区已满，生产者还想向其中放一个新数据项的情况。其解决方案是让生产者睡眠，待消费者从缓冲区中取出一个或多个数据项再唤醒它，同样当消费者试图从缓冲区取出数据而发现缓冲区为空时，消费者就睡眠，直到生产者向其中放入一些数据时再将其唤醒。

![image-20200803234400300](image-20200803234400300.png)

这里有可能会出现竞争条件，原因是未对count加锁。有可能出现以下情况：

缓冲区为空，消费者刚读取count值发现它为0。此时调度程序决定暂停消费者并启动生产者。生产者向缓冲区加入一个数据项，count加1。现在count变成1它推断消费者此时在睡眠，于是调用wakeup唤醒消费者。而此时**消费者在逻辑上并未睡眠（没法理解，调度程序还没暂停消费者？）**，所以wakeup信号丢失。当消费者下次运行时，它将测试**先前读到的count的值（测试的count不是全局变量吗？为啥时之前读到的值）**。发现它为0，于是睡眠，生产者迟早会填满整个缓冲区，然后睡眠，最后这两个进程就永远沉睡下去了。

**问题的实质在于发给一个尚未睡眠进程的wakeup信号 丢失了。如果它没有丢失，则一切很正常。一种快速的方法时加上一个唤醒等待位，当一个wakeup信号发送给一个清醒的进程信号时将该位置1，随后当该进程要睡眠时，如果唤醒等待位为1，则将该位清楚，进程仍然保持清醒。**

#### 信号量

信号量使用一个整型变量累计唤醒次数。一个信号量的取值可以为0或者正数。

Dijkstra建议设立两种操作：down和up。对一信号量执行down操作时先检查其值是否大于0，若大于0则减一并继续。若为0，则进程睡眠，并且此时down操作并未结束。检查数值，修改变量，以及可能发生的睡眠操作均作为一个单一的原子操作完成。保证一个信号量一旦操作开始，则在该操作完成或阻塞之前，其他进程均不可访问该信号量。所谓原子操作是指一组关联的操作要么都不间断的执行要么都不执行。

up是对信号量的值增1。对一个有进程在其上睡眠的信号量执行一次up操作后，该信号量的值仍为0，但在其上睡眠的进程却少了一个。

**用信号量解决生产者消费者问题**

up和down通常作为系统调用实现，并且在执行以下操作时操作系统需要屏蔽所有中断：**测试信号量，更新信号量，在需要时使某个进程睡眠**

该解决方案使用了三个信号量：full用来记录充满的缓冲区槽目数；empty记录空的缓冲槽目数；mutex确保生产者和消费者不会同时访问缓冲区。**供两个或多个进程使用的信号量，其初值为1，保证同时只有一个进程可以进入临界区，称作二元信号量。**如果每个进程在进入临界区前执行一个down操作，退出时执行一个up操作，就能实现互斥。

启动一个I/O设备之后，管理进程就立即对相关联的信号量执行一个down操作，于是进程被立即阻塞。当终端到来时，中断处理程序随后对相关信号量执行一个up操作

![image-20200804231014571](image-20200804231014571.png)

当消费者消费数据过慢会导致生产者一直阻塞吗？不会消费者是离开临界区进行消费的。

信号量的另一种用途是用于实现**同步**，信号量full和empty可以保证某种事件的顺序执行。

#### 互斥量

如果不需要信号量的计数能力，可以使用信号量的一个简化版本，称为互斥量。互斥量在实现时即容易又有效，所以它在实现**用户空间线程包**时非常有用。

互斥量是一个可以处于两态之一的变量：**解锁和加锁**。所以只需要一个二进制位就可以表示。互斥量是两个过程，当一个线程或进程需要访问临界区时调用mutex_lock。如果该互斥量当前时解锁的，则调用成功，调用线程可以自由进入该临界区。

如果互斥量被加锁，则调用线程被阻塞。如果多个线程被阻塞在该互斥量上，将随机选择一个线程并允许他获得锁。

![image-20200805231755423](image-20200805231755423.png)

mutex_lock和enter_region的区别时：enter_region进入临界区失败时始终重复测试锁，但由于超时时钟的作用会调度其他进程运行，这样迟早拥有锁的进程会运行并释放锁。当在线程中情况不太一样，没有时钟停止运行过长的线程。结果时通过忙等待方式来获取锁的线程将永远运行下去，不会得到锁，因为这个线程不会让其他线程得到运行机会

mutex_lock在获取锁失败的时候会将cpu放弃给另一个线程，这样就没有忙等待，这样的调用返回失败或者锁，不阻塞线程，这样就给了调用线程一个灵活性，可以决定下一步做什么。

##### 1、快速用户区互斥量futex

随着并行的增加，有效的同步和锁机制对于性能来说非常重要。**等待时间短，自旋锁会很快，等待时间长，浪费CPU周期。如果有很多竞争，那么阻塞此进程，并且仅当锁被释放的时候让内核解除阻塞会更加有效。然而这也带来了相反的问题，如果竞争很小，上下文切换带来的开销就会变得很大。**

futex是linux的一个特性，它实现了基本的锁，但避免陷入内核，除非它真的不得不这样做。一个futex包含两部分：一个内核服务和一个用户库。内核服务提供一个等待队列，它允许许多进程在一个锁上等待。将一个进程放在等待队列需要系统调用，我们应该避免这种情况。

当线程检查结果，看锁是否被释放，如果处于被锁状态，那么线程成功夺取该锁，如果锁被另一个线程持有，那么线程必须等待，这种情况下futex不自旋，而是使用一个系统调用把这个线程放在等待队列上。可以期望的是，切换到内核的开销已经和合乎情理的了，因为无论如何线程被阻塞了。

##### 2、pthread中的互斥量

pthread提供许多来同步线程的函数，其基本机制是使用一个可以被锁定和解锁的互斥量来保护每个临界区。**这些互斥锁不是强制性的，而是由程序员来保证线程正确的使用它们。**

除了互斥量外，pthread还提供了另一种同步机制：**条件变量，互斥量在允许或阻塞对临界区的访问上是很有用的，条件变量允许线程由于一些未达到的条件而阻塞。**

一个线程将产品放在一个缓冲区内，由另一个线程将它取出。如果生产者发现没有空槽可以使用，它不得不阻塞直到有一个空槽可以使用，生产者使用互斥量可以进行原子性检查，而不受其它线程干扰。但是当发现缓冲区已经满了以后，生产者需要一种方法来阻塞自己并在以后被唤醒，这就是条件变量做的事情了。

![image-20200810223814634](image-20200810223814634.png)

条件变量的主要操作是phread_cond_wait和pthread_cond_signal，前者阻塞调用线程直到另一其它线程向它发信号。条件变量允许这种等待与阻塞原子性的进行，当有多个线程被阻塞并等待同一个信号时，可以使用phread_cond_broadcast。

条件变量与互斥量一起使用。这种模式用于让一个线程锁住一个互斥量，然后当他不能获得它期待的结果时等待一个条件变量。最后另一个线程会向他发信号，使它可以继续运行。phread_cond_wait原子性的调用并解锁它持有的互斥量。

**条件变量不会再内存中保存，如果将一个信号量传递给一个没有线程在等待的条件变量，那么信号就会丢失。**

![image-20200810225620942](image-20200810225620942.png)

只是串行的吗？不能并行？保证了数据安全，降低了效率？

#### 管程
一个管程是一个由过程、变量及数据结构组成的一个集合，他们组成一个特殊的模块或软件包，进程可在任何需要的时候调用管程中的过程，但他们不能再管程之外声明的过程中直接访问管程内的数据结构。

管程有一个很重要的特性，即任一时刻管程中只能有一个活跃进程，这一特性使得管程能有效地完成互斥。

## 死锁

操作系统都具有授权一个进程排他的访问某一种资源的能力

### 资源

在进程对设备、文件等取得了排他性访问权时，有可能会出现死锁，为了尽可能使死锁的讨论通用，我们把这类需要排他性使用的对象称为资源，简单的说资源就是随着时间推移，必须能获得，使用以及释放的任何东西。

#### 可抢占资源和不可抢占资源

资源可分为两类：**可抢占的和不可抢占的**

- **可抢占资源**：可以从拥有它的进程中抢占而不会产生任何副作用，存储器就是一类可抢占的资源
- **不可抢占资源**：指在不引起相关的计算失败的情况下，无法把它从占有它的进程处抢过来，比如一个进程开始刻光盘，突然将蓝光光盘刻录机分配给另一个进程，那么将划坏蓝光光盘。在任何时刻蓝光光盘刻录机都是不可抢占的。

使用一个资源的事件顺序可以用抽象的方式表示如下：

1. **请求资源**
2. **使用资源**
3. **释放资源**

### 死锁简介

死锁定义：如果一个进程集合中的每个进程**都在等待只能由该进程集合中的其它进程才能引发的事件（互相等待释放资源）**，那么该进程集合就是死锁的。

由于所有的进程都在等待，所以没有一个进程能引发可以唤醒该进程集合中的其他进程的事件，这样所有的进程都只好无限期的等待下去。大多数情况下每个进程所等待的事件使释放进程集合中其他进程所占用的资源，这种死锁称为**资源死锁**。

#### 资源死锁的条件

发生资源死锁的四个必要条件：

- **互斥条件**。每个资源要么已经分配给了一个进程，要么就是可用的。
- **占有和等待条件**。已经得到了某个资源的进程可以再请求新的资源。
- **不可抢占条件**。已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式的释放。
- **环路等待条件**。死锁发生时，系统中一定有两个或两个以上的进程组成的一条环路，该环路的每个进程都在等待下一个进程所占有的资源。

死锁发生时，上述四个条件一定是同时满足的。如果其中任何一个条件不成立，死锁就不会发生。

#### 死锁建模

![image-20200829102258545](image-20200829102258545.png)

圆形表示进程，方形表示资源，从资源节点到进程节点的有向边代表该资源已经被请求、授权并被进程占用。

![image-20200829102512826](image-20200829102512826.png)

四种处理死锁的策略：

- 忽略该问题。emm，可以忽略这个策略。
- **检测死锁并恢复**。让死锁发生，检测他们是否发生，一旦发生死锁，采取行动解决问题。
- **仔细对资源进行分配，动态的避免死锁。**
- **通过破坏死锁产生的四个条件之一，防止死锁发生。**

### 死锁检测和死锁恢复

在使用这种技术时，系统并不试图阻止死锁的产生，而是允许死锁发生，当检测到死锁发生之后，采取措施进行恢复。

#### 每种资源类型只有一个资源的死锁检测

可以通过构建资源分配图来进行死锁检测

![image-20200829103259420](image-20200829103259420.png)

这时候死锁检测就可以抽象成有向图环路检测的问题，有很多有向图环路检测的算法。

#### 每种资源类型多个资源的死锁检测

书中提供了一种基于矩阵的算法检测从P1到Pn这n个进程中的死锁。假设资源的类型数为m，E1代表资源类型1，E2代表资源类型2，Ei代表资源类型i（1<=i<=m）。E是**现有资源向量**，代表每种已存在的资源总数。比如资源1代表磁带机，则E1=2代表有两台磁带机。

假设A是**可用资源向量**，则Ai表示当前可供使用的资源数。

现在我们需要两个数组：C代表**当前分配矩阵**，R代表**请求矩阵**。C的第i行表示Pi当前所持有的每一种资源类型的资源数。Cij代表**进程i所持有的资源j**的数量。同理Rij代表**Pi需要的资源j**的数量。

![image-20200829104152614](image-20200829104152614.png)

这四种数据结构间有一个重要的恒等式：

![image-20200829104245754](image-20200829104245754.png)

死锁检测算法就是基于向量的比较。每个进程初始都是没有被标记过的。算法开始会对进程做标记，进程被标记后就表明他们可以被执行，不会进入死锁。当算法结束时，任何没有标记的进程都是死锁进程。该算法假定了一个最坏情形：**所有的进程再退出以前都会不停的获取资源**。

死锁检测算法如下：

- 寻找一个没有被标记的进程Pi，对于它而言R矩阵的第i行向量小于或等于A。
- 如果找到了这样一个进程，那么将C矩阵的第i行向量加到A中标记该进程并转到第一步
- 如果没有这样的进程，那么算法终止。

算法结束时没有被标记的进程都是死锁进程。

算法的第一步是寻找可以运行完毕的进程，该进程的特点是**它有资源并且该请求可被当前的可用资源满足**。这一选中的进程随后就被运行完毕，在这段时间内它释放自己的资源并将它们返回到可用资源库中。然后这一进程被标记为已完成。如果所有的进程都可以运行完毕就不存在死锁的情况。如果其中某些进程一直不能运行，那么他们就是死锁进程。**当前请求的资源一直不能被满足，互相需要其他进程去释放资源，进入了环路等待，一直无法运行**。

#### 从死锁中恢复

- **利用抢占恢复**：在不通知原进程的情况下，将某一资源从一个进程强行取走给另一个进程使用，接着又送回，这种做法是否可行主要取决于该资源本身的特性。用这种方法恢复通常比较困难或者说不太可能。若选择挂起某个进程，则在很大程度上取决于那个进程拥有比较容易回收的资源。
- **利用回滚恢复**：周期性的将进程的资源状态、存储映像进行保存，当发生死锁回滚到死锁发生前的一个时间点
- **通过杀死进程恢复**：杀掉环中的一个或多个进程，直到死锁环被破坏。

### 死锁避免

#### 资源轨迹图

避免死锁的主要算法是基于一个**安全状态**的概念。

![image-20200829110313511](image-20200829110313511.png)

上图横轴表示进程A执行的一个命令，纵轴代表进程B执行的一个命令。进程A在I1出请求一台打印机，在I3处释放，在I2处请求一台绘图仪在I4处释放。

图中的阴影部分是我们感兴趣的，画着从左下到右上线的部分表示该区域两个进程都拥有打印机，而互斥的规则决定了不可能进入该区域。另一种斜线的区域表示两个进程都拥有绘图仪，且同样不可进入。

**如果系统一旦进入由I1、I2和I5、I6组成的矩形区域那么最后一定会到达I2和I6的交叉点，在该点处A请求绘图仪，B请求打印机，而且这两种资源都已经被分配。**

#### 安全状态和不安全状态

死锁避免算法使用了6-6中的相关信息，在任何时刻当前状态包括了E、A、C、R矩阵。**如果没有死锁发生，并且即使所有进程突然请求对资源的最大需求，也仍然存在某种调度次序能够使得每一个进程运行完毕，则称该状态是安全的。**

需要注意的是不安全状态并不是死锁。如下图，系统能运行一段时间。实际上甚至有一个进程可以完成。并且在A请求其他资源实例前可能先释放一个资源实例，这就可以让C先完成，从而避免了死锁。因而，安全状态和不安全状态的区别是：**从安全状态出发，系统能保证所有进程完成，而从不安全状态出发，就没有这样的保证。**

![image-20200829111705569](image-20200829111705569.png)

#### 单个资源的银行家算法

算法做的就是**判断对请求的满足是否会导致系统进入不安全状态**，如果是就拒绝请求；如果满足后系统仍是安全的就予以分配。

![image-20200829112518310](image-20200829112518310.png)

6-11b这个状态是安全的，由于还有两个单位，它可以让C先完成，然后释放C所占的4个单位资源，有了这4个单位资源，银行家就可以给D或B分配所需的资源。

银行家算法就是对每一个请求进行检查，检查如果满足这一请求是否会达到安全状态。**若是，则满足该请求，否则就推迟对该请求的满足**。为了检查状态是否安全，银行家**需要考虑他是否有足够的资源满足某一客户**，如果可以，那么这笔贷款就是可以收回的，并且**接着检查最大限额的客户**，以此类推，**如果所有的投资都可以被收回，那么该状态就是安全的**，最初的请求可以被批准。

#### 多个资源的银行家算法

待补充

### 死锁预防

- 破坏互斥资源：避免分配那些不是绝对必须的资源，尽量做到尽可能少的进程可以真正请求资源。
- 破坏占有并等待条件：禁止已持有资源的进程再等待其它资源，一种思路是规定所有进程在开始执行前请求所需的全部资源，如果有一个或多个资源在使用就不进行分配，进程等待。这种方法的问题是很多进程直到运行时才知道它需要多少资源；另一种是当一个进程请求资源时，先暂时释放当前占有的资源，然后再尝试一次获取所需的全部资源。
- 破坏不可抢占条件：假如一个进程已经分配到一个打印机，且正在进行打印输出，如果由于他需要的绘图仪无法获取而强制性的把他占有的打印机抢掉，会引起一片混乱。但是一些资源可以通过虚拟化的方式来避免发生这样的情况。
- 破坏环路等待条件：消除环路有几种方法，一种是保证每一个进程在任何时刻只能占用一个资源，如果要请求另外要给资源，它必须先释放一个资源。另一种方法是将所有资源统一编号，进程可以在任何时刻提出资源请求，但是所有请求必须按照资源编号的顺序提出。进程可以先请求打印机再请求磁带机，但是不能先请求绘图仪后请求打印机。

![image-20200830224408680](image-20200830224408680.png)

在任何情况下总有一个已分配的资源是编号最高的。占用该资源的进程不可能请求其他已分配的各种资源。它或者会执行完毕，或者最坏的情况是取请求编号更高的资源，而编号更高的资源肯定是可用的。

### 其它问题

#### 两阶段加锁

在很多数据库中一个经常发生的操作是请求锁住一些记录，然后更新所有锁住的记录。当同时有多个进程运行时就会有死锁的危险。

常用的方法是**两阶段加锁**。在第一个阶段，进程试图对所有需要的记录进行加锁，一次锁一个记录。如果第一阶段加锁成功，就开始第二阶段，完成更新后释放锁，在第一阶段并没有做实际的工作。

#### 通信死锁

资源死锁是**竞争性同步**的问题。**进程将资源加锁，是为了防止交替访问资源而产生不一致的资源状态。**

通信死锁是指两个或两个以上进程利用发送信息来通信时，一种普遍的形式是进程A向进程B发送请求信息，然后阻塞直到B回复，假设请求信息丢失，A将阻塞以等待回复，而B将阻塞等待一个向其发送命令的请求，因此发生死锁。中断通信死锁有一种最简单的方案：**超时**

#### 活锁

在某些情况下，当进程意识到不能获取所需要的下一个锁时，就会尝试礼貌的释放已经获得锁，然后等待1ms，再尝试一次，但是如果另一个进程在相同的时刻做了相同的操作，那么就像两个人在一条路上相遇并同时给对方让路一样，相同的步调将导致双方都无法前进。

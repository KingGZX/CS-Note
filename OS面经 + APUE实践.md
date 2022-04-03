## 多线程编程

**Little Tip: APUE手册中一些API在我的Windows底下GCC8.1.0是有提示语句并且编译通过的，但在我的Ubuntu GCC 11.2.0版本下VS Code并没有给我跳提示语句，但是编译运行却也是完全没问题。**

#### 1. pthread_create 和 pthread_self 

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <stdlib.h>

pthread_t ntid;

void printds(const char* s){
    pid_t pid;
    pthread_t tid;

    pid = getpid();
    tid = pthread_self();

    printf("%s pid %lu tid %lu (0x%lx)\n", s, (unsigned long)pid, (unsigned long)tid, (unsigned long)tid);
}

void* thr_fn(void* arg){
    printds("new thread: ");
    return (void *)0;
}

int main(){
    int err;
    err = pthread_create(&ntid, NULL, thr_fn, NULL);
    if(err != 0)
        printf("error, can't create thread");
    printds("main thread:");
    sleep(0);
    exit(0);
}
```

printds函数用于打印进程标识符和线程标志符，Linux系统下线程标志符就是无符号长整数。如下图：

![1641723064339](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641723064339.png)

其中用到了两个获取标志符的api，getpid和pthread_self。

然后主要看pthread_create，看看如何创建一个线程：

![1641727616606](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641727616606.png)

param1：成功时此参数指向的内存单元被设置成新创建的线程ID。

param2：定制线程的不同属性，我们会在后续来讨论属性的问题。

param3：新创建的线程从 start_routine这个函数指针指向的首地址开始运行。

param4：函数所需要的无类型指针参数，如果需要的参数有一个及以上，则将所有放入一个结构体中。



上例中还有一点值得注意的是，主线程我们执行了 sleep，主线程有一个休眠时间以供新创建的线程执行完其任务。最后，我们在gcc编译时要再最后跟上"-lpthread"加上动态线程库才行。



#### 2. pthread_exit 和 pthread_join

线程有多种退出的方式：

1.从启动例程(我们在创建时有一个start_routine)中返回。

2.可以被同一进程中的其他线程取消。  调用api:   pthread_cancel(pthread_t tpid);

3.线程本身调用pthread_exit。

![1641738915041](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641738915041.png)

此rval_ptr可以通过pthread_join获得。

如果线程从它的启动例程简单返回则rval_ptr包含了返回码。如果线程是被取消的，则此内存被设为"PTHREAD_CANCELED"。

可以通过pthread_join将线程置于分离状态，这样资源就可以恢复。如果对线程返回值不感兴趣则可以将指针设为NULL，此时调用pthread_join则获取不了返回状态。

下面一个简单的例子获取线程的返回值：

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void* thr_fn1(void* arg){
    printf("thread1 returning\n");
    return ((void*) 1);
}

void* thr_fn2(void* arg){
    printf("thread2 exiting\n");
    pthread_exit((void*)2);
}

int main(){
    pthread_t tpid1, tpid2;
    int err;
    void* tret;
    err = pthread_create(&tpid1, NULL, thr_fn1, NULL);
    err = pthread_create(&tpid2, NULL, thr_fn2, NULL);

    err = pthread_join(tpid1, &tret);
    printf("thread1 exit code %ld\n", (long)tret);
    err = pthread_join(tpid2, &tret);
    printf("thread2 exit code %ld\n", (long)tret);
    return 0;
}
```

![1641739965316](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641739965316.png)

根据代码我们利用pthread_join去获取等待线程(第一个参数就是线程ID)的返回值。

自然我们有些复杂的任务或许会返回一个结构体，一个类，anyway，此时要注意返回值那块内存是否已经被回收？比如我们的代码如下：

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

struct foo{
    int x, y, z, h;
};

void printfoo(const char* s, const struct foo* ptr){
    printf("%s", s);
    printf("structrue at 0x%lx\n", (unsigned long)ptr);
    printf("foo.x = %d\n", ptr->x);
    printf("foo.y = %d\n", ptr->y);
    printf("foo.z = %d\n", ptr->z);
    printf("foo.h = %d\n", ptr->h);
}

void* thr_fn1(void* arg){
    struct foo fp = {1, 2, 3, 4};
    printfoo("thread 1: \n", &fp);
    pthread_exit((void*)&fp);
}

void* thr_fn2(void* arg){
    printf("thread 2: ID is %lu\n", (unsigned long)pthread_self());
    pthread_exit((void*)0);
}

int main(){
    pthread_t tpid1, tpid2;
    int err;
    void* tret;
    struct foo* fp;

    err = pthread_create(&tpid1, NULL, thr_fn1, NULL);
    err = pthread_join(tpid1, (void*)&fp);
    sleep(1);
    printf("parent starting second thread\n");
    err = pthread_create(&tpid2, NULL, thr_fn2, NULL);
    printf("thread1 exit code %ld\n", (long)tret);
    sleep(1);
    printfoo("parent:\n", fp);
    return 0;
}
```

我们创建两个线程，第一个线程在函数段建了一个结构体fp，我们理应知道局部变量会建立在栈上。而栈，会在函数退出时内容被覆盖，这也就是为什么最后我们主函数内的fp得不到我们想要的结果的原因。

第二个线程做的事很简单，打印自己的ID，返回0。输出结果图如下所示：

![1641741246060](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641741246060.png)

针对这样的bug，有两种解决方案，一是用堆内存，二就是设置全局变量。我们简单尝试堆内存(全局较简单)：

```c
void* thr_fn1(void* arg){
    struct foo* fp = (struct foo*)malloc(sizeof(struct foo));
    fp->x = 1;
    fp->y = 2;
    fp->z = 3;
    fp->h = 4;
    printfoo("thread 1: \n", fp);
    pthread_exit((void*)fp);
    // free(fp);
}
```

结果如下图所示，成功了(其实打开代码里的free也是可以的，因为线程已经退出，不会执行free)：

![1641741552844](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1641741552844.png)

如果线程已经被分离，则我们无法通过pthread_join获得其终止状态。对于"分离状态"，我们也有专门的api:

int pthread_detach(pthread_t tid);

#### 3. 线程同步

###### 锁（互斥量）

1.第一个就是我们在OS里涉及到的经典的"读写者"问题。我们允许多个进程共同进行"读"操作。而读操作会需要一把锁，当锁上后，读者想要进入可以直接进入但写者想要进入则需要等待锁释放。

进程里我们有wait & signal原语，对于线程我们有专门的 pthread_mutex_t 锁。我们可以通过静态声明和动态创建的方式来创建锁。对于动态创建的锁我们需要做init操作设置其属性(可以直接把参数置为NULL声明默认属性)，在销毁(free操作)之前需要调用pthread_mutex_destroy。

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>


int main(){
    pthread_mutex_t lock1 = PTHREAD_MUTEX_INITIALIZER;
	// 静态锁
    pthread_mutex_t* lock2 = (pthread_mutex_t*)malloc(sizeof(pthread_mutex_t));
    int err = pthread_mutex_init(lock2, NULL);
    err = pthread_mutex_destroy(lock2);
    free(lock2);
    // 动态锁
    return 0;
}
```



然后就是"上锁"状态，如果互斥量已经上锁则线程会阻塞至互斥量被解锁。因此我们也配套有互斥量解锁的api。如果不希望互斥量被阻塞的，可以调用一个很抽象的API  pthead_mutex_trylock，如果互斥量已经被锁则会返回EBUSY，否则的话不会阻塞直接返回0。

我们同样根据一个案例来观察。设一个结构体foo，内置id、引用计数count、锁lock。我们用函数foo_hold来增加计数，在对count操作前，我们要先"上锁"，因为"++"操作并不是原子操作，如果多线程并行访问了此结构体，不知道最后是加了n或是n-1······甚至只加了1。  用函数foo_rele来减少计数，当引用次数为0了则应该释放资源。

这里要考虑这么一种情况，假如一个线程正阻塞等解锁此资源，但资源却因计数为0即将被释放，那么此线程将永远阻塞。因此我们采用的技术是先解锁，再将锁destroy，最后释放结构体资源。详细代码如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

struct foo{
    int count;
    pthread_mutex_t lock;
    int id;
    // more attributes
};

struct foo* foo_alloc(){
    struct foo* fp;
    if((fp = (struct foo*)malloc(sizeof(foo))) != NULL){
        fp->count = 1;
        if(pthread_mutex_init(&fp->lock) != 0){
            free(fp);
            return(NULL);
        }
    }
    return fp;
}

void foo_hold(struct foo* fp){
    pthread_mutex_lock(&fp->lock);
    fp->count++;
    pthread_mutex_unlock(&fp->lock);
}

void foo_rele(struct foo* fp){
    pthread_mutex_lock(&fp->lock);
    if(--fp->count == 0){   // last reference
        pthread_mutex_unlock(&fp->lock);
        pthread_mutex_destroy(&fp->lock);
        free(fp);
    }
    else{
        pthread_mutex_unlock(&fp->lock);
    }
}

int main(){

    return 0;
}
```





###### 避免死锁

线程对同一资源加锁两次会导致死锁，但我认为更常见的是**两个线程互相索取对方占有的资源**。可以通过仔细控制加锁的顺序来避免死锁发生，但在锁较多的时候，我们很难做出精确的判断。可以用我们上面提及的 pthread_mutex_trylock来避免死锁发生。

首先我们利用方法一，来解决上面代码问题的升级版，就是在全局变量有一个散列表，结构体多一个指针：

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/time.h>

#define NHASH 29
#define HASH(id) (((unsigned long)id) % NHASH)

struct foo* fh[NHASH];

pthread_mutex_t hashlock = PTHREAD_MUTEX_INITIALIZER;

struct foo{
    int count;
    pthread_mutex_t lock;
    int id;
    struct foo* f_next;
    // more attributes
};
```

首先声明这些要用的东西，我们预定义了HASH算法和结构体指针数组，一把hashlock锁，结构体本身。hashlock锁是用于"锁住全局变量fh数组的，同时也可以锁住f_next指针，因为我们任意线程都可以通过线程访问这个数组，因此我们在用到数组中的元素时应该锁住数组防止被修改。同理，由于f_next串成的链表中的所有元素和数组中的其实是同一些元素，所以在用到这个时也需要hashlock锁。"而另一个结构体内的锁就是锁访问结构体元素时用到的了，和上面是一样的。

获得结构体指针操作：

```c
struct foo* foo_alloc(int id){
    struct foo* fp;
    int idx;
    // 刚创建时不会有线程可以使用它，因此可以不用考虑锁的问题
    if((fp = (struct foo*)malloc(sizeof(struct foo))) != NULL){
        fp->count = 1;
        fp->id = id;
        if(pthread_mutex_init(&fp->lock, NULL) != 0){
            free(fp);
            return(NULL);
        }
    }
    // 现在要将创建好的放置于数组中，所以要为数组上锁
    idx = HASH(id);
    pthread_mutex_lock(&hashlock);
    fp->f_next = fh[idx];    // hash冲突时，在同一位置上挂出一根链表
    fh[idx] = fp; 
    pthread_mutex_lock(&fp->lock);
    pthread_mutex_unlock(&hashlock);
    // other initialization operations
    pthread_mutex_unlock(&fp->lock);
    return fp;
}
```

我们将指针放进数组时要为数组上锁。**还有很重要的一个顺序就是，先上结构体内部锁再解锁数组，因为一旦先解开数组锁，其他线程就能访问到它并对其进行操作，而我们可能还有其他初始化操作要做，所以顺序很重要！**

hold增加计数的操作仍旧是一样的，和hashlock锁无关。

我们为数组增加了一个find操作：

```c
struct foo* foo_find(int id){
    struct foo* fp;
    // 因为要进入数组索引，我们锁住数组
    pthread_mutex_lock(&hashlock);
    // 由于哈希冲突的存在，数组的那个位置可能挂着的是一个链表
    for(fp = fh[HASH(id)] ; fp != NULL ; fp = fp->f_next){
        if(fp->id == id){
            foo_hold(fp);
            break;
        }
    }
    pthread_mutex_unlock(&hashlock);
    return fp;
}
```



foo_rele操作就显得复杂许多，因为我们要考虑释放指针，要进去数组操作。

```c
void foo_rele(struct foo* fp){
    struct foo* tfp;
    int idx;
    // 同样是先检测此结构体的引用计数
    pthread_mutex_lock(&fp->lock);
    if(fp->count == 1){   // last reference
        // 需要先释放这把锁，因为我们要进去数组中删掉此元素，所以先解锁再给数组上锁
        // 否则不先锁上数组，别的线程一直以为此结构体可以被find，会阻塞在hold处
        // 而如果不先解锁，刚刚的情况仍旧存在，有线程会阻塞着。
        // 但解锁后再重新上两把锁后就需要考虑计数值是不是仍旧为1，因为可能有线程find成功了，计数变动。
        
        /*
        well, 最大问题是，如果你不先解锁：而又像我们刚刚讨论的，有线程已经占据了互斥量hashlock而被阻塞		  在fp->lock处，这就满足形成死锁的条件了。那显然不可行！
        */
        pthread_mutex_unlock(&fp->lock);
        pthread_mutex_lock(&hashlock);
        pthread_mutex_lock(&fp->lock);
        if(fp->count != 1){			// 有可能在没锁上fh数组前被执行了find
            --fp->count;
            pthread_mutex_unlock(&fp->lock);
            pthread_mutex_unlock(&hashlock);
            return;
        }
        else{
            // remove the element from hasharray
            idx = HASH(fp->id);
            tfp = fh[idx];
            if(tfp == fp)
                fh[idx] = fp->f_next;
            else{
                while(tfp->f_next != fp)
                    tfp = tfp->f_next;
                tfp->f_next = fp->f_next;
            }
            pthread_mutex_unlock(&hashlock);
            pthread_mutex_unlock(&fp->lock);   // 不能直接free fp 应该先把锁资源进行释放
            pthread_mutex_destroy(&fp->lock);
            free(fp);
        }
    }
    else{
        fp->count --;
        pthread_mutex_unlock(&fp->lock);
    }
}
```

我们可以用hashlock来控制对计数的操作，这样一来foo_rele操作会简单很多。首先foo_hold函数应该上hashlock锁，同时在find函数里不要再去调用hold函数，否则会对同一互斥量上两次锁造成死锁。所以find函数里直接计数增加即可。

最关键的，在foo_rele函数里，我们直接通过锁上hashlock来访问计数次数，此时如果为最后一个索引**(当然，外面可能有线程等待hashlock锁来访问此资源我们不予理睬，否则整体都会变得过于复杂)**，我们直接从fp数组中移除他，然后销毁对象内部的锁，最后释放资源。这样那些阻塞着的线程就能继续运行了。

而不像我们用lock锁控制计数次数时，需要先解锁再获取hashlock锁：否则很有可能因为有别的线程占据了hashlock而等到lock，形成死锁。因此整体会方便很多！





API：[pthread_mutex_timedlock(pthread_mutex_t * restrict mutex, const struct timespec * restrict tsptr)]

此函数允许绑定线程阻塞时间。此函数和pthread_mutex_lock基本是等价的，只不过多了一个参数，而且在达到超时时间时就不会再对互斥量加锁。 超时时间指定愿意等待的绝对时间，结构体timespec用秒和微秒表示。

```c
#include <stdio.h>
#include <time.h>
#include <pthread.h>
#include <sys/time.h>

int main(){
    int err;
    struct timespec tout;
    struct tm* tmp;
    char buf[64];
    pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

    pthread_mutex_lock(&lock);   // fisrt lock
    clock_gettime(CLOCK_REALTIME, &tout);
    tmp = localtime(&tout.tv_sec);
    strftime(buf, sizeof(buf), "%r", tmp);
    printf("current time is %s\n", buf);
    // printf("%ld\n", tout.tv_sec);
    tout.tv_sec += 10;                // 10 seconds from now (actually，I can't understand it well)
    // pthread_mutex_unlock(&lock);
    err = pthread_mutex_timedlock(&lock, &tout);   // try to lock the locked mutex again
    clock_gettime(CLOCK_REALTIME, &tout);
    tmp = localtime(&tout.tv_sec);
    strftime(buf, sizeof(buf), "%r", tmp);
    printf("current time is %s\n", buf);
    // printf("%ld\n", tout.tv_sec);
    if(err == 0)
        printf("mutex locked again\n");
    else
        printf("can't lock mutex again\n");
    return 0;
}
```

###### 读写锁

其实在"**锁(互斥量)**"部分就已经对读写者问题进行了阐述。

这相比于互斥量会有更大的并行性。操作系统实现层面会对问题实现层面进行一定改进，即如果在读模式上锁状态下，如果有写锁的请求那么读写锁通常会阻塞后面到来的读模式锁请求，防止读者一直占据而写得不到相应。

APUE Page340有一个操作读写锁进行工作任务分配的实例，代码实际上并不复杂(操作锁部分)，甚至不及互斥量。其实主要是对"双向链表"的操作比较复杂，如果有兴趣可以自己看看，要经常判断NULL的问题。



这里有提供和上面一样的带超时锁。如下所示：

![1642063316761](C:\Users\user\AppData\Roaming\Typora\typora-user-images\1642063316761.png)

###### 条件变量





###### 自旋锁

自旋锁与互斥量类似，但并不是通过"阻塞"进行同步，而是让线程在获取锁之前处于忙等状态。自旋锁适用的两种情况是: 1.锁被持有的时间短， 2.线程不希望在重新调度上花费太多成本。









### 进程/线程通信方式

1.管道通信。

 管道这种通讯方式有两种限制，一是半双工的通信，数据只能单向流动，二是只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。 

 管道可以分为两类：匿名管道和命名管道。匿名管道是单向的，只能在有亲缘关系的进程间通信；命名管道以磁盘文件的方式存在，可以实现本机任意两个进程通信。

匿名管道通过系统调用pipe，有名管道通过mkfifo实现，前者包含于unistd.h头文件后者包含于sys/stat.h下。

2.共享内存

共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。**共享内存是最快的 IPC 方式**，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信。

系列函数均存在与sys/shm.h路径下：用shmget开辟一块共享内存，用shmat将本进程连接(映射)上共享内存，然后可以在共享内存上根据权限进行读写，用shmctl同样可以对共享段进行操作其中设置IPC_RMID可以删除共享段当且仅当当前进程是最后一个连接此共享内存的进程，用shmdt可以直接进行删除，具体参数设置见APUE。

可使用命令 ipcs -m查看共享内存信息。

3.SOCKET

利用套接字通信，可以实现局域网不同主机间通信。当然也可以在本地创建两个端口进行通信。

详细见Linux~Kingjames/TCPIP文件夹下的client/server实现。

4.消息队列

整体操作和共享内存十分相似，有一点值得注意的是msgctl使用标志符IPC_RMID(含义见上共享内存)时因为消息队列并没有维护计数器会直接进行删除！

调用完msgrecv后就会发现原消息队列占用的字节数变为0，相当于被取走了！

5.信号量



6.信号



### 进程与线程的区别

1.调度：进程是资源管理的基本单位，线程是程序执行的基本单位。

2.切换：线程上下文切换比进程快得多。(因为各个线程用的都是同一块内存，即都是共享进程的那块内存。但两者同样需要做的事寄存器的切换。)

3.拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，只能访问隶属于进程的资源。

4.系统开销： 创建或撤销进程时，系统都要为之分配或回收系统资源，如内存空间，I/O设备等，OS所付出的开销显著大于在创建或撤销线程时的开销，进程切换的开销也远大于线程切换的开销。 





### 为什么需要线程






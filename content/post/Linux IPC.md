---
title: "Linux IPC"
date: 2024-02-23T13:28:32+08:00
draft: false
categories: [linux]
---

1. IPC（进程间通信），是 Linux 系统中不同进程间进行数据交换的机制。主要包括

   - 管道（Pipe）：主要用于父子进程通信，分为匿名管道和命名管道（FIFO）两种。
   - 信号（Signal）：一种异步通信方式，用于通知接收信号的进程某个事件已经发生。
   - 消息队列（Message Queue）：一种队列结构，可以存储要发送的消息。Linux 提供了 System V 和 POSIX 两种消息队列。
   - 共享内存（Shared Memory）：最快的 IPC 机制，允许多个进程共享一段内存区域。Linux 同样提供了 System V 和 POSIX 两种共享内存。
   - 信号量（Semaphore）：主要用于同步和互斥，防止多个进程同时访问共享资源。Linux 提供了 System V 和 POSIX 两种信号量。
   - 套接字（Socket）：可以用于不同机器之间的进程通信，也可以用于同一机器上的进程通信（Unix Domain Socket）。

   这些 IPC 机制各有优缺点，选择哪种机制取决于具体的应用需求。例如，如果需要跨网络通信，那么套接字可能是最好的选择。如果需要高速通信，那么共享内存可能是最好的选择。如果只是简单的父子进程通信，那么管道可能就足够了。

   例如在常用在命令行环境中的 `|`，就是匿名管道，它能够将一个命令的输出作为另一个命令的输入，从而使两个命令能够协同工作，如`ls -l | grep "txt"`、`cat file.txt | wc -l`。

   # 管道

   在 Linux 系统中，管道通常是匿名的，这意味着他们没有在文件系统中显示的路径，而是通过在内核中创建的特殊文件描述符来实现。

   创建匿名管道（Anonymous Pipes），需使用系统调用 `int pipe(int fd[2])` ，这里表示创建一个匿名管道，并返回两个描述符。一个是管道的读取端描述符`fd[0]`，另一个是写入端描述符`fd[1]`。**匿名管道是特殊的文件，只存在于内存，不存在于文件系统中**。

   这里需要注意的是，在某个进程创建匿名管道，该管道的两个描述符在同一进程中，如何起到跨进程通信的作用？

   实际上，匿名管道用于具具有亲缘关系的进程之间的通信，当我们在父进程中调用 `fork()` 系统调用创建子进程并需要父子进程通信时，父进程和子进程可以通过共享的文件描述符进行通信。

   一般情况下，父进程关闭管道的读取端，子进程关闭管道的写入端。这样，父进程只写入，子进程只读取，避免了同时写或同时读的情况。

   通信完成后，父子进程应该关闭不再需要的文件描述符。关闭管道的写入端（读取端）会导致对应读取端（写入端）收到一个文件结束符，这样读取操作就会返回 0，表述数据已经全部读取完毕。

   ```c
   //匿名管道示例
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <string.h>
   
   #define BUFFER_SIZE 25
   #define READ_END 0
   #define WRITE_END 1
   
   int main() {
       char write_msg[BUFFER_SIZE] = "Hello, pipe!";
       char read_msg[BUFFER_SIZE];
       int fd[2];
       pid_t pid;
   
       // 创建管道
       if (pipe(fd) == -1) {
           fprintf(stderr, "Pipe failed");
           return 1;
       }
   
       // 创建子进程
       pid = fork();
   
       if (pid < 0) {
           fprintf(stderr, "Fork failed");
           return 1;
       }
   
       if (pid > 0) {  // 父进程
           close(fd[READ_END]);  // 关闭读取端
   
           // 写入数据到管道
           write(fd[WRITE_END], write_msg, strlen(write_msg) + 1);
           close(fd[WRITE_END]);  // 关闭写入端
           printf("Parent process wrote to the pipe: %s\n", write_msg);
       } else {  // 子进程
           close(fd[WRITE_END]);  // 关闭写入端
   
           // 从管道中读取数据
           read(fd[READ_END], read_msg, BUFFER_SIZE);
           printf("Child process read from the pipe: %s\n", read_msg);
           close(fd[READ_END]);  // 关闭读取端
       }
   
       return 0;
   }
   ```

   对于命名管道（Named Pipes，或 FIFOs），它是一种具有名称的特殊文件，存储在文件系统中，可以通过文件系统路径进行访问，允许无关进程之间进行通信，通过 `mkfifo` 命令或 `mkfifo()` 函数创建。

   一个进程可以向命名管道中写入数据，另一个进程可以从命名管道中读取数据。其读写操作和普通文件的读写操作类似，可以使用文件 I/O 函数（`opoen()`/`read()/`/`write()`/`close()`）来进行。

   命名管道区别于匿名管道的主要特征在于持久性，即使创建它的进程终止，命名管道仍然存在于文件系统中，直到显示地被删除。

   所谓管道，其实就是内核里面的一串缓存。

   当进程向管道写入数据时，内核会将数据缓存到管道的内部缓冲区。如果管道的缓冲区已满，写入操作会被阻塞，直到有足够的空间可以写入数据。类似的，当进程从管道读取数据时，内核会从管道的内部缓冲区中读取数据，并将其提供给进程。如果管道的缓冲区为空，读取操作会被阻塞，直到有数据可供读取。

   管道的读写操作通常会涉及到进程的同步和通知机制。内核需要保证在多个进程同时访问管道时，数据的读写操作是安全和正确的。为了实现进程的同步和通知，Linux 内核可能会使用信号量、锁或者其它同步机制来保护管道的读写操作，以确保数据的一致性和正确性。

   因此，管道这种通信方式效率低，不适合进程间频繁地交换数据。

   总的来说，管道作为一种基本的进程间通信机制，在使用时虽然简单方便，但也存在一些弊端，主要体现在：

   1. 单向通信：这意味着如果需要双向通信，需要创建两个管道，增加复杂度和开销；
   2. 容量限制：管道有一个固定容量限制，一般取决于操作系统的设置。一旦缓冲区达到容量上限，写入操作将被阻塞，可能导致进程间通信的延迟或死锁；
   3. 阻塞式读写：管道空或满时，读写操作会被阻塞，这可能导致进程被挂起，降低系统的响应速度；
   4. 没有数据完整性保证：管道进提供数据流传输，对完整性和可靠性没有保证，通常需要在应用层面实现额外的机制，如校验和、确认应答等；
   5. 无法在网络上使用：管道只能在同一台主机的进程间通信，无法用于网络通信。

   # 消息队列

   对于更高效、频繁传递数据的场景，可以选择使用消息队列。一般在 Linux 中，消息队列是保存在内核中的消息链表。

   数据传输单位是用户自定义的数据，收发双方需要在传输前约定好消息体的数据类型，每个消息体都是固定大小的存储块，不像管道是无格式的字节流数据。

   生命周期跟随系统内核，如果没有主动释放消息队列或没有关闭操作系统，消息队列会一直存在。

   虽然消息队列可以便捷地在进程间传递数据，但它**不适合大数据传输**，因为在内核中每个消息体都有一个最大长度限制，同时所有队列所包含的全部消息体的总长度也有上限。内核中有两个宏定义，`MSGMAX` 和 `MSGMNB`，以字节为单位，分别定义了一条消息的最大长度和一个队列的最大长度。

   另外在运行效率上，**通信过程存在用户态和内核态之间的数据拷贝开销**，因为进程写入数据到内核中的消息队列时，会发生从用户态拷贝数据到内核态的过程，反之亦然。

   因此，基于以上两点，消息队列更适合应用于小规模、低频率数据传输，比如任务队列（每个消息代表一个待处理的任务）、日志数据、状态更新、请求响应、数据流（数据在多个进程之间流式传输，每个消息包含流的一部分）等。

   主要应用场景：

   1. **异步处理**：消息队列可以用于实现异步处理，这样一来，一个进程可以将任务放入队列中，然后立即返回，而不需要等待任务完成。这对于需要长时间运行的任务特别有用，例如大数据计算或复杂的文件操作。
   2. **负载均衡**：如果有大量的任务需要处理，可以使用消息队列来分发这些任务。每个工作进程可以从队列中取出一个任务，处理它，然后再取出下一个任务。这样可以确保所有的工作进程都保持忙碌，而且可以根据需要添加更多的工作进程。
   3. **解耦**：消息队列可以用于解耦系统的不同部分。这意味着一个部分的改变不会直接影响到其他部分。例如，一个服务可以发布消息到队列，而不需要知道哪些消费者会接收和处理这些消息。
   4. **容错**：如果处理消息的进程失败，消息可以留在队列中，然后由另一个进程重新处理。这可以提高系统的可靠性。
   5. **日志记录**：消息队列可以用于收集系统的日志信息。应用程序可以将日志消息发送到队列，然后由专门的日志服务从队列中读取和处理这些消息。

   消息队列的常用系统调用接口：`mq_open()` / `mq_send()` / `mq_receive()` / `mq_close()` / `mq_unlink()`

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <fcntl.h>
   #include <sys/stat.h>
   #include <mqueue.h>
   
   #define QUEUE_NAME "/my_message_queue"
   #define MAX_MSG_SIZE 256
   #define MAX_MSG_COUNT 10
   
   int main() {
       mqd_t mq;
       struct mq_attr attr;
       char buffer[MAX_MSG_SIZE + 1];
       int msg_flags = O_CREAT | O_RDWR;
       mode_t mode = S_IRUSR | S_IWUSR; // Permissions for the message queue
   
       // Set up the attributes of the message queue
       attr.mq_maxmsg = MAX_MSG_COUNT;
       attr.mq_msgsize = MAX_MSG_SIZE;
       attr.mq_flags = 0;
   
       // Create the message queue
       mq = mq_open(QUEUE_NAME, msg_flags, mode, &attr);
       if (mq == (mqd_t)-1) {
           perror("mq_open");
           exit(1);
       }
   
       printf("Message queue created.\n");
   
       // Send a message to the queue
       printf("Enter message to send: ");
       fgets(buffer, MAX_MSG_SIZE, stdin);
   
       if (mq_send(mq, buffer, strlen(buffer), 0) == -1) {
           perror("mq_send");
           exit(1);
       }
   
       printf("Message sent.\n");
   
       // Receive a message from the queue
       ssize_t bytes_read = mq_receive(mq, buffer, MAX_MSG_SIZE, NULL);
       if (bytes_read == -1) {
           perror("mq_receive");
           exit(1);
       }
   
       buffer[bytes_read] = '\0';
       printf("Received message: %s\n", buffer);
   
       // Close the message queue
       mq_close(mq);
   
       // Remove the message queue
       mq_unlink(QUEUE_NAME);
   
       return 0;
   }
   ```

   # 共享内存

   现代操作系统的内存管理，采用虚拟内存技术，即每个进程有自己独立的虚拟内存空间，不同进程的虚拟内存映射到不同的物理内存中。所以，即使进程 A 和进程 B 的虚拟地址一样，其访问的物理内存地址也是不同的，对于数据的增删改查互不影响。

   而共享内存则是**将相同的虚拟地址空间映射到同一的物理内存中**，以实现不同进程在同一内存区域共同增删改查，从而无需拷贝、无需进行用户态内核态切换，大大提升了 IPC 速度，且相对于消息队列而言，其传递数据的大小仅受限于系统的物理内存大小，处理大数据的能力远大于消息队列。常用于数据库系统、图像处理和科学计算等。

   使用共享内存作为 IPC 方式，需要重点关注**同步**和**安全**问题。多个进程同时访问共享内存可能导致数据不一致，因此需要使用同步机制（如信号量）来确保数据一致性。另外，任何可以访问共享内存的进程都可以修改数据，容易导致安全问题（如意外或恶意篡改）。因此在创建共享内存时，可以通过设置权限进行访问控制，读写数据时进行适当的完整性校验，以增强安全性。

   这里提供一个简单示例，首先是创建和向共享内存写数据的进程：

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <fcntl.h>
   #include <sys/shm.h>
   #include <sys/stat.h>
   #include <sys/mman.h>
   #include <semaphore.h>
   
   int main()
   {
       const int SIZE = 4096;
       const char *name = "OS";
       const char *message_0 = "Hello";
       const char *message_1 = "World!";
       const char *sem_name = "sem";
   
       int shm_fd;
       void *ptr;
       sem_t *sem;
   
       /* 创建共享内存对象 */
       shm_fd = shm_open(name, O_CREAT | O_RDWR, 0666);
       if (shm_fd == -1) {
           perror("In shm_open");
           exit(1);
       }
   
       /* 配置共享内存对象的大小 */
       if (ftruncate(shm_fd, SIZE) == -1) {
           perror("In ftruncate");
           exit(1);
       }
   
       /* 将共享内存对象映射到内存 */
       ptr = mmap(0, SIZE, PROT_WRITE, MAP_SHARED, shm_fd, 0);
       if (ptr == MAP_FAILED) {
           perror("In mmap");
           exit(1);
       }
   
       /* 创建信号量 */
       sem = sem_open(sem_name, O_CREAT, 0666, 0);
       if (sem == SEM_FAILED) {
           perror("In sem_open");
           exit(1);
       }
   
       /* 将数据写入共享内存对象 */
       sprintf(ptr, "%s", message_0);
       ptr += strlen(message_0);
       sprintf(ptr, "%s", message_1);
       ptr += strlen(message_1);
   
       /* 通过信号量通知其他进程可以读取数据 */
       if (sem_post(sem) == -1) {
           perror("In sem_post");
           exit(1);
       }
   
       return 0;
   }
   ```

   然后是读数据和删除共享内存的进程：

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <fcntl.h>
   #include <sys/shm.h>
   #include <sys/stat.h>
   #include <sys/mman.h>
   #include <semaphore.h>
   
   int main()
   {
       const int SIZE = 4096;
       const char *name = "OS";
       const char *sem_name = "sem";
   
       int shm_fd;
       void *ptr;
       sem_t *sem;
   
       /* 打开共享内存对象 */
       shm_fd = shm_open(name, O_RDONLY, 0666);
       if (shm_fd == -1) {
           perror("In shm_open");
           exit(1);
       }
   
       /* 将共享内存对象映射到内存 */
       ptr = mmap(0, SIZE, PROT_READ, MAP_SHARED, shm_fd, 0);
       if (ptr == MAP_FAILED) {
           perror("In mmap");
           exit(1);
       }
   
       /* 打开信号量 */
       sem = sem_open(sem_name, 0);
       if (sem == SEM_FAILED) {
           perror("In sem_open");
           exit(1);
       }
   
       /* 等待信号量 */
       if (sem_wait(sem) == -1) {
           perror("In sem_wait");
           exit(1);
       }
   
       /* 从共享内存中读取数据 */
       printf("%s\n", (char *)ptr);
   
       /* 删除共享内存对象 */
       if (shm_unlink(name) == -1) {
           perror("In shm_unlink");
           exit(1);
       }
   
       /* 删除信号量 */
       if (sem_unlink(sem_name) == -1) {
           perror("In sem_unlink");
           exit(1);
       }
   
       return 0;
   }
   ```

   在这个示例中，使用了一个信号量来同步两个进程的操作。写入进程在写入数据后通过信号量通知读取进程可以读取数据。读取进程在读取数据前等待信号量。另外，添加了一些错误处理和安全性检查，例如检查 `shm_open`、`ftruncate`、`mmap`、`sem_open`、`sem_post`、`sem_wait`、`shm_unlink` 和 `sem_unlink` 的返回值，并在出错时打印错误消息并退出。

   这里可以明显看出，相较于前两种 IPC 机制，要想获得更高的效率，就要关注更多的细节，这里有几个要点需要关注：

   1. **同步访问**：当多个进程访问共享内存时，必须使用某种同步机制（如信号量或互斥锁）来确保数据的一致性和完整性。
   2. **清理共享内存**：当所有进程都不再需要共享内存时，应该删除它以释放系统资源。在 Linux 中，可以使用 `shmctl` 函数（对于 System V 共享内存）或 `shm_unlink` 函数（对于 POSIX 共享内存）来删除共享内存。
   3. **错误处理**：在使用共享内存的过程中，应该检查所有可能的错误条件，并适当地处理错误。
   4. **避免使用过大的共享内存区域**：虽然共享内存是一种高效的 IPC 机制，但是使用过大的共享内存区域可能会消耗大量的系统资源，并可能导致性能问题。
   5. **使用适当的数据结构**：在共享内存中，应该使用适合并发访问的数据结构。例如，如果多个进程需要同时读写一个数据结构，那么应该使用一个可以支持并发访问的数据结构，如链表或哈希表。
   6. **避免使用指针**：在共享内存中，不应该使用指向非共享内存区域的指针，因为这些指针在其他进程中可能无效。
   7. **安全性**：共享内存可以被任何具有适当权限的进程访问，因此应该考虑数据的安全性。如果需要，可以使用加密和解密机制来保护数据。

   # 信号量

   在上面的示例代码中，除了共享内存，还引入了另一种 IPC 机制，信号量（Semaphore），以解决共享内存的访问同步问题。

   **信号量其实是一个整形的计数器，主要用于实现进程间的互斥与同步，而不是用于缓存进程间通信的数据。**

   其在 Linux 内核中的定义为：

   ```c
   struct semaphore {
       raw_spinlock_t      lock;
       unsigned int        count;
       struct list_head    wait_list;
   };
   ```

   其中：

   - `lock` 是一个原始的自旋锁，用于保护信号量的数据结构。
   - `count` 是信号量的当前值。当一个进程调用 `down` 或 `sem_wait` 函数试图获取信号量时，如果 `count` 大于 0，那么 `count` 将减 1，进程将继续执行。如果 `count` 等于 0，那么进程将被阻塞，直到 `count` 变为非 0.
   - `wait_list` 是一个链表，包含了所有等待这个信号量的进程。

   控制信号量的方式有两种原子操作，P 操作和 V 操作。

   - P 操作，也被称为 ”wait” 或 “down” 操作。当一个进程需要访问共享资源时，它会执行 P 操作，在 P 操作中，信号量的值会减 1。
   - V 操作，也被称为 “signal” 或 ”up“ 操作。当一个进程完成对共享资源的访问后，它会执行 V 操作，在 V 操作中，信号量的值会加 1。

   > P 操作和 V 操作这两种术语源自荷兰语，P 操作来自 “Proberen”，意为“尝试”，V 操作来自 “Verhogen”， 意为“增加“。
   >
   > 原子操作是指在多线程环境中，一个不可被中断的操作，也就是说这个操作要么完全执行，要么完全不执行，不会出现执行一半的情况。在执行过程中，不会被其他线程打断。

   既然信号量相当于一个计数器，那么当信号量被初始化为 1 时，先到来的 P 操作进程先占用资源，后来的 P 操作都得阻塞等待，保证共享内存在任何时刻只有一个进程在访问，**此时的信号量是作为互斥信号量**。那么如果初始化为 0 呢？

   通常在多进程里，每个进程各自独立运行，先后顺序不可知，当我们希望多个进程能合作实现一个任务时，可以将信号量初始化为 0。例如，进程 A 是数据生产者，进程 B 是数据消费者，显然 B 依赖 A，此时若 B 比 A 先执行，当其执行到 P 操作时，由于信号量为 0，B 阻塞，直到 A 执行了 V 操作，相当于唤醒了阻塞在 P 操作的 B 进程，**此时的信号量是作为同步信号量**，保证进程 A 的 V 操作先于进程 B 的 P 操作执行。

   # POSIX & System V

   POSIX (Portable Operating System Interface) 和 System V 是两种不同的 Unix 标准。它们在很多方面都有所不同，包括它们提供的进程间通信（IPC）机制。

   以下是它们在 IPC 方面的一些主要区别：

   1. **消息队列**：System V 提供了 `msgget`，`msgsnd`，`msgrcv` 和 `msgctl` 系统调用来操作消息队列。而 POSIX 提供了 `mq_open`，`mq_send`，`mq_receive`，`mq_close` 和 `mq_unlink` 函数来操作消息队列。POSIX 消息队列支持优先级，而 System V 消息队列支持消息类型。
   2. **信号量**：System V 提供了 `semget`，`semop` 和 `semctl` 系统调用来操作信号量。而 POSIX 提供了 `sem_open`，`sem_wait`，`sem_post`，`sem_close` 和 `sem_unlink` 函数来操作信号量。POSIX 信号量可以在进程间或线程间使用，而 System V 信号量主要用于进程间。
   3. **共享内存**：System V 提供了 `shmget`，`shmat`，`shmdt` 和 `shmctl` 系统调用来操作共享内存。而 POSIX 提供了 `shm_open`，`mmap`，`munmap`，`shm_unlink` 函数来操作共享内存。
   4. **命名和生命周期**：System V IPC 对象通过 key 和 id 来标识，需要显式删除，否则会一直存在。而 POSIX IPC 对象通过名字来标识，可以设置自动删除，当最后一个引用关闭后，对象会被自动删除。
   5. **接口**：POSIX 的接口通常更简单，更易于使用，而 System V 的接口则更复杂，提供了更多的选项和功能。

   至于选择哪种类型的 IPC 机制，主要取决于具体的应用需求和开发者的偏好。

   一般情况下，我们可能会偏向于选择 POSIX IPC 机制，主要原因在于：

   1. **接口一致性**：POSIX IPC 机制的接口在不同的 Unix-like 系统（包括 Linux）之间保持一致，这有助于提高代码的可移植性。
   2. **更现代的特性**：相比于 System V，POSIX IPC 机制提供了一些更现代的特性，例如更好的线程支持、更灵活的命名机制以及更好的资源管理等。

   然而，这并不意味着 System V IPC 机制没有用武之地。在一些特定的场景下，System V IPC 机制可能会是更好的选择，例如：

   1. **更丰富的功能**：System V IPC 机制提供了一些 POSIX IPC 机制所不具备的功能，例如消息队列中的消息类型、信号量操作的原子性等。
   2. **更广泛的兼容性**：由于 System V IPC 机制的历史更为悠久，因此它在一些较旧的系统中可能会得到更好的支持。

   总的来说，Linux 系统并没有默认采取 POSIX 还是 System V，而是提供了这两种 IPC 机制供开发者根据具体的需求和偏好进行选择。
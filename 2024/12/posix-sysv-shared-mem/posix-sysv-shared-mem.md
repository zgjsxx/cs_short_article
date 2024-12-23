# 探索共享内存：POSIX vs SYSV，哪个更适合你的应用？

共享内存是一种允许多个进程直接访问同一块物理内存区域的技术，能够极大地提高进程间通信的效率。POSIX和SYSV共享内存都是实现共享内存的标准，但它们在API设计、功能、灵活性和兼容性等方面存在一些重要的区别。下面我们将详细探讨它们的不同之处，并介绍各自的使用方法。

## 1. **共享内存的概述**

共享内存（Shared Memory）是进程间通信的一种机制，允许多个进程直接访问同一块内存区域。这种方式通常比管道、消息队列等其他 IPC 机制更高效，因为它避免了数据在进程间的拷贝，而是直接让进程访问物理内存。

在 UNIX 中，进程通过共享内存区来共享数据，通常需要配合互斥锁（mutex）或者信号量（semaphore）来保证数据一致性，避免数据竞争。

## 2. **SYSV（System V）共享内存**

System V 是早期 UNIX 系统中的一组标准接口，后来被许多 UNIX 和类 UNIX 系统采用，包括 Linux。SYSV 共享内存机制基于内核的一个全局标识符来管理共享内存段。

### 2.1. **创建和操作**

在 SYSV 共享内存中，共享内存区通过 `shmget()` 系统调用来创建。其他进程则通过 `shmat()` 来附加共享内存，使用完毕后通过 `shmdt()` 来分离。

- **`shmget()`**：创建共享内存段。
  ```c
  int shmget(key_t key, size_t size, int shmflg);
  ```
  - `key`: 一个唯一的标识符（通常由 `ftok()` 或手动指定）。
  - `size`: 要创建的共享内存区的大小。
  - `shmflg`: 控制共享内存段的权限和标志。

- **`shmat()`**：将共享内存段映射到当前进程的地址空间。
  ```c
  void* shmat(int shmid, const void* shmaddr, int shmflg);
  ```
  - `shmid`: 共享内存段的标识符（由 `shmget()` 返回）。
  - `shmaddr`: 映射地址，通常设置为 `NULL`，让系统选择合适的地址。
  - `shmflg`: 映射标志（通常设置为 0）。

- **`shmdt()`**：将共享内存段从进程的地址空间分离。
  ```c
  int shmdt(const void* shmaddr);
  ```

- **`shmctl()`**：对共享内存段进行控制（如删除共享内存段、获取信息等）。
  ```c
  int shmctl(int shmid, int cmd, struct shmid_ds* buf);
  ```
  - `cmd`: 操作命令（如 `IPC_RMID` 删除共享内存）。
  - `shmid_ds`: 存储共享内存状态的结构体。

### 2.2. **优缺点**
- **优点**：
  - 高效的进程间通信，不需要进行数据拷贝。
  - 灵活，可以在多个进程之间共享数据。
- **缺点**：
  - 管理复杂：需要手动管理内存块的大小、权限以及删除共享内存等。
  - 安全性较差：没有直接的内存保护机制，容易出现内存泄漏或数据冲突。

下面是一个对共享内存进行读写的例子：

**示例 1：写入共享内存的程序**

这个程序会创建一个共享内存对象，并将数据写入其中。

```cpp
// writer.c
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

#define SHM_KEY 1234  // 共享内存的键值
#define SIZE 4096     // 共享内存的大小

int main() {
    // 创建共享内存段
    int shmid = shmget(SHM_KEY, SIZE, IPC_CREAT | 0666);  // 创建共享内存
    if (shmid == -1) {
        perror("shmget failed");
        return 1;
    }

    // 将共享内存附加到进程的地址空间
    void *shm_ptr = shmat(shmid, NULL, 0);
    if (shm_ptr == (void *)-1) {
        perror("shmat failed");
        return 1;
    }

    // 写入数据到共享内存
    const char *message = "Hello from writer process!";
    strcpy(shm_ptr, message);

    printf("Writer: Written to shared memory: %s\n", message);

    // 分离共享内存
    if (shmdt(shm_ptr) == -1) {
        perror("shmdt failed");
        return 1;
    }

    return 0;
}
```

**示例 2：读取共享内存的程序**

这个程序将从共享内存中读取数据并显示出来。

```cpp
// reader.c
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <string.h>

#define SHM_KEY 1234  // 共享内存的键值
#define SIZE 4096     // 共享内存的大小

int main() {
    // 获取已存在的共享内存段
    int shmid = shmget(SHM_KEY, SIZE, 0666);  // 获取共享内存
    if (shmid == -1) {
        perror("shmget failed");
        return 1;
    }

    // 将共享内存附加到进程的地址空间
    void *shm_ptr = shmat(shmid, NULL, 0);
    if (shm_ptr == (void *)-1) {
        perror("shmat failed");
        return 1;
    }

    // 读取共享内存中的数据
    printf("Reader: Read from shared memory: %s\n", (char *)shm_ptr);

    // 分离共享内存
    if (shmdt(shm_ptr) == -1) {
        perror("shmdt failed");
        return 1;
    }

    return 0;
}
```

步骤解释
writer.c 程序：
- 使用 shmget() 创建或获取一个共享内存段。
- 使用 shmat() 将共享内存段附加到进程的地址空间。
- 使用 strcpy() 将字符串 "Hello from writer process!" 写入共享内存。
- 完成后，使用 shmdt() 分离共享内存。

reader.c 程序：
- 使用 shmget() 获取共享内存段。
- 使用 shmat() 将共享内存段附加到进程的地址空间。
- 从共享内存读取数据，并输出到终端。
- 完成后，使用 shmdt() 分离共享内存。

编译和运行
编译代码：

```shell
g++ -o writer writer.cpp
g++ -o reader reader.cpp
```

运行程序：

首先运行 writer 程序来写入共享内存：

```shell
./writer
```

它会将字符串 "Hello from writer process!" 写入共享内存。

然后运行 reader 程序来读取共享内存中的数据：

```shell
./reader
```

输出将会是：

```shell
Reader: Read from shared memory: Hello from writer process!
```

sysv的共享内存可以使用工具ipcs进行查看:

```shell
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x000004d2 0          root       666        4096       0
```

## 3. **POSIX 共享内存**

POSIX 是一个由 IEEE 提出的标准接口，旨在提升不同操作系统间的兼容性。POSIX 共享内存（`shm_open()` 和 `mmap()`）提供了一种基于文件的共享内存机制，它通过虚拟文件系统（VFS）来管理共享内存。

### 3.1. **创建和操作**

- **`shm_open()`**：创建或打开一个共享内存对象。
  ```c
  int shm_open(const char* name, int oflag, mode_t mode);
  ```
  - `name`: 共享内存对象的名称，通常以 `/` 开头（如 `/myshm`）。
  - `oflag`: 文件访问标志，通常为 `O_CREAT` 或 `O_RDWR`。
  - `mode`: 权限设置。

- **`ftruncate()`**：设置共享内存的大小。
  ```c
  int ftruncate(int fd, off_t length);
  ```

- **`mmap()`**：将共享内存映射到进程的地址空间。
  ```c
  void* mmap(void* addr, size_t length, int prot, int flags, int fd, off_t offset);
  ```
  - `addr`: 映射到的地址（通常为 `NULL`，让系统选择地址）。
  - `length`: 映射的大小。
  - `prot`: 保护标志，通常为 `PROT_READ` 或 `PROT_WRITE`。
  - `flags`: 映射标志（`MAP_SHARED` 表示共享内存）。
  - `fd`: 文件描述符，指向共享内存对象。
  - `offset`: 从共享内存对象的哪个位置开始映射。

- **`munmap()`**：解除映射。
  ```c
  int munmap(void* addr, size_t length);
  ```

- **`shm_unlink()`**：删除共享内存对象。
  ```c
  int shm_unlink(const char* name);
  ```

### 3.2. **优缺点**
- **优点**：
  - 更加现代和灵活的接口，支持命名和无命名共享内存。
  - 支持文件描述符和虚拟内存映射，允许内存共享和文件操作结合。
  - 内存管理相对简单，操作符可以基于文件系统进行。
- **缺点**：
  - 由于共享内存是通过文件描述符管理的，所以在跨进程访问时，需要确保所有进程都拥有适当的文件描述符。
  - 对于不熟悉文件系统的开发人员来说，接口相对复杂。

示例 1：写入共享内存的程序
这个程序会创建一个共享内存对象，并将数据写入其中。

```cpp
// writer.c
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

#define SHM_NAME "/my_shm"
#define SIZE 4096

int main() {
    // 创建共享内存对象，并以可读写模式打开
    int fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (fd == -1) {
        perror("shm_open failed");
        return 1;
    }

    // 调整共享内存大小
    if (ftruncate(fd, SIZE) == -1) {
        perror("ftruncate failed");
        return 1;
    }

    // 映射共享内存到进程的地址空间
    void *shm_ptr = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (shm_ptr == MAP_FAILED) {
        perror("mmap failed");
        return 1;
    }

    // 将数据写入共享内存
    const char *message = "Hello from writer process!";
    strcpy(shm_ptr, message);

    printf("Writer: Written to shared memory: %s\n", message);

    // 解除映射
    munmap(shm_ptr, SIZE);

    // 关闭共享内存
    close(fd);

    return 0;
}
```

示例 2：读取共享内存的程序
这个程序将从共享内存中读取数据并显示出来。

```cpp
// reader.c
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

#define SHM_NAME "/my_shm"
#define SIZE 4096

int main() {
    // 打开已存在的共享内存对象
    int fd = shm_open(SHM_NAME, O_RDWR, 0666);
    if (fd == -1) {
        perror("shm_open failed");
        return 1;
    }

    // 映射共享内存到进程的地址空间
    void *shm_ptr = mmap(NULL, SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (shm_ptr == MAP_FAILED) {
        perror("mmap failed");
        return 1;
    }

    // 从共享内存中读取数据
    printf("Reader: Read from shared memory: %s\n", (char *)shm_ptr);

    // 解除映射
    munmap(shm_ptr, SIZE);

    // 关闭共享内存
    close(fd);

    return 0;
}
```

步骤解释:

writer.c 程序：
- 使用 shm_open 创建或打开一个名为 /my_shm 的共享内存对象，并将其映射到进程的地址空间。
- 使用 strcpy 将 "Hello from writer process!" 字符串写入共享内存。
- 映射结束后，调用 munmap 来解除映射，并关闭文件描述符。

reader.c 程序：
- 使用 shm_open 打开已有的共享内存对象 /my_shm。
- 将共享内存映射到进程的地址空间，并读取共享内存中的数据。
- 输出读取的内容，并解除映射，关闭文件描述符。

编译和运行
编译代码：

编译这两个程序时，使用如下命令：

```shell
g++ -o writer writer.cpp 
g++ -o reader reader.cpp
```

运行程序：

首先运行 writer 程序来写入共享内存。

```shell
./writer
```

它会在共享内存中写入字符串 "Hello from writer process!"。

然后运行 reader 程序来读取共享内存中的数据。

```shell
./reader
```

你会看到输出：

```shell
Reader: Read from shared memory: Hello from writer process!
```

## 4. **POSIX vs SYSV 共享内存的差异**

| 特性                  | POSIX 共享内存                              | SYSV 共享内存                              |
|-----------------------|--------------------------------------------|--------------------------------------------|
| **命名方式**           | 通过文件名（如 `/myshm`）进行标识              | 通过共享内存 ID（`shmget()` 的返回值）进行标识 |
| **创建方式**           | `shm_open()`                               | `shmget()`                                 |
| **内存映射**           | 使用 `mmap()` 将共享内存映射到进程地址空间       | 使用 `shmat()` 映射共享内存                  |
| **删除共享内存**       | 使用 `shm_unlink()` 删除                     | 使用 `shmctl()` 删除（通过 `IPC_RMID`）     |
| **跨平台兼容性**       | 支持 POSIX 标准，跨平台性好                     | 只适用于支持 System V 的操作系统（如 Linux）|
| **内存保护**           | 使用 `mmap()` 时可以指定访问权限（如只读、只写等）| `shmat()` 不支持直接控制访问权限             |
| **灵活性**             | 更灵活，支持更广泛的文件操作与内存映射结合      | 不支持文件描述符，功能更为单一               |
| **清理机制**           | 不会自动清理（需要手动调用 `shm_unlink()`）   | 需要手动删除共享内存（通过 `shmctl()`）      |

### 5. **总结**

- **SYSV 共享内存** 更加原始且底层，依赖内核分配的共享内存块，操作起来相对简单但灵活性较低，适用于老旧系统和传统应用。
- **POSIX 共享内存** 提供了更加现代和灵活的接口，能够通过文件系统进行共享内存管理，并支持内存映射和更精细的权限控制，适用于新的应用程序开发。

对于开发者来说，POSIX 共享内存在功能上通常更优越，尤其是在需要跨平台兼容和处理复杂内存映射的情况下；而 SYSV 共享内存则依然在某些较老的系统或特定的应用场景中得到了广泛应用。

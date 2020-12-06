title: '[C++] Eventfd & Epoll'
date: 2020-12-6 17:02
tags:
- eventfd
- epoll
categories:
- C++

## eventfd & epoll

实现跨线程的唤醒。一个线程往fd中写入uint64_t的数据唤醒另一个epoll_wait上的线程

```c++
#include <iostream>
#include <assert.h>
#include <poll.h>
#include <signal.h>
#include <sys/eventfd.h>
#include <unistd.h>
#include <string.h>
#include <thread>

static int s_efd = 0;

int createEventfd()
{
  int evtfd = eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);

  std::cout << "createEventfd() fd : " << evtfd << std::endl;

  if (evtfd < 0)
  {
    std::cout << "Failed in eventfd\n";
    abort();
  }

  return evtfd;
}

void testThread()
{
  int timeout = 0;
  while(timeout < 3) {
    sleep(1);
    timeout++;
  }

  uint64_t one = 1;
  ssize_t n = write(s_efd, &one, sizeof one);
  if(n != sizeof one)
  {
    std::cout << " writes " << n << " bytes instead of 8\n";
  }
}

int main()
{
  s_efd = createEventfd();

  fd_set rdset;
  FD_ZERO(&rdset);
  FD_SET(s_efd, &rdset);

  struct timeval timeout;
  timeout.tv_sec = 0;
  timeout.tv_usec = 500000;

  std::thread t(testThread);

  while(1)
  {
    if(select(s_efd + 1, &rdset, NULL, NULL, &timeout) == 0)
    {
      std::cout << "tick!!!!\n";
      timeout.tv_sec = 0;
      timeout.tv_usec = 500000;
      FD_SET(s_efd, &rdset);
        continue;
    }

    uint64_t one = 0;

    ssize_t n = read(s_efd, &one, sizeof one);
    if(n != sizeof one)
    {
      std::cout << " read " << n << " bytes instead of 8\n";
    }

    std::cout << " wakeup ！\n";

    break;
  }

  t.join();
  close(s_efd);

  return 0;
}
```


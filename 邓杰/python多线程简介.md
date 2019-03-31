# python多线程编程简介

### 概念解释

进程：进程就是一个执行中的程序，如打开的office软件，游戏等都是一个进程，每个进程都有自己的地址空间、内存、数据栈等。

线程：线程与进程相似，但都是在进程下执行，即有了进程才有线程，一个进程至少包括一个线程，一个进程中的各个线程共享同一片数据空间。

多线程：多线程即一个进程内有多个线程工作，可以同时干很多事，例如打开一个浏览器，可以同时进行下载资源、浏览网页等；用软件播放视频时就需要一个线程播放音频另一个线程播放视频，这也是多线程的体现。

### python3中的多线程实现
python3中提供了多个模块来支持多线程编程，包括_thread、threading和queue等。对于_thread和threading来说，一般选择使用threading就好，因为threading模块更加先进、对多线程支持更好。

首先用_thread实现多线程为例，直观感受多线程相比单线程的效率

单线程

    def download():
        print('start download at: ',ctime())
        sleep(4)
        print('download done at:',ctime())

    def browse():
        print('start browse at: ',ctime())
        sleep(6)
        print("browse done at: ",ctime())

    def main():
        print('starting at: ',ctime())
        download()
        browse()
        print('all done at: ',ctime())

    if __name__=='__main__':
        main()

运行结果： 

    starting at:  Sun Mar 31 10:34:30 2019
    start download at:  Sun Mar 31 10:34:30 2019
    download done at: Sun Mar 31 10:34:34 2019
    start browse at:  Sun Mar 31 10:34:34 2019
    browse done at:  Sun Mar 31 10:34:40 2019
    all done at:  Sun Mar 31 10:34:40 2019

可以看到单线程工作的时候，先进行下载，待下载完成后才进行浏览，总耗时为下载耗时和浏览耗时相加，而实际上下载的时候我们就可以浏览其他网页，总耗时应该等于两个流程中耗时较长的那一项。

多线程

    import _thread
    from time import ctime,sleep

    things=['download','browse']
    nsec=[6,4]

    def do(thing,nsec,lock):                        #事件的执行函数
        print('start '+thing+'at:',ctime())
        sleep(nsec)
        print(thing+'done at:',ctime())
        lock.release()                              #事件执行完以后释放锁

    def main():
        print('starting at:',ctime())
        locks=[]
        nthing=range(len(things))

        for i in nthing:                            #获得一个锁列表，每个事件有一个锁
            lock = _thread.allocate_lock()
            lock.acquire()
            locks.append(lock)
        
        for i in nthing:
            _thread.start_new_thread(do,(things[i],nsec[i],locks[i]))   #为每一个事件开启一个线程，第一个参数为函数，第二个参数为调用的函数的参数

        for i in nthing:
            while locks[i].locked():pass            #不断遍历锁列表，查看是否所有的锁都释放了
        
        print('all done at: ',ctime())

    if __name__=='__main__':
        main()

代码运行结果

    starting at: Sun Mar 31 11:22:16 2019
    start download at: Sun Mar 31 11:22:16 2019start browse at:
    Sun Mar 31 11:22:16 2019
    browse done at: Sun Mar 31 11:22:20 2019
    download done at: Sun Mar 31 11:22:22 2019
    all done at:  Sun Mar 31 11:22:22 2019

从结果可以看出来，下载和浏览两个事件同时进行，整个流程的总耗时等于下载的总耗时（下载所需时间较长）

这段代码中用到了锁，锁的作用就是用 '_thread.all.lock()'语句获得锁了以后直到release释放锁之前，这段代码只能有一个线程进行执行，锁可以用来确保多线程中对变量的更改，也可以用来同步各个线程执行情况，如上面这段代码中

    for i in nthing:
        while locks[i].locked():pass 

就是遍历锁列表中的所有锁，如果都释放了才执行下面的语句，都释放了则代表了所有事件执行完成可以结束主线程了，如果没有锁则会出现，下载和浏览都未完成却输出了'all done'语句的情况。

## 注意事项
python大部分情况执行环境是cpython,而cpython因为历史原因而有GIL，GIL的作用就是保证同一时间下，只会有一个线程被python解释器执行而其他线程被挂起，这并不能达到真正的并发所谓的多线程也只能是快速地在不同线程间切换。多线程更适合I/O密集型应用，因为I/O释放了GIL，可以有更多的并发，如果是计算密集型应用则应该采用多进程来提高并行性。
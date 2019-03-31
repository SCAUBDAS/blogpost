## python多进程简介

因为python的GIL的限制，很多时候多线程并不能实现真正的并发，所以要提高程序并行性，充分利用处理器多核优势，就需要使用多进程编程

### multiprocessing
利用python的multiprocessing模块产生子进程,其语法与threading相似

    from time import ctime,sleep
    from multiprocessing import Process

    def download():
        print('start download at: ',ctime())
        sleep(4)
        print('download done at:',ctime())

    def browse():
        print('start browse at: ',ctime())
        sleep(6)
        print("browse done at: ",ctime())

    def main():
        p1=Process(target=download,args=())
        p2=Process(target=browse,args=())
        p=[p1,p2]
        for i in range(2):
            p[i].start()
        for i in range(2):
            p[i].join()
        print('all done')

    if __name__=='__main__':
        main()

运行结果:

    start download at:  Sun Mar 31 15:54:41 2019
    start browse at:  Sun Mar 31 15:54:41 2019
    download done at: Sun Mar 31 15:54:45 2019
    browse done at:  Sun Mar 31 15:54:47 2019
    all done

和多线程的情况一样，下载和浏览同时进行，且总耗时取决于子进程中较慢的那一个。其中的join方法和lock的功能类似，进程调用了join方法后，程序会等待子进程结束以后再往下执行，这样保证了进程间的同步。

### queue
进程之间的通信，如交换数据等，可以通过queue模块实现

    from multiprocessing import Process,Queue
    import time,random
    def write(q):
        print('start write')
        for i in range(5):
            print('write %d to queue'%i)
            q.put(i)
            time.sleep(randint(1,4))


    def read(q):
        print('start read')
        while True:
            temp = q.get(True)
            print('read %d from queue.' % temp)
            

    if __name__=='__main__':
        
        q = Queue()
        w = Process(target=write, args=(q,))
        r = Process(target=read, args=(q,))
        w.start()
        r.start()
        w.join()
        r.terminate()
        print('all done')

运行结果：

    start read
    start write
    write 0 to queue
    read 0 from queue.
    write 1 to queue
    read 1 from queue.
    write 2 to queue
    read 2 from queue.
    write 3 to queue
    read 3 from queue.
    write 4 to queue
    read 4 from queue.
    all done

不同进程本来是拥有自己独立的内存，存储空间等资源的，通过queue模块则可以实现不同进程之间的数据交换，一些进程对数据进行写入，另一些进程进行数据的取出等操作。
.. title: C 的 Thread Pool
.. slug: c-thread-pool
.. date: 2013/06/07 21:43:28
.. tags: 
.. link: 
.. description: 

C 是一個什麼都要自己來的世界 ....
=========================================================================

最近被丟到 FreeBSD 跟 C 的世界裡面，沒有 Scala 的 Actor 可以用。
所以只好參考別人的 Thread Pool 來看一下，在 C 的世界裡面 Thread Pool 是怎麼實作的。

首先先去 StackOverFlow 中尋找是否有人問過類似的問題

`Existing threadpool C implementation`_

裡面有提到幾個 C Thread Pool 的實作範例與可參考的文件

1. threadpool-mbrossard_
2. threadpool-jmatthew_
3. cthreadpool_
4. C-Thread-Pool_

我這邊是直接應該是會以 threadpool-mbrossard 作為第一個研究的版本，因為他一直有在維護。
而且作者就是 `Existing threadpool C implementation`_ 的發文者，感覺他還蠻熱心的。

threadpool
--------------------------------------------------------------------------

**A simple C thread pool implementation**

Currently, the implementation:

* Works with pthreads only, but API is intentionally opaque to allow
  other implementations (Windows for instance).
* Starts all threads on creation of the thread pool.
* Reserves one task for signaling the queue is full.
* Stops and joins all worker threads on destroy.

.. figure:: http://upload.wikimedia.org/wikipedia/commons/0/0c/Thread_pool.svg

from wikipedia

首先 Thread Pool 要有的東西就是 job 或者是 task 讓 Thread 知道他們要做什麼事情。

.. code-block:: c

    typedef struct {
        void (*function)(void *);
        void *argument;
    } threadpool_task_t;

接下來就是 Thread Pool 本身，他必須存放所有的 Thread 與 Job Queue  

.. code-block:: c

    struct threadpool_t {
        pthread_mutex_t lock;
        pthread_cond_t notify;
        pthread_t *threads;
        threadpool_task_t *queue;
        int thread_count;
        int queue_size;
        int head;
        int tail;
        int count;
        int shutdown;
        int started;
    };

這邊他使用了一個 pthread_t 的 pointer 來紀錄所有的 Thread，簡單來說就是一個 pthread_t 的 array，而 head, tail 就是紀錄 array 的 offset。
threadpool_task_t 也是一樣的原理，真是出乎意料的簡單。

.. code-block:: c

    threadpool_t *threadpool_create(int thread_count, int queue_size, int flags)
    {
        threadpool_t *pool;
        int i;

        /* TODO: Check for negative or otherwise very big input parameters */

        if((pool = (threadpool_t *)malloc(sizeof(threadpool_t))) == NULL) {
            goto err;
        }

        /* Initialize */
        pool->thread_count = 0;
        pool->queue_size = queue_size;
        pool->head = pool->tail = pool->count = 0;
        pool->shutdown = pool->started = 0;

        /* Allocate thread and task queue */
        pool->threads = (pthread_t *)malloc(sizeof(pthread_t) * thread_count);
        pool->queue = (threadpool_task_t *)malloc
            (sizeof(threadpool_task_t) * queue_size);

        /* Initialize mutex and conditional variable first */
        if((pthread_mutex_init(&(pool->lock), NULL) != 0) ||
           (pthread_cond_init(&(pool->notify), NULL) != 0) ||
           (pool->threads == NULL) ||
           (pool->queue == NULL)) {
            goto err;
        }

        /* Start worker threads */
        for(i = 0; i < thread_count; i++) {
            if(pthread_create(&(pool->threads[i]), NULL,
                              threadpool_thread, (void*)pool) != 0) {
                threadpool_destroy(pool, 0);
                return NULL;
            }
            pool->thread_count++;
            pool->started++;
        }

        return pool;

     err:
        if(pool) {
            threadpool_free(pool);
        }
        return NULL;
    }

再來就是 Thread Pool 的建立，由於剛剛提到的他其實是使用一個 pthread array 與一個 job array 來存放所有的 thread 與 jobs。
因此需要在一開始的時候就決定 Thread Pool 與 Jobs 的最大數量。

.. code-block:: c

        /* Allocate thread and task queue */
        pool->threads = (pthread_t *) malloc(sizeof(pthread_t) * thread_count);
        pool->queue = (threadpool_task_t *) malloc(sizeof(threadpool_task_t) * queue_size);

而每個 Thread 要執行的 Function 是

.. code-block:: c

   static void *threadpool_thread(void *threadpool)
    {
        threadpool_t *pool = (threadpool_t *)threadpool;
        threadpool_task_t task;

        for(;;) {
            /* Lock must be taken to wait on conditional variable */
            pthread_mutex_lock(&(pool->lock));

            /* Wait on condition variable, check for spurious wakeups.
               When returning from pthread_cond_wait(), we own the lock. */
            while((pool->count == 0) && (!pool->shutdown)) {
                pthread_cond_wait(&(pool->notify), &(pool->lock));
            }

            if((pool->shutdown == immediate_shutdown) ||
               ((pool->shutdown == graceful_shutdown) &&
                (pool->count == 0))) {
                break;
            }

            /* Grab our task */
            task.function = pool->queue[pool->head].function;
            task.argument = pool->queue[pool->head].argument;
            pool->head += 1;
            pool->head = (pool->head == pool->queue_size) ? 0 : pool->head;
            pool->count -= 1;

            /* Unlock */
            pthread_mutex_unlock(&(pool->lock));

            /* Get to work */
            (*(task.function))(task.argument);
        }

        pool->started--;

        pthread_mutex_unlock(&(pool->lock));
        pthread_exit(NULL);
        return(NULL);
    }

在 **for(;;)** 裡面，Thread 第一件要做的事情就是去搶奪 lock
 
.. _threadpool-mbrossard: https://github.com/mbrossard/threadpool 
.. _threadpool-jmatthew: http://people.clarkson.edu/~jmatthew/cs644.archive/cs644.fa2001/proj/locksmith/code/ExampleTest/
.. _cthreadpool: http://sourceforge.net/projects/cthpool/
.. _C-Thread-Pool: https://github.com/Pithikos/C-Thread-Pool
.. _Existing threadpool C implementation: http://stackoverflow.com/questions/6297428/existing-threadpool-c-implementation

---
layout: post
comments: true
title:  "Semaphore"
excerpt: "A brief introduction to semaphore"
date:   2018-03-06 12:00:00
mathjax: false
---

Dijkstra invented semaphores, when studying the consumer/producer problem. Under the surface of the dry definition, we should always keep in mind the intuition behind semaphore and its relation to the bounded buffer problem.

A semaphore is an object with an integer value that can be manipulated with two methods: sem_wait() and sem_post(). 

    Initializing Code
    sem_t s;
    sem_init(&s, 0, 1)
    //0 indicates that the semaphore is shared between threads in the same process
    //1 indicates that the initial value of the associated integer is 1

When the second parameter is not zero, a semaphore can be used to synchronize access across different processes.



    int sem_wait(sem_t *s);
    // decrement the value of semaphore by one
    // wait if value of semaphore s is negative
    int sem_post(sem_t *s);
    // increment the value of semaphore by one
    // if there is one or more threads waiting, wake one

We can see that sem_wait() will return right away if the value of the semaphore was one or higher, or sem_wait() will suspend execution. Also notice that the value of the semaphore, when negative, is equal to the number of waiting threads. 



Semaphore as lock

    sem_init(&m, 0, 1);
    sem_wait(&m);
    	//critical section
    sem_post(&m)

The first thread call this piece of code is able to execute the critical section right away, meanwhile, change the semaphore to zero. The following threads have to wait on a queue. The size of the queue is the negation of the semaphore value.



Semaphore as conditional variable

    void *child(void *arg) {
        printf("child");
        sem_post(&s);
    }
    
    int main(int argc, char *argv[]) {
        sem_init(&s, 0, 0);
        printf("parent");
        pthread_t c;
        Pthread_create(c, NULL, child, NULL);
        sem_wait(&s);
        printf("prarent: end");
        return 0;
    }

Semaphore with initial value 0 is very similar to a conditional variable. But they are different subtly. When using a semaphore, there's no need to use a shared variable and a lock. When using a conditional variable, the reason why we need a lock and a shared variable is to prevent cond_signal() to be called before cond_wait(). 

But for a semaphore, even if we can sem_post() before sem_wait(), the system still work correctly. sem_post() will increase the semaphore value by one.



The Producer/ Consumer Problem

    sem fill, empty, lock;
    sem_init(lock, 0, 1)
    sem_init(fill, 0, 0);
    sem_init(empty,0,MAX)
    void producer(void *arg) {
        int loop = (int) arg;
        for (int i = 0; i < loops; i++) {
            sem_wait(&empty);    // empty is at first MAX meaning there are MAX empty chunks
            sem_wait(&lock);
            	put(i);
            sem_post(&lock);
            sem_post(&fill);
        }
    }
    
    void *consumer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++){
            sem_wait(&fill);
            sem_wait(&lock);
            	int temp = get();
            sem_post(&lock);
            sem_post(&empty);
            printf("%d\n", temp);
        }  
    }

The semaphore implementation has some subtle differences from the conditional variable implementation. At first, there is no shared variable, meaning no explicit counter to track the size of available empty space. Because a semaphore its self is able to record such information.

Meanwhile, there is no mechanism for a semaphore to release lock like what conditional variable can do.  So we have to put   



Reader-Writer Locks

Writer and reader problem is only slightly different from Producer/Consumer problem. The difference a reader will only read a buffered value, but not remove it.

    typedef struct _rwlock_t {
        sem_t readlock;
        sem_t writelock;
        int readers;
    } rw_lock_t;
    
    void rwlock_init(rw_lock_t *rw) {
        rw->readers = 0;
        sem_init(&rw->writelock,0,1);
        sem_init(&rw->readlock,0,1);
    }
    void rwlock_get_readlock(rw_lock_t *rw){
        sem_wait(&rw->readlock);           // rw is a shared resource
        rw->readers++;					 // multiply readers could access it
        if(rw->readers == 0)			 // so this critical section
            sem_wait(&rm->writelock)	  // should be enclosed by a readlock
        sem_post(&rw->readlock)
    }
    void rwlock_release_readlock(rw_lock_t *rw){
        sem_wait(&rw->readlock);
        rw->readers--;
        if(rw->readers == 0)
            sem_post(&rm->writelock)
        sem_post(&rw->readlock)
    }
    void rwlock_get_writelock(rw_lock_t *rw){
        sem_wait(&rw->writelock);
    }
    void rwlock_release_writelock(rw_lock_t *rw){
        sem_post(&rw->writelock);
    }



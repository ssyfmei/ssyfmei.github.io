---
layout: post
comments: true
title:  "Conditional Variable I"
excerpt: "A brief introduction on how to use Conditional Variable"
date:   2018-02-01 11:00:00
mathjax: false
---

Conditional Variable is a mechanism used by a thread to inform another thread to begin execution. For example, a main thread might need to wait a child to finish before continuing execution. A child thread can make use of conditional variable to signal the main thread that it is done.



A conditional variable has two methods, wait() and signal(). Calling wait() will put a thread into waiting state. Calling signal() will wake up another waiting thread. A sample code is like this:

    int done = 0;
    pthread_mutex_t mlock = PTHREAD_MUTEX_INITIALIZER;
    Pthread_cond_t  cv    = PTHREAD_COND_INITIALIZER;
    
    void thr_exit() {
        Pthread_mutex_lock(&mlock);
        done = 1;
        Pthread_cond_signal(&cv);
        Pthread_mutex_unlock(&mlock);
    }
    void *another_thread(void *arg) {
        printf("anthor_thread_running\n");
        thr_exit();
    }
    void thr_join() {
        Pthread_mutex_lock(&mlock);
        while(done == 0)
        	Pthread_cond_wait(&cv, &mlock);
        Pthread_mutex_unlock(&mlock);
    }
    int main(int argc, char *argv[]) {
        printf("main thread: begin\n");
        pthread_t p;
        Pthread_create(&p, NULL, another_thread, NULL); // start a child thread
        thr_join();
        printf("main thread: end\n");
        return 0;
    }



There are several things need attention:

1. Why do we need a shared variable done?
2. Why wait() method take a lock parameter?
3. Why not just to use a shared variable?



Let's first consider the first question. If we get rid of the shared variable, the code will be like

    void thr_exit() {
        Pthread_mutex_lock(&mlock);
        Pthread_cond_signal(&cv);
        Pthread_mutex_unlock(&mlock);
    }
    void thr_join() {
        Pthread_mutex_lock(&mlock);
        Pthread_cond_wait(&cv, &mlock);
        Pthread_mutex_unlock(&mlock);
    }

Unfortunately, this code is broken. Consider the case that the child thread is scheduled before the main thread, the signal() function will be called before the wait() function. Thus when the main thread calls the wait() function, no thread is going to wake it up and it will sleep forever.



Let's consider the second question, why the wait function need the lock as a parameter?

 The answer is that the main thread need to release the lock  when it is put to sleep, otherwise the child thread is not able to acquire the lock and call signal().



Naturally, then, we would ask why do we need the lock at the first place? Isn't a shared variable enough? Let us consider a lock-free implementation.

    void thr_exit() {
        done = 1;
        Pthread_cond_signal(&cv);
    }
    
    void thr_join() {
        while(done == 0)
        	Pthread_cond_wait(&cv);
    }

The existence of lock guarantees that a sequence of commands protected by lock is atomic.  Without lock, consider the case where a system interruption happens when the first line of join() just gets execution,  the system switches to execute the child thread and call the signal() before the main thread call wait(). Again, the main thread will sleep forever.



A final issue is that why are we using a while statement instead of a if statement which is more natural. 

    void thr_join() {
        if(done == 0)
        	Pthread_cond_wait(&cv);
    }

A short answer is the while statement implementation is more robust. Even if the main thread is accidentally waked up,  it will sleep again if the incident the main thread is waiting for is not done. 

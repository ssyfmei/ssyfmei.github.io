---
layout: post
comments: true
title:  "Conditional Variable II"
excerpt: "The comsumer/producer problem"
date:   2018-02-08 11:00:00
mathjax: false
---

Conditional Variable II



Let us use conditional variable to solve the classic producer/consumer problem.



The producer/consumer problem occurs in many real-world applications. For example, in a multi-thread web server, one or more producer threads put HTTP requests into a pool and one or more consumer threads take HTTP requests from the pool and process them.

This problem also appears when we pip the output of one program into another one. 

    grep keyword foo.txt | wc

In this scenario, the output stream of grep is redirected to the input stream of wc. Under the surface, the output of grep is put in an in-kernel bounded buffer and wc acts as a consumer.



solution 1

    int buffer;
    int count = 0;
    
    void put(int value) {
        assert(count == 0);
        count = 1;
        buffer = value;
    }
    int get() {
        assert(count == 1);
        count = 0;
        return buffer;
    }

									get() and put() for a buffer of size 

    void *producer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++)
            put(i);
    }
    void *consumer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++)
            printf("%d\n", get());
    }

This naive solution is obviously broken. If put() ran twice in a row, this program would abort. Let's add lock and conditional variable. 

solution 2

    cond_t cond;
    mutex_t mutex;
    
    void *producer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++) {
            Pthread_mutex_lock(&mutex);
        	while(count == 1)
                Pthread_cond_wait(&cond, &mutex);
            put(i);
        	Pthread_cond_signal(&cond);
        	Pthread_mutex_unlock(&mutex);
        }
    }
    void *consumer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++){
            Pthread_mutex_lock(&mutex);
            while(count == 0)
                Pthread_cond_wait(&cond, &mutex);
            int temp = get();
            Pthread_cond_signal(&cond);
            Pthread_mutex_lock(&mutex);
            printf("%d\n", temp);
        }       
    }

This solution is good when only one consumer is present. But when there are multiple consumers,  this code still has a bug. The problem is that one consumer is able to wake another consumer. This will cause problems.

Consider the following execution order, two consumers are sleeping, the producer put one item into the buffer, call signal(). One consumer wake, take an item from the buffer, then call signal(), and then call wait() to go to sleep. Then instead of the producer, another consumer is waken up. It will find the buffer to be empty and call wait() to put itself to sleep. Now all threads are sleeping and will sleep forever.

To fix the problem, we need two conditional variables. A filled conditional variable used by a producer to signal consumers. An empty conditional variable used by consumers to signal the producer. See the solution 3.



solution 3

    cond_t filled, empty;
    mutex_t mutex;
    
    void *producer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++) {
            Pthread_mutex_lock(&mutex);
        	while(count == 1)
                Pthread_cond_wait(&empty, &mutex);
            put(i);
        	Pthread_cond_signal(&filled);
        	Pthread_mutex_unlock(&mutex);
        }
    }
    void *consumer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++){
            Pthread_mutex_lock(&mutex);
            while(count == 0)
                Pthread_cond_wait(&filled, &mutex);
            int temp = get();
            Pthread_cond_signal(&empty);
            Pthread_mutex_lock(&mutex);
            printf("%d\n", temp);
        }       
    }

This is a correct solution for a buffer of size 1, but cannot handle a buffer of general size.  Let us fix this by using a cyclic array representation of the buffer.



ultimate solution

    int buffer[MAX];
    int fill_ptr = 0;
    int use_ptr  = 0;
    int count = 0;
    
    void put(int value) {
        buffer[fill_ptr] = value;
        fill_ptr = (fill_ptr + 1) % MAX;
        count++;
    }
    int get() {
       int value = buffer[use_ptr];
       use_ptr = (use_ptr + 1) % MAX;
       count--;
       return value;
    }

    cond_t filled, empty;
    mutex_t mutex;
    void producer(void *arg) {
        int loop = (int) arg;
        for (int i = 0; i < loops; i++) {
            Pthread_mutex_lock(&mutex);
        	while(count == MAX)
                Pthread_cond_wait(&empty, &mutex);
            put(i);
        	Pthread_cond_signal(&filled);
        	Pthread_mutex_unlock(&mutex);
        }
    }
    
    void *consumer(void *arg) {
        int loops = (int) arg;
        for (int i = 0; i < loops; i++){
            Pthread_mutex_lock(&mutex);
            while(count == 0)
                Pthread_cond_wait(&filled, &mutex);
            int temp = get();
            Pthread_cond_signal(&empty);
            Pthread_mutex_lock(&mutex);
            printf("%d\n", temp);
        }  
    }



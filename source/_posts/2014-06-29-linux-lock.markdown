---
layout: post
title: "linux_lock"
date: 2014-06-29 23:49:52 +0800
comments: true
categories: Linux
---

#Linux下互斥锁,读写锁区别

##目录
1.互斥锁  
2.读写锁   

互斥锁 实现多个线程之间的同步,互斥锁主要函数  
1.pthread_mutex_init ->初始化   
2.pthread_mutex_destroy -->销毁   
3.pthread_mutex_lock -->加锁  
4.pthread_mutex_unlock -->解锁   

读写锁 比mutex有更高的适用性，可以多个线程同时占用读模式的读写锁，但是只能一个线程占用写模式的读写锁。  
当读写锁是读加锁状态,所有以读模式对它进行加锁的线程都可以获得到访问权,但是以写模式对它加锁的线程将阻塞  
当读写锁是写加锁状态,所以对该资源进行操作的线程都会阻塞,直到该锁解锁.  
读写锁函数  
1.pthread_rwlock_init -->初始化  
2.pthread_rwlock_destroy -->销毁  
3.pthread_rwlock_rdlock -->读锁定  
4.pthread_rwlock_unlock -->解锁,写锁定的解锁也是调用该函数  
5.pthread_rwlock_rwlock -->写锁定   
6.pthread_rwlock_tryrdlock -->尝试是否能获得到读锁  
7.pthread_rwlock_trywrlock -->尝试是否能获得到写锁  

以下是测试的例子

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
pthread_t id;
pthread_t id1;
pthread_rwlock_t _lock;
pthread_mutex_t mutex;
void ReadLog( void *ptr )
{

    unsigned int pid = (unsigned int )ptr;
    printf("Thread %u lock--\n",pid);
    FILE* fp = fopen("log.txt","rb");
    if(fp)
    {
        fseek(fp,0,SEEK_END);
        int nlen = ftell(fp);
        fseek(fp,0,SEEK_SET);
        char* pData = new char[nlen+1];
        memset(pData,0,nlen+1);
        fread(pData,1,nlen,fp);
        fclose(fp);
        printf("%s",pData);
        delete [] pData;
    }
    sleep(2);
    printf("Thread %u unlock--\n",pid);

}


void *rd_lock_thread( void* ptr )
{
    pthread_rwlock_rdlock( &_lock );
    printf("读锁,其他线程才可以进入\n");
    ReadLog( ptr );
    pthread_rwlock_unlock( &_lock );
}

void *rw_lock_thread( void* ptr )
{

    pthread_rwlock_wrlock( &_lock );
    printf("写锁,只有该线程解锁 %u ,其他线程才可以进入\n",(unsigned int ) ptr );
    ReadLog( ptr );
    pthread_rwlock_unlock( &_lock );
}

void *mutex_thread(void *ptr)
{

    pthread_mutex_lock(&mutex);
    printf("只有当 Thread %u 解锁,其他线程才可以进入--\n",(unsigned int ) ptr);
    ReadLog( ptr );
    pthread_mutex_unlock(&mutex);
}



int main(int argc, const char *argv[])
{
    int i,ret;
    printf("演示:mutex-互斥锁--\n");
    pthread_mutex_init( &mutex, NULL );
    ret=pthread_create(&id,NULL,mutex_thread,(void*)1);
    ret=pthread_create(&id1,NULL,mutex_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);
    pthread_mutex_destroy( &mutex );
    printf("-------------------------------------\n\n");

    printf("演示:读写锁--读锁--\n");
    pthread_rwlock_init(&_lock,0);
    ret=pthread_create(&id,NULL,rd_lock_thread,(void*)1);
    ret=pthread_create(&id1,NULL,rd_lock_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);
    pthread_rwlock_destroy(&_lock);
    printf("-------------------------------------\n\n");


    printf("演示:读写锁--写锁--\n");
    pthread_rwlock_init(&_lock,0);
    ret=pthread_create(&id,NULL,rw_lock_thread,(void*)1);
    ret=pthread_create(&id1,NULL,rw_lock_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);
    pthread_rwlock_destroy(&_lock);

    return 0;
}
```

{% img /../images/linux_lock/linux_lock_pic_1.png %}  
{% img /../images/linux_lock/linux_lock_pic_2.png %}  
{% img /../images/linux_lock/linux_lock_pic_3.png %}  

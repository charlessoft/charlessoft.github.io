---
layout: post
title: "linux_lock_package"
date: 2014-06-30 00:58:59 +0800
comments: true
categories: linux
---

#读写锁,互斥锁封装

炼数成金的课程中把互斥锁和读写锁封装了一个类,一起分享下,适用windows/linux,

```
#ifndef OSSLATCH_HPP__
#define OSSLATCH_HPP__

#include "core.hpp"

#ifdef _WINDOWS
#define oss_mutex_t						CRITICAL_SECTION
#define oss_mutex_init(__lock, __attribute)		InitializeCriticalSection( (__lock) )
#define oss_mutex_destroy				DeleteCriticalSection
#define oss_mutex_lock					EnterCriticalSection
#define oss_mutex_trylock(__lock)		(TRUE == TryEnterCriticalSection( (__lock) ) )
#define oss_mutex_unlock				LeaveCriticalSection

#define oss_rwlock_t					SRWLOCK
#define oss_rwlock_init(__lock, __attribute)		InitializeSRWLock( (__lock) )
#define oss_rwlock_destroy(__lock)		(1)	
#define oss_rwlock_rdlock				AcquireSRWLockShared
#define oss_rwlock_rdunlock				ReleaseSRWLockShared
#define oss_rwlock_wrlock				AcquireSRWLockExclusive
#define oss_rwlock_wrunlock				ReleaseSRWLockExclusive
#define oss_rwlock_rdtrylock(__lock)	(false)
#define oss_rwlock_wrtrylock(__lock)	(false)

#else

#define oss_mutex_t						pthread_mutex_t
#define oss_mutex_init					pthread_mutex_init
#define oss_mutex_destroy				pthread_mutex_destroy
#define oss_mutex_lock					pthread_mutex_lock
#define oss_mutex_trylock(__lock)	(pthread_mutex_trylock( (__lock) ) == 0 )
#define oss_mutex_unlock				pthread_mutex_unlock

#define oss_rwlock_t					pthread_rwlock_t
#define oss_rwlock_init					pthread_rwlock_init
#define oss_rwlock_destroy				pthread_rwlock_destroy
#define oss_rwlock_rdlock				pthread_rwlock_rdlock
#define oss_rwlock_rdunlock				pthread_rwlock_unlock
#define oss_rwlock_wrlock				pthread_rwlock_wrlock
#define oss_rwlock_wrunlock				pthread_rwlock_unlock
#define oss_rwlock_rdtrylock(__lock)	(pthread_rwlock_tryrdlock( (__lock) ) == 0 )
#define oss_rwlock_wrtrylock(__lock)	(pthread_rwlock_trywrlock ( ( __lock) ) == 0 )

#endif

enum OSS_LATCH_MODE
{
   SHARED ,
   EXCLUSIVE
} ;

class ossXLatch
{
private :
   oss_mutex_t _lock ;
public :
   ossXLatch ()
   {
      oss_mutex_init ( &_lock, 0 ) ;
   }
   ~ossXLatch ()
   {
		oss_mutex_destroy(&_lock);
   }
   void get ()
   {
		oss_mutex_lock(&_lock);
   }
   void release ()
   {
		oss_mutex_unlock(&_lock);
   }
   bool try_get ()
   {
		return oss_mutex_trylock(&_lock);
   }
} ;

class ossSLatch
{
private :
   oss_rwlock_t _lock ;
public :
   ossSLatch ()
   {
      oss_rwlock_init ( &_lock, 0 ) ;
   }

   ~ossSLatch ()
   {
      oss_rwlock_destroy ( &_lock ) ;
   }

   void get ()
   {
      oss_rwlock_wrlock ( &_lock ) ;
   }

   void release ()
   {
      oss_rwlock_wrunlock ( &_lock ) ;
   }

   bool try_get ()
   {
      return ( oss_rwlock_wrtrylock ( &_lock ) ) ;
   }

   void get_shared ()
   {
      oss_rwlock_rdlock ( &_lock ) ;
   }

   void release_shared ()
   {
      oss_rwlock_rdunlock ( &_lock ) ;
   }

   bool try_get_shared ()
   {
      return ( oss_rwlock_rdtrylock ( &_lock ) ) ;
   }
} ;

#endif
```

```
//调用
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "ossLatch.hpp"
#include   <unistd.h>
ossXLatch Mutex_lock;
ossSLatch Shared_lock;
pthread_t id;
pthread_t id1;

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

    Shared_lock.get_shared();
    printf("读锁,其他线程才可以进入\n");
    ReadLog( ptr );
    Shared_lock.release();
}

void *rw_lock_thread( void* ptr )
{

    Shared_lock.get();
    printf("写锁,只有该线程解锁 %u ,其他线程才可以进入\n",(unsigned int ) ptr );
    ReadLog( ptr );
    Shared_lock.release();
}

void *mutex_thread(void *ptr)
{

    Mutex_lock.get();
    printf("只有当 Thread %u 解锁,其他线程才可以进入--\n",(unsigned int ) ptr);
    ReadLog( ptr );
    Mutex_lock.release();
}



int main(int argc, const char *argv[])
{
    int i,ret;


    printf("演示:mutex-互斥锁--\n");
    ret=pthread_create(&id,NULL,mutex_thread,(void*)1);
    ret=pthread_create(&id1,NULL,mutex_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);
    printf("-------------------------------------\n\n");

    printf("演示:读写锁--读锁--\n");
    ret=pthread_create(&id,NULL,rd_lock_thread,(void*)1);
    ret=pthread_create(&id1,NULL,rd_lock_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);

    printf("-------------------------------------\n\n");


    printf("演示:读写锁--写锁--\n");
    ret=pthread_create(&id,NULL,rw_lock_thread,(void*)1);
    ret=pthread_create(&id1,NULL,rw_lock_thread,(void*)2);
    pthread_join(id,NULL);
    pthread_join(id1,NULL);

    return 0;
}


```

[toc]

# 重点函数讲解

## waitpid

```c
pid_t waitpid(pid_t pid, int *wstatus, int options) {
    
}

/*
[pid]
 < -1: meaning wit for ant child process whose process group ID is equal to the [absolute] value of pid;
 
 -1 : meaning wait for any child process;
 
 0 : meaning wait for any child process whose process group ID is equal to that of the calling process at the time of the call to waitpid();
 
 > 0 : meaning wait for the child whose process ID is equal to the value of pid.

*/

/*
[wstatus]
表示waitpid的退出状态
*/
```





DESCRIPTION
       All of these system calls are used to wait for state changes in a child
       of the calling process, and obtain information about  the  child  whose
       state  has changed.  

A state change is considered to be: 

* the child terminated;  被终止

* the child was stopped by a signal;  被stop掉
* or the child was resumed by a  signal. 被激活

```shell

If a child has already changed state, then these calls  return  immedi‐
ately.   Otherwise,  they block until either a child changes state or a
signal handler interrupts the call (assuming that system calls are  not
automatically restarted using the SA_RESTART flag of sigaction(2)). 
```

 ### Notes

A child that terminates, but has not been waited for  becomes  a  "zom bie".  

The kernel maintains a minimal set of information about the zombie process (PID, termination status, resource  usage  information)  

in order to allow the parent to later perform a wait to obtain information about the child.  

As long as a zombie is not removed  from  the  system via  a wait, it will consume a slot in the kernel process table, and if this table fills, it will not be possible to create further  processes.





​       If a parent process terminates, then its "zombie" children (if any) are
​       adopted by init(1), (or by the nearest "subreaper" process  as  defined
​       through  the  use  of  the  prctl(2) PR_SET_CHILD_SUBREAPER operation);
​       init(1) automatically performs a wait to remove the zombies.

```shell
A child that terminates, but has not been waited for  becomes  a  "zom bie".  

The kernel maintains a minimal set of information about the zombie process (PID, termination status, resource  usage  information)  

in order to allow the parent to later perform a wait to obtain information about the child.  

As long as a zombie is not removed  from  the  system via  a wait, it will consume a slot in the kernel process table, and if this table fills, it will not be possible to create further  processes.

If a parent process terminates, then its "zombie" children (if any) are
adopted by init(1), 

or by the nearest "subreaper" process  as  defined
through  the  use  of  the  prctl(2) PR_SET_CHILD_SUBREAPER operation);

init(1) automatically performs a wait to remove the zombies.

# 如果init(1)进程，后面也不收这些zombie，这时候就该重启系统了
```



## kill函数

kill  - send a signal to a process

```c
kill(pid, SIGKILL); // 表示向当前pid进程发送 SIGKILL 信号，但是不单单可以发送KILL信号
```



## sigprocmask 函数







```shell
sigprocmask, rt_sigprocmask - examine and change blocked signals

sigprocmask()  is  used  to  fetch and/or change the signal mask of the calling thread.
```



```c
/* Prototype for the glibc wrapper function */
int sigprocmask(int how, const sigset_t *new_mask, sigset_t *old_mask);

/*
[how]
SIG_BOLCK: 简单理解就是,开始阻塞new_mask里面的信号集合，将之前的信号保存在old_mask里面；

SIG_UNBLOCK: 解除信号的阻塞

SIG_SETMASK: 重新设置mask

*/


```






---
title: grep-line-buffer
date: 2019-02-12 19:07:53
tags:
---
#### 
           --line-buffered       flush output on every line
           

##### 下面这条命令不生效

    node [commend1] | grep 'xxx' | grep 'xxx'
    
##### 只有一个grep的时候， 内容能够正常打出来，但是加上第二个就打不出来。找了好久的原因，最后发现是因为管道的全缓冲，也就是前面一个grep后的管道内容没有超过默认的buffer_size就不会输出给后面的管道，
加上--line-buffered之后，每输出一行就刷新输出。

##### 但假如commend1运行结束后会退出，就不需要加--line-buffered，我理解缓存针对的是动态的输出情况下。

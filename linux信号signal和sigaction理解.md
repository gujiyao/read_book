---
title: linux 信号signal和sigaction理解
created: '2020-01-11T02:31:41.000Z'
modified: '2020-01-11T02:32:33.000Z'
---

# linux 信号signal和sigaction理解

<p align="start"><span style="color: rgb(0, 0, 0);"><small>signal，此函数相对简单一些，给定一个信号，给出信号处理函数则可，当然，函数简单，其功能也相对简单许多，简单给出个函数例子如下：</small></span></p>

<p align="start">**\[cpp\]** [view plain](http://blog.csdn.net/beginning1126/article/details/8680757 "view plain")[copy](http://blog.csdn.net/beginning1126/article/details/8680757 "copy")

1. 1 #include <signal.h>  
    
2. 2 #include <stdio.h>  
    
3. 3 #include <unistd.h>  
    
4. 4   
    
5. 5 <span style="color: rgb(0, 102, 153);">**void ouch(**</span><span style="color: rgb(0, 0, 0);">**int sig)  **</span>
    
6. 6 {  
    
7. 7     printf(<span style="color: rgb(0, 0, 255);">"I got signal %d\\n", sig);  </span>
    
8. 8     <span style="color: rgb(0, 130, 0);">// (void) signal(SIGINT, SIG\_DFL);  </span>
    
9. 9     <span style="color: rgb(0, 130, 0);">//(void) signal(SIGINT, ouch);  </span>
    
10. 10   
    
11. 11 }  
    
12. 12   
    
13. 13   
    
14. 14   
    
15. 15 <span style="color: rgb(0, 0, 0);">int main()  </span>
    
16. 16 {  
    
17. 17     (<span style="color: rgb(0, 102, 153);">**void) signal(SIGINT, ouch);  **</span>
    
18. 18   
    
19. 19     <span style="color: rgb(0, 102, 153);">**while(1)  **</span>
    
20. 20     {  
    
21. 21         printf(<span style="color: rgb(0, 0, 255);">"hello world...\\n");  </span>
    
22. 22         sleep(1);  
    
23. 23     }  
    
24. 24 }</p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>当然，实际运用中，需要对不同到signal设定不同的到信号处理函数，SIG\_IGN忽略/SIG\_DFL默认，这俩宏也可以作为信号处理函数。同时SIGSTOP/SIGKILL这俩信号无法捕获和忽略。注意，经过实验发现，signal函数也会堵塞当前正在处理的signal，但是没有办法阻塞其它signal，比如正在处理SIG\_INT，再来一个SIG\_INT则会堵塞，但是来SIG\_QUIT则会被其中断，如果SIG\_QUIT有处理，则需要等待SIG\_QUIT处理完了，SIG\_INT才会接着刚才处理。</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>sigaction，这个相对麻烦一些，函数原型如下：</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>int sigaction(int sig, const struct sigaction \*act, struct sigaction \*oact)；</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>函数到关键就在于struct sigaction</small></span></p>

<p align="start">**\[cpp\]** [view plain](http://blog.csdn.net/beginning1126/article/details/8680757 "view plain")[copy](http://blog.csdn.net/beginning1126/article/details/8680757 "copy")

1. stuct sigaction  
    
2. {  
    
3. <span style="color: rgb(0, 102, 153);">**void (\*)(**</span><span style="color: rgb(0, 0, 0);">**int) sa\_handle;  **</span>
    
4. sigset\_t sa\_mask;  
    
5. <span style="color: rgb(0, 0, 0);">int sa\_flags;  </span>
    
6. }</p>

<p align="start">**\[cpp\]** [view plain](http://blog.csdn.net/beginning1126/article/details/8680757 "view plain")[copy](http://blog.csdn.net/beginning1126/article/details/8680757 "copy")

1. 1 #include <signal.h>  
    
2. 2 #include <stdio.h>  
    
3. 3 #include <unistd.h>  
    
4. 4   
    
5. 5   
    
6. 6 <span style="color: rgb(0, 102, 153);">**void ouch(**</span><span style="color: rgb(0, 0, 0);">**int sig)  **</span>
    
7. 7 {  
    
8. 8     printf(<span style="color: rgb(0, 0, 255);">"oh, got a signal %d\\n", sig);  </span>
    
9. 9   
    
10. 10     <span style="color: rgb(0, 0, 0);">int i = 0;  </span>
    
11. 11     <span style="color: rgb(0, 102, 153);">**for (i = 0; i < 5; i++)  **</span>
    
12. 12     {  
    
13. 13         printf(<span style="color: rgb(0, 0, 255);">"signal func %d\\n", i);  </span>
    
14. 14         sleep(1);  
    
15. 15     }  
    
16. 16 }  
    
17. 17   
    
18. 18   
    
19. 19 <span style="color: rgb(0, 0, 0);">int main()  </span>
    
20. 20 {  
    
21. 21     <span style="color: rgb(0, 102, 153);">**struct sigaction act;  **</span>
    
22. 22     act.sa\_handler = ouch;  
    
23. 23     sigemptyset(&act.sa\_mask);  
    
24. 24     sigaddset(&act.sa\_mask, SIGQUIT);  
    
25. 25     <span style="color: rgb(0, 130, 0);">// act.sa\_flags = SA\_RESETHAND;  </span>
    
26. 26     <span style="color: rgb(0, 130, 0);">// act.sa\_flags = SA\_NODEFER;  </span>
    
27. 27     act.sa\_flags = 0;  
    
28. 28   
    
29. 29     sigaction(SIGINT, &act, 0);  
    
30. 30   
    
31. 31   
    
32. 32     <span style="color: rgb(0, 102, 153);">**struct sigaction act\_2;  **</span>
    
33. 33     act\_2.sa\_handler = ouch;  
    
34. 34     sigemptyset(&act\_2.sa\_mask);  
    
35. 35     act.sa\_flags = 0;  
    
36. 36     sigaction(SIGQUIT, &act\_2, 0);  
    
37. 37   
    
38. <span style="color: rgb(0, 102, 153);">**while(1)  **</span>
    
39. {  
    
40. sleep(1);  
    
41. }  
    
42. 38     <span style="color: rgb(0, 102, 153);">**return;  **</span>

44. }</p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>1\. 阻塞，sigaction函数有阻塞的功能，比如SIGINT信号来了，进入信号处理函数，默认情况下，在信号处理函数未完成之前，如果又来了一个SIGINT信号，其将被阻塞，只有信号处理函数处理完毕，才会对后来的SIGINT再进行处理，同时后续无论来多少个SIGINT，仅处理一个SIGINT，sigaction会对后续SIGINT进行排队合并处理。</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>2\. sa\_mask，信号屏蔽集，可以通过函数sigemptyset/sigaddset等来清空和增加需要屏蔽的信号，上面代码中，对信号SIGINT处理时，如果来信号SIGQUIT，其将被屏蔽，但是如果在处理SIGQUIT，来了SIGINT，则首先处理SIGINT，然后接着处理SIGQUIT。</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>3.</small></span><span style="color: rgb(0, 0, 0);"><small>sa\_flags如果取值为0，则表示默认行为。还可以取如下俩值，但是我没觉得这俩值有啥用。</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>SA\_NODEFER，如果设置来该标志，则不进行当前处理信号到阻塞</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>SA\_RESETHAND，如果设置来该标志，则处理完当前信号后，将信号处理函数设置为SIG\_DFL行为</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>下面单独来讨论一下信号屏蔽，记住是屏蔽，不是消除，就是来了信号，如果当前是block，则先不传递给当前进程，但是一旦unblock，则信号会重新到达。</small></span></p>

<p align="start">**\[cpp\]** [view plain](http://blog.csdn.net/beginning1126/article/details/8680757 "view plain")[copy](http://blog.csdn.net/beginning1126/article/details/8680757 "copy")

1. <span style="color: rgb(128, 128, 128);">#include <signal.h>  </span>
    
2. <span style="color: rgb(128, 128, 128);">#include <stdio.h>  </span>
    
3. <span style="color: rgb(128, 128, 128);">#include <unistd.h>  </span>

7. <span style="color: rgb(0, 102, 153);">**static **</span><span style="color: rgb(0, 102, 153);">**void sig\_quit(**</span><span style="color: rgb(0, 0, 0);">**int);  **</span>

9. <span style="color: rgb(0, 0, 0);">int main (</span><span style="color: rgb(0, 102, 153);">**void) {  **</span>
    
10. sigset\_t <span style="color: rgb(0, 102, 153);">**new, old, pend;  **</span>

12. signal(SIGQUIT, sig\_quit);

14. sigemptyset(&<span style="color: rgb(0, 102, 153);">**new);  **</span>
    
15. sigaddset(&<span style="color: rgb(0, 102, 153);">**new, SIGQUIT);  **</span>
    
16. sigprocmask(SIG\_BLOCK, &<span style="color: rgb(0, 102, 153);">**new, &old);  **</span>

18. sleep(5);

20. printf(<span style="color: rgb(0, 0, 255);">"SIGQUIT unblocked\\n");  </span>
    
21. sigprocmask(SIG\_SETMASK, &old, NULL);

23. sleep(50);  
    
24. <span style="color: rgb(0, 102, 153);">**return 1;  **</span>
    
25. }

27. <span style="color: rgb(0, 102, 153);">**static **</span><span style="color: rgb(0, 102, 153);">**void sig\_quit(**</span><span style="color: rgb(0, 0, 0);">**int signo) {  **</span>
    
28. printf(<span style="color: rgb(0, 0, 255);">"catch SIGQUIT\\n");  </span>
    
29. signal(SIGQUIT, SIG\_DFL);  
    
30. }</p>

gcc -g -o mask mask.c 

<p align="start"><span style="color: rgb(0, 0, 0);"><small>./mask</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>\========这个地方按多次ctrl+\\</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>SIGQUIT unblocked</small></span></p>

catch SIGQUIT

Quit (core dumped)

<p align="start"><span style="color: rgb(0, 0, 0);"><small>\======================</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>注意观察运行结果，在sleep的时候，按多次ctrl+\\，由于sleep之前block了SIG\_QUIT，所以无法获得SIG\_QUIT，但是一旦运行sigprocmask(SIG\_SETMASK, &old, NULL);则unblock了SIG\_QUIT，则之前发送的SIG\_QUIT随之而来。</small></span></p>

<p align="start"><span style="color: rgb(0, 0, 0);"><small>由于信号处理函数中设置了DFL，所以再发送SIG\_QUIT，则直接coredump。</small></span></p>

<p align="start">原文地址： [https://www.cnblogs.com/lidabo/p/4581065.html](https://www.cnblogs.com/lidabo/p/4581065.html)</p>

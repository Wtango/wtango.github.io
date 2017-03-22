---
id: 199
title: 段错误(Segmentation fault)产生的原因以及调试方法
date: 2015-08-30T21:48:52+00:00
author: Tango
layout: post
guid: http://wtango.com/?p=199
permalink: '/%e6%ae%b5%e9%94%99%e8%af%afsegmentation-fault%e4%ba%a7%e7%94%9f%e7%9a%84%e5%8e%9f%e5%9b%a0%e4%bb%a5%e5%8f%8a%e8%b0%83%e8%af%95%e6%96%b9%e6%b3%95/'
categories:
  - c\c++
  - Linux
tags:
  - c\c++
---
#### 1. 什么是段错误

可能每个程序员在Linux环境下使用C/C++编程时都曾遇到过“段错误”(Segmentation fault)，段错误是由于内存管理单元(负责支持虚拟内存的硬件)的异常所导致，而该异常则通常是由于解除引用一个未初始化或非法的指针引起的。产生段错误的地方有可能在自己编写的代码中，也有可能出现在库函数中(传递一个非法指针给库函数)。令人很不爽的是，段错误除了只有一个简单的错误信息之外，没有任何额外的提示，并且有的段错误不是每次都会出现。

<!--more-->

#### 2. 导致段错误的直接原因

  * 解除引用一个包含非法值
  * 解除引用一个空指针(常常由函数返回，并未经检查就使用)
  * 在未得到正确的权限时进行访问。例如，试图往一个只读的文本段储存值就会引发段错误。
  * 用完了栈或者堆空间(虚拟内存虽然巨大，但也有可能使用殆尽)

#### 3. 段错误调试方法

##### 3.1 dmesg + nm

dmesg可以在应用程序crash时，显示内核中保存的相关信息。如下所示，通过dmesg命令可以查看发生段错误的程序名称、引起段错误发生的内存地址、指令指针地址、堆栈指针地址、错误代码、错误原因等。

<pre>tango@Tango-PC:~/cfun$ dmesg 
[14253.910681] a.out[4927]: segfault at 0 ip 00000000004004f6 sp 00007fffebcf5260 error 6 in a.out[400000+1000]</pre>

然后使用nm查看其指针信息：

<pre>0000000000400570 T __libc_csu_fini
0000000000400500 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
00000000004004ed T main
0000000000400460 t register_tm_clones
0000000000400400 T _start
0000000000601038 D __TMC_END__
</pre>

段错误地址4004f6在main函数之后，在_\_libc\_csu_init之前，所以“段错误”应该是在main中产生的。

##### 3.2 使用fprintf

根据代码执行的情况，确定大致产生“段错误”的地方，使用fprintf(stderr,&#8221;&#8221;,&#8230;);来确定具体产生的位置。切记调试信息要输出到错误输出，因为错误输出默认不缓存，标准输出是有缓存的，程序崩溃可能导致缓存数据没有输出，所以建议调试信息都输出到错误输出。

##### 3.3 gdb

编译程序时加上-ggdb选项，使用gdb调试。

<pre>gcc -ggdb foo.c
gdb a.out
(gdb) run
Starting program: ./a.out 

Program received signal SIGSEGV, Segmentation fault.
0x00000000004004f6 in main ()
(gdb) 
</pre>

可以看到“段错误”实在main函数中产生的，那么可以在产生段错误的函数上打上断点，之后单步调试，便可以找出产生段错误的真正原因。

##### 3.4 signal(SIGSEGV,handler)

细心的网友可能发现了，产生段错误时会产生一个SIGSEGV信号。我们可以添加SIGSEGV的处理函数在程序发生段错误退出之前做一些事情，我们甚至可以使用backtrace(3)函数来保存当前栈内容.

<pre class="brush: cpp; title: ; notranslate" title="">void dump(int signo)
{
        fprintf(stderr,"catch Segmentation fault!!!\n");
#define SIZE 100
        FILE *fh;
        if(!(fh = fopen("/tmp/dbg_msg.log", "w+")))
                exit(0);
        void *buffer[100];
        int nptrs;
        nptrs = backtrace(buffer,SIZE);
        backtrace_symbols_fd(buffer, nptrs, fileno(fh));
        fflush(fh);
        exit(-1);
}

main (void)
{
        signal(SIGSEGV, &dump);
        *((int*)NULL) = 0;

        return 0;
}
</pre>
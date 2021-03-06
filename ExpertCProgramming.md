--------------------------------------------------------------------------------

## 3. 分析C语言的声明

* 参数在传递过程中首先尽可能的存放到寄存器中。一个int型参数一般会放到寄存
  器中，而一个只包含int型成员的结构体变量则很可能会被传递到堆栈中。


--------------------------------------------------------------------------------

## 4. 令人震惊的事实

* 对于数组，其地址在**编译**时可知；而指针必须首先在**运行**期取得它的当前
  值，然后才能进行解除引用操作。


--------------------------------------------------------------------------------

## 5. 对链接的思考

* 链接器一般只提供几个基本的I/O服务，就是被称作BIOS的程序。他们存在于内存
  中固定的地点，并不是每个可执行文件的一部分。

* 即使是在静态链接中，整个libc.a文件也没有被全部装入到可执行文件中，所装入
  的只是所需要的函数。

* `cc -o libfruit.so -G tomato.c`

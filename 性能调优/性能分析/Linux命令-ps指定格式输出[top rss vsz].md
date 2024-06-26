# Linux命令-ps 指定格式输出

参考文章

1. [ps命令的-o选项使用](http://fantefei.blog.51cto.com/2229719/1425304)

`ps`命令的`-o`选项可以输出的列, 格式为`ps -o format`. 其中`format`参数是以空格或逗号分隔的列表. 

常用列属性的关键字如下(使用`man ps`查看`STANDARD FORMAT SPECIFIERS`小节会有更详细的说明...太多了, 无法一一列举)

- `%cpu`: 在第1行的列名显示为`%CPU`, 表示当前进程CPU利用率.
    - 百分数(没有百分号`%`), 精确到小数点后1位. 
    - 又名`c`, 不过在第1列的列名显示的就不是`%CPU`而是`C`了.
    - 还有其他别名, 虽然信息是一样的, 但是列名会有所区别.
    - `100%`的话意为单核 cpu 满载, 其实最大值与服务器 cpu 核心数有关.
- `%mem`: 第1行的列名显示为`%MEM`, 表示当前进程对**物理内存**的利用率, 即`rss`列的值与服务器总内存的比值
    - 百分数(没有百分号`%`), 精确到小数点后1位. 
- `command`: 完整的启动命令, 其另一种形式为`comm`, **只显示命令名, 无启动参数**.
    - 打印这个的时候就不要用`-ef`了, 只用`-e`好了, 不然`command`列会有很多东西显示出来的.
- `rss`: `resource size`的缩写形式, 表示当前进程占用的, 未被放在swap空间的内存大小, 即活动内存.
    - 单位`KB`. 
    - 同`top`中的`RES`列.
    - Resident Set Size(常驻内存集大小), 哪一个正确???
    - 该值取自`/proc/$pid/status`文件中的`VmRSS`字段.
- `vsz`: `virtual size`的缩写, 表示当前进程占用的虚拟内存大小, 不一定是存储在swap空间的大小, 很可能是非活动性内存(比如缓存之类<???>)
    - 单位`KB`. 
    - 同`top`中的`VIRT`列.
    - 该值取自`/proc/$pid/status`文件中的`VmSize`字段.
- `start_time`: 进程启动的时间, 从启动开始到当前的**人类时间**(相对于占用CPU的总时间). 
    - 进程启动的时间, 1天之内可以精确到分钟, 格式[hh:mm], 超过1天貌似就只显示几月几日了
    - 又名`bsdstart`, `start`, `lstart`和`stime`, 都是同一个意思. 
    - `ps`列名为`START`
- `cputime`: 当前进程占用CPU的总时间, 并不是从启动到现在一共过了多长时间, 因为在这期间 cpu 时间片还分给了其他进程.
    - 又名`time`.
    - `ps`列名为`TIME`
    - 24小时之内格式为[小时数:分钟数:秒数], 超过24小时的格式为[天数-小时数:hh:mm], 目前还没遇到更大的.
    - 与`top`命令的`TIME+`列含义相同, 不过格式不太一样, 后者的格式为[分钟数:秒数.xx], 后面精确到小数点后两位, 如果时间很长的话, 小数点后面的可能会舍去, 但冒号前面一定是分钟数, 目前还没遇到更大的.
- `etime`: 这个才是进程自启动到目前一共的时间.
    - `ps`列名`ELAPSED`
    - 格式为`[DD-]hh:]mm:ss`
    - 还有个`etimes`, 表示总秒数
- `stat`: 当前进程运行状态, man手册中`PROCESS STATE CODES`章节有对状态码的详细描述.
- `drs`: data resident set size 物理内存中, trs之外的部分
- `trs`: text resident set size 可执行代码所占用的物理内存
    - 看缩写好像是和rss相关, 但实际貌似 drs + trs (+1) = vsz

另外, `ps`的`-o`还可以指定输出结果在第一行的显示的列名, 如下

```log
## 查看init进程的cpu和内存状况
$ ps -o pid,%cpu=abc,%mem=123 -p 1
   PID  abc  123
     1  0.3  1.3
```

不过这种自定义有时好像也不是那么好使

```log
## 偏移得有点多...
$ ps -o pid=jinchenghao,%cpu=abc,%mem -p 1
jinchenghao,%cpu=abc,%mem
                        1
```

## 常用格式

```
ps -e -o pid,ppid,%cpu,%mem,start_time,command
```

线程相关

```
ps -eT -o pid,ppid,tid,%cpu,%mem,start_time,command
```

资源相关

```
ps -eT -o pid,%cpu,%mem,rss,vsz,drs,trs,size,start_time,command
```

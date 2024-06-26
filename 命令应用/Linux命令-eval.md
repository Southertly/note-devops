# Linux命令-eval

参考文章

1. [shell 中的 eval](http://www.cnblogs.com/huzhiwei/archive/2012/03/14/2395956.html)
2. [shell eval命令使用](http://blog.csdn.net/w_ww_w/article/details/7075867)
3. [shell模板变量替换](https://www.cnblogs.com/woshimrf/p/shell-template-variable.html)

## 1. 基本认识

shell中的`eval`后可以接多个参数, 在最简单的情况下就和`eval`不存在一样, 后面的命令该执行的执行, 该显示的显示, 不会有任何不同. 

如下 

```
$ eval touch tmpfile
$ ls
tmpfile

$ eval echo 'hello world'
hello world
```

如果其后面的参数是一个字符串, 或者说其本质是字符串, 则`eval`会尝试将其当作命令执行. 这里说的"本质"即变量的值, 如果`eval`后接的参数是一个变量, 而这个变量的值是一个字符串, 则`eval`会尝试将其当做命令执行.

如下

```log
## eval是把'ls -al'当成命令执行的.
$ eval 'ls -al'
-rw-r--r--  1 root root         0 Dec  9 15:09 tmpfile
## eval后接一个字符串变量
$ ab='ls -al'
$ eval $ab
-rw-r--r--  1 root root         0 Dec  9 15:09 tmpfile
```

好了, 入题完成, 接下来揭示`eval`工作的本质. 

前面两组示例中, 第一组中`eval`后的参数`touch tmpfile`中不存在变量, `eval`将其全部执行; 第二组中`eval`会尝试将后面的变量`$ab`解析成字符串`ls -al`, 也就是完成变量的替换, 然后将其作为命令执行.

把这两者结合起来, 就是`eval`真正的工作流程: **`eval`会把其后面的所有参数当作一条shell命令去执行, 但是在执行之前会把其中存在的变量解析出来.**

如下示例

```
$ ab=tmpfile
$ eval touch $ab
$ ls
tmpfile
```

我们把上述操作写在脚本中查看它具体的执行流程. 可以看到, eval首先将`$ab`变量替换成`tmpfile`, 然后正常执行`touch`命令.

```
$ cat ./st.sh
#!/bin/bash
ab=tmpfile
set -x
eval touch $ab

$ bash ./st.sh 
+ eval touch tmpfile
++ touch tmpfile
```

------

很多人可能会问, 要它有什么用? `touch`一个文件而已, 用`eval`做什么?

其实很多脚本语言都有`eval()`函数, 如javascript, python等, 目的是为了可以动态执行一些由函数, 或是由服务器端返回的代码. 也就是说, 这个代码没有存在于本地, 而是从其他环境得到的, 而且一般来说, 通过这些方法得到的动态代码, 都是字符串类型(这个应该不容质疑吧...).

这种场景比较难想像, 举个例子来说吧, 一个python语言写的web应用, 它的后台程序中有一个功能可以由很多不同的模块实现, 不同的实现有各自的特性, 用户需要根据他们本身的需要来选择哪一种实现. 就像在安装操作系统的时候, 用户可以选择合适的文件系统(ext4, ext3, ntfs等), 安装程序会加载所有可选的文件系统的源码, 但是对于服务器端应用来说, 为每个用户会话都加载实现同一功能的所有模块的确有些得不偿失. 所以python程序可以通过`eval("__import__('os').system('whoami')")`这种形式加载用户指定的模块并执行响应. 相关场景介绍可以参考[这篇文章](http://python.jobbole.com/82770/)的第1节.

~~但是, shell的语法还是没有js, python这种高级语言中那么强大. 首先, shell中的函数是不能返回复杂数据的, 最多是通过`return`返回一个状态码, 所以动态生成一条命令并返回给`eval`使用的; 另外, 其实脚本语言甚至支持`eval`嵌套, 即`eval`的执行结果还是可以被另一个`eval`执行.~~ 试验未成功, eval的输出结果不能被另外的`eval`使用(像`eval(eval('动态代码'))`), 而且似乎没什么意义.

需要注意的是, `eval`语句中, 单双引号依然是有所区别的, 反斜线`\`也依然会被当作转义字符特殊处理, `eval`本身也是通过分号分隔的, 即, 出现分号之后的命令, 不再属于`eval`的执行范围.

## 2. 使用实例

### 2.1 取得传入脚本的最后一个参数

我们知道, 传入脚本的参数可以通过`$1`, `$2`这种形式取得, 但当传入的参数个数不能确定, 而脚本中更好需要得到最后一个参数的确切值时, `eval`的使用就显得很有必要.

参数的个数可以通过`$#`这个特殊变量获得, 我们需要首先取得这个变量的值, 然后才能获取第`$#`个参数的值.

```bash
#!/bin/bash
## 获取传入脚本的参数个数
sum=$#
## 这句显然只能得到参数个数的值
echo $sum
## 这句会把$$当成特殊变量处理, 将得到脚本执行时所在的进程pid和sum字符串
echo $$sum
## 这句则会得到一个$字符和一个数值, 就是参数的个数, 它们的组却没有办法得到解析
echo \$$sum
## 这句会报错
## echo ${$sum}

## 这句可以正常执行, eval首先得到了$sum的值, 然后执行echo $n(n就是$sum的值, 它是一个数字)
eval echo \$$sum
```

实际执行一下这个脚本试试? 记得命令行中带上几个参数...

### 2.2 配置解析

假设有如下文件, 我们希望将第1列的值当作变量名, 第2列的值当作变量值, 该如何做?

```
name general
age 24
sex male
```

其实这就已经相当于一个简易的配置文件了, 也许在实际使用中会有用(当然, 也可以将这种文件初始化成字典变量, 反而会更简单更直观一点). 参考脚本

```
while read key val
do
eval "${key}=${val}"
done < 文件名
echo "$name $age $sex"
```

### 2.3 模板渲染

```bash
eval "cat <<EOF
$(< 模板文件路径)
EOF
" > 结果文件路径
```

我从来没用过`<`重定向, 这一次算是长见识了. 其实就是从目标文件中读取数据, 与`>`相反.

如果直接执行`< 文件`, 什么也不会显示.

```log
$ cat requirements.txt
requests==2.24.0
$ < requirements.txt ## 这里什么也没有.
```

为了获取到这个文件的内容, 可以使用反引号或双小括号

```log
$ cnt=$(< requirements.txt)
/tmp
$ echo $cnt
requests==2.24.0

$ cnt2=`< requirements.txt`
$ echo $cnt2
requests==2.24.0
```

> 按行读取文件时可能会用到, `while read line do echo $line; done < 待读取的文件`.


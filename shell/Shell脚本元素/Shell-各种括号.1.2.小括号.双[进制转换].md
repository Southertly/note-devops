# Shell-各种括号.1.2.小括号.双

参考文章

1. [shell中的括号（小括号, 中括号, 大括号）](http://blog.csdn.net/tttyd/article/details/11742241)
2. [shell中的各种括号](http://blog.csdn.net/weihongrao/article/details/17007575)
3. [linux中双括号和双中括号, 括号和中括号](http://blog.csdn.net/weihongrao/article/details/17006931)
4. [shell脚本中的几个括号总结(小括号/大括号/花括号)](http://blog.csdn.net/lee244868149/article/details/38422437)
5. [shell编程中常用的比较、判断和删除等语法](http://blog.csdn.net/lee244868149/article/details/38424267)
6. [GNU Linux shell中如何进行各进制编码间(二进制、8进制、16进制、base64)的转换](https://blog.csdn.net/yygydjkthh/article/details/50699913)

> 小括号 == (圆括号)

> `((exp))`这种计算是符合C语言的运算符, 也就是说只要符合C的运算符都可用在`((exp))`, 甚至是三目运算符. 注意: 这种扩展计算是整数型的计算, 不支持浮点型. 若是逻辑判断, 表达式exp为真则为1, 假则为0. 

## 1. 整数运算

1. 四则运算: 加减乘除, 求余. (不过没有办法设置精度, 除法只能保留整数部分), 可用括号
2. 位运算: 与(&), 或(|), 非(^); 
3. 数值比较: `>` `<`, `>=` `>=`, `==` `!=`, 与(&&), 或(||), 返回值为1/0(真或假).
4. 三目运算符: (( 1 ? 2: 3))
5. 进制转换

**注意**

1. **内容只可以是数字, 且只可以是整数**
2. 如果要将双小括号的结果赋值给变量, 需要使用`$`运算符, 如`$((1 + 2))`; 或是直接在双小括号内部完成赋值, 如`((num = 1 + 2))`
3. 双括号内外共享变量, 内部还可以使用自增(`++`), 自减(`--`)运行符; 另外, **双括号内部引用变量时不可以加`$`符号**
4. 双小括号内的表达式支持空格分隔
5. 双小括号也可以判断字符串是否相等, 但对于空字符串的判断有些问题.

```
$ abc=''
$ if (( abc == '' )); then echo empty; fi
-bash: ((: abc ==  : 语法错误: 期待操作数 （错误符号是 "==  "）
```

> 数值比较用双小括号, 字符串比较, 可以使用双中括号.

### 1.1. 简单运算

简单运算并赋值

```
$ echo $((1 + 2))
3
$ ((num = 1 + 2))
$ echo $num
3
```

真值运算, 返回值只能是0或1.

```
$ echo $(( 2 || 1))
1
$ echo $((1 > 2))
0
```

自增运算

```
$ a=1
$ ((a ++))
$ echo $a
2
```

### 1.2. 三目运算

三目运算, 不过好像没多大用, 只能进行数据运算

```
$ echo $(( 1==2 ? 4 : 7))
7
```

如果第一个参数是空字符串或0, 就返回第3个参数的值(即认为这个条件为假);反之, 就返回第2个参数串的值. 

但前提是, 第2, 3个参数为数值类型, 如果它们是字符串, 那就会返回0

```
$ a=123
## 在双小括号内部, 变量a不用加$符
$ echo $(( a ? 8 : 9 ))
8
$ a=
$ echo $(( a ? 8 : 9 ))
9

$ a=123
## 貌似原因是abc被当作了变量, 所以返回0
$ echo $(( a ? abc : def ))
0
## 但是不能将abc, def赋值为字符串, 双小括号会认为存在语法错误...这个可能比较难以理解, 不建议日后使用
$ abc='abc'
$ echo $(( a ? abc : def ))
-bash: abc: expression recursion level exceeded (error token is "abc")
```

> 三目运算中, 空字符串及只包含空白字符的字符串被视为false.

### 1.3. 进制转换

按照参考文章6, 十分有效.

双小括号内部的数值默认以10进制处理, 不过可以通过与其他高级语言相似的前缀来声明变量的进制. 如以`0`开头即为8进制, 以`0x`开头即为16进制. 使用 `BASE#NUMBER`这种形式可以表示其它进制.

> BASE取值范围：2-64

八进制转十进制

```bash
((num = 020)); echo $num    ## 16
((num = 8#20)); echo $num   ## 16
```

十六进制转十进制

```bash
((num = 0xff)); echo $num     ## 255
((num = 16#ff)); echo $num    ## 255
```

这里只是其他进制转十进制的方法, 至于其他进制互转的方法, 可以使用`bc`命令.

## 2. 扩展C语言流程控制语法

这一节与单中括号与单大括号相关, 建议先了解这两者的用法.

默认情况下, shell中的条件判断如`if`, `while`, `for`的语法类似于

```bash
## if [ 1 = 1 ]; then shell命令; fi
if [ 1 == 1a ]; then echo yes; else echo no; fi

## for i in {1..10}; do shell命令; done
for i in {1..10}; do touch test$i.txt; done

## while [ a -lt 10 ]; do shell命令; done
while [ $a -lt 10 ]; do echo $a; ((a++)); done
```

有了`(())`, 可以使用类似于C语言的循环语句

**if**

```log
## 默认[]中是不可以使用`&&`与`||`操作符, 并且数值比较不能使用> < =等符号的, 双小括号中可以, 这一点与双中括号类似
$ if (( 'abc' == 'abc' || 1 == 2 )); then echo yes; else echo no; fi
yes
$ if [[ 'abc' == 'abc' || 1 == 2 ]]; then echo yes; else echo no; fi
yes
```

**for**

```log
$ for ((a=0; a<5; a++)); do echo $a; done
0
1
2
3
4
```

```log
$ a=0
$ while ((a<5)); do echo $a; ((a++)); done
0
1
2
3
4
```

> 在作判断条件时, 双括号中的内容不再局限于数字, 也可以是字符串.

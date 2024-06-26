# Linux计算命令.2.expr

参考文章

1. [Linux下的计算器(bc、expr、dc、echo、awk)知多少？](http://blog.chinaunix.net/uid-24673811-id-1760837.html)

`expr`可以对表达式进行计算, 其基本格式为

```
expr 表达式1 操作符 表达式2 操作符...
```

相应的语法规则如下:

1. 表达式与操作符用空格隔开, 所以表达式不可以包含空格
2. shell的特定字符需要使用反斜线转义
3. 数值计算有优先级但不能出现括号
4. 包含空格和其他特殊字符的项要用引号包裹起来(好像不对啊)

------

表达式包括

- 条件判断
- 数值比较
- 数值计算: 加减乘除, 求余
- 字符串比较, 索引, 截取, 长度计算

**其中数值运算要求数字必须为整数**

**示例**

```shell
## 必须要有空格
$ expr 1 + 2 + 3
6
## 否则...
$ expr 1+2+3
1+2+3
## 试试这个
$ x='1 + 2 + 3'
$ expr $x
6
## 特殊字符需要转义
$ expr 3 \* 9
27
## 计算优先级
$ expr 3 + 2 \* 2
7
## 不能用括号
$ expr ( 3 + 2 ) \* 2
-bash: syntax error near unexpected token `3'
```

## 1. 条件判断

1. `expr 表达式1 \| 表达式2`: 如果表达式1不为0或空则返回`表达式1`, 否则返回`表达式2`.
2. `expr 表达式1 \& 表达式2`: 如果两个表达式都不为0或空则返回`表达式1`, 否则返回数值0.

```log
$ x='0'
$ y='结果呢'
$ expr $x \| $y
结果呢

$ x=0
$ expr $x \| $y
结果呢

x='abc'
$ expr $x \| $y
abc
$ x='a bc'
$ expr $x \| $y
expr: syntax error
```

```
$ x=123
$ y=abc
$ expr $x \& $y
123
$ y=0
$ expr $x \& $y
0
$ x=0
$ y=abc
$ expr $x \& $y
0
```

## 2. 数值比较

`表达式1 { =, \>, \>=, \<, \<=, != } 表达式2`

如果两个表达式都是整数，返回整数比较的结果(0或1)；否则它返回的是字符串比较的结果。

```
$ x=2
$ y=3
$ expr $x \< $y
1
$ expr $x = $y
0
```

字符串比较就算了.

## 3. 数值计算

包括`+`, `-`, `\*`, `/`, `%`, 分别为加减乘除, 求余.

运算符仍然有优先级, 但不能使用括号.

## 4. 字符串操作

不在这里介绍

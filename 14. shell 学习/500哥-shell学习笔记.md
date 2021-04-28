```bash
#!/bin/bash
```

`#!`跟shell命令的完全路径，用于显示后期命令以哪种shell来执行这些命令



shell变量： 临时变量和永久变量

临时变量： shell程序内部定义，使用范围仅限于定义它的程序，对其他程序不可见

永久变量： 环境变量，其值不随shell脚本的执行结束而消息



用户自定义变量： 由字母或下划线大头，由字母、数字或者下划线组成，并且区分字母大小写

使用变量值时，要在变量名前加上前缀"$"

变量赋值，赋值号"="两边应没有空格

将一个命令的执行结果赋值给变量

```shell
A=`date`
B=$(ls -l)
```

 单引号和双引号的区别：

单引号之间的内容原封不动地指定给了变量，双引号取消了空格的作用，特殊符号的含义会保留

删除变量 : `unset` 变量名

位置变量和特殊变量：

位置变量： Shell解释执行用户的命令时，将命令行的第一个字作为命令名，而其他字作为参数。由出现在命令行上的位置确定的参数称为位置参数，使用`$N`来表示。

例如：

```shell
./example.sh file1 file2 file3
$0 这个程序的文件名 example.sh
$N 这个程序的第n个参数值
```

特殊变量： 

一些变量是一开始执行脚本时就会设定，且不能被修改，但我们不叫它只读的系统变量，而叫它特殊变量。这些变量当一执行程序时就有了，用户无法将一般的系统变量设定成只读的。例如：

```shell
$*   # 这个程序的所有参数
$#	# 这个程序的参数个数
$$   # 这个程序的PID
$!   #执行上一个后台程序的PID
$?   # 执行上一个指令的返回值


#!/bin/bash
echo "$* 表示这个程序的所有参数"
echo "$# 表示这个程序的参数个数"

touch /tmp/a.txt
echo "$$ 表示程序的进程PID"

touch /tmp/b.txt &
echo "$! 执行上一个后台指令的PID"
echo "$$ 表示程序的进程PID"
```



## 2. 算数表达式

`read`命令，从键盘读入数据，赋给变量

```shell
read a b c  # 从键盘读取三个值并赋给a b c
```

`expr`命令，Shell变量的算数运算,对**整数型变量**进行算数运算，运算符之间需要有空格

```shell
expr 1 + 2 
expr 2 \* 3 # *是特殊符号，乘号需要转义 
```

复杂运算：

```shell
var4=8
expr `expr 5 + 11` / $var4
```

```shell
var1=8
var2=2
var4=`expr $var1 / $va2`
```



变量测试语句： `test 测试条件`  , 测试范围： 整数、字符串、文件

字符串和变量：

```shell
test $str1==$str2  # 是否相等
test $str1!=$str2  # 是否不相等

test $str1 # 是否不为空

test -n $str1 # 是否为空
或者
test -z $str1
引用的过程中字符串需要加上双引号  [ -n "$str1" ]; then
```

整数：

```shell
test int1 -eq int2
test int1 -ge int2 >=
test int1 -gt int2 >
test int1 -le int2 <=
test int1 -lt int2 <
```

也可以省略`test`写成`[int -lt int2]`

文件测试：

```
test -d file #测试是否为目录
test -f file #测试是否为文件
test -x file # 是否为可执行文件
test -r file
test -w file

test -e file # 测试文件是否存在
test -s file # 测试大小是否为空，是否为空文件
```



## 3. 流程控制

`if`语句：

```shell
if 条件
then
语句
fi
```

`;`表示两个命令写在一行，互不影响，与`&&`不同，后者出现某一个命令错误时会将后面的命令中断

```shell
#!/bin/bash

if [ -x /bin/ls ]
then
/bin/ls
fi

或者

if [ -x /bin/ls ]; then
/bin/ls
fi
```

 `if/else`语法:

```shell
if 条件1 ; then
	 命令1
else 
	 命令2
fi	 
```

例如：

```shell
#!/bin/bash

if [ -x /etc/password ]; then
/bin/ls
else
pwd
fi
```



多个条件的联合：

`-a`或`&&` : 逻辑与，仅当两个条件都成立时，结果为真

`-o`或`||`:逻辑或，两个条件有一个城里，结果为真

更复杂的if语句：

```shell
if 条件1 ; then
	命令1
elif 条件2 ; then
	命令2
elif 条件3 ; then
 	命令3
else 
	命令4
fi	
```

例如：

```shell
#!/bin/bash

echo "input a file name:"
read file_name

if [ -d $file_name ]; then
echo "$file_name is a dir"
elif [ -f $file_name ]; then
echo "$file_name is a file"
elif [ -c $file_name -o -b $file_name ]; then
echo "$file_name is a device file"
else
echo "$file_name is an unknow file"
fi  
```

  

`case`流程控制语句，适用于多分支

```shell
case 变量 in 
字符串1) 命令列表1
;;
...
字符串n) 命令列表n
;;
esac
```

例如：

```shell
#!/bin/bash

echo "**************"
echo "Please selet your command:"
echo "1"
echo "2"
echo "3"
read op
case $op in
1)
echo "11111"
;;
2)
echo "22222"
;;
3)
echo "333333"
;;
*) # 参数*，匹配所有参数
echo "*******"
;;
esac
```



循环语句 `for...done`语句：

```shell
for 变量 in 名字表
do
命令列表
done
```

例如:

```shell
#!/bin/bash

for DAY in Sunday Monday Tuesday Wednesday Thursday Friday Staurday
do
	echo "The day is:$DAY"
done
```



`while`:

```shell
while 条件
do
命令
done
```

例如：

```shell
#!/bin/bash

num=1
while [ $num -le 10 ]
do
square=`expr $num \* $num`
echo $square
num=`expr $num + 1`
done
```



使用`(())`扩展`shell`算数运算的使用方法

使用`[]`时候，必须保证运算符与算数之间有空格。四则运算也能借助：`expr`命令完成。

`(())`结构语句，就是对`shell`中算数及赋值运算的扩展

语法：

```
((表达式1，表达式2......))
```

1. 在双括号中，所有表达式可以像C语言一样，如`a++,b--`
2. 在双括号中，所有变量可以不加入`$`符号前缀
3. 双括号可以进行逻辑运算，四则运算
4. 双括号扩展了`for`、`while`、`if`条件测试运算
5. 支持多个表达式运算，各个表达式之间用逗号`,`分开

例如：

```shell
#!/bin/bash

var1=1
while((var1<100))
do
echo "Valueof the variable is :$var1"
((var1=var1*2))
done

-------------
Valueof the variable is :1
Valueof the variable is :2
Valueof the variable is :4
Valueof the variable is :8
Valueof the variable is :16
Valueof the variable is :32
Valueof the variable is :64
```



循环语句嵌套：

例如：

```shell
#!/bin/bash

echo -n "请输入行数:"
read line
echo -n "请输入分隔符:"
read char
a=1
while [ $a -le $line ]
do
	b=1
	while [ $b -le $a ]
		do
			echo -n "$char"
			b=`expr $b + 1`
		done
	 echo	
   a=`expr $a + 1`
 done
 
```

```shell
#!/bin/bash

read -p "请输入行号：" line
for((i=1;i<line;i++))
do
	for((j=$line-$i;j>0;j--))
		do
			echo -n " "
		 done
   for((k=1;k<=$((2*$i-1));k++))
   	do
   		echo -n '*'
   	done
    echo
done    
   		
```



`break`和`continue`：

```shell
#!/bin/bash

echo -n "请输入命令值："
read char
while true
do
if [ $char != "Q" ]; then
echo -n "请输入正确的命令值："
read char
continue
else
echo -n "恭喜你，输入成功!"
break
fi
done
```



## 4. shell函数

`Shift`：每执行一次，参数序列顺次左移一个位置，$#的值减1，用于分别处理每个参数，移出去的参数，不再可用

```shell
做一个加法计算器，求出所有参数的和

#!/bin/bash

if [ $# -le 0 ]; then
echo "error!参数不足!"
exit 124
fi
sum=0
while [ $# -gt 0 ]
do
sum=`expr $sum + $1`
shift
done
echo $sum
```

`shell函数`

函数：把一个功能封装起来，使用时直接调用

语法：

```shell
函数名()
{
命令序列
}

或
function 函数名()
{
命令序列
}

注意： 函数调用过程不需要加()
```

调用语法:

```shell
函数名 参数1 参数2 参数3
```

函数中的变量均为全局变量，没有局部变量，调用函数时，可以传递参数，使用`$N`来获取

例如：

```shell
#!/bin/bash
abc=123
echo $abc

example()
{
   abc=456
}
example
echo $abc
```

```shell
#!/bin/bash

example()
{
echo $1
echo $2
}

example aaa bbb
```



案例1 ： 自动备份Mysql脚本

```shell
#!/bin/bash

BAKDIR=/data/backup/mysql/`date +%Y-%m-%d`
MYSQLDB=test
MYSQLUSER=root
MYSQLPW=root

if [ $UID -ne 0 ]; then
echo "这个脚本需要管理员权限"
sleep 2
exit 0
fi
if [ ! -d $BAKDIR ]; then
mkdir -p $BAKDIR
else
	echo "文件夹已存在，开始备份...."
fi
/usr/bin/mysqldump -u$MYSQLUSR -p$MYSQLPW -d > $BAKDIR/webapp_db.sql
cd $BAKDIR ; tar -czf webapp_mysql_db.tar.gz *.sql

find . type -f -name *.sql | xargs rm -rf
或
find . -type -f -name *.sql -exec rm -rf {} \ ;
[ $? -eq 0 ] && echo "This `date +%Y-%m-%d` Mysql Backup is Success"
cd /data/backup/mysql ; find . -type d -mtime +30 | xargs rm -rf
echo "The mysql backup successfully"
```



案例2: 自动解压`ZIP`包

`xargs` : 能够捕获一个命令的输出，然后传递给另外一个命令。

```shell
#!/bin/bash

PATH1=/data/backup
PATH2=/data/test
cd $PATH1
for i in `find . -name "*.zip" | awk -F . '{print $2}'`
do
upzip -o .$i.zip -d $PATH2$i
done
```

`awk -F . '{print $2}'` :    awk操作列，以`.`作为分隔符输出第2列   $2 表示第二列

## 5. 正则表达式 、 sed 、 awk

`grep`   

`-v`  ： 不匹配

`-n`： 显示行号

1. 正则表达式中特殊字符
   - `^word`：待搜寻的字符串(word)在行首
   - `word$`：待搜寻的字符串(word)在行尾
   - `\`： 去掉特殊符号的意义   
   - `*`：重复零个到无穷多个前一个字符
   - `[list]`：字符集合，里面列出想要的字符
   - `^[^#]`：不以`#`开头
   - `[n-m]`：搜索字符集合内的字符范围
   - `^[a-z]`：以小写字母开头
   - `^[^a-zA-Z]`:不以英文字母开头
2. `.`代表绝对有一个任意字符的意思，而`*`代表重复前一个字符到无穷的,  任意一个长度的字符表示的方法:`.*`



`sed` : `stream editor`流编辑器，是一行一行的处理文件内容的，正在处理的内容存放在模式空间(缓冲区)内，处理完成后按照选项的规定进行输出或文件( `>`)的修改

`-n`：抑制自动(默认的)输出，读取下一个输入行

`-i`:编辑文件内容

`a`:在匹配后追加

`i`:在匹配后插入
`p`:打印

`c`:替换

`d`：删除

`w`：另存

`h/H`：拷贝

`g/G`:粘贴，从存放空间取回/追加到模式空间

例如：

输出第三行:	`sed -n '3p' /test.sh`

输出前三行： `sed` -n '1,3p' /test.sh 

输出除了前三行以外的内容： `sed -n '1,3!p' /test.sh`

输出第三行和之后三行的内容 `sed -n '3,+3p' /test.sh`

在文件首部插入`**` :   `sed '1i**' /test.sh`

在文件末尾插入`**` :  `sed '$a**' /test.sh`

替换第三行内容 ： `sed '3c%%%' /test.sh`

第二行到第四行的内容复制到文件末尾： `ser=d '2,4H;$G' /test.sh`

将`fstab`中包含`ext4`的记录行写入新的文件中： `sed '/ext4/w newfstab2' /etc/fstab`



`awk`：是一种优良的文本处理工具，Linux及Unix环境中现有的功能最强大的数据处理引擎之一。任何awk语句都由模式和动作组成，一个awk脚本可以有多个语句，模式决定动作语句的触发条件和出发时间

`awk -F分隔符 'BEGIN { 初始化 } { 循环逐行执行部分 } END { 结束处理 }' file_list1 file_list2`

特殊字段：

`BEGIN`:语句设置计数和打印头部信息，在任何动作之前进行

`END`:语句输出统计结果，在完成动作之后执行

在awk中，分隔符默认为空格，可以用`-F`进行改变， 例如改变成逗号 `-F`

NF = 字段数量(列数) 、 NR = 记录数量(行数)

例如：

在定义年月日显示方式：  ` date | awk '{print "Year:"$6 "\tmonth:"$2 "\tday:"$3 }` 

显示所有内容 ` awk '{print $0}'`

以`:`作为分隔符，显示第一行`awk -F: '{print $1}'`

显示第一列到第三列  ` awk '{print $1,$3}'`

打印一个文件头，打印一个文件尾： `awk 'BEGIN {print "name level result \n"}{print $1,$2,$3} END{print "end of class1"}' result`

第二列大于5的数据： `awk $2 >=5 {print $0}`



LeetCode: 

```shell
#答案1：
cat words.txt|tr -s ' ' '\n'|sort |uniq -c|sort -r|awk '{print $2,$1}'

#答案2：
[root@localhost ~]# awk '{for(i=1;i<=NF;i++){print $i}}' words.txt 
the
day
is
sunny
the
the
the
sunny
is
is

#答案：
awk '{for(i=1;i<=NF;i++){asso_array[$i]++;}};END{for(w in asso_array){print w,asso_array[w];}}' words.txt
```

 **awk按行读取文件,有如下例子进行说明：**

```shell
#!/bin/bash

awk '{print $1}'  # 读取第一列，如果有多行，那么每一行的第一列都会被输出
awk '{for(i=1;i<=NF;i++){print $i}}' # 从第一行开始运行for循环语句，获取最大列数，逐个输出，接着第二行运行for循环语句

```


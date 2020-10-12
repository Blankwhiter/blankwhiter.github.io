shell学习
写在前面：#!/bin/bash开头都是以.sh 执行脚本的内容

# 1. 变量替换
- ${变量名#匹配规则}  从头开始匹配，最短删除
- ${变量名##匹配规则}  从头开始匹配，最长删除
- ${变量名%匹配规则}  从尾开始匹配，最短删除
- ${变量名%%匹配规则}  从尾开始匹配，最长删除
- ${变量名/旧字符串/新字符串}  替换旧的字符串为新字符串，只替换第一个
- ${变量名//旧字符串/新字符串}  替换旧的字符串为新字符串，替换所有

### 1.1 定义变量并输出显示
```
[root@iZbp1d0jkped5bei97jkheZ ~]# varivable_1="I love you,Do you love me"
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${varivable_1}
I love you,Do you love me
```
###  1.2 ${变量名#匹配规则} 
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var1=${varivable_1#*ov}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $var1
e you,Do you love me
```
###  1.3 ${变量名##匹配规则} 
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var2=${varivable_1##*ov}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $var2
e me
```

### 1.4 ${变量名%匹配规则} 
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var3=${varivable_1%ov*}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${var3}
I love you,Do you l
```
###  1.5 ${变量名%%匹配规则} 
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var4=${varivable_1%%ov*}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${var4}
I l
```

###  1.6 ${变量名/旧字符串/新字符串}
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var5=${varivable_1/l/L}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${var5}
I Love you,Do you love me
```

###  1.7 ${变量名//旧字符串/新字符串} 
```
[root@iZbp1d0jkped5bei97jkheZ ~]# var6=${varivable_1//l/L}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${var6}
I Love you,Do you Love me
```


# 2. 字符串处理
## 2.1 获取字符串长度
- ${#string}
- expr length "$string"

###  2.1.1
```
[root@iZbp1d0jkped5bei97jkheZ ~]# echo ${#varivable_1}
25
```

###  2.1.2
```
[root@iZbp1d0jkped5bei97jkheZ ~]# expr length "$varivable_1"
25
[root@iZbp1d0jkped5bei97jkheZ ~]# len=`expr length "$varivable_1"`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $len
25
```


## 2.2 获取子串在字符串中的索引位置
- expr index "$string" "$substring"  [不是子串，而是子串切分成单个字符，查找第一个出现的字符的位置]

```
[root@iZbp1d0jkped5bei97jkheZ ~]# index=`expr index "$varivable_1" Do`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $index
4
[root@iZbp1d0jkped5bei97jkheZ ~]# index=`expr index "$varivable_1" DoD`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $index
4
```
注： 可以看出它是拆分一个个的字符一旦匹配到第一个就返回的


## 2.3 获取子串长度
- expr match "$string" "$substring"  [必须从头开始匹配，能匹配到的子串返回长度，支持正则]

```
[root@iZbp1d0jkped5bei97jkheZ ~]# index=`expr match "$varivable_1" Do`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $index
0

[root@iZbp1d0jkped5bei97jkheZ ~]# index=`expr match "$varivable_1" I`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $index
1
```
注：可以看出它是从头开始匹配的 


## 2.4 子串抽取
- ${string:position}  从string的position位置开始
- ${string:position:length}  从position位置开始抽取length长度
- ${string: -position}  从右边开始匹配
- ${string:(position)}  从左边开始匹配
- expr substr "$string" "$position" "$length"  从position位置开始抽取length长度，索引从0开始


```
[root@iZbp1d0jkped5bei97jkheZ ~]# substr=${varivable_1:11}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $substr
Do you love me
```

```
[root@iZbp1d0jkped5bei97jkheZ ~]# substr=${varivable_1:11:2}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $substr
Do
```

```
[root@iZbp1d0jkped5bei97jkheZ ~]# substr=${varivable_1: -7}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $substr
love me
```

```
[root@iZbp1d0jkped5bei97jkheZ ~]# substr=${varivable_1:(-7)}
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $substr
love me
```


```
[root@iZbp1d0jkped5bei97jkheZ ~]# substr=`expr substr "${varivable_1}" 12 15`
[root@iZbp1d0jkped5bei97jkheZ ~]# echo $substr
Do you love me
```
注：这里的12索引计算从1开始计算
 

## 3. 字符串练习
 >string="bigdata process framework is hadoop, hadoop is an open source project"
执行脚本后，打印输出string字符串变量，并给出用户以下选项：
(1) 打印string长度
(2) 在整个字符串中删除Hadoop
(3) 替换第一个Hadoop为Mapreduce
(4) 替换全部Hadoop为Mapreduce
用户输入对应的数字会执行相应的功能，输入q|Q退出操作

例子：
```
#!/bin/bash
string="Bigdata process framework is Hadoop,Hadoop is an open source project"

function print_tips {
    echo "******************************************"
    echo "***  (1) 打印string长度"
    echo "***  (2) 在整个字符串中删除Hadoop"
    echo "***  (3) 替换第一个Hadoop为Mapreduce"
    echo "***  (4) 替换全部Hadoop为Mapreduce"
    echo "******************************************"
}

function print_len {
    echo "${#string}"
}

function del_Hadoop {
    echo "${string//Hadoop/}"
}

function rep_Hadoop_to_Mapreduce_first {
    echo "${string/Hadoop/Mapreduce}"
}

function rep_Hadoop_to_Mapreduce_alll {
    echo "${string//Hadoop/Mapreduce}"
}

while true
do
    echo
    echo
    echo "【string=$string】"
    print_tips

    read -p "please input your choice(1|2|3|4|q|Q): " choice

    case $choice in
            1)
                    echo
                    print_len
                    ;;
            2)
                    echo
                    del_Hadoop
                    ;;
            3)
                    echo
                    rep_Hadoop_to_Mapreduce_first
                    ;;
            4)
                    echo
                    rep_Hadoop_to_Mapreduce_alll
                    ;;
            q|Q)
                    exit
                    ;;
            *)
                    echo
                    echo "error input!"
                    ;;
    esac
done
```



# 4. 命令替换
- \`command\`
- $(command)

$(())主要用来进行整数运算，包括加减乘除

###  例1：获取系统中的所有用户并输出(/etc/passwd)
```
cat /etc/passwd | cut -d ":" -f 1
```

###  command.sh
```
#!/bin/bash
index=1
for user in `cat /etc/passwd | cut -d ":" -f 1`
do
    echo "this is $index user: $user"
    index=$(($index+1))
done
```

注：cut命令：从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的
范围之内，该字符将被写出；否则，该字符将被排除


###  例2：根据系统时间计算今年、明年
```
echo "this is $(date +%Y), next year is $(($(date +%Y)+1))"
```
注：打印年月日 `echo $(date +%Y%m%d)` ，等同于 `date +%Y%m%d`

###  例3：根据系统时间获取今年还剩下多少个星期，已经过了多少个星期
```
echo "今年已经过了$(date +%j)天，合$(($(date +%j) / 7))周，还剩下$(((365-$(date +%j)) / 7))周"
```

###  例子4：判断nginx进程是否存在，不存在的话拉起该进程
```
#!/bin/bash
nginx_process_num=$(ps -ef | grep nginx | grep -v grep | wc -l)
if [ $nginx_process_num -eq 0 ]; then
    systemctl start nginx
fi
```

# 5. 有类型变量
- declare、typeset  [+/-] [变换选项] 变量名


选项
>-r 只读
-i 整数
-a 数组
-f 在脚本中显示定义的函数和函数体
-F 在脚本中显示定义的函数
-x 申明环境变量
-p 打印除所有的变量
如果要取消类型声明，减号变加号就行了

例子：
```
[root@localhost home]# declare -r variable_2="test read"
[root@localhost home]# variable_2="test write"
-bash: variable_2: readonly variable
[root@localhost home]# declare +r variable_2
-bash: declare: variable_2: readonly variable
"
```
注：一旦设置成只读，就完全不能对它操作了。好在这个变量只是临时生效的，系统一旦重启就消失了。

```
[root@localhost home]# num1=1
[root@localhost home]# echo $num1+1
1+1
[root@localhost home]# declare -i num2=2
[root@localhost home]# echo $num2+1
2+1
[root@localhost home]# declare -i num3
[root@localhost home]# num3=$num2+1
[root@localhost home]# echo $num3
3
```
注：可以看出来 当你需要声明了整数类型时需要计算 仍需要一个整数类型来接收结果，不然会被当成字符串拼接。如果变量被声明成整数，把一个结果不是整数的表达式赋值给它时，就会变成0.如果变量被声明成整数，把一个小数（浮点数）赋值给它时，也是不行的。因为bash并不内置对浮点数的支持。 

```
[root@localhost home]# array=("jane" "jone" "jack" "jordan")
[root@localhost home]# echo ${array[@]}
jane jone jack jordan  // 输出数组内容
 

[root@localhost home]# echo ${array[0]}
jane    //输出下标对应的内容
[root@localhost home]# echo ${#array}
4     // 获取数组长度 
[root@localhost home]# echo ${#array[0]}
4 //获得对应内容的长度

```

```
[root@localhost home]# declare -x jdk_version=java8
[root@localhost home]# env
XDG_SESSION_ID=45
HOSTNAME=localhost.localdomain
TERM=xterm
SHELL=/bin/bash
OLDPWD=/root
SSH_TTY=/dev/pts/2
USER=root
........
PWD=/home
jdk_version=java8
LANG=en_US.UTF-8
HISTCONTROL=ignoredups
SHLVL=1
HOME=/root
```
注：`declare -x 变量名=变量值` 等价于 `export 变量名=变量值`

```
[root@localhost home]# declare -p
declare -- BASH="/bin/bash"
declare -ir BASHPID
declare -A BASH_ALIASES='()'
declare -a BASH_ARGC='()'
declare -a BASH_ARGV='()'
declare -A BASH_CMDS='()'
declare -- BASH_COMMAND
declare -a BASH_LINENO='()'
declare -a BASH_SOURCE='()'
......
```


# 6. 数字运算
- expr num1 operator num2
- ((num1 operator $num2))

注意：expr只支持整型运算
- expr 操作符
>num1 | num2     num1不为空且不为0，返回num1，否则返回num2
num1 & num2     num1不为空且不为0，返回num1，否则返回0
num1 < num2     num1小于num2，返回1，否则返回0
num1 <= num2    num1小于等于num2，返回1，否则返回0
num1 = num2     num1等于num2，返回1，否则返回0
num1 != num2    num1不等于num2，返回1，否则返回0
num1 > num2     num1大于num2，返回1，否则返回0
num1 >= num2    num1大于等于num2，返回1，否则返回0
num1 + num2
num1 - num2
num1 * num2
num1 / num2
num1 % num2

例子：
```
[root@localhost home]# num1=10
[root@localhost home]# num2=20 
[root@localhost home]# expr $num1 + $num2
30
[root@localhost home]# echo $(($num1 + $num2))
30
[root@localhost home]# 
[root@localhost home]# expr $num1 \| $num2
10
[root@localhost home]# expr $num1 \& $num2
10
[root@localhost home]# expr $num1 \> $num2
0
[root@localhost home]# expr $num1 \>= $num2
0
[root@localhost home]# expr $num1 \< $num2
1
[root@localhost home]# expr $num1 \<= $num2
1
[root@localhost home]# expr $num1 = $num2
0
[root@localhost home]# expr $num1 + $num2
30
[root@localhost home]# expr $num1 - $num2
-10
[root@localhost home]# expr $num1 \* $num2
200
[root@localhost home]# expr $num1 / $num2
0
[root@localhost home]# expr $num1 % $num2
10

```
注：或 与 大于 小于 乘法需要转义

###  练习：提示用户输入一个正整数num，然后计算1+2+3+...+num的值，必须判断num是否为正整数，不符合允许再次输入
```
#!/bin/bash
sum=0
while true
do
    read -p "please input: " num
    expr $num + 1 &> /dev/null

    if [ $? -eq 0 ]; then
        if [ `expr $num \> 0` -eq 1 ]; then

            for ((i=0;i<=$num;i++))
            do
                sum=$(($sum + $i))
            done

            echo $sum
            exit
        else
            echo "小于等于0"
            continue
        fi
    else
        echo "不是整数"
        continue
    fi
done
```


# 7. 函数定义和使用
## 7.1 函数定义
```
第一种：
function name1 {
}

第二种：
name2() {
}

```

###  例：写一个nginx的守护脚本
```
#!/bin/bash

this_pid=$$
while true
do
        ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null
        if [ $? -eq 0 ]; then
                echo "nginx is running well!"
                sleep 3
        else
                echo "starting!"
                systemctl nginx start
                sleep 1
        fi
done
```

注：
>$$ ：当前线程pid
     grep -v grep ： 过滤去掉查询ngxin命令所占的线程 
     grep -v $this_pid ： 过滤去掉本身脚本所占的线程
     sleep 3 ：睡眠3秒
     $? ： 获取函数的返回值




## 7.2 传递参数

### 例：写一个脚本支持加减乘除四种运算（cal.sh）
```
#!/bin/bash

function cal {
    case $2 in
        +)
            echo $(($1 + $3))
            ;;
        -)
            echo $(($1 - $3))
            ;;
        \*)
            echo $(($1 * $3))
            ;;
        /)
            echo $(($1 / $3))
            ;;
        *)
            echo "error input!"
            ;;
    esac
}

cal $1 $2 $3
```

测试
```
[root@localhost home]# sh cal.sh 1 + 2
3
```

注：方法内传参 跟 方法调用直接使用$1 $2 顺序获取参数。

## 7.3 返回值
- return返回值，只能返回1-255之内的整数。使用return返回值，通常供其他地方调用获取状态，因此通常返回0或者1，0表示成功，1表示失败。return表示return 0
- echo返回值，可以返回任何字符结果，通常用于返回数据，比如一个字符串或列表值

###  例子：判断nginx是否正在执行的脚本
```
#!/bin/bash

this_pid=$$
function is_nginx_running {
    
    ps -ef | grep nginx | grep -v grep | grep -v $this_pid &> /dev/null

    if [ $? -eq 0 ]; then
        return
    else
        return 1
    fi
}

is_nginx_running && echo "nginx is running" || echo "nginx is down"
```



### 例子：输出用户列表
```
#!/bin/bash
function get_user_list {
    users=$(cat /etc/passwd | cut -d ":" -f1)
    echo $users
}

index=1
user_list=$(get_user_list)
for u in $user_list
do
    echo "this is the $((index++)) user: $u"
done
```

## 7.4 变量
- 使用local定义变量表示局部变量，否则一般的变量都是全局变量
- 函数内部的变量如果跟外部变量同名，则函数内部的变量替换外部的变量

## 7.5 函数库
例：定义一个函数库，该函数库实现以下几个函数
>(1)加法函数 add
(2)减法函数 reduce
(3)乘法函数 multiple
(4)除法函数 dived
(5)打印系统运行情况的函数sys_load，该函数可以显示系统内存运行情况

### 文件名 base_function.lib(放在了/root/script目录下，读者可定义放置位置)，内容如下：
```
#!/bin/echo
function add {
    echo "`expr $1 + $2`"
}
function reduce {
    echo "`expr $1 - $2`"
}
function multiple {
    echo "`expr $1 \* $2`"
}
function divide {
    echo "`expr $1 / $2`"
}
function sys_load {
    echo "---memory info---"
    free -m

    echo
    echo "---disk info---"
    df -h
}
```

### calculate.sh
```
#!/bin/bash
. /root/script/base_function.lib
add 1 2
reduce 11 33
multiple 3 44
divide 20 2
sys_load
```
执行只需 `sh calculte.sh`

- 库文件的后缀是任意的，但是一般以.lib使用
- 库文件通常没有执行权限
- 库文件无需和脚本在同级目录，只需在脚本中引用时指定绝对路径即可
- 第一行一般使用#!/bin/echo,输出警告信息，避免用户执行

 注：sh -x calculate.sh 查看shell执行过程


# 8. 常用查找命令
## 8.1 find [路径] [选项] [操作]

###  选项
```
 -name                根据名字
-iname                根据名字，不区分大小写
-perm                 根据权限
-prune                该选项可以排除某些查找目录
-user                 根据文件用主
-group                根据文件属组
-mtime -n|+n          根据文件更改时间，-n表示n天以内修改的文件，+n表示n天以外修改的文件
-mmin -n|+n           根据文件更改时间，-n表示n分钟以内修改的文件，+n表示n分钟以外修改的文件
-newer file1 ! file2  比file1新但是比file2旧的文件
-type                 根据文件类型    f-文件，d-目录，l-管道文件，c-字符设备文件，b-块设备文件，p-管道文件
-size -n +n           根据文件大小 -n 文件大小大于n的文件 ，+n 文件大小小于n的文件 
-mindepth n           从n级子目录开始搜索
-maxdepth n           最多搜索到n级子目录
-a                    与
-o                    或
!|-not                非
```

### 操作
```
-print         默认操作
-exec          对搜索到的文件执行特定操作，格式为 -exec command {} \; ，其中{}代表搜索到的文件，如：find . -name "*.conf" -exec rm -rf {} \;
-ok            跟-exec一样，只是每次操作都会给用户提示
```

### 例：将/var/log目录下以log结尾，且更改时间在7天以上的删除
```
find /var/log -name "*log" -mtime +7 -exec rm -rf {} \;
```

注：该命令可以达到很多效果，在此处不再一一赘述，总的来说 就是从哪里 按照什么样的规则 找到符合条件的结果 而后又想对这些结果进行什么操作的效果。

## 8.2 locate which whereis

```
locate         不同于 find，find会查找整个磁盘（名称全部匹配），而locate命令会在数据库中查找（名称部分匹配），只能查找单个文件。可以用updatedb更新数据库文件，该文件是 /var/lib/mlocate/locate.db
whereis        -b，只返回二进制文件，-m，只返回帮助文档文件，-s，只返回源码文件
which          只查找二进制文件
```
注：本人比较少用到 ，请自行实验。

命令场景对比
- find  查找某一类文件 功能强大 速度慢
- locate 只能查找单个文件 功能单一 速度快
- whereis 查找程序的可执行文件，帮助文档等 不常用
- which 值查找程序的可执行文件 常用于查找文件的绝对路径


## 接下来会介绍三种命令  grep的查找、 sed的编辑、 awk的报表

# 9. grep
- grep [option] [pattern] [file1,file2]
- command | grep [option] [pattern]
- egrep 和 grep -E 等价

### 参数（option）
>-i     不区分大小写
-v     反向选择 只打印没有匹配的，而匹配的反而不打印。
-n     显示行号
-r     递归
-E     支持扩展正则表达式
-F     不按正则表达式，按字符串字面匹配
-x     匹配整行
-w     匹配整词 被匹配的文本只能是单词，而不能是单词中的某一部分，如文本中有liker，而我搜寻的只是like，就可以使用-w选项来避免匹配liker
-c     只输出匹配到的总行数，不显示具体内容
-l     只列出文件，不显示具体内容
--color :将匹配到的内容以颜色高亮显示。
-A  n：显示匹配到的字符串所在的行及其后n行，after
-B  n：显示匹配到的字符串所在的行及其前n行，before
-C  n：显示匹配到的字符串所在的行及其前后各n行，context

例子：
```
[root@localhost home]# grep "root" /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost home]# grep -i "ROOT" /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost home]# grep -n "root" /etc/passwd
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nologin
[root@localhost home]# grep -c "root" /etc/passwd
2
[root@localhost home]# grep -o "roo*" /etc/passwd
roo
roo
roo
roo
ro
ro

```

注： grep 更多的使用，请参考 https://www.cnblogs.com/flyor/p/6411140.html


### 例：grep -E "java|JAVA" file(某个文件)


# 10. sed(stream editor|流编辑器的缩写)
- sed [option] "pattern command" file
- stdout | sed [option] "pattern command" file

sed工作模式** 逐行匹配操作**

### 先在本地准备2份测试文件 
### 1.sed.txt
内容：
```
i love java
I love JAVA
so what ?
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.

```
### 2.edit.sed
```
/java/p
/so/p
```

###  option
>-n     只打印模式匹配行。
-e     直接在命令行编辑。多重处理的时候用-e连接
-f     指定编辑处理的 pattern command 内容。 
-r     pattern支持扩展正则表达式。
-i     对源文件进行修改

```
[root@localhost home]# sed -n "/java/p" sed.txt
i love java
[root@localhost home]# sed -n -e "/JAVA/p" -e "/java/p" sed.txt
i love java
I love JAVA
[root@localhost home]# sed -n -f edit.sed sed.txt 
i love java
so what ?
[root@localhost home]# sed -n -r "/java|JAVA/p" sed.txt
i love java
I love JAVA

```



### 例：替换文件中love为like
```
[root@localhost home]# sed -n "s/love/like/g;p" sed.txt
i like java
I like JAVA
so what ?
[root@localhost home]# cat sed.txt 
i love java
I love JAVA
so what ?
//可以看出来这里并不会直接影响到原文本文件

[root@localhost home]# sed -i "s/love/like/g" sed.txt
[root@localhost home]# cat sed.txt 
i like java
I like JAVA
so what ?

```
注：`s/aa/bb/g`  s/ / /g 写法是固定的,可参考`command`

### pattern
```
10command                匹配第10行
10,20command             匹配第10-20行
10,+5command             匹配第10-16行
/pattern1/command        匹配pattern1的行。sed -n "/\/spool\//p" /etc/passwd，匹配带有/spool/的行。sed -n "/^daemon/p" /etc/passwd，匹配daemon开头的行
/pattern1/,/pattern2/command     匹配pattern1到pattern2的行结束。sed -n "/^daemon/p" /etc/passwd
10,/pattern1/command        从第10行开始匹配到第一个pattern1的行结束
/pattern1/,10command        从匹配到pattern1的行到第10行，如果匹配到pattern1的行大于10行，则只会选择匹配到pattern1的那一行
```

例子
```
[root@localhost home]# sed -n '2p' sed.txt 
I like JAVA

[root@localhost home]# sed -n '2,5p' sed.txt 
I like JAVA
so what ?
but C languge is the best ,
i say the C languge is the best,

[root@localhost home]# sed -n '2,+5p' sed.txt 
I like JAVA
so what ?
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.

[root@localhost home]# sed -n '/but/p' sed.txt 
but C languge is the best ,

[root@localhost home]# sed -n '/but/,/words/p' sed.txt 
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.

[root@localhost home]# sed -n '2,/but/p' sed.txt 
I like JAVA
so what ?
but C languge is the best ,

[root@localhost home]# sed -n '/but/,3p' sed.txt 
but C languge is the best ,

[root@localhost home]# sed -n '/but/,10p' sed.txt 
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.

[root@localhost home]# sed -n '/but/,2p' sed.txt 
but C languge is the best ,

```

### command
```
查询
    p     打印
增加
    a     行后追加。sed -i "/\/bin\/bash/a this user can login to system" passwd
    i     行前追加
    r     外部文件读入，行尾追加
    w     匹配行写入外部文件
删除
    d     删除不能登录的用户，sed -i "/\/sbin\/nologin/d" passwd。删除从mail开头的行到ftp开头的行，sed -i "/^mail/,/^ftp/d" passwd
    例：删除配置文件中的所有空行和注释行。sed -i "/^$/d;/[:blank:]*#/d;/\t/d" nginx.conf，[:blank:]匹配空格，\t匹配tab
    例：在配置文件中所有不以#开头的行前面添加*符号（#开头的行不添加）。sed -i "s/^\([^#]\)/\*\1/g" nginx.conf 或者 sed -i "s/^[^#]/*&/g" nginx.conf
更改
    s/old/new/      将行内第一个替换
    s/old/new/g     将行内所有替换
    s/old/new/2     将行内第二个替换
    s/old/new/2g    从第二个开始替换所有的
    s/old/new/ig    忽略大小写
    例：删掉所有的数字。sed -i "s/[0-9]+//g" sed.txt
其它
    =        显示行号 sed -n "/\/sbin\/nologin/=" passwd
```
例子：
```
[root@localhost home]# sed -i '/?/a ??' sed.txt 
[root@localhost ~]# cat sed.txt 
cat: sed.txt: No such file or directory
[root@localhost ~]# cat /home/sed.txt 
i like java
I like JAVA
so what ?
??
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.

```
上方输出可以看出文本中第四行上 多加了一行 `？？`



```
[root@localhost ~]# cat /home/sed.txt 
i like java
I like JAVA
then
so what ?
??
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.
```
上方输出可以看到在第三行上 多了一行 `then`


```
[root@localhost home]# sed -i '/so/r edit.sed' sed.txt 
[root@localhost ~]# cat /home/sed.txt 
i like java
I like JAVA
then
so what ?
/java/p
/so/p
??
but C languge is the best ,
i say the C languge is the best,
i say the C languge is the best,
i say the C languge is the best.
the impotant words , say three times.

```

上方输出可以看到`so what` 下面多了两行,即`edit.sed`文件的内容，文件名并不是指定要sed后缀。


```
[root@localhost home]# sed -i '/so/w out.txt' sed.txt 
[root@localhost home]# cat out.txt 
so what ?
/so/p
```

上方可以看到当前目录下会多处一个out.txt，内容就是匹配到的内容


```
[root@localhost home]# sed -i '/so/d' out.txt 
[root@localhost home]# cat out.txt 
```
上方可以看到out.txt内容又被删除掉了

```
[root@localhost home]# sed -i 's/the/THE/g' sed.txt 
[root@localhost home]# cat sed.txt 
i like java
I like JAVA
THEn
so what ?
/java/p
/so/p
??
but C languge is THE best ,
i say THE C languge is THE best,
i say THE C languge is THE best,
i say THE C languge is THE best.
THE impotant words , say three times.
```
上方可以看到内容都`the`被替换成`THE`。其他的替换就不一一测试了。



## 10.2 反向引用
- & 和 1 引用模式匹配到的整个串（1的时候要替换的模式匹配中的串要用小括号包围起来）
  
# 例：单词中追加
将这句话中的C ： ` i say THE C languge is THE best,` 替换成C++ ： `i say THE C++ languge is THE best,`


```
[root@localhost home]# sed -i 's/TH..C/&++/g' sed.txt 
[root@localhost home]# cat sed.txt 
i like java
I like JAVA
THEn
so what ?
/java/p
/so/p
??
but C languge is THE best ,
i say THE C++ languge is THE best,
i say THE C++ languge is THE best,
i say THE C++ languge is THE best.
THE impotant words , say three times.

```
上方看到给能匹配到Th..C的全替换成上C++


注：sed更多的例子 请参考 https://www.cnblogs.com/ctaixw/p/5860221.html

### 例：统计mysql配置文件各配置段的数量
```
#!/bin/bash

FILE_NAME=/root/script/my.cnf
function get_all_segament {
    sed -n '/\[.*\]/p' $FILE_NAME | sed -e "s/\[//g" | sed -e "s/\]//g"
    # 查找[开头]结尾的行，并且删除掉[和]
}

function get_all_segament_count {
    count=`sed -n "/\[$1\]/,/\[.*\]/p" $FILE_NAME | grep -v "^#" | grep -v "^$" | grep -v "\[.*\]" | wc -l`
    # 查找[$1]开头的行到发现[.*]的行结束，去掉#开头和空行，并去掉[.*]的行（即开头和结束的行），统计数量
    echo $count
}

index=0
for segament in $(get_all_segament)
do
    index=`expr $index + 1`
    count=`get_all_segament_count $segament`
    echo "$index: $segament $count"
done
```

输出：
```
1: client 1
2: mysql 1
3: mysqld 6
4: mysqldump 3
```

# 11. awk
- awk 'BEGIN{}pattern{commands}END{}' file_name
- stdout | 'BEGIN{}pattern{commands}END{}'

```
BEGIN{}       正式处理之前执行的
pattern       匹配模式
{commands}    执行命令（可能多行）
END{}         处理完所有的匹配数据之后执行
```

## 11.1 内置变量
```
$0          整行内容
$1-$n       本行中按照某个字符分隔后的第n个变量
NF          当前行的字段个数，也就是列的个数（Number Field）
NR          当前行的行号，从1开始计算（Number Row）
FNR         多文件处理时，每个文件行号单独计数，都是从1开始（File Number Row）
FS          输入字段分隔符，不输入默认是空格或者tab键分隔（Field Separator)
RS          输入行分隔符。默认回车换行（Row Separator)
OFS         输出字段分隔符。默认空格
ORS         输出行分隔符。默认回车
FILENAME    处理的文件名
ARGC        命令行参数个数
ARGV        命令行参数数组
```

### 例：打印/etc/passwd文件的内容
test.txt 内容如下：
```
this is the test -- document 
so we can see it the changes -- changes
we can learn some code  by using this  -- using
the - code - is - shell -  languge 
grep & sed & awk
```


```
awk '{print $0}' /etc/passwd
awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
awk 'BEGIN{FS=":"}{print NF}' /etc/passwd
awk '{print NR}' /etc/passwd nginx.conf
awk '{print FNR}' /etc/passwd nginx.conf
awk 'BEGIN{RS="--"}{print $0}' test.txt
echo "a-b-c--d-e-f--g-h-i" | awk 'BEGIN{RS="--";FS="-"}{print $3}'
echo "a-b-c--d-e-f--g-h-i" | awk 'BEGIN{RS="--";FS="-";ORS="|";OFS="&"}{print $1,$2,$3}' #必须用逗号分隔，否则输出字段分隔符不会起作用。a&b&c|d&e&f|g&h&i
awk '{print FILENAME}' nginx.conf #文件有多少行就会输出多少次
awk '{print ARGC}' /etc/passwd test.txt
```

### 11.2 printf 格式化输出
```
%s         打印字符串
%d         打印十进制数字
%x         打印十六进制
%f         打印浮点型
%o         打印八进制
%e         打印科学计数法
%c         打印ascii码
-          左对齐
+          右对齐
#          显示八进制在前面加0，十六进制在前面加0x
```

例：
```
awk 'BEGIN{FS=":"}{printf "%s\n",$1}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%-20s %-20s\n",$1,$7}' /etc/passwd
awk 'BEGIN{FS=":"}{printf "%#o\n",$3}' /etc/passwd
```
## 11.3 模式匹配的两种方式
- 正则表达式
- 按关系运算符匹配

```
>
<
==        可以用于数值和字符串
<=
>=
!=
~         匹配正则表达式
!~        不匹配正则表达式
&&
||
!
```

例：
```
匹配/etc/passwd中包含root的行。    awk '/root/{print $0}' /etc/passwd
匹配/etc/passwd中以root开头的行。    awk '/^root/{print $0}' /etc/passwd

匹配/etc/passwd中第3个字段大于50的行。    awk 'BEGIN{FS=":"}$3>50{printf "%s\n",$0}' /etc/passwd

匹配/etc/passwd中第7个字段等于/sbin/nologin的行。    awk 'BEGIN{FS=":"}$7=="/sbin/nologin"{printf "%s\n",$7}' /etc/passwd

匹配/etc/passwd中第7个字段不等于/sbin/nologin的行。    awk 'BEGIN{FS=":"}$7!="/sbin/nologin"{printf "%s\n",$7}' /etc/passwd

匹配/etc/passwd中第3个字段包含三个以上数字的行（匹配正则表达式）。    awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{printf "%d\n",$3}' /etc/passwd

匹配/etc/passwd中包含root或nologin的所有行。awk '/root/ || /nologin/{print $0}' /etc/passwd

匹配/etc/passwd中第3个字段包含小于50并且第4个字段大于60并且第7行包含/sbin/nologin的所有行
    awk 'BEGIN{FS=":"}$3>50 && $4<60 && $7~/\/sbin\/nologin/{printf "%s %s %s\n",$3,$4,$7}' /etc/passwd
```

## 11.4 动作表达式中的算术运算符
```
+
-
*
\
^或**    乘方
++x
x++
--x
x--
```

例：
```
awk 'BEGIN{var1=10;var2="hello";print var1,var2}'
awk 'BEGIN{num1=10;num2+=num1;print num1,num2}'
awk 'BEGIN{num1=10;num2=29;print num1+num2}'
awk 'BEGIN{num1=10;num2=29;print num1-num2}'
awk 'BEGIN{num1=10;num2=29;print num1*num2}'
awk 'BEGIN{num1=10;num2=29;print num1/num2}'
awk 'BEGIN{num1=10;num2=29;print num1^num2}'
awk 'BEGIN{x=10;y=20;print x++;y++}'
awk 'BEGIN{x=10;y=20;print ++x;++y}'
awk 'BEGIN{num1=10;num2=29;printf "%0.2f\n",num1/num2}' #保留两位小数
```

### 例：使用awk计算某文件中空白行的数量
```
awk '/^$/{sum++}END{print sum}' my.cnf
```

### 例：计算课程分数平均值
score.txt 内容如下：
```
Allen      90 99 93 73
Jone       83 23 38 97
Monica     99 77 89 43
Jerry      77 44 32 91
```

```
awk 'BEGIN{printf "%-8s %-8s %-8s %-8s %-8s\n","姓名","语文","数学","物理","平均分"}{total=$2+$3+$4+$5;avg=total/4;printf "%-8s %-8d %-8d %-8d %-8d %0.2f\n",$1,$2,$3,$4,$5,avg}' score.txt
```

## 11.5 条件
```
if (条件表达式1)
    动作1
else if (条件表达式2)
    动作2
else
    动作3
```

例：
```
awk 'BEGIN{FS=":"}{if($3<50) {printf "%-10s %-4d\n","小于50的uid",$3} else if($3<80) {printf "%-10s %-4d\n","小于80的uid",$3} else {printf "%-10s %-4d\n","其它uid",$3}}' /etc/passwd
```

### awk的代码可能很长，这个时候可以写成脚本用-f来调用
script.awk  内容如下：
```

BEGIN {
    FS=":"
}

{
    if($3<50) {
        printf "%-10s %-4d\n","小于50的uid",$3
    } else if($3<80) {
        printf "%-10s %-4d\n","小于80的uid",$3
    } else {
        printf "%-10s %-4d\n","其它uid",$3
    }
}
```

```
awk -f script.awk /etc/passwd
```

### 例：计算课程分数平均值，并且只打印分数大于70的同学的姓名和分数
score.txt 内容如下：
```
Allen     90 99 93 73
Jone      83 23 38 97
Monica     99 77 89 43
Jerry      77 44 32 91
```
```
awk '{total=$2+$3+$4+$5;avg=total/4;if(avg>70){printf "%-10s %-0.2f\n",$1,avg}}' score.txt
```

## 11.6 循环
```
while(条件表达式)
    动作

do
    动作
while(条件表达式)

for(初始化计数器;测试计数器;计数器变更)
    动作
```

### 例：计算1+2+3+...+100的和，分别使用do-while、while、for实现
```
awk 'BEGIN{for(i=1;i<=100;i++){sum+=i};printf "sum=%d\n",sum}'
awk 'BEGIN{do{sum+=i++}while(i<=100);printf "sum=%d\n",sum}'
awk 'BEGIN{while(i<=100){sum+=i++};printf "sum=%d\n",sum}'
```

### 11.7 字符串函数
```
函数名                   解释         函数返回值
-----------------------------------------------------------------------------------------------------------
length(str)              计算字符串长度                                                 整数返回值
index(str1,str2)         在str1中查找str2的位置                                         返回值为索引，从1开始
toupper(str)             转换为大写                                                     转换后的大写字符串
tolower(str)             转换为小写                                                     转换后的小写字符串
substr(str,m,n)          从str的m个字符截取n位                                             截取后的子串 
split(str,arr,fs)        按fs切割字符串，结果保存到arr（分隔符默认是空格，可以省略）         切割后的子串的个数
match(str,RE)            在str中按照RE查找，返回位置                                     返回索引位置，从1开始

sub(RE,RepStr,str)       在str中按RE搜索字符串并将其替换为RepStr，只替换第一个，返回替换的个数
gsub(RE,RepStr,str)      在str中按RE搜索字符串并将其替换为RepStr，替换所有
```

例：
```
awk 'BEGIN{print length("abcd")}'
awk 'BEGIN{print index("abcd","c")}'
awk 'BEGIN{print toupper("abCd")}'
awk 'BEGIN{print tolower("abCd")}'

awk 'BEGIN{print substr("hello,world",3,6)}'
awk 'BEGIN{print split("root:x:0:0:root",arr,":");print arr}'
awk 'BEGIN{print match("hello,world", /lo/)}'
```

###  例：返回/etc/passwd中每个字段的长度
```
awk 'BEGIN{FS=":";OFS=":"}{print length($1),length($2),length($3),length($4),length($5),length($6),length($7)}' /etc/passwd

搜索"i have a dream"中ea的位置
awk 'BEGIN{print index("i have a dream","ea")}'
awk 'BEGIN{print match("i have a dream","ea")}'

将"Hadoop is a bigdata framework"转换为小写
awk 'BEGIN{print tolower("Hadoop is a bigdata framework")}'

将"Hadoop is a bigdata framework"转换为大写
awk 'BEGIN{print toupper("Hadoop is a bigdata framework")}'

将"Hadoop is a bigdata framework"按空格分割后保存在数组中
awk 'BEGIN{str="Hadoop is a bigdata framework";split(str,arr," ");for(a in arr){print arr[a]}}'

找出字符串"Transaction 23345 start: select * from master"中第一个数字出现的位置

awk 'BEGIN{str="Transaction 23345 start: select * from master";print match(str,/[0-9]/)}'

截取字符串"Transaction 23345 start: select * from master"，从第4个开始截取5位
awk 'BEGIN{str="Transaction 23345 start: select * from master";print substr(str,4,5)}'

替换"Transaction 23345 start, Event ID: 9002"中出现的第一个数字串为$
awk 'BEGIN{str="Transaction 23345 start, Event ID: 9002";count=sub(/[0-9]+/,"$",str);print count,str}'
```

### 11.8 选项
```
-v     参数传递
-f     指定awk脚本文件
-F     指定分隔符，可以连着写
-V     查看awk版本
```

例：
```
awk -v arg1=12 -v arg2="hello world" 'BEGIN{print arg1,arg2}'
awk -F ":" '{print $1}' /etc/passwd
awk -F: '{print $1}' /etc/passwd
```

## 11.9 awk中数组的用法

- 数组使用（索引从0开始）

```
array=("janee" "jone" "jacek" "jordan")
打印元素             echo ${array[0]}、echo ${array[@]}
打印元素个数         echo ${#array[@]}
打印元素长度         echo ${#array[0]}
给元素赋值        array[2]="messi"
删除元素            unset array[2]、unset array
分片访问            echo ${array[0]:1:3}
元素替换            ${array[@]/e/E}（替换元素中的第一个e为E）、${array[@]//e/E}（替换元素中的所有e为E）
元素便利            for a in ${array[@]}; do echo $a; done
```

- awk中数组的用法（索引从1开始）

```
awk 'BEGIN{str="hadoop spark yarn storm flume";split(str,array," ");for(i=1;i<=length(array);i++){print array[i]}}'

awk的数组中可以使用字符串作为数组的下标

awk 'BEGIN{array["var1"]="zhangsan";array["var2"]="lisi";array["var3"]="wangwu";for(i in array){print array[i]}}'
```

### 例：统计各种tcp状态连接状态数
```
netstat -an | grep tcp | awk '{array[$6]++}END{for(i in array){print i,array[i]}}'
```

## 例：计算纵向横向总和
score.txt 内容如下：
```
Allen     90 99 93 73
Jone     83 23 38 97
Monica     99 77 89 43
Jerry     77 44 32 91

awk '{line++;line_sum=0;for(i=1;i<=NF;i++){line_sum+=$i;col_sum[i]+=$i;printf "%-6s ",$i};print line_sum}END{printf "%-6s","";for(i=2;i<=length(col_sum);i++){printf "%-6s ",col_sum[i]};printf "\n"}' score.txt
```
输出：
```
Allen   90     99     93     73     355
Jone    83     23     38     97     241
Monica  99     77     89     43     308
Jerry   77     44     32     91     244
       349    243    252    304
``` 

## 11.10 awk中数组的用法

### 例：用awk脚本处理数据并生成报告

生成数据的脚本：insert.sh
```
#!/bin/bash
function create_random {
    min=$1
    max=$(($2-$min+1))
    num=$(date +%s%N)
    echo $(($num%$max+$min))
}

INDEX=1
while true
do
    for user in Mike Allen Jerry Tracy Hanmeimei Lilei
    do
        COUNT=$RANDOM
        NUM1=`create_random 1 $COUNT`
        NUM2=`expr $COUNT - $NUM1`
        echo "`date "+%Y-%m-%d %H:%M:%S"`" $INDEX Batches: $user insert $COUNT data into table 'test1', insert $NUM1 records successfully, failed insert $NUM2 records >> /root/script/data.txt
        INDEX=`expr $INDEX + 1`
    done
done

数据格式：
2019-04-17 23:44:36 495 Batches: Jerry insert 7658 data into table test1, insert 1008 records successfully, failed insert 6650 records
2019-04-17 23:44:36 496 Batches: Tracy insert 17609 data into table test1, insert 10348 records successfully, failed insert 7261 records
2019-04-17 23:44:36 497 Batches: Hanmeimei insert 14256 data into table test1, insert 1599 records successfully, failed insert 12657 records
2019-04-17 23:44:36 498 Batches: Lilei insert 9279 data into table test1, insert 7856 records successfully, failed insert 1423 records
2019-04-17 23:44:36 499 Batches: Mike insert 22652 data into table test1, insert 6291 records successfully, failed insert 16361 records

(1)、统计每个人员插入了多少条数据进数据库
awk 'BEGIN{printf "%-10s %-10s\n","name","total"}{stat[$5]+=$7}END{for(i in stat){printf "%-10s %-10s\n",i,stat[i]}}' data.txt

(2)、统计每个人员插入成功和失败了多少条数据进数据库
awk 'BEGIN{printf "%-10s %-10s %-10s %-10s\n","User","Total","Succeed","Failed"}{sum[$5]+=$7;suc_sum[$5]+=$13;fail_sum[$5]+=$18}END{for(i in sum){printf "%-10s %-10s %-10s %-10s\n",i,sum[i],suc_sum[i],fail_sum[i]}}' data.txt

(3)、在(2)的基础上统计全部插入记录数
awk 'BEGIN{printf "%-10s %-10s %-10s %-10s\n","User","Total","Succeed","Failed"}{sum[$5]+=$7;suc_sum[$5]+=$13;fail_sum[$5]+=$18}END{for(i in sum){all_sum+=sum[i];all_suc_sum+=suc_sum[i];all_fail_sum+=fail_sum[i];printf "%-10s %-10s %-10s %-10s\n",i,sum[i],suc_sum[i],fail_sum[i]};printf "%-10s %-10s %-10s %-10s\n","",all_sum,all_suc_sum,all_fail_sum}' data.txt

(4)、查找丢失数据，也就是成功+失败的记录数不等于总共插入的记录数
awk '{if($7!=$13+$18){print $0}}' data.txt
```

# 12. mysql操作
## 12.1 安装启动mariadb
```
yum install mariadb mariadb-server mariadb-libs -y
systemctl start mariadb
```

# 13. 脚本工具

>脚本工具功能概述
(1)、实现一个脚本，该脚本提供类似supervisor的功能，可以对进程进行管理
(2)、一键查看所有进程运行状态
(3)、单个或批量启停进程
(4)、提供进程分组功能，可以按组查看进程运行状态，可以按组启停进程

```
配置文件 process.cfg

[GROUP_LISE]
WEB_LIST
DB_LIST
HADOOP_LIST
YARN_LIST

[WEB_LIST]
nginx
httpd

[DB_LIST]
mysql
postgresql

[HADOOP_LIST]
datanode
namenode

[YARN_LIST]
resourcemanager
nodemanager

[nginx]
description="Web Server 1"
program_name=tail
parameter=-f /root/tmp/web-nginx.conf


[httpd]
description="Web Server 2"
program_name=tail
parameter=-f /root/tmp/web-httpd.conf

[mysql]
description="High Perfrmance Database"
program_name=tail
parameter=-f /root/tmp/db-mysql.conf

[postgresql]
description="High Perfrmance Database"
program_name=tail
parameter=-f /root/tmp/db-postgresql.conf

[datanode]
description="Hadoop datanode"
program_name=tail
parameter=-f /root/tmp/hadoop-datanode.conf

[namenode]
description="Hadoop namenode"
program_name=tail
parameter=-f /root/tmp/hadoop-namenode.conf

[resourcemanager]
description="yarn resourcemanager"
program_name=tail
parameter=-f /root/tmp/yarn-resourcemanager.conf

[nodemanager]
description="yarn nodemanager"
program_name=tail
parameter=-f /root/tmp/yarn-nodemanager.conf
#!/bin/bash

THIS_PID=$$
GROUP_LIST=GROUP_LIST
CFG_FILE=/root/script/tmp/process.cfg

function group_list {
        group_list=`sed -n "/\[$GROUP_LIST\]/,/^\[.*\]/p" $CFG_FILE | grep -v "^$" | grep -v "\[.*\]" | grep -v "\#"`
        echo $group_list
}

function get_all_process {
        for group in $(group_list)
        do
                p_list=$(get_all_process_by_group $group)
                echo $p_list
        done
}

function get_all_process_by_group {
        group_process=`sed -n "/\[$1\]/,/\[.*\]/p" $CFG_FILE | grep -v "^$" | grep -v "\[.*\]" | grep -v "\#"`
        echo $group_process
}

function is_group_exists {
        count=`sed -n "/\[$1\]/p" $CFG_FILE | grep -v "$GROUP_LIST" | wc -l`
        if [ $count -eq 1 ]; then
                return 0
        else
                return 1
        fi
}

function get_group_by_process_name {
        for g in $(group_list)
        do
                for p in `get_all_process_by_group $g`
                do
                        if [ $p == $1 ]; then
                                echo "$g"
                        fi
                done
        done
}

function get_process_info_by_pid {
        ps -ef | awk -v pid=$1 '$2==pid{print}' &> /dev/null
        if [ $? -eq 0 ]; then
                proc_status="RUNNING"
        else
                proc_status="STOPPED"
        fi

        proc_cpu=`ps aux | awk -v pid=$1 '$2==pid{print $3}'`
        proc_mem=`ps aux | awk -v pid=$1 '$2==pid{print $4}'`
        proc_start_time=`ps -p $1 -o lstart | grep -v "STARTED"`
}

function get_pid_by_process_name {
        if [ $# -ne 1 ]; then
                return 1
        else
                pid=`ps -ef | grep "$1" | grep -v grep | grep -v "$0" |  awk '{print $2}'`
                echo $pid
        fi
}

function format_print {
        group=`get_group_by_process_name $1`
        ps -ef | grep $1 | grep -v grep | grep -v $THIS_PID &> /dev/null
        if [ $? -eq 0 ]; then
            for pids in `get_pid_by_process_name $1`
            do
                for _pid in $pids
                do
                    get_process_info_by_pid $_pid

                    awk -v p_name="$1" \
                    -v p_group="$group" \
                    -v p_id="$_pid" \
                    -v p_status="$proc_status" \
                    -v p_cpu="$proc_cpu" \
                    -v p_mem="$proc_mem" \
                    -v p_start_time="$proc_start_time" \
                    'BEGIN{printf "%-20s%-10s%-10s%-10s%-10s%-10s%-10s\n",p_name,p_group,p_status,p_id,p_cpu,p_mem,p_start_time;}'
                done
            done
        else
                    awk -v p_name="$1" \
                    -v p_group="$group" \
                    'BEGIN{printf "%-20s%-10s%-10s%-10s%-10s%-10s%-10s\n",p_name,p_group,"NULL","NULL","NULL","NULL","NULL";}'
        fi
}

echo "********************************************************************************************************"
echo `group_list`

echo `get_all_process`

echo `get_all_process_by_group DB_LIST`

echo `is_group_exists WEB_LIST`

echo $(get_group_by_process_name mariadb)

format_print $1
echo "********************************************************************************************************"
```

# 14. 其它
```
ps -ef | grep nginx | awk '{print $2}' | xargs kill

$?        命令执行的结果，0表示成功，其它表示有异常
$$        脚本执行的子进程的pid
$#         参数数量
$0      shell文件名
$@      shell执行的时候传入的所有参数
shift    shell执行的时候跳过一个传入的参数
netstat
    -l[listening]
    -a[all]
    -t[tcp]
    -p[program] Show the PID and name of the program to which each socket belongs.
```


# 附录：
## 已知一个程序的端口 怎么找到程序运行位置

1.首先得到端口所占的进程id （PID）
```
[root@localhost ~]# lsof -i:8088
COMMAND   PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
java    81278 root  230u  IPv6 193305309      0t0  TCP *:radan-http (LISTEN)

```
2.1 使用`ps`命令 获得所在位置 
```
 ps -aux |grep -v grep|grep 81278
[root@localhost slapp_web_jar3]# ps -aux |grep -v grep|grep 81278
root      81278  0.0  3.1 16277464 1289992 ?    Sl   May12  21:22 java -jar slapp-web.jar
 ```

 2.2 进入线程所在文件夹查看具体信息 
 ```
[root@localhost 81278]# cd /proc/81278
[root@localhost 81278]# ll -ail
total 0
293933453 dr-xr-xr-x   9 root root 0 May 28 15:53 .
        1 dr-xr-xr-x 344 root root 0 Apr  8 16:35 ..
297546802 dr-xr-xr-x   2 root root 0 May 29 08:46 attr
297546788 -rw-r--r--   1 root root 0 May 29 08:46 autogroup
297546784 -r--------   1 root root 0 May 29 08:46 auxv
297546807 -r--r--r--   1 root root 0 May 29 08:46 cgroup
297546799 --w-------   1 root root 0 May 29 08:46 clear_refs
293945644 -r--r--r--   1 root root 0 May 28 15:53 cmdline
297546789 -rw-r--r--   1 root root 0 May 29 08:46 comm
297546813 -rw-r--r--   1 root root 0 May 29 08:46 coredump_filter
297546806 -r--r--r--   1 root root 0 May 29 08:46 cpuset
297546794 lrwxrwxrwx   1 root root 0 May 29 08:46 cwd -> /home/quotation/slapp_web_jar3
293945643 -r--------   1 root root 0 May 28 15:53 environ
293999317 lrwxrwxrwx   1 root root 0 May 28 15:58 exe -> /usr/local/src/jdk1.8.0_151/bin/java
```

那么我就得出了在`/home/quotation/slapp_web_jar3`下 有一个`slapp-web.jar` 使用这个端口。

注：读者也可以直接使用 `lsof -p PID`

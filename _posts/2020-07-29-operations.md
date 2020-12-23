---
layout: article
title: "运维学习笔记"
tags: 
	- 运维
	- shell
toc: true
---
## **运维常用命令**
清屏 clear, ctrl + l
创建文件夹 mkdir abc
创建文件 touch abc
设置权限 chmod 755 abc, chmod -x abc, chmod a+x abc, chmod a+w abc  
注意  
r 代表读权限，用数值4表示  
w 代表写权限，用数字2表示  
x 代表执行权限，用数字1表示  
\+ 代表增加权限  
\- 代表减少权限  
因此755代表 rwxr-xr-x，7代表文件拥有者权限，5代表文件所属组权限，5代表文件组外权限
## **find**
用途：基于文件属性查找  
用法：
```
find [查找路径] 寻找条件 操作
```
### 常用参数：  
```
-name
-type f/d(文件/目录)
-size +/-30M(大于/小于)
-perm 
-exec
-ok (比exec多一个询问)
-maxdepth 指定搜索的目录层级
```
### -exec参数详解
```
find [目录] -[参数] -exec [命令] {} \;
find [目录] -[参数] -exec [命令] {} +
分号和加号都是-exec参数的终结符
\;  每找到一个，就执行一次[命令]
+   找到多个，一次性执行[命令]
```
### -exec特点
这两种-exec中的tar命令的使用等效
```
find . -type f -exec tar -czvf n.tar.gz {} +
find . -type f -exec tar -uvf n.tar.gz {} \;
```
### xargs
这种使用方式效果和上述两种方式一致
```
find . -type f | xargs tar -czvf n.tar.gz
```
## **tar**
```
tar -xzvf [压缩文件名] -C [指定卸压目录]（卸压）
tar -czvf [压缩文件名] [要进行压缩的文件] （压缩）
tar -uvf [压缩文件名] [要进行压缩的文件] （更新式压缩）
tar -tvf [压缩文件名] （不卸压查看内容）
--remove-files （在压缩完文件后删除源文件）
```
## **sed**
### s：字符替换
```
sed 's///' file
sed 's/abc.net/efg.com/' text.txt 
```
### c：整行替换
```
sed '//c/' file
sed '/^SELINUX=/cSELINUX=disabled/' file
^代表开头
```
### g：全局替换
```
sed 's///g' file
sed 's/abc.net/efg.com/g' test.txt
```
### -n：定位到某一行
```
sed -n '2p' file （定位到第二行）
sed -n '/broadcast/p' file （定位到/broadcast这一行）
```
### -e：允许在同一行命令中执行多个命令
```
sed -e '///' -e '///' file
sed -e 's/11/aa/' -e 's/bb/gg/' file
```
### -i：生效到文本；不加只是屏显测试结果  
### sed可以同时对多个文件执行同一个'///'
```
sed 's/aa/11/' file1 file2 ...
```
## **awk**
### 功能：报告生成器（不修改文本）  
行：record  
列：field  
NR：number of record  
NF：number of field  
### 基本操作：
```
awk '{print $0}' access.log (打印所有列)
awk '{print $1}' access.log (打印第一列)
awk '{print $NF}' access.log (打印最后一列)
awk '{print $NF-1}' access.log (最后一列-1)
awk '{print $(NF-1)}' access.log (打印倒数第二列)
```
### 列对齐：
```
awk '{print $1" \t "$2" \t "$3}' access.log (列对齐)
awk '{print $1,$2,$3}' access.log (输出默认分隔符为逗号，代表一个空格，若把逗号改成空格，输出效果表现为空)
echo 1:2 3*4|awk -F":" '{print $1}' (-F表示指定输入分隔符，输入默认分隔符为空格)
```
### 显示行数/列数：
```
awk '{print NR "\t" $0}' awk.txt (打印行号)
awk '{print NF "\t" $0}' awk.txt (打印每行的列数)
awk '$2=="2012"{print $0}' staff.txt (当第二列的值==2012时，打印这一行)
awk '{print NF}' staff.txt (只打印每行的列数)
awk '{print NR}' staff.txt (只打印每行的行号)
awk 'NR=="3"{print NR,$0}' staff.txt (只打印第3行)
awk 'NF=="5"{print $0}' staff.txt (只打印列数为5的行)
注意：如果不指定文件名，awk会等待输入
```
### 自定义分隔符
```
awk 'BEGIN{FS=","}{print $1}'
BEGIN表示全局变量，FS表示输入分隔符，OFS表示输出分隔符
awk默认的输入分隔符和输出分隔符是分别定义的
awk 'BEGIN{FS=",";OFS=","}{print $1,$2}'
awk '{print NR,FILENAME,$0}' staff.txt data.txt (会把data的内容追加打印在staff的下方显示)
```
### 隐藏某列内容
```
awk '{$3="xxxx";print $0}' staff.txt
```
### awk做计算
```
awk '{a=1;b=3;print a+b}' (回车给出结果，但不结束)
awk遇到字符：
awk '{a=2;b="56ass";c=3;print b+c}' (56+3=59)
awk '{a=2;b="ass56";c=3;print b+c}' (3)
```
## **grep/egrep**
### 功能：基于文本内容的查找
```
grep -E等效于egrep
```
### 排除#号行（注释）和空行
```
egrep -v "^$|#" file (-v取反，^以什么开头，$以什么结尾，|或者)
egrep -v '(#|^$)' file
或者
grep -Ev "#|^$" file
```
### 查找目录下的所有文件中是否包含指定字符串
```
find . | xargs grep -ri "IBM"
```
### 查找目录下的所有文件中是否包含指定字符串，并只打印文件名
```
find . | xargs grep -ril "IBM"
```
## **cut**
### 功能  
文本/屏显的切割命令；能接收管道，也能直接操作文本，按行处理  
默认以制表符为分隔符
b 按字节切割（1byte=8bit）
c 按字符切割（character，全角是2个byte，半角是1个byte）
f 按字段切割（field列）  
d 指定分隔符（输入、输出一样）
```
who | cut -b 3 (切割who的输出，并只显示每行的第3个字符)
```
### 范围的表示方法

|表示|备注|
|---|---|
|N|只有第N项|
|N-|从第N项一直到行尾|
|N-M|从第N项到第M项(包括M)|
|-M|从一行的开始到第M项(包括M)|
```
cut -d: -f 1,3-5,8- file.txt (-d指定分隔符:,-f指定输出第1列、3-5列、第8列及以后)
```
### cut和awk的一些区别
cut默认不会忽略空格和制表符(全是field字段)
awk默认把空格、制表符看作分隔符(非field字段)  
![例子](/operations/cut.png)
### cut如何输入制表符
先按ctrl+v，再按tab健，"\t"这种分隔符都是无效的
## **重定向**
标准输入0 standardinput STDIN
标准输出1 standardoutput STDOUT
标准错误2 standarderror STDERR
### 标准输出重定向

|符号命令|用途|
|---|---|
|\> 或 1> | 把**标准输出**重定向到新文件(会覆盖)，**标准错误**不适用|
|&> 或 2> | 标准错误输出|
|\>> | 追加，不覆盖原有文件|
|&>> | 标准错误追加|
|2>&1 | 将标准错误重定向到标准输出|
|命令 >> 文件 2>&1 或 命令 &>> 文件 | 将标准输出与错误输出共同(追加)写入文件(不管命令执行结果如何)|
|命令 > 文件 2>&1 或 命令 &> 文件 | 同上(不追加)|

```
systemctl status httpd > httpd (无屏显，内容存到httpd)
systemctl status abc > abc (屏显错误，abc存在并空白)
```
tee : 类似于大于号，但**只从管道(或输入重定向)接收数据**  
tee -a : 类似于追加符(-a append)  
tee和tee -a都会屏显，>>和>不屏显  
tee可以同时重定向到多个文件(而>和>>没有这个功能，123 > file1 file2 结果file1里面是123 file2)  
命令1|tee file.txt|命令2: 将命令1的结果既保存到file.txt中，也传给命令2，并屏显命令2的结果(若有第三个管道，就正常传值)
### 标准输入重定向

|符号命令|用途|
|---|---|
|<|指定输入文件|
|<<|等待用户输入，指定输入结束符|
|tr|只处理字符，而非单行单列，只显示处理结果，**不修改源文件**，**只从管道(或输入重定向)接收数据**(同tee)|

```
cat > file <<end (一直等待输入，直到输入end)
echo 1234 | tr 123 AB (结果为ABB4)
echo 1::2::3 | tr -s : (-s压缩重复字符，结果为1:2:3)
tr -d "^$" < file (不能删除空行，因为tr只对字符操作)
tr -d " " < file (删除空格)
tr 1-9 a-z < file (数字变字母)
```
## **sort**

默认ASCII码排列，从左往右读取字符abcdefg

|参数|用途|
|---|---|
|-r|倒排序|
|-n|按数字排列|
|-t|指定分隔符，如-t:指定分隔符为冒号|
|-k|指定列，如-k3n代表第3列按数字排序|
## **uniq**
去除重复行  
-c：去除重复行并统计重复行的数量
## **wc**
wc -l：统计行数
## **锚定符**
与正则的区别：锚定符前后不能有其他字符，正则可以有
```
\<字符串\>
\b字符串\b
```
## **head/tail**
head -5 前5行  
tail -5 最后5行  
tail -f 表示一直刷新最后的
## **shell脚本编写**
```shell
#!/bin/bash
if [ $# -ge 1 ];then
        systemctl status $1 &> /dev/null
        if [ $? -eq 0 ];then
                echo "$1 is `systemctl status $1|sed -n 3p|awk '{print $2,$3}'`"
        else
                echo "$1 服务不存在或未正常运行"
        fi
else
        echo "请按如下格式执行命令:bash $0 服务名称"
fi

```

```shell
#!/bin/bash
if [ $# -ge 1 ];then
        for i in $@
        do
                systemctl status $i &> /dev/null
                if [ $? -eq 0 ];then
                        echo "$i is `systemctl status $i|sed -n 3p|awk '{print $2,$3}'`"
                else
                        echo "$i 服务不存在或状态错误"
                fi
        done
else
        echo "请按如下格式执行命令:bash $0 服务名称"
fi

```
```shell
#!/bin/bash
read -p "Please input your record: " cj
if [ $cj -ge 0 ] && [ $cj -le 59 ];then
	echo "补考"
elif [ $cj -ge 60 ] && [ $cj -le 70 ];then
	echo "一般"
elif [ $cj -ge 71 ] && [ $cj -le 85 ];then
        echo "良好"
elif [ $cj -ge 86 ] && [ $cj -le 100 ];then
        echo "优秀"
else
	echo "Please input a valid record(0-100)"
fi
```
```shell
#!/bin/bash
case $1 in
1)
        echo "god bless me"
;;
2)
        echo "cat is cute"
;;
*)
        echo "bye"
;;
esac
```
```
#!/bin/bash
set -xv
read -p "Please input your number: " NU
[ $NU = "1" ] && echo "You are the best" || ([ $NU = "2" ] && echo "Oh my god!")

```
## **zabbix**
```
[zabbix-server]服务端：监控端，就是监控者
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
#上述操作还会在/etc/yum.repos.d/下面生成一个新的仓库文件zabbix.repo
#但这个文件还是指向zabbix官网，往往因为网络问题还是会导致下面的安装错误
#所以，可以修改一下：
sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo
yum clean all
yum makecache
yum install deltarpm vim wget -y


yum install -y zabbix-server-mysql zabbix-web-mysql mariadb-server
systemctl enable mariadb --now
mysql_secure_installation
mysql -uroot -proot
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'root';
mysql> quit;
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -proot zabbix
mysql -uzabbix -proot -D zabbix -se "show tables;"

sed -i.ori '116a DBPassword=root' /etc/zabbix/zabbix_server.conf
sed -i.ori '18a php_value date.timezone  Asia/Shanghai' /etc/httpd/conf.d/zabbix.conf
systemctl disable firewalld --now
setenforce 0;sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config
systemctl enable zabbix-server --now
systemctl enable httpd --now

#解决web中文乱码：
yum -y install wqy-microhei-fonts
cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf

#浏览器打开：http://server的ip/zabbix
#账户：Admin/zabbix


[zabbix-agent]客户端：被监控端，就是被监控者，或者叫代理端
#若agent和server同主机，下面步骤就不用重复操作了：
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
sed -i 's#http://repo.zabbix.com#https://mirrors.aliyun.com/zabbix#' /etc/yum.repos.d/zabbix.repo
yum clean all
yum makecache
yum install deltarpm vim wget -y

#下面这一步是安装agent包，是必备操作，无论agent和server是否同主机：
yum install -y zabbix-agent 

#下面这5步是在agent的配置文件上配置服务端的IP，若agent和server同主机，可做可不做：
#请根据自己的实际情况，修改“zabbix服务端的IP”：
sed -i.ori 's#Server=127.0.0.1#Server=zabbix服务端的IP#' /etc/zabbix/zabbix_agentd.conf
sed -i.ori 's#ServerActive=127.0.0.1#ServerActive=zabbix服务端的IP#' /etc/zabbix/zabbix_agentd.conf
sed -i '/^Hostname=/cHostname=zabbix-agent1' /etc/zabbix/zabbix_agentd.conf
systemctl disable firewalld --now
setenforce 0;sed -i '/^SELINUX=/cSELINUX=disabled' /etc/selinux/config


#这一步是要做的：
systemctl enable zabbix-agent --now
```
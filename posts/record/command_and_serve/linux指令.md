---
title: linux指令和操作
date: 2017-07-17 21:28:17
tags: ["linux"]
categories: ["记录"]
draft: true
---

> 最要记录我平时用的linux指令，方便查找

vim 命令：

* ls : 显示当前目录的文件
* cd : 进入某个文件
* service nginx / mysql / vsftpd restart : 重启服务
* cat .... : 查看某个文件
* vi .... : 编辑某个文件
* :q .... : 编辑文件时，不保存退出
* :w .... : ....保存
* :q, :w ... : 强制退出或保存
* :qw ... : ....保存并退出
* rm -rf ... : 删除文件夹，包括本身
* rm -f ... : 删除文件
* unzip : 解压zip压缩包（在服务器解压，更快）
* clear l : 清屏  
* 查看状态下执行：dd，删除当前行

查看命令：

* netstat -plntu : 查看端口

* du -h : 查看当前文件夹的大小

* du -ah : 查看所有文件的大小

* du -h --max-depth=1： 查看当前文件夹的大小，深度为 1

* ps -ef|grep xxx : 查看 xxx 的进程

* top -H -p {pid}: 查看某进程下的线程数

* kill -9 123123 : 杀掉 123123 的进程

* vi filename : 新建打开一个文件

* chmod -R 777 xxx : 给权限

* netstat -tunpl : 查看端口

* tree:  tree -I '*git|*svn|*node_modules*|dist' -C 查看当前文件夹的文件列表

* tail: 查看日志

  *  tail:  
           -n  是显示行号；相当于nl命令；例子如下：
                tail -100f test.log      实时监控100行日志
                tail  -n  10  test.log   查询日志尾部最后10行的日志;

                tail -n +10 test.log    查询10行之后的所有日志;

  * head:  

            跟tail是相反的，tail是看后多少行日志；例子如下：
            
                head -n 10  test.log   查询日志文件中的头10行日志;
            
                head -n -10  test.log   查询日志文件除了最后10行的其他所有日志;

  * cat： 

    tac是倒序查看，是cat单词反写；例子如下：

     cat -n test.log |grep "debug"   查询关键字的日志

查看进程

```

```



查看内存：

```
free -m // 查看内存信息
df -h // 查看硬盘信息
top // 查看内存使用情况
dmidecode | more // 查看系统硬件配置信息

// 下面的解析，来自(https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316399.html)

// top 命令，各个参数的作用

total 进程总数
running 正在运行的进程数
sleeping 睡眠的进程数
stopped 停止的进程数
zombie 僵尸进程数
Cpu(s): 
0.3% us 用户空间占用CPU百分比
1.0% sy 内核空间占用CPU百分比
0.0% ni 用户进程空间内改变过优先级的进程占用CPU百分比
98.7% id 空闲CPU百分比
0.0% wa 等待输入输出的CPU时间百分比
0.0%hi：硬件CPU中断占用百分比
0.0%si：软中断占用百分比
0.0%st：虚拟机占用百分比

Mem:
191272k total    物理内存总量
173656k used    使用的物理内存总量
17616k free    空闲内存总量
22052k buffers    用作内核缓存的内存量
Swap: 
192772k total    交换区总量
0k used    使用的交换区总量
192772k free    空闲交换区总量
123988k cached    缓冲的交换区总量,内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小,相应的内存再次被换出时可不必再对交换区写入。

序号  列名    含义
a    PID     进程id
b    PPID    父进程id
c    RUSER   Real user name
d    UID     进程所有者的用户id
e    USER    进程所有者的用户名
f    GROUP   进程所有者的组名
g    TTY     启动进程的终端名。不是从终端启动的进程则显示为 ?
h    PR      优先级
i    NI      nice值。负值表示高优先级，正值表示低优先级
j    P       最后使用的CPU，仅在多CPU环境下有意义
k    %CPU    上次更新到现在的CPU时间占用百分比
l    TIME    进程使用的CPU时间总计，单位秒
m    TIME+   进程使用的CPU时间总计，单位1/100秒
n    %MEM    进程使用的物理内存百分比
o    VIRT    进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
p    SWAP    进程使用的虚拟内存中，被换出的大小，单位kb。
q    RES     进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
r    CODE    可执行代码占用的物理内存大小，单位kb
s    DATA    可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
t    SHR     共享内存大小，单位kb
u    nFLT    页面错误次数
v    nDRT    最后一次写入到现在，被修改过的页面数。
w    S       进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程)
x    COMMAND 命令名/命令行
y    WCHAN   若该进程在睡眠，则显示睡眠中的系统函数名
z    Flags   任务标志，参考 sched.h
```



### 修改防火墙端口

```
vi /etc/sysconfig/iptables
service iptables restart
```





## centos

我用的是`centos` ， `centos` 使用 `yum` 进行包的管理

`yum -y install nginx` 直接按照 `nginx` 和 所依赖的包

`service nginx start` 就能开启服务，不用带路径那样输入命令开启服务

## nginx 配置

yum install nginx

`yum` 安装的 `nginx` 在 `etc/nginx`

## linux 安装git

如果有`yum`

`yum install git-core`

安装完毕后，设置用户

`git config --global user.name "Your Name"`

`git config --global user.email "youremail@domain.com"`

设置ssh`ssh-keygen -t rsa -C "youremail@163.com"`，连续回车即可

`cat ~/.ssh/id_rsa.pub`，复制ssh，在`github`添加

## 安装nvm

`curl https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash`

查看所有`node`版本：`nvm ls-remote node `

安装指定版本`node`：`nvm install v6.9.4`

使用指定版本：`nvm use v6.9.4`

设置默认版本：`nvm alias default v4.6.0`

#### 安装 443 问题

add '199.232.68.133 raw.githubusercontent.com' to /etc/hosts



## 安装node包

安装淘宝镜像：`npm install -g cnpm --registry=https://registry.npm.taobao.org`

## 安装mongodb

设置配置文件`vim /etc/yum.repos.d/mongodb-org-3.4.repo`

添加以下内容：

    [mongodb-org-3.4]  
    name=MongoDB Repository  
    baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/  
    gpgcheck=1  
    enabled=1  
    gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc  

开始安装`mongodb`：`yum -y install mongodb-org`

安装完后，`log`和`db`位置都配置好了

`systemctl start mongod.service` // `service mongod start` ，开启mongodb服务

`systemctl status mongod.service`，查看mongodb开启状态

如果`Active`不是`active`，就是失败了。查看`/var/log/mongodb/mongod.log`日志，看看是否有这样的错误`Failed to unlink socket file /tmp/mongodb-27017.sock Operation not permitted`，如果有，就去`/tmp`下，删除那个`.sock`文件。然后重新运行成功

## 操作mongodb

`mongo`，运行`shell`



## nodejs - nginx配置

阿里云配置`DNS`


    server {
      listen 80;
      server_name canvas-Image-processing.rni-l.com;
    
      charset utf-8;
    
      location / {
        proxy_pass  http://localhost:6363/;
    
        proxy_set_header  Host  $host;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
      }
    }

pkill php-fpm: 关闭php-fpm
https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/



## 编写脚本

常见用法：

```shell
# 声明变量
test="haha"

# 输出字符串变量的长度
# 重点在 #test
${#test}

# let，let 命令可以直接进行基本算术操作，且声明的变量无需加 $
n1=5
n2=4
let res=n1+n2
echo ${res}

# echo 快速生成文件；这命令会清空改文件内容，并输出对应文字
echo "hah test" > test.txt
# >> 会在源文件后面追加内容
echo "hah test2" >> test.txt

# 文件描述符有三种：0，1，2，分别对应输入、输出和 标准错误
# 我们可以命令的内容输出到一个文件里面；下面的命令会把 ls 命令操作后的结果，追加到文件里面
# 同样可以使用 0,1,2
echo ls 1>> test.txt

# tee 可以将命令输出到可视区，并生成文件；-a 是追加内容
ls | tee test.txt
ls | tee -a test.txt

```

数组

```shell
# 定义数组
arr=(1 2 3 55)
arr2[0]="1"
arr2[1]="2"
echo ${arr[2]} 

# 打印全部
echo ${arr[*]}
# 打印长度
echo ${#arr[*]}
```


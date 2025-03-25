

### idea

```
ctrl + alt +l 格式化代码
```

```
ctrl+alt+t 找try catch
```

```
选中代码，ctrl+alt+m抽取方法
```

**插件** love your eyes theme

ctrl+d是复制上一行

### git

```
本机配置
1.官网下载，一直下一步即可
2.配置账号和邮箱
git config --global user.name "name"
git config --global user.email "email"
3.本地生成ssh key （ssh key是机器之间传输消息的证明，每一台主机都应该各自生成ssh key，并绑定到gitlab账号）
ssh-keygen -t rsa -C "email"
4.gitlab上关联key
gitlab settings sshKeys 
```

```
git branch <branch_name>
git checkout <branch_name> 创建并切换分支
git branch -d <branch_name> 删除new
git checkout pre 切换回原来的分支
git merge new 合并new到pre上
```

```
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:qingcaikeke/aa.git
git push -u origin main
```

#### 网易实习相关：

merge合并后的历史记录保留了所有分支的历史，呈现出分叉后再合并的结构。

rebase将当前分支的提交按顺序应用到另一个分支的顶端，这相当于在另一个分支的基础上重新应用当前分支的更改。

```
小明和小张同时基于main1开发，小明修改push，merge到main后，main1边为main2；小王完成修改，试图merge，发生冲突，因为小王是基于main1开发的，而远程现在是main2，所以冲突
这时需要rebase
第一步 拉最新的main2分支
第二步 把小王的修改分支代码rebase到main2分支
第三步 此时有冲突，手动解决，以=====为分割， 上面是main，下面是小王的分支修改(与main不同的)，手动解决冲突
第四步 git add . git rebase --continue 然后跳入vim，按i进入编辑模式，写清本次rebase，然后esc+：wq
第五步 git push -f 强推
```

```
git fetch
git rebase origin/master
git add .
git rebase --continue
git push -f
```

```
其他常用操作
git log
git status
git branch
git checkout feature-yjy
```

```
Commit Message
基本语法： <type>[scope]:<subject> <body>
type: 提交的commit类型，如feat(添加新特性), fix（修复bug）, docs（仅修改了文档）
scope: 本次commit的影响范围，可不写
subject: 简要描述
body: 详细描述
```

### docker

docker pull <name镜像名> 如：docker pull mysql

docker run -d --name <Myname> -p 6379:6379 <image>（-d后台运行）

docker exec -it my-redis-container（容器名） redis-cli（容器中执行的实际命令）（-it 交互式终端（interactive））



docker --help  #Docker帮助

 docker --version  #查看Docker版本

 docker search <image> #搜索镜像文件，如：docker search mysql

 docker images  #查看已经拉取下来的所以镜像文件

 docker rmi <image> #删除指定镜像文件

docker ps   #查看正在运行的所有镜像

docker ps -a  #查看所有发布的镜像

 docker rm <image>  #删除执行已发布的镜像

docker exec -it <container_name_or_id> /bin/bash #进入redis docker容器

redis-cli 在容器中使用redis命令行工具

docker exec -it（与容器进行交互式终端连接） my-mysql-container（容器名） mysql -uroot  -p（）



cd /tmp -> mkdir mysql -> cd mysql -> docker run -p 3306:3306 --name mysql -v $PWD/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123  mysql:5.7.25

**挂载**

-v把主机上名为xx的卷挂载到容器的xx目录，目的是为了持久化

卷挂载是双向的，对主机文件修改，容器内会变，对容器内修改，主机会变

### linux

[Linux 命令大全 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-command-manual.html)

- 文件相关(mv mkdir rmdir cd ls pwd)
- 文件编辑（vi cat rm-rf cp rm）
- 进程相关( ps top netstat )
- 权限相关(chmod chown useradd groupadd)
- 网络相关(netstat ip ifconfig addr ping)
- 测试相关(测试网络连通性:ping 测试端口连通性:telnet)

vim：文本编辑器，可以在命令行中编辑文本文件

vi+文件：编辑文件

cat：查看文件，显示内容，concatenate

pwd：print work directory

vi/vim是linux原始的文本编辑器，可以用于创建文件

:wq(保存并退出)，:wq! (强制保存并退出)，:q!(直接退出)，:q(未修改)

grep：文件中查正则表达式或字符串

##### 系统更新与包管理

yum：：安装 httpd 服务：`yum install httpd`。更新系统所有软件包：`yum update`。

##### 文件操作命令

**cd**：用于切换目录。如进入项目目录：`cd /home/user/project`。

**ls**：列出目录内容。查看当前目录下的文件和文件夹：`ls`，以详细信息列出：`ls -l`。

**cp**：复制文件或目录。将`source_file`复制到`destination_file`：`cp source_file destination_file`。

**mv**：移动或重命名文件、目录。移动文件到另一个目录：`mv file /new/directory`。重命名文件：`mv old_name new_name`。

**rm**：删除文件或目录。删除文件：`rm file`。删除目录及其所有内容：`rm -r directory`。

##### 压缩与解压缩

**tar**：用于打包和解包文件。打包目录为`archive.tar`：`tar -cvf archive.tar directory`。解压`archive.tar`到当前目录：`tar -xvf archive.tar`。

**gzip**：压缩和解压缩文件，通常与`tar`命令结合使用。压缩文件：`gzip file`。解压`file.gz`：`gzip -d file.gz`。

**unzip**：用于解压 zip 格式的文件。解压`archive.zip`：`unzip archive.zip`。

##### 进程管理

**ps**：查看系统当前运行的进程的快照，重点在于查看需要查看的进程。查看所有进程：`ps -ef`。查看与当前终端相关的进程：`ps -l`。

Process Status

**top**：**动态**显示系统中各个进程的资源占用情况，按 CPU 使用率、内存使用率等排序，重点在于资源占用情况。

Table Of Processes

**kill**：用于终止进程。根据进程 ID（PID）杀死进程：`kill -9 PID`。

##### 网络命令

**ifconfig**：查看和配置网络接口信息，如 IP 地址、子网掩码等。

**ping**：测试网络连接，检查与目标主机的连通性。`ping www.example.com`。

**netstat**：查看网络连接、路由表、网络接口等信息。查看所有网络连接：`netstat -an`。

#### linux目录结构

 [Linux 系统目录结构 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-system-contents.html)

可使用：ls / 查看

![image-20240530145846710](C:\Users\16776\AppData\Roaming\Typora\typora-user-images\image-20240530145846710.png)





##### 命令实战

用户主目录：`~` = `/home/yjy`，如果是root，`~`=`/root`，每个用户的主目录路径都是唯一的，并以 `/home/` 开头（除了 `root` 用户)

[yjy@linux ~]$：用户yjy，主机名linux，当前工作目录~，$代表权限级别为普通用户，#为超级用户

若使用：cd /usr/local，则变为[yjy@linux /usr/local]

目录后面是否带/：带着代表目录下的内容，不带代表包括目录本身

cp -r /src /dest/：dset目录下会多一个src目录，将 `/src` 目录（包括目录本身）复制到 `/dest` 目录中

mongoDB安装使用：

[yjy@linux ~]$ wget https://fastdl.mongodb.org/linux/mogodb-linux-x86_64-3.4.3.tgz 	:wget，命令行工具，用于从网上下载文件 

[yjy@linux ~]$ tar -xf XX.tgz -C ~/	tar：处理归档文件的命令，-x：解压，-f：指定文件夹，-C：指定解压缩的目录

[yjy@linux ~]$ mv XX/ /usr/local/mogodb	mv：移动

[yjy@linux mogodb]$ mkdir /usr/local/mogodb/data/

[yjy@linux mogodb]$ mkdir /usr/local/mogodb/data/logs/

[yjy@linux mogodb]$ touch /user/local/mogodb/data/logs/mogodb.log

[yjy@linux mogodb]$ touch /usr/local/mongodb/data/mongodb.conf  `touch`：创建一个文件

[yjy@linux mogodb]$ vim ./data/mogodb.conf	`.`：相对路径，`vim`：编辑

### maven

下载一个新项目，可以在聚合项目的根目录执行mvn clean install,相当于按聚合中指明的模块顺序，先去执行clean，再去install

mvn所在的命令必须在pom.xml所在的文件使用

compile：编译，生成target包

clean：清理之前编译的结果（删除target）

maven识别测试方法不止对注解有要求，还对测试类有要求XxxTest，对方法名有要求testXxx

package:打成jar包（会编译核心程序，测试程序，并进行测试）保存到target文件下

补：build=compile+test+package

install将当前工程生成的jar包保存到本地仓库，会按照坐标保存到指定位置

**mvnrepository.com**中去找依赖，不要想当然的去改版本号

下载错误解决：可能下载过程存在污染或损害，要先删除缓存（根据坐标去本地仓库中找对应的 . lastUpdated文件）




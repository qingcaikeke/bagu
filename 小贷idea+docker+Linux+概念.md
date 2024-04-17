```
ctrl+alt+t 找try catch
```

```
选中代码，ctrl+alt+m抽取方法
```

ctrl+d是复制

FTP上传文件?

### git

git branch <branch_name>

git checkout <branch_name> 创建并切换分支

git branch -d <branch_name> 删除new

git checkout pre 切换回原来的分支

git merge new 合并new到pre上

```
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:qingcaikeke/aa.git
git push -u origin main
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

### linux

[Linux 命令大全 | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-command-manual.html)

- 文件相关(mv mkdir rmdir cd ls pwd)
- 文件编辑（vi cat rm-rf）
- 进程相关( ps top netstat )
- 权限相关(chmod chown useradd groupadd)
- 网络相关(netstat ip ifconfig addr ping)
- 测试相关(测试网络连通性:ping 测试端口连通性:telnet)

ls：list directory contents

ps：Process Status：是一个动态的、实时更新的进程列表，并包含了各种系统资源的使用情况，如 CPU 利用率、内存使用情况等

top：Table Of Processes：是一个静态的进程列表，可以显示更详细的信息，如进程的状态、优先级（类似任务管理器）

vim：文本编辑器，可以在命令行中编辑文本文件

vi+文件：编辑文件

cat：查看文件，显示内容

pwd：print work directory



### 概念

toB（bussiness）：其目标客户是其他公司、组织或企业。这样的软件公司与其他企业建立合作关系，为其提供定制的解决方案，以满足特定的业务需求。

toC（consumer）：一家在线零售商在其网站上销售电子产品。它直接面向个体消费者，提供在线购物体验，接受个人订单，并直接向消费者交付产品。






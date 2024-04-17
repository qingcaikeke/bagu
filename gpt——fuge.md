Docker的镜像可以理解为一种特殊的文件系统，它包含了运行容器所需的所有文件，包括程序、库、资源、配置等文件，以及一些为运行时准备的一些配置参数，如匿名卷、环境变量、用户等。但是，镜像不包含任何动态数据，其内容在构建之后也不会被改变。

Docker镜像可以用来创建Docker容器，它类似于虚拟机镜像，是一个面向Docker引擎的只读模板。例如，一个镜像可以只包含一个完整的Ubuntu操作系统环境，也可以安装了Apache应用程序或其他需要的软件，这可以分别称为一个Ubun阳镜像和一个Apache镜像。

docker run --name Nginx -d -p 80:80 nginx

`\` 是一个特殊的转义字符，被称为"换行转义"。它的作用是告诉解释器，当前命令未结束，下一行是当前命令的一部分。

命令的目的是创建一个名为"Nginx"的Docker容器，该容器使用nginx镜像，并在后台运行

Docker daemon是Docker最核心的后台进程，也被称为守护进程。它负责响应来自Docker Client的请求，然后将这些请求翻译成系统调用完成容器管理操作。该进程会在后台启动一个API Server，负责接收由Docker Client发送的请求，接收到的请求将通过Docker daemon内部的一个路由分发调度，由具体的函数来执行请求。



docker：快速部署应用

需要什么环境，只需要做出一份镜像，基于这个镜像启动容器，把项目丢进去

镜像（.iso）容器（相当于操作系统）仓库（同一管理镜像）

0.gitcode创建仓库/项目

1.vcs->create git repository(相当于git init)

2.左侧commit/ctrl（mac的command）+k 	（相当于commit）

3.ctrl+shift+k(添加远程仓库url) 	（相当于push）

ETC是Etcetera的缩写，这个文件夹是用来存放所有的系统管理所需要的配置文件和子目录。在Unix和Linux操作系统中，ETC是一个重要的配置文件夹。



Manage multiple Spring Boot run configurations in the Services tool windo

你可以在IntelliJ IDEA的Services工具窗口中管理你的Spring Boot应用程序的多个运行配置。

centos7.9

nuoshell termius（） electerm
backend后端integration整合credential可以信任的证书



硬编码的，直接在代码中指定了它们的值。在实际开发中，可以通过使用配置文件、常量或通过外部输入来替代硬编码，以提高代码的可维护性和灵活性。



 Nginx是一个轻量级、高性能的反向代理服务器。其事件驱动的架构使得它能够高效地处理大量并发连接。从而分发客户端请求到多个后端服务器

Nginx 利用其高效的、异步非阻塞（reactor模式）的设计，以及事件驱动（红黑树储存socket，有事件发生，回调，加入就绪链表），多路复用（一个进程管理多个socket）等特性，能够在单一进程中有效地处理大量并发连接。

前面提到的 TCP Socket 调用流程是最简单、最基本的，它基本只能一对一通信，因为使用的是同步阻塞的方式，当服务端在还没处理完一个客户端的网络 I/O 时，或者 读写操作发生阻塞时，其他客户端是无法与服务端连接的。



I/O多路复用是一种通过单一的事件循环机制来管理多个I/O操作的技术。它使得程序能够同时监控多个文件描述符（sockets、files等），在任一文件描述符就绪时进行相应的I/O操作，而无需为每个I/O操作创建单独的线程或进程。
#### （一）Dockerfile

> Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

- ①编写Dockerfile

> 创建Dockerfile

```bash
mkdir mynginx
cd mynginx
vi Dockerfile
```



![img](https:////upload-images.jianshu.io/upload_images/11223715-e1eb2602c7a10552.png?imageMogr2/auto-orient/strip|imageView2/2/w/426/format/webp)



> 编辑内容

```bash
FROM nginx
RUN echo '<h1>Hello,World,Dockerfile</h1>' > /usr/share/nginx/html/index.html
```



![img](https:////upload-images.jianshu.io/upload_images/11223715-8e8cd23770003e2a.png?imageMogr2/auto-orient/strip|imageView2/2/w/636/format/webp)



- ②构建镜像

> 命令： docker build -t 名称:版本号 .
>  提取Dockerfile，将Dockerfile按行进行分析 Dockerfile每行第一个单词，如CMD、FROM等，这个叫做command。根据command，将之后的字符串用对应的数据结构进行接收。处理完所有的命令，如果需要打标签，则给最后的镜像打上tag，结束。

```bash
docker build -t nginx:v0 .
```



![img](https:////upload-images.jianshu.io/upload_images/11223715-94da3a496b2b155b.png?imageMogr2/auto-orient/strip|imageView2/2/w/994/format/webp)





![img](https:////upload-images.jianshu.io/upload_images/11223715-01285fbc3e5f3613.png?imageMogr2/auto-orient/strip|imageView2/2/w/771/format/webp)



- ③该镜像历史

> 镜像的历史来源

```bash
docker history nginx:v0
```



![img](https:////upload-images.jianshu.io/upload_images/11223715-680c4893f6965d85.png?imageMogr2/auto-orient/strip|imageView2/2/w/1052/format/webp)



#### （二）Dockerfile命令合集

- ①FROM

> 所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而FROM就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

> 选择dockerhub中官方镜像为基础，这样更加的安全稳定，而且可持续性，有人维护。

> 对于scratch 就是空白镜像，有老铁奇怪一个空白的没有基础的，我如何执行我的程序，对于linux系统来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，比方使用go语言开发的应用编译打包成为二进制的问题，在用scratch进行打包，这就是为什么go语言更核心容器下面的微服务架构的原因。

```bash
FROM <image>
FROM <image>:<tag>
FROM <image>@<digest>
FROM scratch #制作base Image
FROM centos #使用base Image
FROM centos:7.9
FROM mysql:5.6
```

- ②LABEL

> 给镜像添加信息。使用docker inspect可查看镜像的相关信息

```bash
LABEL maintainer="394498036@qq.com"
LABEL version="1.0"
LABEL description="This is description \
欢迎关注：编程坑太多"
```

- ③RUN

> 指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。基本就是shell脚本。

```bash
#不建议使用
RUN yum update
RUN yum install -y vim
RUN python-dev

#建议使用
RUN yum update && yum install -y vim \
          python-dev #反斜线换行
RUN  apt-get update && apt-get install -y perl \
          pwgen --no-install-recommends && rm -rf \
          /var/lib/apt/lists/*   #注意清理cache
```

> 每一个指令都会创建一层，并构成新的镜像。当运行多个指令时，会产生一些非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。因此，在很多情况下，我们可以合并指令并运行。

> 注意初学docker容易出现的2个关于RUN命令的问题：
>  1.RUN代码没有合并。
>  2.每一层构建的最后一定要清理掉无关文件。

- ④ENV

> 方便编写比较复杂的Dockerfile，主要为了方便维护。

```bash
ENV MYSQL_VERSION 5.6
E-NV apt-get install -y mysql-server = "${MYSQL_VERSION}" \
&& rm -rf /var/lib/apt/lists/* #引用常亮
```

- ⑤COPY

> 将文件和目录复制到容器的文件系统。文件和目录需位于相对于 Dockerfile 的路径中。尽量使用COPY不使用ADD。这里ADD就不做讲解。

```bash
COPY ["", ""]
COPY nginx.conf /etc/nginx/nginx.conf
```

- ⑥WORKDIR

> 工作目录

```bash
WORKDIR /test #如果没有会自动创建test目录
WORKDIR idig8
RUN pwd          #输出结果应该是/test/idig8
```

> 用WORKDIR，不要用RUN cd 尽量使用绝对目录！

- ⑦ENTRTYPOINT

> 设置容器启动时运行的命令
>  让容器以应用程序或者服务的形式运行
>  不会被忽略，一定会执行

- ⑧CMD

> 设置容器启动后默认执行的命令和参数
>  容器启动时默认执行的命令
>  如果docker run 指定了其他命令，CMD命令被忽略
>  如何定义了多个CMD，只有最后一个会执行

PS：一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个.dockerignore，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。
 基本思路：
 1.编写.dockerignore文件
 2.容器只运行单个应用
 3.将多个RUN指令合并为一个
 4.基础镜像的标签不要用latest
 5.每个RUN指令后删除多余文件
 6.选择合适的基础镜像(alpine版本最好)
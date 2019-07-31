# 镜像构建上下文（Context）

docker build命令最后有一个.。

.表示当前目录，而Dockerfile就在当前目录，因此不少初学者以为这个路径是在指定Dockerfile所在路径，这么理解其实是不准确的。

如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。

那么什么是上下文呢？

首先我们要理解docker build的工作原理。

Docker在运行时分为Docker引擎（也就是服 务端守护进程）和客户端工具。Docker的引擎提供了一组RESTAPI，被称为Docker Remote API，而如	docker命令这样的客户端工具，则是通过这组API与Docker引擎交 互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种docker功能，但实际上，一切都是使用的远程调用形式在服务端（Docker引擎）完成。也因为这种C/S设计， 让我们操作远程服务器的Docker引擎变得轻而易举。当我们进行镜像构建的时候，并非所有定制都会通过RUN指令完成，经常会需要将一些本地 文件复制进镜像，比如通过COPY指令、ADD指令等。而docker	build命令构建镜像，其 实并非在本地构建，而是在服务端，也就是Docker引擎中构建的。

那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，docker build命令得知这个路径后，会将路径下的所有内容打包，然后上传给Docker引擎。这样 Docker引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如果在Dockerfile中这么写：`COPY ./package.json /app/`这并不是要复制执行docker build命令所在的目录下的package.json，也不是复制Dockerfile所在目录下package.json，而是复制上下文（context）目录下的package.json。因此，COPY这类指令中的源文件的路径都是相对路径。这也是初学者经常会问的为什么`COPY ../package.json /app`或者`COPY /opt/xxxx /app`无法工作的原因，因为这些路径已经 超出了上下文的范围，Docker引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

现在就可以理解刚才的命令`docker build -t nginx:v3 .`中的这个.，实际上是在指定上下 文的目录，docker build命令会将该目录下的内容打包交给	Docker引擎以帮助构建镜像。如果观察docker build输出，我们其实已经看到了这个发送上下文的过程：`$docker build -t nginx:v3 . Sending build context to Docker	daemon 2.048 kB ...`理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现`COPY /opt/xxxx	/app`不工作后，于是干脆将Dockerfile放到了硬盘根目录去构建，结果发现docker build执行后，在发送一个几十GB的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让docker build打包整个硬盘，这显然是使用错误。一般来说，应该会将Dockerfile置于一个空目录下，或者项目根目录下。

如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传 给Docker引擎，那么可以用.gitignore一样的语法写一个.dockerignore，该文件是用于 剔除不需要作为上下文传递给Docker引擎的。

那么为什么会有人误以为.是指定Dockerfile所在目录呢？

这是因为在默认情况下，如果 不额外指定Dockerfile的话，会将上下文目录下的名为Dockerfile的文件作为 Dockerfile。这只是默认行为，实际上Dockerfile的文件名并不要求必须为Dockerfile，而且并不要求 必须位于上下文目录中，比如可以用 -f ../Dockerfile.php参数指定某个文件作为Dockerfile	。

当然，一般大家习惯性的会使用默认的文件名Dockerfile，以及会将其置于镜像构建上下文目录中。

>注意:docker-compose.yaml到Dockfile中上下文具有传递性

```bash
Fusionshen@Fusionshen MINGW64 /d/go-work/src/github.com/fusionshen/dentist-expert
$ tree -L 3
.
|-- de-api
|   |-- Dockerfile
|   |-- Dockerfile-dev
`-- docker
    |-- README.md
    `-- docker-compose.yml

```

我尝试去用docker-compose.yml和Dockerfile去构建容器,最初的设计是这样的
```yml
#docker/docker-compose.yml
version: '2'
services:
  dentist_expert_api:
    build:
      context: .
      dockerfile: ../de-api/Dockerfile
    container_name: dentist_expert_api
    ...
```

```DockerFile
#de-api/DockerFile
FROM library/golang:1.12.7

ENV APP_DIR $GOPATH/src/github.com/fusionshen/dentist-expert

RUN go get github.com/astaxie/beego && go get github.com/beego/bee

RUN mkdir -p $APP_DIR
ADD . $APP_DIR/de-api
WORKDIR $APP_DIR/de-api

RUN go get -v -d ./
```

单独构建`docker bulid .`是可以的，因为此时的context默认取的就是/de-api/,但是利用`docker-compose up -d`就是会报错，因为`context: .`传递过去了，这个构建问题断断续续花了我一两天的时间，等我查看了docker_practice.pdf，发现了上面的文字，理解了其中的道理，所以修改yml文件如下：
```yml
#docker/docker-compose.yml
version: '2'
services:
  dentist_expert_api:
    build:
      context: $GOPATH/src/github.com/fusionshen/dentist-expert/de-api
      dockerfile: Dockerfile
    container_name: dentist_expert_api
    ...
```
>另外，通过以前.netcore自动生成的DockerFile内容可以，DockerFile中可以WORKDIR可以随时改变
```DockerFile
FROM microsoft/dotnet:2.2-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 5000

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src
COPY DataPlatformSI.WebAPI/DataPlatformSI.WebAPI.csproj DataPlatformSI.WebAPI/
RUN dotnet restore DataPlatformSI.WebAPI/DataPlatformSI.WebAPI.csproj
COPY . .
WORKDIR /src/DataPlatformSI.WebAPI
RUN dotnet build DataPlatformSI.WebAPI.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish DataPlatformSI.WebAPI.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENV ASPNETCORE_URLS http://*:5000
ENTRYPOINT ["dotnet", "DataPlatformSI.WebAPI.dll"]

```

**What a fucking stupid mistake!**
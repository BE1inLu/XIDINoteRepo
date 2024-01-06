nestjs 随笔 04
===

- **rxjs**
-  **nest** 集成 **docker** 部署 & 测试
## rxjs

工作第一天开始就有在接触这个东西，不过没留意。直到我最近写项目，看着工友在我维护的代码 `.submit()` 内还要再次手动创建 `promise` 来返回数据。

```ts
// 演示返回的数据
const stream = of(4);

// 两眼一黑
function promiseResult(){
	return new Promise((resolve,reject)=>{
		stream.submit((res)=>{
			resolve(res)
		})
		...
	})
}
```

绝了。

另一个是，我在写 `grpc` 服务接口的时候，给我定义的格式很繁琐，应该有优化空间在的。

```ts
function grpcControllerDemo(request: grpcRequest): grpcResponse{
	return defer(() => {
		// 因为这里返回的是 promise ，所以用 defer 来做处理了
		return service.demoservice();
	}).pipe(
		map((item) => ({
			param:item, 
		}as grpcResponse))
	)
}
```

最后还是狠狠补课看文档，现在知道了个大概。

文档看下来主要是围绕 `Observable` 来做文章的。

我的理解来看，现在有一条时间线，这个时间线就是 `Observable` ，创建可以用对应的操作符来实现，比如说 `of()` , `interval()` 或者直接返回一个新 `Observable` 对象。

如果说想要返回数据，就是订阅返回。即`subscripe()` 我把他看做是 `promise` 的 `then` 。

举个例子，在一个书店里，我想买《芒格之道》和 《财新周刊》，如果是 `promise` 的话，用`Promise.all([])` 就能解决这个问题。

```ts
const bookList = ["bookA","bookB"];
// promise
const firstReq = axios.get('/buybook',{ book: bookList[0] });
const secondReq = axios.get('/buybook',{ book: bookList[1] });
Promise.all([firstReq,secondReq]).then(([ firstresp, secondresp ])=>{ ... });
```

rx 的话，遵循流的概念。

```ts
const booklist = ["bookA","bookB"];
const stream = from(booklist).pipe(
	mergeMap((val)=>{
		fromPromise(axios.get('/buybook',{ book: val }));
	}),
	mergeAll()
);
stream.subscripe(res => { ... });
```

我也是刚起步，只知道这么点简单的写法。但是有句话我是很认可的

> **Everthing is a stream. (万物皆流)**

对应的，这句话背后的含义我试着解读一下，可能就是以下几点：
- **各自独立，1个流做1件事。**
- **函数式编程少状态定义。**
- **以更直观的方式处理数据。**

## Docker 的集成、部署和测试

### 中间件的服务搭建 & 部署

最早的时候，搭建数据库和集成一些中间件（比如说 Redis）直接下载安装到本地的电脑上用。后面了解到docker可以单独拉数据库和其他的一些第三方中间件做服务。

`docker` 部署就不多提了，`window` 的 `wsl`，然后安装`docker desktop` 。以及设置好镜像源。

先把主要的镜像拉取下来，不用部署容器的时候还得要下。

开发用的东西先拉取下来：

```powershell
docker pull mysql:latest | docker pull redis:alpine | docker pull nginx:stable-perl
```

镜像后缀代表着想要拉取的版本，这里最好去对应的 `dockerHub` 看下，拉取适合自己的版本 `tag`。

部署完毕后就可以写 `docker-compose` 了。

> docker-compose.playload.yaml

```yaml
version: "3"
services:
  mysql:
    restart: always
    image: mysql:latest
    container_name: mysql-server
    volumes:
      - mysqldatadir:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=${PASSWORD}"
      - "MYSQL_DATABASE=${DBNAME}"
      - "TZ=Asia/Shanghai"
  redis:
    restart: always
    image: redis:alpine
    container_name: redis-server
    volumes:
      - redisdatadir:/data
  nginx:
    restart: always
    image: nginx:stable-perl
    container_name: nginx-server
    depends_on:
      - redis
      - mysql
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 3309:3309
      - 6379:6379
volumes:
  mysqldatadir:
  redisdatadir:
```

这个文件主要是做部署第三方中间件和 `SQL` 数据库用的。`nginx` 是作为容器代理将接口代理出去用的。

关于 3309 这个端口，因为我不想和现有的数据库端口冲突，所以 `mysql` 的端口从 3306 映射到了 3309。

这个文件不写 `nginx` 也行，但就是得要自己手动将对应的映射写到 `ports` 上。我写 `nginx` 目的是可以方便我去做端口管理。

> docker-compose.playload.yaml

```yaml
version: "3"
services:
  mysql:
    restart: always
    image: mysql:latest
    container_name: mysql-server
+   ports：
+	  - 3309：3306
    volumes:
      - mysqldatadir:/var/lib/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=${PASSWORD}"
      - "MYSQL_DATABASE=${DBNAME}"
      - "TZ=Asia/Shanghai"
  redis:
    restart: always
    image: redis:alpine
    container_name: redis-server
+   ports:
+     - 6379:6379
    volumes:
      - redisdatadir:/data
- nginx:
-   restart: always
-   image: nginx:stable-perl
-   container_name: nginx-server
-   depends_on:
-     - redis
-     - mysql
-   volumes:
-     - ./nginx.conf:/etc/nginx/nginx.conf
-   ports:
-     - 3309:3309
-     - 6379:6379
volumes:
  mysqldatadir:
  redisdatadir:
```

> nginx.conf

```nginx
events {
    worker_connections 1024;
}
stream {
    upstream mysqlImpl {
        server mysql:3306;
    }
    upstream redisImpl {
        server redis:6379;
    }
    server {
        listen 3309;
        proxy_pass mysqlImpl;
    }
    server {
        listen 6379;
        proxy_pass redisImpl;
    }
}
```

这个 `nginx.conf` 文件创建了一个流，监听外部的 3306 端口和 6379 端口。其中，`mysql:3306` 的 `mysql` 是 `docker` 对应的容器名，就是 `docker-compose` 文件下对应定义的服务名。

```yaml
...
services:
  mysql: <-- 这个
    restart: always
	...
```

另一点是，创建容器成功后，容器自己本身也会单独定义一个默认网络。这个网在创建的时候就已经联通了。可以试试在容器内的 `bash`， `ping` 一下对应的容器地址试试。比如说在 mysql 容器内的 `bash` ，`ping` 一下 `redis` 的IP地址，是可以有消息的。

### 自己的服务部署到docker

上面的文件也只是解决了我的开发问题，但如果说我想把我的服务推送到服务器的 `docker` 运行咋办？

问题不是很大，单独写 `dockerfile` 文件，自己在 `docker` 编译好推送过去就完事。

以我写过的项目为例，这个项目是以 monorepo 文件组织类型的项目。结构大概长这样。

```text
app
  - service-account
    - src
    - tsconfig.app.json
  - service-project
    - src
    - tsconfig.app.json
  - web-api
    - src
    - tsconfig.app.json
lib
  - core
  - util
...
```

在以 `monorepo` 组织的项目架构中，启动程序和单体项目不同，启动只启动 `nest-cli` 文件下的主应用，其他的应用需要自己手动启动。

我现在想要把我的服务部署到服务器上的同时，还想先运行一遍测试，不报错的话就直接打包成容器，推送到 `docker` 做服务。

`dockerfile` 就可以解决我提出来的问题。

但是，在此之前，先把不用的东西给剔除！

先写 `.dockerignore` 把 `node_module` 和打包无关紧要的东西去掉。重点是去掉 `node_module`。

在主目录下创建 `dockerignore`。

> .dockerigonore

```text
.git
*Dockerfile*
*docker-compose*
node_modules
pnpm-lock.yaml
.vscode
```

这一步做完了，下一步才是开始写 `dockerfile`。

以 service-account 为例。

```text
app
  - service-account
    - src
    - tsconfig.app.json
+   - dockerfile
  - web-api
    - src
    - tsconfig.app.json
+   - dockerfile
```

>service-account/dockerfile

```dockerfile
FROM node:20.10.0-alpine3.18 as base
ARG NODE_ENV=dev

RUN mkdir -p /app
ADD . /app
WORKDIR /app

RUN npm install -g pnpm
RUN npm install -g @nestjs/cli
RUN pnpm i

RUN nest build protolib
RUN nest build core
RUN nest build modules

COPY . .

FROM base as test
RUN pnpm test:service-room

FROM base as build
EXPOSE 5002
CMD [ "node","dist/apps/web-api/main.js" ]
```

这个文件就做了3件事情，编译文件，运行测试，然后暴露服务对应的端口和执行运行服务的指令。

服务对应的 `lib` 同样记得编译一遍。

同样，web-api 下也创建一个`dockerfile` ，和上面大体相同，注意启动目录和 lib 的编译。

这样一个容器服务就算完成了。然后新建一个用来开发测试 docker-compose ，在原 docker-compose 的基础上加上这个容器并做出修改。

>docker-compose.dev.yaml

```yaml
version: "3"
services:
  account:
    build:
      context: .
      dockerfile: ./apps/service-account/Dockerfile
    image: "service-account:latest"
    restart: always
	depends_on:
      - mysql
      - redis
  webapi:
    build:
      context: .
      dockerfile: ./apps/web-api/Dockerfile
    image: "web-api:latest"
    restart: always
	depends_on:
      - mysql
      - redis
  mysql:
	...
  redis:
	...
  nginx:
	...
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 3000:3000
  ...
```

这里 nginx.conf 注意下，因为要代理的端口变为了一个，而且对外的端口是http，这里需要做修改。

不嫌麻烦的话可以新建多的一个 conf ，复制到 conf.d 里面，然后在 nginx.conf 导入这个新建的 `conf`就好了。

这里我省事，直接重写这个文件了。

```nginx
events {
    worker_connections 1024;
}

server {
	listen 3000;
	server_name webApi;

	location /api/ {
		# 修改请求头, 设置dns
		# ...
		proxy_pass webApi;
	}
}

```

如果是容器内，容器之间用grpc来传信息，可以在 docker-compose 设置对应的环境服务端口参数就好。

就这样吧。
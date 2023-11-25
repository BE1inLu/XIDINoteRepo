# nestjs 随笔02

## 关于测试

项目初始化的 test demo

> user.service.spec.ts

```ts
describe("UserService", () => {
  let service: UserService;
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UserService],
    }).compile();
    service = module.get<UserService>(UserService);
  });
  it("should be defined", () => {
    expect(service).toBeDefined();
  });
});
```

最简单的测试, 只判断有没有定义生成 service 模块。

## 使用 typeorm 的情况下对 service 进行测试

两种办法，一个是创建一个新的数据库进行测试  
第二种办法是，自己 mock 一个数据库出来   

这里说的是第二种办法  

> user.service.ts

```ts
/** createUserDto */
class CreateUserDto {
  permissionName: string;
}

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User) private readonly userRepo: Repository<User>
  ) {}

  async create(createUserDto: CreateUserDto) {
    return await this.userRepo.save(createUserDto);
  }

  async findAll() {
    return await this.userRepo.find();
  }
}
```

**Mode 1**

在 \*.spec.ts 文件写 repository 的方法

> user.service.spec.ts

```ts
describe("UserService", () => {
  let service: UserService;
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UserService],
    })
      .useMocker(() => {
        // return jest.fn({})
        return {
          find: jest.fn().mockImplementation((result) => {
            return result;
          }),
        };
      })
      .compile();
    service = module.get<UserService>(UserService);
  });

  it("should be defined", () => {
    expect(service).toBeDefined();
  });

  it("should findAll", async () => {
    const mockPermission = {
      //...Arg
    };
    expect(await this.service.findAll()).toEqual(mockPermission);
  });
});
```

**Mode 2**

通过创建一个工厂类返回 repository 的方法

```ts

// mocktype.ts

export type MockType<T> = {
  [P in keyof T]?: jest.Mock<{}>;
};

// repositoryMockFactory.ts

export const repositoryMockFactory: () => MockType<Repository<any>> = jest.fn(
  () => ({
    save: jest.fn((entity) => entity),
    find: jest.fn((entity) => entity),
    update: jest.fn((entity) => entity),
    remove: jest.fn((entity) => entity),
  })
);

// user.service.ts

describe("UserService", () => {
  let service: UserService;
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        {
          permission: getRepository(User),
          useFactory: repositoryMockFactory,
        },
      ],
    }).compile();
    service = module.get<UserService>(UserService);
    mockRepository = module.get(getRepositoryToken(User));
  });

  it("should be defined", () => {
    expect(service).toBeDefined();
  });

  it("should findAll permission", async () => {
    mockRepository.find.mockReturnValue(mockPermission);
    expect(await service.findAll()).toEqual(mockPermission);
  });
});
```
## 关于微服务

grpc 远过程调用  
redis 缓存中间件  
rebbitmq 消息队列  

最近在对之前写的鉴权 demo 做了微服务的实践，把单体项目转成微服务的形式。

结构大概长这样：

- .../app/mainapp
- .../app/web-api
- .../app/service-account
- .../app/service-peoject
- .../lib/protolib
- .../lib/core

web-api 负责暴露外部的通信方法，即 http 协议的 api ，同时做鉴权拦截处理

service-account 负责账户服务的处理，就是处理登录注册的方法，token生成，还有认证鉴权的处理方法

service-project 这里作为一个测试的服务，比如说后续扩展业务需要执行其他的方法可以写在这

protolib 保存 grpcbuf 文件，和生成 proto 文件的接口调用

core 用来存放泛用的数据，util，decorator，config，common 这些

### 怎么用 grpc？

- 定义 \*.proto 文件
- 在服务端 main.ts 注册 grpc 微服务
- 写对应的 server 服务
- 在客户端 \*.module.ts 注册 grpc 服务
- 在需要调用的 server.ts 方法上导入对应的方法 

#### 一些心得

- \*.proto 生成对应的调用接口，定义方法可以用 protoc 来生成  

>[!note]
>需要导入包 protoc ，protobufjs，ts-proto

```bash
npx protoc --plugin=protoc-gen-ts_proto=".\\node_modules\\.bin\\protoc-gen-ts_proto.cmd" --ts_proto_opt=nestJs=true --ts_proto_out=...\protolib\src\outdir .../proto/accountservice.proto
```
- 






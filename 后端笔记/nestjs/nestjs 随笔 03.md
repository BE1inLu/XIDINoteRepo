nestjs 随笔 03
===

- 动态模块
- 测试 II

## 动态模块

因为最近在写数据库集成，和 `grpc` 相关的模块服务，翻找文章找到了一种实践方法，记录一下。

比如说，在一个以 `monorepo` 形式组织的项目里面，因为模块内的 `entity` 互相引用有点混乱，想把它们提取出来处理。

我想要实现的是：

- 把 `entity` 提取出来
- 每个模块下可以根据需要自动导入所需的 `entity`

先从第一个问题解决，第一个问题简单，我把 `account` 文件夹下的 `entity` 全部提取出来，单独创建一个新的 lib 并放入进去。

```bash
nest g lib repository
```

项目就多了一个 lib/repository 文件夹

然后再进行搬移...

```text
lib
 - repository
    - src
        - entity
           - foo.entity.ts
           - bar.entity.ts
           - index.ts
        - repository.module.ts
        - ...     
    
```

第二个问题，因为我是用 `TypeORM` 来进行数据库导入注册的，看博文发现可以通过动态模块的方式来进行自动导入，不必每个 `module` 下写 `forFeature([foo,bar])`

修改的方法是在 `repository.module.ts` 下重写成动态模块，让外部的模块引入此模块以此来实现导入。

再重新组织下文件夹...

```text
lib
 - repository
    - src
        - constant
            - repository.constant.ts
        - entity
            - foo.entity.ts
            - bar.entity.ts
            - index.ts
        - repository.module.ts
        - ...     
    
```

定义好导入模块的常量

> repository.constant.ts

```ts
import { foo, bar } from '../entity'

export const FOO = Symbol('foo')
export const BAR = Symbol('bar')

export const repository=[
    [FOO,foo],
    [BAR,bar]
] as [symbol,typeof foo][]

```

> repository.module.ts

```ts
@Module({
    imports: [TypeOrmModule.forFeature(repository.map((repository) => repository[1]))],
    providers: repository.map(([symbol, entity]) => ({
        provide: symbol,
        useFactory: (datasource: DataSource) => datasource.getRepository(entity),
        inject: [DataSource],
    })),
    exports: repository.map((repository) => repository[0]),
})
export class RepositoryModule{ }
```

自此，动态模块就写好了。

使用的话，先在 `service` 对应的 `module` 模块中导入 `RepositoryModule`。

然后在 `service` 中改写注入方法。

> test.service.ts

```ts

@injectable()
export class testService{

    constructor(
       /* @InjectRepository(foo) private readonly foorepo: Repository<foo>, */
       @Inject(FOO) private readonly foorepo: Repository<foo>
    ){}

    // ...
}
```

我的理解是，重点是在 provider 这里。这里的 provider 返回注入的参数。

同样的，因为写 `grpc` 服务也需要大量的 `client` 服务需要注册，这时候改写成动态模块的方式导入就变得异常舒适。

> protolib.module.ts

```ts
@Module({
    providers: ProtoClient.map(([string, option]) => ({
        provide: string,
        useFactory: () => {
            return ClientProxyFactory.create({
                transport: Transport.GRPC,
                options: option,
            });
        },
    })),
    exports: ProtoClient.map((item) => item[0]),
})
export class ProtolibModule {}
```

`grpc` 的动态模块关键是要通过工厂返回 `ClientProxyFactory.create()` 这个方法。

官方文档也提供了一种通过 `config` 来注入的思路，我也是根据官方文档改写的这个 `client` 方法。

# 测试 II

因为改了注入的方式，所以在对应的测试模块中也要改写模块的注入方式

> foo.spec.ts

```ts
describe("FooService", () => {
  let service: FooService;
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        FooService,
        {
          permission: Foo,
          useFactory: repositoryMockFactory,
        },
      ],
    }).compile();
    service = module.get<FooService>(FooService);
    mockRepository = module.get(getRepositoryToken(Foo));
  });

  it("should be defined", () => {
    expect(service).toBeDefined();
  });

  it("should findAll Foo", async () => {
    mockRepository.find.mockReturnValue(mockFoo);
    expect(await service.findAll()).toEqual(mockFoo);
  });
});

```


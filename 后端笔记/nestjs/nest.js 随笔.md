
## 关注的特性

和spring很像，也有angular的特性。共同之处是都是靠依赖注入这个功能。  
写一段时间下来很 angular ，模块化这个特性很突出。

- 管道

```ts

@Injectable()
export class ValidationPipe implements PipeTransform {
    async transform(value: any, { metatype }: ArgumentMetadata) {
        if (!metatype || !this.toValidate(metatype)) {
            return value;
        }
        const object = plainToClass(metatype, value);
        const errors = await validate(object);
        if (errors.length > 0) {
            throw new BadRequestException('验证错误');
        }
        return value;
    }
    private toValidate(metatype: Function): boolean {
        const types: Function[] = [String, Boolean, Number, Array, Object];
        return !types.includes(metatype);
    }
}

// 使用管道

// ...controller

@Port('test')
@UsePipes(ValidationPipe)
controllerFunc(...){
// ...
}

```

- 守卫

```ts
/**
 *	守卫定义
 */
export class defaultguard extends AuthGuard('jwtAuthStrategy') implements CanActivate{
	    canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
        return super.canActivate(context)
    }
    handleRequest(err,user,info){
	    if(err || !user){
		    throw err || new Exception('err')
	    }
	    return user
    }
}

//usepipe
@UseGuard(JwtAuthGuard)

```

- 依赖注入（DI）和控制反转（IOC） -> angular

依赖注入的最大优点是，解耦，和复用。

> 如果你饿了，就不要去楼下的饭店买东西吃，不然你就会进了服装店，或者是去到了电器城，又或者是去了一个工厂，然后捣鼓别人运行良好的东西产生bug
> 正确的做法是，掏出手机，打开外卖应用，选择想吃的外卖并美美享用

写过 spring ，每次在 service 层注入的时候就会使用 @bean 方法进行对 service 注入使用。@bean 就是依赖注入的一种体现。
而在 angular 里，用 @injectable 来声明注入文件，通过在 \*.module.ts 文件导入注入文件，再在service 层 constructer 使用注入文件。

- typeorm

oneToMany<-->manyToOne

```ts
    async findRole(mail: string) {
        const userrole = await this.userRoleRepository.find({
            where: {
                user: {
                    email: '123@123.com',
                },
            },
            relations: { role: true, user: true },
        })
        const role = userrole.map((userrole) => userrole.role)
        Logger.log(JSON.stringify(userrole))
        Logger.log(JSON.stringify(role))
    }
```

写鉴权查到的


## 项目架构?

单体项目以模块来进行组织
微服务的话就要用 monorepo这种组织结构

## 环境
dotenv
appmoudle 设置configmoudle

> app.moudle.ts

```ts
imports:[
	ConfigModule.forRoot({
		isGlobal: true,
		load: [defaultConfig],
	}),
	 // ...
]
```

> config/default.config.ts

```ts

setConfig()
setConfig({ path: './to/path/.env' })
const defaultConfig = {
    DEV: process.env.ENV_MODE || 'DEV',
    // ...
}

export default registerAs('defaultConfig', () => ({
    ...defaultConfig,
}))
```

- grpc
	还在研究

## 认证和鉴权？

passport、passport-jwt

user-> role -> permission 

自己的单体项目
src
src/config -- 存放config
src/core/auth -- 权限认证
src/core/result -- 返回状态信息格式 ｛data,statucode,msg｝
module/user
module/role
module/permission
module/project
模块化，获取用户、权限等相关信息

auth 里面设置 strategy 和 guard 来设置拦截判断权限等信息



看到别人的做法是 拆分出来单独做一个微服务来调用获取信息
权限和认证分离 access 和 auth 分开独立做服务

app/api-control  <- 做外部的api调用
app/service-auth <-认证相关
app/service-admin <- 用户服务
...

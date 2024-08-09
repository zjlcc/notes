# Day23笔记

## 认证和授权

**流程**：登录操作后向前端发送token，前端进行保存，下一次请求时在请求头的Authorization携带token，然后通过认证中间件判断token的有效性，CurrentUser通过访问HttpContext.User获取用户信息，在每次请求都会重新获取，因为生命周期为Scope，然后在访问控制器时通过授权中间件判断该用户是否具有权限。

## 授权中间件

#### 1、简单授权

##### （1）控制器或操作添加[Authorize]标识

  ASP.NET Core 中的授权由AuthorizeAttribute及其各种参数控制。 在其最基本的形式中，将`[Authorize]`属性应用于控制器、操作或 Razor 页面，将对该组件的访问权限限制为经过身份验证的用户。还可以使用`AllowAnonymous`属性来允许未经身份验证（经过身份验证、未经过身份验证、匿名）的用户访问单个操作。

#### 2、基于角色的授权

##### （1）将角色服务添加到 Identity

  一个用户可以绑定多个角色，一个角色可以绑定多个用户。

```csharp
builder.Services.AddDefaultIdentity<IdentityUser>( ... )   // 注册基于用户的Identity授权服务IdentityUser
      .AddRoles<IdentityRole>()  // 注册基于角色的Identity授权服务IdentityRole
      ...
```

##### （2）添加角色检查标识（使用特性标识）

  ① 基础使用：

```
[Authorize(Roles = "Administrator")]  // Administrator角色的用户才能调用该控制器中的内容；多个角色可以指定为逗号分隔的列表，如：[Authorize(Roles = "HRManager,Finance")]
public class AdministrationController : Controller
{
    public IActionResult Index() =>
        Content("Administrator");
}
```

  ② 限制用户既是 角色“PowerUser”又是角色“ControlPanelUser”：

```
// 控制器-限制用户既是 角色“PowerUser”又是角色“ControlPanelUser”
[Authorize(Roles = "PowerUser")]
[Authorize(Roles = "ControlPanelUser")]
public class ControlPanelController : Controller
{
    public IActionResult Index() => Content("PowerUser && ControlPanelUser");
}
```

  ③ 限制某个操作只有用户是角色“Administrator”才能访问；其他操作用户角色可以是“Administrator”或“PowerUser”

```
[Authorize(Roles = "Administrator, PowerUser")]
public class ControlAllPanelController : Controller
{
    public IActionResult SetTime() => Content("Administrator || PowerUser");

    // 操作-限制该操作只有用户是角色“Administrator”才能访问；其他操作用户角色可以是“Administrator”或“PowerUser”
    [Authorize(Roles = "Administrator")]
    public IActionResult ShutDown() => Content("Administrator only");
}
```

#### 3、基于策略的授权

* 基于角色的授权：和基于声明的授权均使用预配置的形式，他们都是策略授权的一种实现；修改这些策略需要重新编译程序。
* 基于策略的授权：允许管理员在应用程序中定义精细的授权策略，并修改这些策略而无需重新编译或重启应用程序。

##### （1）自定义策略授权-游戏登录接口_用户年龄限制在13岁

  ① 注册授权

```
//自定义授权策略-游戏登录接口_用户年龄限制在13岁
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();  // 一般使用AddScoped达到不需要重新启动的目的
  builder.Services.AddAuthorization(options =>
  {
      options.AddPolicy("AtLeast13", policy =>
          policy.Requirements.Add(new MinimumAgeRequirement(13)));
  });
```

  ② 创建“自定义授权策略-游戏登录接口_用户年龄限制在13岁”的标记 `MinimumAgeRequirement`

```
using Microsoft.AspNetCore.Authorization;

namespace fly_chat1_net7.Middlewares.UserAuthorization_Diy
{
    public class MinimumAgeRequirement : IAuthorizationRequirement  // 实现空的标记接口 IAuthorizationRequirement
    {
        public MinimumAgeRequirement(int minimumAge) => MinimumAge = minimumAge;

        public int MinimumAge { get; }
    }
}
```

  ③ 重写 AuthorizationHandler 的允许授权方法 `HandleRequirementAsync`

```
using Microsoft.AspNetCore.Authorization;
using System.Security.Claims;

namespace fly_chat1_net7.Middlewares.UserAuthorization_Diy
{
    public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
    {
        protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, MinimumAgeRequirement requirement)
        {
            var dateOfBirthClaim = context.User.FindFirst(c => c.Type == ClaimTypes.DateOfBirth && c.Issuer == "http://contoso.com");  // 获取用户的出生年月

            if (dateOfBirthClaim is null)
            {
                return Task.CompletedTask;  // 忽略-不进行授权
            }

            // 计算年龄
            var dateOfBirth = Convert.ToDateTime(dateOfBirthClaim.Value);
            int calculatedAge = DateTime.Today.Year - dateOfBirth.Year;
            if (dateOfBirth > DateTime.Today.AddYears(-calculatedAge))
            {
                calculatedAge--;
            }

            if (calculatedAge >= requirement.MinimumAge)  // 比较
            {
                context.Succeed(requirement);  // 符合条件的用户进行授权
            }

            return Task.CompletedTask;  // 忽略-不进行授权
        }
    }
}
```

  ④ 添加角色检查标识

```
[Authorize(Policy = "AtLeast13")]
public class GameLoginController : Controller
{
    public ActionResult Login()
    {
        return View();
    }

    [AllowAnonymous]  // 免授权
    public ActionResult Show()
    {
        return View();
    }
}
```

## 认证中间件

Authentication 在 ASP.NET Core 中，对于 Authentication（认证） 的抽象实现，此中间件用来处理或者提供管道中的 HttpContext 里面的 AuthenticationManager 相关功能抽象实现。

HttpContext 中的 User 相关信息同样在此中间件中被初始化。

* `AuthenticationOptions` 对象主要是用来提供认证相关中间件配置项，后面的 OpenIdConnect，OAuth，Cookie 等均是继承于此。
* `AuthenticationHandler` 对请求流程中（Pre-Request）中相关认证部分提供的一个抽象处理类，同样后面的其他几个中间件均是继承于此。

在 **`AuthenticationHandler`** 中, 有几个比较重要的方法:

* `HandleAuthenticateAsync` ：处理认证流程中的一个核心方法，这个方法返回 `AuthenticateResult`来标记是否认证成功以及返回认证过后的票据（AuthenticationTicket）。
* `HandleUnauthorizedAsync`：可以重写此方法用来处理相应401未授权等问题，修改头，或者跳转等。
* `HandleSignInAsync`：对齐 HttpContext AuthenticationManager 中的 SignInAsync
* `HandleSignOutAsync`：对齐 HttpContext AuthenticationManager 中的 SignOutAsync
* `HandleForbiddenAsync`：对齐 HttpContext AuthenticationManager 中的 ForbidAsync，用来处理禁用等结果
  在继承Authentication实现中间件的时候只需要重写上面的一个或者多个方法即可。
  
  **JwtBearer 中间件**，同样它重写了 `HandleAuthenticateAsync` 方法。

大致步骤如下：

1. 读取 Http Request Header 中的 Authorization 信息
2. 读取 Authorization 值里面的 Bearer 信息
3. 验证 Bearer 是否合法，会得到一个 ClaimsPrincipal
4. 使用 ClaimsPrincipal 构建一个 Ticket（票据）
5. 调用 Options.Events.TokenValidated(context)，用户可以重写此方法验证Token合法性
6. 返回验证成功

#### 其他的知识点

AuthenticationScheme叫做认证方案，在 MVC 程序中一般通过在 Controller 或者 Action 上 打标记（Attribute）的方式进行授权，最典型的就是新建一个项目的时候里面的AccountController。

AutomaticAuthenticate 是一个bool类型的字段，用来配置是否处理 AuthenticationHandler 是否处理请求。或者你可以理解为中间件是不是自动的处理认证相关的业务。

AutomaticChallenge该配置项是用来设置哪个中间件会是身份验证流程中的默认中间件，当代码运行到 Controller 或者 Action 上的 `[Authorize]` 这个标记的时候，就会触发身份验证流程。



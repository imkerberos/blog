+++
title = "使用 ASP.NET Core 建立 WebApi 服务"
author = ["Evilee"]
date = 2020-03-09
lastmod = 2020-03-10T14:01:09+08:00
tags = ["ssh", "gfw"]
categories = ["计算机"]
draft = true
creator = "Emacs 26.3 (Org mode 9.4 + ox-hugo)"
+++

还是选一个靠谱的吧.

<!--more-->


## <span class="section-num">1</span> 原贴地址 {#原贴地址}

<https://dev.to/%5Fpatrickgod/net-core-3-1-web-api-entity-framework-jumpstart-part-1-4jla>


## <span class="section-num">2</span> 环境 {#环境}

macos
vscode
dotnet-sdk

```text
brew cast install dotnet-core
brew cast install visaul-studio-code
```


## <span class="section-num">3</span> 工程手脚架 {#工程手脚架}

```text
cd ~/ws
mkdir -p dotnet/Dipper && cd dotnet/Dipper
```

创建解决方案

```text
dotnet sln -n Dipper
dotnet new webapi -n Dipper.Dubhe
dotnet sln add Dipper.Dubhe
```

不喜欢缺省的代码格式，修改下

```text
cat > omnisharp.json << END
{
        "FormattingOptions": {
                "NewLinesForBracesInLambdaExpressionBody": false,
                "NewLinesForBracesInAnonymousMethods": false,
                "NewLinesForBracesInAnonymousTypes": false,
                "NewLinesForBracesInControlBlocks": false,
                "NewLinesForBracesInTypes": false,
                "NewLinesForBracesInMethods": false,
                "NewLinesForBracesInProperties": false,
                "NewLinesForBracesInAccessors": false,
                "NewLineForElse": false,
                "NewLineForCatch": false,
                "NewLineForFinally": false
        }
}
END
```

打开工程

```text
code .
```

vscode 中隐藏临时文件

```text
mkdir .vscode
cat > .vscode/settings.json << END
{
    "files.exclude": {
        "*/obj/": true,
        "*/bin/": true
    }
}
END
```

修改 `Properties/launchSettings.json` 文件中，=applicationUrl= 修改为下面代码的样式，http 在前， https 在后。

```json
"profiles": {
    "Dipper.Dubhe": {
        "commandName": "Project",
        "launchBrowser": true,
        "launchUrl": "weatherforecast",
        "applicationUrl": "http://localhost:5000;https://localhost:5001",
        "environmentVariables": {
            "ASPNETCORE_ENVIRONMENT": "Development"
        }
    }
}
```

按下 F5, 开始运行，可以看到一片空白，不过没有出错就表示手脚架成功了。

```sh
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```


## <span class="section-num">4</span> 继续 {#继续}

```sh
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
```

配置好数据库。


## <span class="section-num">5</span> 如何处理 route 参数, query 参数, post 参数 和 body 参数 {#如何处理-route-参数-query-参数-post-参数-和-body-参数}

```csharp
public class SomeQuery {
    public string SomeParameter1 {get; set;}
    public int? SomeParameter2 {get; set;}
}

[HttpGet]
public IActionResult FindSomething([FromRoute] SomeQuery query) {
}

[HttpGet]
public IActionResult GetSomething([FromRoute] int someId, [FromQuery] SomeQuery query) {
}

public class ApiModel {
    [FromRoute]
    public int SomeId {get; set;}
    [FromQuery]
    public string SomeParameter1 {get; set;}
    [FromQuery]
    public int? SomeParameter2 {get; set;}
}
```


## <span class="section-num">6</span> 如何使用 Postgres 的 Enum 数据类型 {#如何使用-postgres-的-enum-数据类型}

假设有如下自定义的枚举类型:

```csharp
public enum MyEnumType {
    FieldA,
    FildB,
}

```

在数据库的 DbContext 中创建 Enum 类型并在静态构造函数中进行映射。

```csharp
using Npgsql;
public class MyDbContext: DbContext {
    protected override void OnModelCreating(ModelBuilder builder) {
        builder.HasPostgresqEnum<MyEnumType>();
    }

    static MyDbContext() {
        NpgsqlConnection.GlobalTypeMapper.MapEnum<MyEnumType>();
    }
}
```

就可以愉快地玩耍了:

```csharp
public class Authror {
    public MyEnumType MyEnum {get; set;}
}

using (var ctx = MyDbContext()) {
    ctx.Authors.Add(new Author{MyEnum = MyEnumType.FieldA});
    ctx.SaveChanges();

    var author = ctx.Authors.Single(b => b.MyEnum == MyEnumType.FieldA);
}
```


## <span class="section-num">7</span> EntityFrameworkCore {#entityframeworkcore}

```sh
dotnet ef migrations add InitialCreate
dotnet ef database update
```


## <span class="section-num">8</span> 代码共享 {#代码共享}

假如有两个应用（例如：用户端和管理后台）需要共享某些代码（如数据模型），可以建立三个工程: Library, Front, Admin.

```text
Library/Library.csproj
Front/Web1.csproj
Admin/Web2.csproj
```

Front 和 Admin 共同依赖 Library.

```sh
dotnet add Front reference Library
dotnet add Admin reference Libaray
```

这样就可以在 Front 和 Admin 中使用 Library 的代码了。

```csharp
using Library.Models;
```


## <span class="section-num">9</span> 多数据库配置 {#多数据库配置}

代码:

```csharp
public class Startup {
    public void ConfigureServices(IServiceCollection services) {
        services.AddDbContext<Extern1DbContext>(options => options.UsePgsql(Configuration.GetConnectionString("DefaultConnection1"),
                                                                            b => b.MigrationsAssembly("CurrentProjectName")));
        services.AddDbContext<Extern2DbContext>(options => options.UsePgsql(Configuration.GetConnectionString("DefaultConnection2"),
                                                                            b => b.MigrationsAssembly("CurrentProjectName")));
    }
}
```

迁移命令:

```sh
dotnet ef migrations --context Extern1DbContext add InitialExtern1 -o Migrations/Extern1
dotnet ef migrations --context Extern2DbContext add InitialExtern1 -o Migrations/Extern2
dotnet ef database update --context Extern1DbContext
dotnet ef database update --context Extern2DbContext
```

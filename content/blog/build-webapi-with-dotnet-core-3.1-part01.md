+++
title = "使用 ASP.NET Core 建立 WebApi 服务"
author = ["Evilee"]
date = 2020-03-09
lastmod = 2020-03-22T00:00:00+08:00
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
dotnet tool install --global dotnet-script
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


## <span class="section-num">10</span> AutoMapper 递归映射 {#automapper-递归映射}

```csharp
public class SourceOuterObject
{
  public SourceSet SourceSet { get; set; }
}

public class SourceSet
{
  public List<SourceObject> SourceList{ get; set; }
}

public class TargetOuterObject
{
  public List<TargetObject> TargetList{ get; set; }
}
```

```csharp
Mapper.CreateMap<SourceObject, TargetObject>();
Mapper.CreateMap<SourceOuterObject, TargetOuterObject>()
    .ForMember(dest => dest.TargetList, opt => opt.MapFrom(src => src.SourceSet.SourceList);
```

另外一个不太优美的方案:

```sharp
Mapper.CreateMap<SourceOuterObject, TargetOuterObject>();
Mapper.CreateMap<SourceObject, TargetObject>();
Mapper.CreateMap<SourceSet, List<TargetObject>>()
    .ConvertUsing(ss => ss.SourceList.Select(bs => Mapper.Map<SourceObject, TargetObject>(bs)).ToList());
```


## <span class="section-num">11</span> Full Text Search for .NetCore {#full-text-search-for-dot-netcore}


### <span class="section-num">11.1</span> 方案一: {#方案一}

假设如下模型:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
}
```

使用 `dotnet ef migrations add ...` 生成迁移文件以后，需要手动修改：

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql(@"CREATE INDEX fts_idx ON ""Product"" USING GIN (to_tsvector('jiebacfg', ""Name"" || ' ' || ""Description""));");
}

protected override void Down(MigrationBuilder migrationBuilder) {
    migrationBuilder.Sql(@"DROP INDEX fts_idx;");
}
```

查询方式：

```csharp
var context = new ProductDbContext();
var npgsql = context.Products
    .Where(p => EF.Functions.ToTsVector("jiebacfg", p.Name + " " + p.Description).Matches("Npgsql"))
    .ToList();
```

> `EF.Functions.ToTsVector` 仅支持在 Linq 闭包中执行，外部直接调用会 `System.Exception`.


### <span class="section-num">11.2</span> 方案二： {#方案二}

直接生成 `Tsv` 列:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public NpgsqlTsVector SearchVector { get; set; }
}
```

在 DbContext 中为 `SearchVector` 创建索引:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder) {
    modelBuilder.Entity<Product>()
        .HasIndex(p => p.SearchVector)
        .HasMethod("GIN"); // Index method on the search vector (GIN or GIST)
}
```

自动生成数据库迁移代码：

```sh
dotnet ef migrations add ....
```

手动编辑生成的迁移代码，目的是为了在更新记录对应字段时，自动触发更新 TSV.

```csharp
public partial class CreateProductTable : Migration {
    protected override void Up(MigrationBuilder migrationBuilder) {
        // Migrations for creation of the column and the index will appear here, all we need to do is set up the trigger to update the column:

        migrationBuilder.Sql(
            @"CREATE TRIGGER product_search_vector_update BEFORE INSERT OR UPDATE
              ON ""Products"" FOR EACH ROW EXECUTE PROCEDURE
              tsvector_update_trigger(""SearchVector"", 'pg_catalog.english', ""Name"", ""Description"");");

        // If you were adding a tsvector to an existing table, you should populate the column using an UPDATE
        // migrationBuilder.Sql("UPDATE \"Products\" SET \"Name\" = \"Name\";");
    }

    protected override void Down(MigrationBuilder migrationBuilder) {
        // Migrations for dropping of the column and the index will appear here, all we need to do is drop the trigger:
        migrationBuilder.Sql("DROP TRIGGER product_search_vector_update");
    }
}
```

在代码中使用:

```csharp
var context = new ProductDbContext();
var npgsql = context.Products
    .Where(p => p.SearchVector.Matches("Npgsql"))
    .ToList();
```


## <span class="section-num">12</span> JWT {#jwt}

1.  Authentication 认证
2.  Authorization 授权


### <span class="section-num">12.1</span> 如何停用 Token? 把 Token/或者 User 存下来(数据库或者 redis) {#如何停用-token-把-token-或者-user-存下来--数据库或者-redis}

对于想屏蔽用户的方案：每个用户创建 JWT 以后都保存下来，接收到 JWT 以后，验证查询其是否在回收的 JWTs 中，不是的话通过，否则回收，屏蔽用户。


## <span class="section-num">13</span> Snippets {#snippets}


### <span class="section-num">13.1</span> json 编码和解码 {#json-编码和解码}

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;
using System.ComponentModel.DataAnnotations;

public class Model {
    [JsonPropertyName("book_id")]
    public string BookId {get; set;}

    public string BookName {get; set;}
}

Model deserialized = JsonSerializer.Deserialize<Model>(json);
string json =  JonsSerializer.Serialize(deserialized);
```


### <span class="section-num">13.2</span> base64 编码和解码 {#base64-编码和解码}

```csharp
using System;
using System.Text;
bytes[] bytes = Convert.FromBase64String("e2NRUZbLCwXu/v8Y9+LZVA==");
string s = Convert.ToBase64String(bytes);
```


### <span class="section-num">13.3</span> AES 加密和解密 {#aes-加密和解密}

```csharp
using System.Security.Cryptography;
using System.IO;

var text = "Hello World";
var buffer = Encoding.UTF8.GetBytes(text);

var iv = GetRandomData(128);
var keyAes = GetRandomData(256);

if (string.IsNullOrEmpty(keyAes))
    throw new ArgumentException("Key must have valid value.", nameof(key));
if (string.IsNullOrEmpty(text))
    throw new ArgumentException("The text must have valid value.", nameof(text));

byte[] result;
using (var aes = Aes.Create()) {
    aes.Key = keyAes;
    aes.IV = iv;
    using (var encryptor = aes.CreateEncryptor(aes.Key, aes.IV))
        using (var resultStream = new MemoryStream()) {
            using (var aesStream = new CryptoStream(resultStream, encryptor, CryptoStreamMode.Write))
                using (var plainStream = new MemoryStream(buffer)) {
                    plainStream.CopyTo(aesStream);
                }
            result = resultStream.ToArray();
        }
}
```

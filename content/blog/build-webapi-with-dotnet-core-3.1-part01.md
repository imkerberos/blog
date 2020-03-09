+++
title = "使用 ASP.NET Core 建立 WebApi 服务"
author = ["Evilee"]
date = 2020-03-09
lastmod = 2020-03-09T22:25:02+08:00
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


## <span class="section-num">5</span> EntityFrameworkCore {#entityframeworkcore}

```sh
dotnet ef migrations add InitialCreate
dotnet ef database update
```

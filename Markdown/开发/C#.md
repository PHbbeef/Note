# 报错指令
```bash
# No usable version of the libssl was found
方案：安装libssl1.0.0版本。
我安装的是 libssl1.0.0_1.0.2g-1ubuntu4.20_amd64.deb
找到你要的版本 http://security.ubuntu.com/ubuntu/pool/main/o/openssl/
```

# VS Code创建ASP.NET Core Web API项目
```bash
# 安装插件 NuGet Package Manager GUI（其他可选择主要是OpenAPI）
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.Design
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Relational
Pomelo.EntityFrameworkCore.MySql
Swashbuckle.AspNetCore （默认被添加，以便支持使用Swagger查看OpenAPI）
# 注：运行之后没有在网址添加 /Swagger/index.html 自行添加
```

## 插件
|插件|备注|
|----|----|
| C# | 代码提示 |
| NuGet Gallery | 包 |
| vscode-solution-explorer  | 创建项目，解决方案 |


## 创建项目
```C#
//创建项目
dotnet new webapi -f net6.0 -o TestApi
```

```C#
//创建解决方案
dotnet new sln -n "1"
```

```C#
//项目添加到解决方案中
dotnet sln "1.sln" add “TestApi"
```

```C#
//添加指定包
dotnet "add" "/root/code/Cshap/Test/Test.csproj" "package" "Swashbuckle.AspNetCore"
```


## 启动和调试
|    |    |
|----|----|
|CTRL+F5|快速启动|
|F5|调试|

注：/swagger/index.html（目录结构）


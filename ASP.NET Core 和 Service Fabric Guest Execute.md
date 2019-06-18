# ASP.NET Core 和 Service Fabric Guest Execute
在不想改动 ASP.NET Core 任何代码的情况将程序部署到 Service Fabric。

## 修改 ASP.NET Core 项目文件
如果在项目文件的`PropertyGroup`节点中找不到则加上如下代码：

``` xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

## 打包
使用 `dotnet publish` 命令打包 ASP.NET  Core 程序。在 ASP.NET Core 项目所在文件夹中执行下面命令。

```
dotnet publish -o ..\out -c release -r win7-x64
```

这样会帮我们生成一个和 Web 项目同名的 exe 文件。程序会被发布到上一级的 `out` 目录中。可以根据需要调整这个目标目录。

## 创建 Service Fabric Guest Execute 服务
使用 Visual Studio 2017 创建或者添加 Guest Execute 的服务。

- Code Package Folder 填上一步的发布目标文件夹。
- Code Package Behavior 选择 Add link to external folder 。
- Program 填上一步生成的 exe 名称 。
- Working Folder 选择 CodePackage 。
- 最后填写 Service Name 。

> 由于发布时指定了运行时为 win7-x64，所以应该不需要在集群节点服务器上安装 .NET Core SDK 或者 Runtime 了。之前发布的时候安装 .NET Core SDK 然后用 dotnet CLI 执行，结果服务总是无法启动。

## Cake 脚本示例

要减少错误的发生最好是把过程程序化，编译过程程序化比较好的方式就是使用各种构建工具。这里我们使用 Cake 。

通过 Cake 发布 ASP.NET Core 网站。

```
Task("PublishWeb")
.IsDependentOn("Build")
.Does(() => 
{
    CleanDirectories("./src/XXX.Web.out/");
     var settings = new DotNetCorePublishSettings
     {
         Runtime = "win7-x64",
         Configuration = "Release",
         OutputDirectory = "./src/XXX.Web.out/"
     };

     DotNetCorePublish("./src/XXX.Web", settings);
});
```
# 写在前面

    我们在开发、使用Service Fabric前必须要先要了解几个问题：

### 开发、生产环境的区别
Service fabric 开发环境与生产环境的安装方式是不一样的，在最初的时候，估计多数人都不会先去认真通读 [Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/ "Service Fabric Documentation") 官方文档，而是在先在网上找一些简化入门的文档，简单读完后就凭借自己的对过往开发、生产环境的安装的经验就马上开始了。但是 Service fabric开发、生产环境的安装与使用并不是跟过往经验一样，一路"Next"就可以配置好的，所以需要大家在上手前，尽量通读熟悉 [Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/ "Service Fabric Documentation") 官方文档后，有一个较熟悉的概览后在进一补动手。开发工具建议选择最新版本的Visual Studio 2019，当前Visual 2015/2017也是可以的，但Service fabric 6.5是最后一个支持Visual studio 2015集成环境开发的版本。
### 安装开发环境的几个种方式
- Windows Web Platform Installer方式

    这种有界面安装方式建议在安装好Visual Studio 2019集成开发环境后进行，下载地址是：[MicrosoftAzure-ServiceFabric-CoreSDK](https://webpihandler.azurewebsites.net/web/handlers/webpi.ashx/getinstaller/MicrosoftAzure-ServiceFabric-CoreSDK.appids "MicrosoftAzure-ServiceFabric-CoreSDK.exe")
- PowerShell脚本安装方式
    脚本安装方式安装方式比以上有界面安装方式简单，首先以管理员方式打开PowerShell，然后进行以下简单几个步骤即可完成；
    + 在PowerShell中安装 [Chocolatey](https://chocolatey.org/install "Installing Chocolatey") 环境
    
        >` Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')) `
    + 安装系统以来的下载工具
        >` choco install webpicmd -y `
    + 安装Service fabric SDK
        >` webpicmd.exe /Install /AcceptEula /SuppressReboot /Products:MicrosoftAzure-ServiceFabric-CoreSDK `
### 创建生产环境方式
生产环境相对于开发环境安装可能更容易一些，但是生产环境有二种模式的集运行与管理方式，一是基于X509证书保护方式运行的群集，另外一种是UnSecure方式下运行的群集，接下来我们分步骤介绍一下基于X509证书保护下的安全群集的安装过程。

# 基于X509安全证书安装生产群集
安装前的准备工作
- Windows Server 2019,服务器数量5台(可向国内、国外云服务商购买)
- 通配符域名证书一个(向正式的安全证书提供商购买)
- 
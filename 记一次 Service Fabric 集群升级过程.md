# 记一次 Service Fabric 集群升级过程

最近 Service Fabric 团队发布了 6.5 版，看到开发集群已经落后4、5个版本，所以决定升级一下。前一次升级比较顺畅，没想到这次升级遇到了点问题。

## 通过 powershell 脚本升级

只需要执行简单几个 powershell 命令就可以执行升级：

### 连接到集群

``` powershell
###### connect to the secure cluster using certs
$ClusterName= "mysecurecluster.something.com:19000"
$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3"
Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $CertThumbprint  `
    -FindType FindByThumbprint `
    -FindValue $CertThumbprint `
    -StoreLocation CurrentUser `
    -StoreName My
```

最简单的方式只需要在 powershell 输入 `Connect-serviceFabricCluster` 然后回车，此时会连接到本地的集群。

### 获取可以升级的版本

``` powershell
###### Get the list of available Service Fabric versions
Get-ServiceFabricRegisteredClusterCodeVersion
```

执行以上命令你会看到类似如下图片的输出：
![](_v_images/20190618133500522_3949.png)

### 开始升级

```
Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

###### Here is a filled-out example

Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback
```

升级需要节点能够访问互联网，可以从微软网站上下载程序文件。如果集群的节点不能访问互联网，则需要我们自己下载程序文件。

### 监控升级进度

我们可以通过 powershell 命令和 Service Fabric Explorer 监控升级进度。

``` powershell
Get-ServiceFabricClusterUpgrade
```

升级过程其实不复杂，不过有时候也会遇到不能成功升级的情况。这次我遇到的情况是升级过程中有节点报错，最终升级过程因为节点出错而回滚。不过这个错误不是比如会发生的，比如逐个版本升级，前几个版本升级时没有出错，有个版本怎么都升级不了。

在 Service Fabric Explorer 里面看到的 Error 信息为：

> 'FabricDCA' reported Error for property 'DataCollectionAgent'.
> Exception occured while creating an object of type FabricDCA.FileShareEtwCsvUploader (assembly FileShareUploader, Version=6.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35) for creating consumer FileShareWinFabEtw.

其中关键词是 FileShare 和 Etw，根据这两个关键词我觉得可能是集群的设置有问题。

果不其然，查看以前创建集群时的 JSON 配置文件，发现有个地方设置了 FileShare，但是实际上这个地址因为共享文件夹没有配置好从而不能访问。

``` json
{
        "diagnosticsStore": 
        {
            "metadata":  "Please replace the diagnostics file share with an actual file share accessible from all cluster machines. For example, \\\\machine1\\DiagnosticsStore.",
            "dataDeletionAgeInDays": "21",
            "storeType": "FileShare",
            "connectionstring": "\\\\machine1\\DiagnosticsStore"
        },
}
```

创建共享文件夹之后再次执行升级命令，成功升级到 6.5 。其实这个模板文件已经提示了要用一个集群中机器都可以访问的文件共享地址，只是检查配置文件的脚本并不会验证这一点。而且大多数时候这个地址不可用也不会有什么影响……谁知道它在升级的路上等着我呢。

> 升级时可以跳过中间版本直接升级到最新版。

> 微软提供了一个配置，可以开启集群自动升级。

## 参考资料

- [Upgrade the Service Fabric version that runs on your cluster](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-upgrade-windows-server)
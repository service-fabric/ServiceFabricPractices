# 使用 Azure DevOps 部署 Service Fabric 程序

使用 Azure Service Fabric Pipeline 部署 Service Fabric 程序可以让事情更加容易，微软的工具对自己的框架支持还是挺给力的，Service Fabric 在 Azure DevOps 上也算是一等公民了。

## 创建 Releases

在 Pipelines 上新增加一个 Release 配置。

把 Artifacts 和 Stages 连接起来，可以设置自动触发，自动触发可以设置一些 Git Branch Filter，这样可以确保只有特定的 Branch 才会触发自动部署。在测试环境自动部署我觉得没什么问题，但是在生产环境可就要慎重了。

## 要注意的事情

如果你自己搭建的 Service Fabric 集群，你可能需要在一台可以控制的、能够访问 Service Fabric 集群的机器上安装 Service Fabric Agent 。Agent 安装很容易，按照 Azure DevOps 的提示进行就好了。如果你的集群在公网上暴露，需要使用证书等安全手段保护集群。

下面是一些准备工作：

1. 创建 PAT Tokon，用于身份验证。
2. 创建 Agent Pool，最好选一个有意义的名字，例如 ProdPool、TestPool，以免创建 Build 和 Release 的时候选错了。

PAT Tokon 和 Agent Pool 名称在配置 Agent 时需要提供。

## Service Fabric 应用程序参数管理

Service Fabric 提供了一个有用的特性，可以根据环境的不同使用不同的应用程序参数。例如 Staging、Test、Prod 分别使用不同的应用程序参数，这些参数可能是数据库连接字符串或者其他应用程序配置。

有些程序参数可以直接放在源代码里面，例如开发时使用的，仅在内网可以使用的一些配置密钥，但是有些放在源代码里面就非常危险了，例如某些可以在公网访问的密钥。

我们可以为每种环境创建一个 Release pipeline 配置，每个配置使用不同的应用程序参数文件。

我们可以把应用程序参数文件和源代码分开放置，例如把应用程序参数放置 Agent 的工作目录。


---
title: "在 Azure Service Fabric 中帮助保护服务的通信 | Microsoft Docs"
description: "概述如何帮助保护 Azure Service Fabric 群集中运行的 Reliable Services 的通信。"
services: service-fabric
documentationcenter: .net
author: suchiagicha
manager: timlt
editor: vturecek
ms.assetid: fc129c1a-fbe4-4339-83ae-0e69a41654e0
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 04/20/2017
ms.author: suchiagicha
ms.openlocfilehash: 0804e43c3f1bb13bea92ebd661ca52c799ff2332
ms.sourcegitcommit: b5c6197f997aa6858f420302d375896360dd7ceb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/21/2017
---
# <a name="help-secure-communication-for-services-in-azure-service-fabric"></a>在 Azure Service Fabric 中帮助保护服务的通信
> [!div class="op_single_selector"]
> * [Windows 上的 C#](service-fabric-reliable-services-secure-communication.md)
> * [Linux 上的 Java](service-fabric-reliable-services-secure-communication-java.md)
>
>

安全是通信最为重视的要素之一。 Reliable Services 应用程序框架提供了一些预先构建的通信堆栈和工具，可用来提高安全性。 本文介绍如何在使用服务远程处理和 Windows Communication Foundation (WCF) 通信堆栈时提高安全性。

## <a name="help-secure-a-service-when-youre-using-service-remoting"></a>使用服务远程处理时帮助保护服务
我们将使用一个现有[示例](service-fabric-reliable-services-communication-remoting.md)来解释如何为 Reliable Services 设置远程处理。 若要在使用服务远程处理时帮助保护服务，请遵循以下步骤：

1. 创建接口 `IHelloWorldStateful`，用于定义可供服务的远程过程调用使用的方法。 服务将使用 `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime` 命名空间中声明的 `FabricTransportServiceRemotingListener`。 这是可以提供远程处理功能的 `ICommunicationListener` 实现。

    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }

    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }

        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. 添加侦听器设置和安全凭据。

    确保要用来帮助保护服务通信的证书安装在群集中的所有节点上。 有两种方式可用于提供侦听器设置和安全凭据：

   1. 在服务代码中直接提供：

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportRemotingListenerSettings  listenerSettings = new FabricTransportRemotingListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }

       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           x509Credentials.RemoteCertThumbprints.Add("9FEF3950642138446CC364A396E1E881DB76B483");
           return x509Credentials;
       }
       ```
   2. 使用[配置包](service-fabric-application-and-service-manifests.md)提供：

       在 settings.xml 文件中添加 `TransportSettings` 节。

       ```xml
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```

       在这种情况下，`CreateServiceReplicaListeners` 方法如下所示：

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportRemotingListenerSettings .LoadFrom("HelloWorldStatefulTransportSettings")))
           };
       }
       ```

        如果在 settings.xml 文件中添加 `TransportSettings` 节，则 `FabricTransportRemotingListenerSettings ` 将按默认加载此节中的所有设置。

        ```xml
        <!--"TransportSettings" section .-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        在这种情况下，`CreateServiceReplicaListeners` 方法如下所示：

        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. 在安全服务上使用远程堆栈（而不是使用 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` 类）调用方法来创建服务代理时，请使用 `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`。 传入包含 `SecurityCredentials` 的 `FabricTransportRemotingSettings`。

    ```csharp

    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "9FEF3950642138446CC364A396E1E881DB76B483",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
    x509Credentials.RemoteCertThumbprints.Add("4FEF3950642138446CC364A396E1E881DB76B48C");

    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        SecurityCredentials = x509Credentials,
    };

    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    如果客户端代码正在作为服务的一部分运行，则可以从 settings.xml 文件中加载 `FabricTransportRemotingSettings`。 创建与服务代码类似的 HelloWorldClientTransportSettings 节，如上所示。 对客户端代码进行以下更改。

    ```csharp
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.LoadFrom("HelloWorldClientTransportSettings")));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    如果客户端不是作为服务一部分运行，可以在 client_name.exe 所在的同一位置中创建 client_name.settings.xml 文件。 然后在该文件中创建 TransportSettings 节。

    类似于服务，如果在客户端 settings.xml/client_name.settings.xml 中添加 `TransportSettings` 节，则 `FabricTransportRemotingSettings` 将默认加载此节中的所有设置。

    在此情况下，上述代码将进一步简化：  

    ```csharp

    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

## <a name="help-secure-a-service-when-youre-using-a-wcf-based-communication-stack"></a>使用基于 WCF 的通信堆栈时帮助保护服务
我们将使用一个现有[示例](service-fabric-reliable-services-communication-wcf.md)来解释如何为 Reliable Services 设置基于 WCF 的通信堆栈。 若要在使用基于 WCF 的通信堆栈时帮助保护服务，请遵循以下步骤：

1. 对于服务，需要帮助保护创建的 WCF 通信侦听器 (`WcfCommunicationListener`)。 为此，请修改 `CreateServiceReplicaListeners` 方法。

    ```csharp
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        return new[]
        {
            new ServiceReplicaListener(
                this.CreateWcfCommunicationListener)
        };
    }

    private WcfCommunicationListener<ICalculator> CreateWcfCommunicationListener(StatefulServiceContext context)
    {
       var wcfCommunicationListener = new WcfCommunicationListener<ICalculator>(
            serviceContext:context,
            wcfServiceObject:this,
            // For this example, we will be using NetTcpBinding.
            listenerBinding: GetNetTcpBinding(),
            endpointResourceName:"WcfServiceEndpoint");

        // Add certificate details in the ServiceHost credentials.
        wcfCommunicationListener.ServiceHost.Credentials.ServiceCertificate.SetCertificate(
            StoreLocation.LocalMachine,
            StoreName.My,
            X509FindType.FindByThumbprint,
            "9DC906B169DC4FAFFD1697AC781E806790749D2F");
        return wcfCommunicationListener;
    }

    private static NetTcpBinding GetNetTcpBinding()
    {
        NetTcpBinding b = new NetTcpBinding(SecurityMode.TransportWithMessageCredential);
        b.Security.Message.ClientCredentialType = MessageCredentialType.Certificate;
        return b;
    }
    ```
2. 在客户端中，在前面[示例](service-fabric-reliable-services-communication-wcf.md)中创建的 `WcfCommunicationClient` 类保持不变。 但是，需要重写 `WcfCommunicationClientFactory` 的 `CreateClientAsync` 方法：

    ```csharp
    public class SecureWcfCommunicationClientFactory<TServiceContract> : WcfCommunicationClientFactory<TServiceContract> where TServiceContract : class
    {
        private readonly Binding clientBinding;
        private readonly object callbackObject;
        public SecureWcfCommunicationClientFactory(
            Binding clientBinding,
            IEnumerable<IExceptionHandler> exceptionHandlers = null,
            IServicePartitionResolver servicePartitionResolver = null,
            string traceId = null,
            object callback = null)
            : base(clientBinding, exceptionHandlers, servicePartitionResolver,traceId,callback)
        {
            this.clientBinding = clientBinding;
            this.callbackObject = callback;
        }

        protected override Task<WcfCommunicationClient<TServiceContract>> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
        {
            var endpointAddress = new EndpointAddress(new Uri(endpoint));
            ChannelFactory<TServiceContract> channelFactory;
            if (this.callbackObject != null)
            {
                channelFactory = new DuplexChannelFactory<TServiceContract>(
                this.callbackObject,
                this.clientBinding,
                endpointAddress);
            }
            else
            {
                channelFactory = new ChannelFactory<TServiceContract>(this.clientBinding, endpointAddress);
            }
            // Add certificate details to the ChannelFactory credentials.
            // These credentials will be used by the clients created by
            // SecureWcfCommunicationClientFactory.  
            channelFactory.Credentials.ClientCertificate.SetCertificate(
                StoreLocation.LocalMachine,
                StoreName.My,
                X509FindType.FindByThumbprint,
                "9DC906B169DC4FAFFD1697AC781E806790749D2F");
            var channel = channelFactory.CreateChannel();
            var clientChannel = ((IClientChannel)channel);
            clientChannel.OperationTimeout = this.clientBinding.ReceiveTimeout;
            return Task.FromResult(this.CreateWcfCommunicationClient(channel));
        }
    }
    ```

    使用 `SecureWcfCommunicationClientFactory` 创建 WCF 通信客户端 (`WcfCommunicationClient`)。 使用客户端调用服务方法。

    ```csharp
    IServicePartitionResolver partitionResolver = ServicePartitionResolver.GetDefault();

    var wcfClientFactory = new SecureWcfCommunicationClientFactory<ICalculator>(clientBinding: GetNetTcpBinding(), servicePartitionResolver: partitionResolver);

    var calculatorServiceCommunicationClient =  new WcfCommunicationClient(
        wcfClientFactory,
        ServiceUri,
        ServicePartitionKey.Singleton);

    var result = calculatorServiceCommunicationClient.InvokeWithRetryAsync(
        client => client.Channel.Add(2, 3)).Result;
    ```

## <a name="next-steps"></a>后续步骤
* [Reliable Services 中使用 OWIN 的 Web API](service-fabric-reliable-services-communication-webapi.md)

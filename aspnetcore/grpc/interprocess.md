---
title: 使用 gRPC 进行进程间通信
author: jamesnk
description: 了解如何使用 gRPC 进行进程间通信。
monikerRange: '>= aspnetcore-5.0'
ms.author: jamesnk
ms.date: 08/24/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/interprocess
ms.openlocfilehash: cfe8c98bb94817035437b2feb127a07bf5cad24b
ms.sourcegitcommit: 47c9a59ff8a359baa6bca2637d3af87ddca1245b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2020
ms.locfileid: "88945455"
---
# <a name="inter-process-communication-with-grpc"></a>使用 gRPC 进行进程间通信

作者：[James Newton-King](https://twitter.com/jamesnk)

客户端和服务之间的 gRPC 调用通常通过 TCP 套接字发送。 TCP 非常适用于网络中的通信，但当客户端和服务在同一台计算机上时，[进程间通信 (IPC)](https://wikipedia.org/wiki/Inter-process_communication) 的效率更高。 本文档介绍如何在 IPC 场景中将 gRPC 用于自定义传输。

## <a name="server-configuration"></a>服务器配置

Kestrel 支持自定义传输。 在 Program.cs 上配置 Kestrel：

```csharp
public static readonly string SocketPath = Path.Combine(Path.GetTempPath(), "socket.tmp");

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(options =>
            {
                if (File.Exists(SocketPath))
                {
                    File.Delete(SocketPath);
                }
                options.ListenUnixSocket(SocketPath);
            });
        });
```

上面的示例：

* 在 `ConfigureKestrel` 中配置 Kestrel 的终结点。
* 调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 来侦听具有指定路径的 [Unix 域套接字 (UDS)](https://en.wikipedia.org/wiki/Unix_domain_socket)。

Kestrel 提供对 UDS 终结点的内置支持。 UDS 在 Linux、macOS 和 [Windows 的新式版本](https://devblogs.microsoft.com/commandline/af_unix-comes-to-windows/)上受支持。

## <a name="client-configuration"></a>客户端配置

`GrpcChannel` 支持通过自定义传输进行 gRPC 调用。 创建通道后，可以使用包含自定义 `ConnectionFactory` 的 `SocketsHttpHandler` 来配置它。 工厂允许客户端通过自定义传输建立连接，然后通过该传输发送 HTTP 请求。

> [!IMPORTANT]
> `ConnectionFactory` 是 .NET 5 候选发布 1 中的新 API。

Unix 域套接字连接工厂示例：

```csharp
public class UnixDomainSocketConnectionFactory : SocketsConnectionFactory
{
    private readonly EndPoint _endPoint;

    public UnixDomainSocketConnectionFactory(EndPoint endPoint)
        : base(AddressFamily.Unix, SocketType.Stream, ProtocolType.Unspecified)
    {
        _endPoint = endPoint;
    }

    public override ValueTask<Connection> ConnectAsync(EndPoint? endPoint,
        IConnectionProperties? options = null, CancellationToken cancellationToken = default)
    {
        return base.ConnectAsync(_endPoint, options, cancellationToken);
    }
}
```

使用自定义连接工厂创建通道：

```csharp
public static readonly string SocketPath = Path.Combine(Path.GetTempPath(), "socket.tmp");

public static GrpcChannel CreateChannel()
{
    var udsEndPoint = new UnixDomainSocketEndPoint(SocketPath);
    var socketsHttpHandler = new SocketsHttpHandler
    {
        ConnectionFactory = new UnixDomainSocketConnectionFactory(udsEndPoint)
    };

    return GrpcChannel.ForAddress("http://localhost", new GrpcChannelOptions
    {
        HttpHandler = socketsHttpHandler
    });
}
```

使用上述代码创建的通道通过 Unix 域套接字发送 gRPC 调用。
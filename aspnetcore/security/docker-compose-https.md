---
title: 使用 docker 组合通过 HTTPS 在容器中托管 ASP.NET Core 映像
author: ravipal
description: 了解如何通过 HTTPS Docker Compose 宿主 ASP.NET Core 映像
monikerRange: '>= aspnetcore-2.1'
ms.author: ravipal
ms.custom: mvc
ms.date: 03/28/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/docker-compose-https
ms.openlocfilehash: 533d86fb17e3c89fdca59685b090645a11ba5473
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/04/2020
ms.locfileid: "82775136"
---
# <a name="hosting-aspnet-core-images-with-docker-compose-over-https"></a><span data-ttu-id="c681c-103">通过 HTTPS Docker Compose 宿主 ASP.NET Core 映像</span><span class="sxs-lookup"><span data-stu-id="c681c-103">Hosting ASP.NET Core images with Docker Compose over HTTPS</span></span>


<span data-ttu-id="c681c-104">[默认情况下](/aspnet/core/security/enforcing-ssl)，ASP.NET Core 使用 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="c681c-104">ASP.NET Core uses [HTTPS by default](/aspnet/core/security/enforcing-ssl).</span></span> <span data-ttu-id="c681c-105">[HTTPS](https://en.wikipedia.org/wiki/HTTPS)依赖于信任、标识和加密的[证书](https://en.wikipedia.org/wiki/Public_key_certificate)。</span><span class="sxs-lookup"><span data-stu-id="c681c-105">[HTTPS](https://en.wikipedia.org/wiki/HTTPS) relies on [certificates](https://en.wikipedia.org/wiki/Public_key_certificate) for trust, identity, and encryption.</span></span>

<span data-ttu-id="c681c-106">本文档介绍如何通过 HTTPS 运行预生成的容器映像。</span><span class="sxs-lookup"><span data-stu-id="c681c-106">This document explains how to run pre-built container images with HTTPS.</span></span>

<span data-ttu-id="c681c-107">若要开发方案，请参阅[通过 HTTPS 上的 Docker 开发 ASP.NET Core 应用程序](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md)。</span><span class="sxs-lookup"><span data-stu-id="c681c-107">See [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md) for development scenarios.</span></span>

<span data-ttu-id="c681c-108">此示例需要 docker [17.06](https://docs.docker.com/release-notes/docker-ce)或更高版本的[docker 客户端](https://www.docker.com/products/docker)。</span><span class="sxs-lookup"><span data-stu-id="c681c-108">This sample requires [Docker 17.06](https://docs.docker.com/release-notes/docker-ce) or later of the [Docker client](https://www.docker.com/products/docker).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c681c-109">先决条件</span><span class="sxs-lookup"><span data-stu-id="c681c-109">Prerequisites</span></span>

<span data-ttu-id="c681c-110">本文档中的某些说明需要[.Net Core 2.2 SDK](https://dotnet.microsoft.com/download)或更高版本。</span><span class="sxs-lookup"><span data-stu-id="c681c-110">The [.NET Core 2.2 SDK](https://dotnet.microsoft.com/download) or later is required for some of the instructions in this document.</span></span>

## <a name="certificates"></a><span data-ttu-id="c681c-111">证书</span><span class="sxs-lookup"><span data-stu-id="c681c-111">Certificates</span></span>

<span data-ttu-id="c681c-112">针对域的[生产主机](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/)需要[证书颁发机构颁发](https://wikipedia.org/wiki/Certificate_authority)的证书。</span><span class="sxs-lookup"><span data-stu-id="c681c-112">A certificate from a [certificate authority](https://wikipedia.org/wiki/Certificate_authority) is required for [production hosting](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) for a domain.</span></span> <span data-ttu-id="c681c-113">[Let's Encrypt](https://letsencrypt.org/)是提供免费证书的证书颁发机构。</span><span class="sxs-lookup"><span data-stu-id="c681c-113">[Let's Encrypt](https://letsencrypt.org/) is a certificate authority that offers free certificates.</span></span>

<span data-ttu-id="c681c-114">本文档使用[自签名开发证书](https://wikipedia.org/wiki/Self-signed_certificate)来托管预生成的映像`localhost`。</span><span class="sxs-lookup"><span data-stu-id="c681c-114">This document uses [self-signed development certificates](https://wikipedia.org/wiki/Self-signed_certificate) for hosting pre-built images over `localhost`.</span></span> <span data-ttu-id="c681c-115">说明类似于使用生产证书。</span><span class="sxs-lookup"><span data-stu-id="c681c-115">The instructions are similar to using production certificates.</span></span>

<span data-ttu-id="c681c-116">对于生产证书：</span><span class="sxs-lookup"><span data-stu-id="c681c-116">For production certificates:</span></span>

* <span data-ttu-id="c681c-117">此`dotnet dev-certs`工具不是必需的。</span><span class="sxs-lookup"><span data-stu-id="c681c-117">The `dotnet dev-certs` tool is not required.</span></span>
* <span data-ttu-id="c681c-118">无需将证书存储在说明中使用的位置。</span><span class="sxs-lookup"><span data-stu-id="c681c-118">Certificates don't need to be stored in the location used in the instructions.</span></span> <span data-ttu-id="c681c-119">将证书存储在站点目录之外的任何位置。</span><span class="sxs-lookup"><span data-stu-id="c681c-119">Store the certificates in any location outside the site directory.</span></span>

<span data-ttu-id="c681c-120">以下部分中包含的说明使用`volumes` *docker-compose.yml. docker-compose.override.yml*中的属性将证书装载到容器中。</span><span class="sxs-lookup"><span data-stu-id="c681c-120">The instructions contained in the following section volume mount certificates into containers using the `volumes` property in *docker-compose.yml.*</span></span> <span data-ttu-id="c681c-121">可以使用`COPY` *Dockerfile*中的命令将证书添加到容器映像中，但不建议这样做。</span><span class="sxs-lookup"><span data-stu-id="c681c-121">You could add certificates into container images with a `COPY` command in a *Dockerfile*, but it's not recommended.</span></span> <span data-ttu-id="c681c-122">由于以下原因，不建议将证书复制到映像：</span><span class="sxs-lookup"><span data-stu-id="c681c-122">Copying certificates into an image isn't recommended for the following reasons:</span></span>

* <span data-ttu-id="c681c-123">这使得使用开发人员证书进行测试变得困难。</span><span class="sxs-lookup"><span data-stu-id="c681c-123">It makes it difficult to use the same image for testing with developer certificates.</span></span>
* <span data-ttu-id="c681c-124">这使得难以使用同一个映像来托管生产证书。</span><span class="sxs-lookup"><span data-stu-id="c681c-124">It makes it difficult to use the same image for Hosting with production certificates.</span></span>
* <span data-ttu-id="c681c-125">证书泄露存在重大风险。</span><span class="sxs-lookup"><span data-stu-id="c681c-125">There is significant risk of certificate disclosure.</span></span>

## <a name="starting-a-container-with-https-support-using-docker-compose"></a><span data-ttu-id="c681c-126">使用 docker 合成启动支持 https 的容器</span><span class="sxs-lookup"><span data-stu-id="c681c-126">Starting a container with https support using docker compose</span></span>

<span data-ttu-id="c681c-127">使用以下有关操作系统配置的说明。</span><span class="sxs-lookup"><span data-stu-id="c681c-127">Use the following instructions for your operating system configuration.</span></span>

### <a name="windows-using-linux-containers"></a><span data-ttu-id="c681c-128">使用 Linux 容器的 Windows</span><span class="sxs-lookup"><span data-stu-id="c681c-128">Windows using Linux containers</span></span>

<span data-ttu-id="c681c-129">生成证书并配置本地计算机：</span><span class="sxs-lookup"><span data-stu-id="c681c-129">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="c681c-130">在上述命令中，将`{ password here }`替换为密码。</span><span class="sxs-lookup"><span data-stu-id="c681c-130">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="c681c-131">创建包含以下内容的_docker-compose.yml docker-compose.override.yml_文件：</span><span class="sxs-lookup"><span data-stu-id="c681c-131">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
<span data-ttu-id="c681c-132">Docker 撰写文件中指定的密码必须与用于证书的密码匹配。</span><span class="sxs-lookup"><span data-stu-id="c681c-132">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="c681c-133">启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：</span><span class="sxs-lookup"><span data-stu-id="c681c-133">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="macos-or-linux"></a><span data-ttu-id="c681c-134">macOS 或 Linux</span><span class="sxs-lookup"><span data-stu-id="c681c-134">macOS or Linux</span></span>

<span data-ttu-id="c681c-135">生成证书并配置本地计算机：</span><span class="sxs-lookup"><span data-stu-id="c681c-135">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="c681c-136">`dotnet dev-certs https --trust`仅在 macOS 和 Windows 上受支持。</span><span class="sxs-lookup"><span data-stu-id="c681c-136">`dotnet dev-certs https --trust` is only supported on macOS and Windows.</span></span> <span data-ttu-id="c681c-137">你需要以发行版支持的方式在 Linux 上信任证书。</span><span class="sxs-lookup"><span data-stu-id="c681c-137">You need to trust certificates on Linux in the way that is supported by your distro.</span></span> <span data-ttu-id="c681c-138">可能需要在浏览器中信任该证书。</span><span class="sxs-lookup"><span data-stu-id="c681c-138">It is likely that you need to trust the certificate in your browser.</span></span>

<span data-ttu-id="c681c-139">在上述命令中，将`{ password here }`替换为密码。</span><span class="sxs-lookup"><span data-stu-id="c681c-139">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="c681c-140">创建包含以下内容的_docker-compose.yml docker-compose.override.yml_文件：</span><span class="sxs-lookup"><span data-stu-id="c681c-140">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
<span data-ttu-id="c681c-141">Docker 撰写文件中指定的密码必须与用于证书的密码匹配。</span><span class="sxs-lookup"><span data-stu-id="c681c-141">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="c681c-142">启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：</span><span class="sxs-lookup"><span data-stu-id="c681c-142">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="windows-using-windows-containers"></a><span data-ttu-id="c681c-143">使用 Windows 容器的 windows</span><span class="sxs-lookup"><span data-stu-id="c681c-143">Windows using Windows containers</span></span>

<span data-ttu-id="c681c-144">生成证书并配置本地计算机：</span><span class="sxs-lookup"><span data-stu-id="c681c-144">Generate certificate and configure local machine:</span></span>

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

<span data-ttu-id="c681c-145">在上述命令中，将`{ password here }`替换为密码。</span><span class="sxs-lookup"><span data-stu-id="c681c-145">In the preceding commands, replace `{ password here }` with a password.</span></span>

<span data-ttu-id="c681c-146">创建包含以下内容的_docker-compose.yml docker-compose.override.yml_文件：</span><span class="sxs-lookup"><span data-stu-id="c681c-146">Create a _docker-compose.debug.yml_ file with the following content:</span></span>

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=C:\https\aspnetapp.pfx
    volumes:
      - ${USERPROFILE}\.aspnet\https:C:\https:ro
```
<span data-ttu-id="c681c-147">Docker 撰写文件中指定的密码必须与用于证书的密码匹配。</span><span class="sxs-lookup"><span data-stu-id="c681c-147">The password specified in the docker compose file must match the password used for the certificate.</span></span>

<span data-ttu-id="c681c-148">启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：</span><span class="sxs-lookup"><span data-stu-id="c681c-148">Start the container with ASP.NET Core configured for HTTPS:</span></span>

```console
docker-compose -f "docker-compose.debug.yml" up -d
```
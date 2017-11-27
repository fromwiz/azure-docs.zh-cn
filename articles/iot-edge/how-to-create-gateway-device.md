---
title: "使用 Azure IoT Edge 创建网关设备 | Microsoft Docs"
description: "使用 Azure IoT Edge 创建可为多个设备处理信息的网关设备"
services: iot-edge
keywords: 
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 11/15/2017
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: e1337ddf5ed84a06a62e2faa198f3e8fb49bc3bd
ms.sourcegitcommit: 3ee36b8a4115fce8b79dd912486adb7610866a7c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2017
---
# <a name="create-an-iot-edge-gateway-device-to-process-data-from-other-iot-devices---preview"></a>创建 IoT Edge 网关设备以处理来自其他 IoT 设备的数据 - 预览

将 IoT Edge 设备用作网关的方法有两种：

* 连接使用 [Azure IoT 设备 SDK][lnk-devicesdk] 的下游设备；
* 连接不使用 IoT 中心协议的设备。

将 IoT Edge 设备用作网关时，有以下好处：

* **连接多路复用**。 通过 IoT Edge 设备连接到 IoT 中心的所有设备将使用同一个基础连接。
* **流量平滑处理**。 在本地保存消息的同时，IoT Edge 设备将在 IoT 中心限制情况下自动执行指数回退。 这将使解决方案能灵活应对流量高峰。
* **有限的脱机支持**。 网关设备将在本地存储不能传递到 IoT 中心的消息。

在本文中，我们将 IoT Edge 网关称为用作网关的 IoT Edge 设备。

>[!NOTE]
>当前：
> * 如果网关与 IoT 中心断开连接，则下游设备无法通过网关进行身份验证；并且
> * IoT Edge 设备无法连接到 IoT Edge 网关。

## <a name="use-the-azure-iot-device-sdk"></a>使用 Azure IoT 设备 SDK

安装在所有 IoT Edge 设备中的 Edge 中心将以下基元公开给下游设备：

* 设备到云和云到设备的消息；
* 直接方法；和
* 设备孪生操作。

>[!NOTE]
>目前，通过 IoT Edge 网关进行连接时，下游设备不能使用文件上传。

使用 Azure IoT 设备 SDK 将设备连接到 IoT Edge 网关时，需要：

* 使用引用网关设备主机名的连接字符串来设置下游设备；并且
* 确保下游设备信任网关设备用于接受连接的证书。

使用控件脚本安装 Azure IoT Edge 运行时的时候，将为边缘中心创建一个证书，正如“在 [Windows][lnk-tutorial1-win] 和 [Linux][lnk-tutorial1-lin] 模拟设备上安装 IoT Edge 这一教程中那样。 该证书由边缘中心用来接受传入 TLS 连接，且连接到网关设备时，该证书必须受到下游设备的信任。

可以创建任何启用设备网关拓扑所需的信任的证书基础结构。 在本文中，我们使用的是用于启用 IoT 中心 [X.509 CA 安全性][lnk-iothub-x509]的证书设置，其中包括与特定 IoT 中心关联的 X.509 CA 证书（IoT 中心所有者 CA）以及一系列安装在 IoT Edge 设备中通过该 CA 签名的证书。

>[!IMPORTANT]
>目前，IoT Edge 设备和下游设备只能使用 [SAS 令牌][lnk-iothub-tokens]来通过 IoT 中心进行身份验证。 该证书将只用于验证叶和网关设备之间的 TLS 连接。

我们的配置将“IoT 中心所有者 CA”用作：
* 所有 IoT Edge 设备上 IoT Edge 运行时的设置的签名证书，以及
* 安装在下游设备中的公钥证书。

这将得出一个解决方案，该方案使得所有设备可使用任意一个 IoT Edge 设备作为网关（只要它们连接到同一个 IoT 中心）。

### <a name="create-the-certificates-for-test-scenarios"></a>创建测试方案的证书

可以使用[管理 CA 证书示例][lnk-ca-scripts]中描述的示例 Powershell 和 Bash 脚本生成自签名 IoT 中心所有者 CA 和用其签名的设备证书。

1. 请执行[管理 CA 证书示例][lnk-ca-scripts]的步骤 1 来安装脚本。
2. 请按照步骤 2 生成 IoT 中心所有者 CA，此文件将由下游设备用于验证连接。

使用以下说明生成网关设备的证书。

#### <a name="bash"></a>Bash

* 运行 `./certGen.sh create_edge_device_certificate myGateway` 创建新的设备证书。  
  这将创建包含公钥和 PFX 的文件 .\certs\new-edge-device.* 和包含设备的私钥的 .\private\new-edge-device.key.pem。  
* `cat new-edge-device.cert.pem azure-iot-test-only.intermediate.cert.pem azure-iot-test-only.root.ca.cert.pem > new-edge-device-full-chain.cert.pem` 以获取公钥。
* `./private/new-edge-device.cert.pem` 包含设备的私钥。

#### <a name="powershell"></a>Powershell

* 运行 `New-CACertsEdgeDevice myGateway` 创建新的设备证书。
  这将创建包含公钥、私钥和该证书的 PFX 的文件 myEdgeDevice*。  在签名过程中提示输入密码时，请输入“1234”。


>[!IMPORTANT]
>此示例仅用于测试目的。 有关生产方案的信息，请参阅[保护 IoT 部署][lnk-iothub-secure-deployment]获取有关如何保护 IoT 解决方案和对证书进行相应预配的 Azure IoT 指南。

### <a name="configure-an-iot-edge-device-as-a-gateway"></a>将 IoT Edge 设备配置为网关

若要将 IoT Edge 设备配置为网关，只需配置为使用上一部分中创建的设备证书。

我们假定以下文件名来自上面的示例脚本：

| 输出 | Bash 脚本 | Powershell |
| ------ | ----------- | ---------- |
| 设备证书 | `certs/new-edge-device.cert.pem` | `certs/new-edge-device.cert.pem` |
| 设备私钥 | `private/new-edge-device.cert.pem` | `private/new-edge-device.cert.pem` |
| 设备证书链 | `certs/new-edge-device-full-chain.cert.pem` | `certs/new-edge-device-full-chain.cert.pem` |
| IoT 中心所有者 CA | `certs/azure-iot-test-only.root.ca.cert.pem` | `RootCA.pem` |

 与在 [Windows][lnk-tutorial1-win] 或 [Linux][lnk-tutorial1-lin] 模拟设备上部署 Azure IoT Edge 中描述的安装相似，必须对 IoT Edge 运行时提供以上信息。 
 
 在 Linux 中：

        sudo iotedgectl setup --connection-string {device connection string}
            --edge-hostname {gateway hostname, e.g. mygateway.contoso.com}
            --device-ca-cert-file {full path}/certs/new-edge-device.cert.pem
            --device-ca-chain-cert-file {full path}/certs/new-edge-device-full-chain.cert.pem
            --device-ca-private-key-file {full path}/private/new-edge-device.key.pem
            --owner-ca-cert-file {full path}/certs/azure-iot-test-only.root.ca.cert.pem

在 Windows 中：

        iotedgectl setup --connection-string {device connection string}
            --edge-hostname {gateway hostname, e.g. mygateway.contoso.com}
            --device-ca-cert-file {full path}/certs/new-edge-device.cert.pem
            --device-ca-chain-cert-file {full path}/certs/new-edge-device-full-chain.cert.pem
            --device-ca-private-key-file {full path}/private/new-edge-device.key.pem
            --owner-ca-cert-file {full path}/RootCA.pem

默认情况下，示例脚本不设置设备私钥的密码。 如果要设置密码，请添加以下参数：

        --device-ca-passphrase {passphrase}

此脚本将提示设置边缘代理证书的密码。
在该命令后必须重启 IoT Edge 运行时。

### <a name="configure-an-iot-hub-device-application-as-a-downstream-device"></a>将 IoT 中心设备应用程序配置为下游设备

下游设备可以是使用 [Azure IoT 设备 SDK][lnk-devicesdk] 的任何应用程序，如[使用 .NET 将设备连接到 IoT 中心][lnk-iothub-getstarted]中描述的简单设备。

首先，下游设备应用程序必须信任 IoT 中心所有者 CA 证书，以便验证与网关设备的 TLS 连接。 通常可通过两种方法执行此步骤：在 OS 级别，或（对于某些语言而言）在应用程序级别。

例如，对于 .NET 应用程序，可以添加下片段来信任以 PEM 格式存储在路径 `certPath` 中的证书。

        using System.Security.Cryptography.X509Certificates;
        
        ...

        X509Store store = new X509Store(StoreName.Root, StoreLocation.CurrentUser);
        store.Open(OpenFlags.ReadWrite);
        store.Add(new X509Certificate2(X509Certificate2.CreateFromCertFile(certPath)));
        store.Close();

请注意，上面引用的示例脚本在文件 `certs/azure-iot-test-only.root.ca.cert.pem` (Bash) 或 `RootCA.pem` (PowerShell) 中生成公钥。

在 OS 级别执行此步骤与在 Windows 之间和 Linux 分发版之间执行是不同的。

第二步是使用引用网关设备主机名的连接字符串来初始化 IoT Hub 设备 sdk。
通过将 `GatewayHostName` 属性追加到设备连接字符串来完成此步骤。 例如，此处为设备的示例设备连接字符串，我们向它追加了 `GatewayHostName` 属性：

        HostName=yourHub.azure-devices-int.net;DeviceId=yourDevice;SharedAccessKey=2BUaYca45uBS/O1AsawsuQslH4GX+SPkrytydWNdFxc=;GatewayHostName=mygateway.contoso.com

这两个步骤将使设备应用程序连接到网关设备。

## <a name="use-other-protocols"></a>使用其他协议

网关设备的主要功能之一是解释下游设备的协议。 要连接不使用 IoT 中心协议的设备，可以开发一个 IoT Edge 模块，该模块执行此转换并代表下游设备连接到 IoT 中心。

可以决定创建一个透明或不透明的网关：

| &nbsp; | 透明网关 | 不透明网关|
|--------|-------------|--------|
| 存储在 IoT 中心标识注册表中的标识 | 所有已连接的设备的标识 | 仅网关设备的标识 |
| 设备孪生 | 每个已连接的设备均有自己的设备孪生 | 仅网关具有设备和模块孪生 |
| 直接方法和云到设备的消息 | 云可以对每个已连接的设备单独寻址 | 云只能对网关设备寻址 |
| [IoT 中心限制和配额][lnk-iothub-throttles-quotas] | 适用于每个设备 | 适用于网关设备 |

使用不透明网关模式时，通过该网关连接的所有设备共享同一个可包含最多 50 条消息的云到设备的队列。 它遵循的原则是，仅当极少设备通过各字段网关进行连接以及云到设备的流量较低时，才使用不透明网关模式。

实现不透明网关时，协议转换模块将使用模块连接字符串。

实现透明网关时，模块使用下游设备的连接字符串来创建 IoT 中心设备客户端的多个实例。

## <a name="next-steps"></a>后续步骤

- [了解开发 IoT Edge 模块的要求和工具][lnk-module-dev]。

[lnk-devicesdk]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-tutorial1-win]: tutorial-simulate-device-windows.md
[lnk-tutorial1-lin]: tutorial-simulate-device-linux.md
[lnk-module-dev]: module-development.md
[lnk-iothub-getstarted]: ../iot-hub/iot-hub-csharp-csharp-getstarted.md
[lnk-iothub-x509]: ../iot-hub/iot-hub-x509ca-overview.md
[lnk-iothub-secure-deployment]: ../iot-hub/iot-hub-security-deployment.md
[lnk-iothub-tokens]: ../iot-hub/iot-hub-devguide-security.md#security-tokens 
[lnk-iothub-throttles-quotas]: ../iot-hub/iot-hub-devguide-quotas-throttling.md
[lnk-iothub-devicetwins]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-iothub-c2d]: ../iot-hub/iot-hub-devguide-messages-c2d.md
[lnk-ca-scripts]: https://github.com/Azure/azure-iot-sdk-c/blob/CACertToolEdge/tools/CACertificates/CACertificateOverview.md
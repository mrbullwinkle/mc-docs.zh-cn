---
title: 在 Linux ARM32 上安装 Azure IoT Edge | Microsoft Docs
description: 在 ARM32 设备（例如 Raspberry PI）上的 Linux 中安装 Azure IoT Edge 的说明
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
origin.date: 07/10/2019
ms.date: 07/29/2019
ms.author: v-yiso
ms.openlocfilehash: eb24c1400ec8a6e2ff9d79bf899c517c2d145eac
ms.sourcegitcommit: 5fea6210f7456215f75a9b093393390d47c3c78d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/19/2019
ms.locfileid: "68337321"
---
# <a name="install-azure-iot-edge-runtime-on-linux-arm32v7armhf"></a>在 Linux 上安装 Azure IoT Edge 运行时 (ARM32v7/armhf)

使用 Azure IoT Edge 运行时可将设备转变为 IoT Edge 设备。 该运行时可以部署在像 Raspberry Pi 一样小的设备上，也可以部署在像工业服务器一样大的设备上。 使用 IoT Edge 运行时配置设备后，即可开始从云中部署业务逻辑。 

若要了解有关 IoT Edge 运行时如何工作以及包含哪些组件的详细信息，请参阅[了解 Azure IoT Edge 运行时及其体系结构](iot-edge-runtime.md)。

本文列出了在 Linux ARM32v7/armhf IoT Edge 设备上安装 Azure IoT Edge 运行时的步骤。 例如，这些步骤适用于 Raspberry Pi 设备。 有关支持的 ARM32 操作系统的列表，请参阅 [Azure IoT Edge 支持的系统](support.md#operating-systems)。 

>[!NOTE]
>Linux 软件存储库中的包受到每个包中的许可条款限制 (/usr/share/doc/*package-name*)。 使用程序包之前请阅读许可条款。 安装和使用程序包即表示接受这些条款。 如果不同意许可条款，则不要使用程序包。

## <a name="install-the-latest-version"></a>安装最新版本

根据以下部分的说明，将最新版 Azure IoT Edge 服务安装到 Linux ARM 设备上。 

### <a name="install-the-container-runtime"></a>安装容器运行时

Azure IoT Edge 依赖于 [OCI 兼容的](https://www.opencontainers.org/)容器运行时。 对于生产方案，强烈建议使用下面提供的[基于 Moby](https://mobyproject.org/) 的引擎。 这是官方唯一支持用于 Azure IoT Edge 的容器引擎。 Docker CE/EE 容器映像与基于 Moby 的运行时兼容。

以下命令用于安装基于 Moby 的引擎和命令行接口 (CLI)。 CLI 对开发非常有用，但对生产部署来说是可选的。

```bash

# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Download and install the moby-engine
curl -L https://aka.ms/moby-engine-armhf-latest -o moby_engine.deb && sudo dpkg -i ./moby_engine.deb

# Download and install the moby-cli
curl -L https://aka.ms/moby-cli-armhf-latest -o moby_cli.deb && sudo dpkg -i ./moby_cli.deb

# Run apt-get fix
sudo apt-get install -f

```

### <a name="install-the-iot-edge-security-daemon"></a>安装 IoT Edge 安全守护程序

**IoT Edge 安全守护程序**提供和维护 IoT Edge 设备上的安全标准。 守护程序在每次开机时启动，并通过启动 IoT Edge 运行时的其余部分来启动设备。 


```bash
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Download and install the standard libiothsm implementation
curl -L https://aka.ms/libiothsm-std-linux-armhf-latest -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb

# Download and install the IoT Edge Security Daemon
curl -L https://aka.ms/iotedged-linux-armhf-latest -o iotedge.deb && sudo dpkg -i ./iotedge.deb

# Run apt-get fix
sudo apt-get install -f
```

IoT Edge 成功安装以后，输出会提示你更新配置文件。 执行[配置 Azure IoT Edge 安全守护程序](#configure-the-azure-iot-edge-security-daemon)部分的步骤，完成设备预配。 

## <a name="install-a-specific-version"></a>安装特定版本

若要安装特定版本的 Azure IoT Edge，可以直接将 IoT Edge GitHub 存储库中的组件文件作为目标。 使用前面部分列出的那些 `curl` 命令，将所有 IoT Edge 组件安装到设备上：首先是 Moby 引擎和 CLI，然后是 libiothsm，最后是 IoT Edge 安全守护程序。 唯一区别是，你将 **aka.ms** URL 替换为你的链接，这些链接直接指向要使用的每个组件的版本。

导航到 [Azure IoT Edge 版本](https://github.com/Azure/azure-iotedge/releases)，找到需要将其作为目标的发行版。 展开版本的 **Assets** 节，选择与 IoT Edge 设备的体系结构匹配的文件。 每个 IoT Edge 版本都包含 **iotedge** 和 **libiothsm** 文件。 并非所有版本都包含 **moby-engine** 或 **moby-cli**。 如果尚未安装 Moby 容器引擎，请在较旧的版本中查找包含 Moby 组件的版本。 

IoT Edge 成功安装以后，输出会提示你更新配置文件。 按照下一部分中的步骤完成设备预配。 

## <a name="configure-the-azure-iot-edge-security-daemon"></a>配置 Azure IoT Edge 安全守护程序

配置 IoT Edge 运行时以将物理设备与 Azure IoT 中心中存在的设备标识相链接。 

可以使用 `/etc/iotedge/config.yaml` 处的配置文件配置守护程序。 默认情况下，该文件有写保护，因此你可能需要提升权限才能对其进行编辑。

可以使用 IoT 中心提供的设备连接字符串手动预配单个 IoT Edge 设备。 或者，可以使用设备预配服务自动预配设备，当需要预配多个设备时这会非常有用。 根据预配选项，选择合适的安装脚本。 

### <a name="option-1-manual-provisioning"></a>选项 1：手动预配

若要手动预配设备，需要为其提供[设备连接字符串](how-to-register-device-portal.md)，可以通过在 IoT 中心注册新的 IoT Edge 设备来创建该设备连接字符串。

打开配置文件。 

```bash
sudo nano /etc/iotedge/config.yaml
```

找到文件的预配配置，并取消注释“手动预配配置”  节。 使用 IoT Edge 设备的连接字符串更新 **device_connection_string** 的值。 请确保注释掉任何其他预配部分。

```yaml
# Manual provisioning configuration
provisioning:
  source: "manual"
  device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  
# DPS TPM provisioning configuration
# provisioning:
#   source: "dps"
#   global_endpoint: "https://global.azure-devices-provisioning.net"
#   scope_id: "{scope_id}"
#   attestation:
#     method: "tpm"
#     registration_id: "{registration_id}"
```

保存并关闭该文件。 

   `CTRL + X`、`Y`、`Enter`

在配置文件中输入预配信息后，重启守护程序：

```bash
sudo systemctl restart iotedge
```

### <a name="option-2-automatic-provisioning"></a>选项 2：自动预配

若要自动预配设备，请[设置设备预配服务并检索设备注册 ID](how-to-auto-provision-simulated-device-linux.md)。 使用自动预配时，IoT Edge 支持多种证明机制，但硬件要求也会影响你的选择。 例如，默认情况下，Raspberry Pi 设备不附带受信任的平台模块 (TPM) 芯片。

打开配置文件。 

```bash
sudo nano /etc/iotedge/config.yaml
```

找到文件的预配配置，并取消注释适用于你的证明机制的部分。 例如，使用 TPM 证明时，请分别使用来自 IoT 中心设备预配服务和带有 TPM 的 IoT Edge 设备的值更新 **scope_id** 和 **registration_id** 的值。

```yaml
   # Manual provisioning configuration
   # provisioning:
   #   source: "manual"
   #   device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  
   # DPS TPM provisioning configuration
   provisioning:
     source: "dps"
     global_endpoint: "https://global.azure-devices-provisioning.net"
     scope_id: "{scope_id}"
     attestation:
       method: "tpm"
       registration_id: "{registration_id}"
```

保存并关闭该文件。 

   `CTRL + X`、`Y`、`Enter`

在配置文件中输入预配信息后，重启守护程序：

```bash
sudo systemctl restart iotedge
```


## <a name="verify-successful-installation"></a>验证是否成功安装

如果使用了上一部分中的**手动配置**步骤，则应在设备上成功预配并运行 IoT Edge 运行时。 或者，如果使用了**自动配置**步骤，则需要完成一些额外的步骤，以便运行时可以代表你向 IoT 中心注册你的设备。 有关后续步骤，请参阅[在 Linux 虚拟机上创建和预配模拟 TPM IoT Edge 设备](how-to-auto-provision-simulated-device-linux.md#give-iot-edge-access-to-the-tpm)。

使用以下命令检查 IoT Edge 守护程序的状态：

```bash
systemctl status iotedge
```

使用以下命令查看守护程序日志：

```bash
journalctl -u iotedge --no-pager --no-full
```

此外，还可使用以下命令列出正在运行的模块：

```bash
sudo iotedge list
```

## <a name="tips-and-suggestions"></a>提示和建议

需要提升的权限才能运行 `iotedge` 命令。 安装运行时后，请从计算机中注销并重新登录以自动更新权限。 在此之前，在任何 `iotedge` 命令前都要使用 **sudo**。

在资源受限的设备上，强烈建议按照[故障排除指南](troubleshoot.md#stability-issues-on-resource-constrained-devices)中的说明将 *OptimizeForPerformance* 环境变量设置为 *false*。

如果网络具有代理服务器，请按照[配置 IoT Edge 设备以通过代理服务器进行通信](how-to-configure-proxy-support.md)中的步骤进行操作。

## <a name="uninstall-iot-edge"></a>卸载 IoT Edge

如果要从 Linux 设备中删除 IoT Edge 安装，请从命令行使用以下命令。 

删除 IoT Edge 运行时。 

```bash
sudo apt-get remove --purge iotedge
```

删除 IoT Edge 运行时以后，已创建的容器会被停止，但仍存在于设备上。 查看所有容器以了解哪些容器仍然存在。 

```bash
sudo docker ps -a
```

从设备中删除容器，包括两个运行时容器。 

```bash
sudo docker rm -f <container name>
```

最后，从设备中删除容器运行时。 

```bash 
sudo apt-get remove --purge moby-cli
sudo apt-get remove --purge moby-engine
```

## <a name="next-steps"></a>后续步骤

预配了安装运行时的 IoT Edge 设备后，现在可以[部署 IoT Edge 模块](how-to-deploy-modules-portal.md)。

如果无法正确安装 IoT Edge 运行时，请参阅[故障排除](troubleshoot.md#stability-issues-on-resource-constrained-devices)页面。

若要将现有安装更新到最新版本的 IoT Edge，请参阅[更新 IoT Edge 安全守护程序和运行时](how-to-update-iot-edge.md)。

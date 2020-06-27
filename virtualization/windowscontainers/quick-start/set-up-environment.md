---
title: Windows- und Linux-Container unter Windows 10
description: Richten Sie Windows 10 oder Windows Server für Container ein, und fahren Sie dann mit dem Ausführen Ihres ersten Containerimages fort.
keywords: Docker, Container, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: quickstart
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 52cc32171ffdaf3e27476923ebdda3bc226ce4c9
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192607"
---
# <a name="get-started-prep-windows-for-containers"></a>Erste Schritte: Vorbereiten von Windows für Container

In diesem Tutorial wird Folgendes beschrieben:

- Einrichten von Windows 10 oder Windows Server für Container
- Ausführen Ihres ersten Containerimages
- Containerisieren einer einfachen .NET Core-Anwendung

## <a name="prerequisites"></a>Voraussetzungen

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Zum Ausführen von Containern unter Windows Server benötigen Sie einen physischen Server oder virtuellen Computer, auf dem Windows Server (halbjährlicher Kanal), Windows Server 2019 oder Windows Server 2016 ausgeführt wird.

Zum Testen können Sie eine Kopie von [Windows Server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) oder eine [Windows Server Insider-Vorschau](https://insider.windows.com/for-business-getting-started-server/) herunterladen.

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

Zum Ausführen von Containern unter Windows 10 benötigen Sie Folgendes:

- Ein physisches Computersystem mit Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) sollte aktiviert sein.

> [!NOTE]
>  Ab dem Windows 10-Update von Oktober 2018 ist es nicht mehr zulässig, dass Benutzer einen Windows-Container im Prozessisolationsmodus unter Windows 10 Enterprise oder Professional für Entwicklungs-/Testzwecke ausführen. Weitere Informationen finden Sie in den [FAQ](../about/faq.md).
>
> Windows Server-Container verwenden standardmäßig Hyper-V-Isolation unter Windows 10, damit Entwickler die gleiche Kernelversion und -konfiguration nutzen können, die in der Produktion verwendet wird. Weitere Informationen zur Hyper-V-Isolation finden Sie im Bereich [Konzepte](../manage-containers/hyperv-container.md) unserer Dokumentation.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installieren von Docker

Der erste Schritt ist die Installation von Docker. Docker ist für die Arbeit mit Windows-Containern erforderlich. Docker bietet eine Standardlaufzeitumgebung für Container mit einer gemeinsamen API und Befehlszeilenschnittstelle (CLI).

Weitere Informationen zur Konfiguration finden Sie unter [Docker-Engine unter Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# <a name="windows-server"></a>[Windows Server](#tab/Windows-Server)

Zum Installieren von Docker unter Windows Server können Sie ein [OneGet-Anbieter-PowerShell-Modul](https://github.com/oneget/oneget) verwenden, das von Microsoft veröffentlicht wurde und den Namen [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider) trägt. Dieser Anbieter aktiviert das Containerfeature in Windows und installiert die Docker-Engine und den -Client. Gehen Sie dazu wie folgt vor:

1. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und installieren Sie den Docker-Microsoft-PackageManagement-Anbieter aus dem [PowerShell-Katalog](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Wenn Sie zum Installieren des NuGet-Anbieters aufgefordert werden, geben Sie `Y` ein, um auch diesen zu installieren.

2. Verwenden Sie das PackageManagement-PowerShell-Modul, um die neueste Version von Docker zu installieren.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie `A` ein, um die Installation fortzusetzen.
3. Nachdem die Installation vollständig ist, starten Sie den Computer neu.

   ```powershell
   Restart-Computer -Force
   ```

Wenn Sie Docker später aktualisieren möchten:

- Überprüfen Sie die installierte Version mit `Get-Package -Name Docker -ProviderName DockerMsftProvider`.
- Ermitteln Sie die aktuelle Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`.
- Sobald Sie bereit sind, aktualisieren Sie mithilfe von `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, gefolgt von `Start-Service Docker`.

# <a name="windows-10"></a>[Windows 10](#tab/Windows-10-Client)

Mithilfe der folgenden Schritte können Sie Docker unter Windows 10 Professional und Enterprise Edition installieren.

1. Laden Sie [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, installieren Sie die Anwendung, und erstellen Sie ein kostenloses Docker- Konto, wenn Sie noch kein Konto besitzen. Weitere Informationen finden Sie in der [Docker-Dokumentation](https://docs.docker.com/docker-for-windows/install).

2. Legen Sie während der Installation den Standardcontainertyp auf Windows-Container fest. Wenn Sie den Typ nach Abschluss der Installation ändern möchten, können Sie entweder das Docker-Element in der Windows-Taskleiste (wie unten gezeigt) oder den folgenden Befehl in einer PowerShell-Eingabeaufforderung verwenden:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Docker-Taskleistenmenü mit dem Befehl „Auf Windows-Container umstellen“.](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie Ihre Umgebung ordnungsgemäß konfiguriert haben, folgen Sie dem Link, um zu erfahren, wie ein Container ausgeführt wird.

> [!div class="nextstepaction"]
> [Ausführen Ihres ersten Containers](./run-your-first-container.md)

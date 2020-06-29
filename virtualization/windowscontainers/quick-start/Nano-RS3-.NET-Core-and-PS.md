---
title: Erstellen und Ausführen einer Anwendung mit oder ohne .NET Core 2.0 oder PowerShell Core 6
description: Erstellen und Ausführen einer Anwendung mit oder ohne .NET Core 2.0 oder PowerShell Core 6.
keywords: Docker, Container
ms.topic: quickstart
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 3963be400fc4df0f64145acff2ab0d18f06ece5e
ms.sourcegitcommit: 1bafb5de322763e7f8b0e840b96774e813c39749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/22/2020
ms.locfileid: "85192497"
---
# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>Erstellen und Ausführen einer Anwendung mit oder ohne .NET Core 2.0 oder PowerShell Core 6

Beim Nano Server Container-Basisbetriebssystemimage wurden in dieser Version .NET Core und PowerShell entfernt, obwohl .NET Core und PowerShell als mehrschichtige Add-On-Container auf dem Nano Server-Basiscontainer unterstützt werden.

Wenn Ihr Container systemeigenen Code oder offene Frameworks wie z. B. Node.js, Python, Ruby usw. ausführt, reicht der Nano Server-Basiscontainer.  Ein Unterschied besteht darin, dass bestimmter systemeigener Code möglicherweise aufgrund von [Speicherbedarfseinsparungen](https://docs.microsoft.com/windows-server/get-started/nano-in-semi-annual-channel) in dieser Version im Vergleich zu Windows Server 2016 nicht ausgeführt wird. Wenn Sie Regressionsprobleme feststellen, teilen Sie uns diese in den [Foren](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers) mit.

Verwenden Sie „docker build”, um den Container von einer Dockerfile zu erstellen und „docker run“, um sie auszuführen.  Der folgende Befehl lädt das Nano Server-Container-Basisbetriebssystemimage herunter, was einige Minuten dauern kann und druckt ein "Hello World!" Nachricht an die Hostkonsole.

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

Sie können komplexere Anwendungen mit [Dockerfile-Dateien unter Windows](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile) erstellen und dabei Dockerfile-Syntax wie FROM, RUN, COPY, ADD, CMD usw. verwenden.  Obwohl Sie nicht in der Lage sind, bestimmte Befehle direkt von diesem Basisimage auszuführen, können Sie nun Containerimages erstellen, die nur die Elemente enthalten, die Sie benötigen, damit Ihre Anwendung funktioniert.

Da .NET Core und PowerShell nicht mehr im Nano Server-Container-Basisbetriebssystemimage verfügbar sind, ist es eine Herausforderung, den Container mit einem Inhalt in komprimiertem Zip-Format zu erstellen. Mit der Funktion für eine [mehrstufige Build](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), die in Docker 17.05 verfügbar ist, können Sie PowerShell in einem anderen Container zum Entzippen und Kopieren des Inhalts in einen anderen Nano-Container nutzen. Dieser Ansatz kann verwendet werden, um einen .NET Core-Container und einen PowerShell-Container zu erstellen.

Sie können das PowerShell-Containerimage mit folgendem Befehl verwenden:

```
docker pull microsoft/nanoserver-insider-powershell
```

Sie können das .NET Core-Containerimage mit folgendem Befehl verwenden:

```
docker pull microsoft/nanoserver-insider-dotnet
```

Im folgenden sind einige Beispiele, wie wir mehrstufige Builds verwendet haben, um diese Containerimages zu erstellen.

## <a name="deploy-apps-based-on-net-core-20"></a>Bereitstellen von Apps basierend auf .NET Core 2.0
Sie können das .NET Core 2.0-Containerimage in der Insider-Version nutzen, um Ihre .NET Core-Apps auszuführen, deren .NET Core-Anwendung an anderer Stelle erstellt wurde und im Container ausgeführt werden soll.  Weitere Informationen zum Ausführen einer .NET Core-Anwendung mit .NET Core-Containerimages finden Sie auf [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly).  Wenn Sie eine Anwendung innerhalb des Containers entwickeln, sollten Sie stattdessen die .NET Core SDK verwenden.  Fortgeschrittene Benutzer können ihren eigenen .NET Core 2.0-Container mit der .NET Core 2.0-Version, der Dockerfile und der URL erstellen, die unter [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0) angegeben sind. Dazu kann ein Windows Server Core-Container für die Funktionen zum Download und Entzippen verwendet werden.  Das Dockerfile-Beispiel gleicht der [.NET Core Runtime Dockerfile](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile).


Mithilfe dieser Dockerfile kann ein .NET Core 2.0-Container mit folgendem Befehl erstellt werden.

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>Führen Sie PowerShell Core 6 in einem Container aus
Mithilfe der gleichen [mehrstufigen Build](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)-Methode kann ein PowerShell Core 6-Container mit [dieser PowerShell Dockerfile](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile) erstellt werden.


Erteilen Sie anschließend den Befehl „docker build”, um das PowerShell-Container-Image zu erstellen.

```
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

Weitere Informationen finden Sie unter [PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release).  Es ist erwähnenswert, dass die PowerShell-ZIP-Datei eine Teilmenge von .NET Core 2.0 enthält, die zum Erstellen von PowerShell Core 6 erforderlich ist.  Wenn Ihre PowerShell-Module von .NET Core 2.0 abhängig sind, können Sie den PowerShell-Container auf dem Nano .NET Core-Container anstatt auf dem Nano-Basiscontainer erstellen, d. h. mithilfe von „FROM microsoft/nanoserver-insider-dotnet“ in der Dockerfile-Datei.

## <a name="next-steps"></a>Nächste Schritte
- Verwenden Sie eines der auf dem Nano Server basierenden neuen Containerimages, das in Docker Hub verfügbar ist wie beispielsweise Nano Server-Basisimage, Nano mit .NET Core 2.0 und Nano mit PowerShell Core 6
- Erstellen Sie mithilfe des Beispielinhalts der Dockerfile in diesem Handbuch Ihr eigenes Containerimage basierend auf dem neuen Nano Server-Container-Basisbetriebssystemimage

---
title: Informationen zu Windows-Containern
description: Bei Containern handelt es sich um eine Technologie zum Verpacken und Ausführen von Apps (einschließlich Windows-Apps), die in verschiedenen Umgebungen lokal und in der Cloud eingesetzt werden können. In diesem Thema wird erläutert, wie Microsoft, Windows und Azure Sie beim Entwickeln und Bereitstellen von Apps in Containern unterstützen, einschließlich der Verwendung von Docker und Azure Kubernetes Service.
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: overview
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 0759eef043ed347622f81c7b54a86c673144465d
ms.sourcegitcommit: 186ebcd006eeafb2b51a19787d59914332aad361
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2020
ms.locfileid: "87984752"
---
# <a name="windows-and-containers"></a>Windows und Container

Bei Containern handelt es sich um eine Technologie zum Verpacken und Ausführen von Windows- und Linux-Anwendungen, die in verschiedenen Umgebungen lokal und in der Cloud eingesetzt werden können. Container stellen eine schlanke, isolierte Umgebung bereit, in der Apps einfacher entwickelt, bereitgestellt und verwaltet werden können. Container werden schnell gestartet und beendet. Sie eignen sich daher ideal für Apps, die sich schnell an einen sich ändernden Bedarf anpassen müssen. Die schlanke Natur von Containern macht sie auch zu einem nützlichen Tool zum Erhöhen der Dichte und Auslastung Ihrer Infrastruktur.

![Abbildung, die zeigt, wie Container in der Cloud oder lokal ausgeführt werden können, wobei monolithische Apps oder in nahezu jeder Sprache geschriebene Microservices unterstützt werden.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Das Microsoft-Containerökosystem

Microsoft bietet eine Reihe von Tools und Plattformen, die Sie beim Entwickeln und Bereitstellen von Apps in Containern unterstützen:

- <strong>Führen Sie Windows-oder Linux-basierte Container unter Windows 10</strong> für Entwicklung und Tests mit [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) aus. Dabei werden in Windows integrierte Containerfunktionen verwendet. Sie können Container auch [nativ unter Windows Server ausführen](../quick-start/set-up-environment.md?tabs=Windows-Server).
- <strong>Entwickeln, Testen, Veröffentlichen und Bereitstellen von Windows-basierten Containern</strong> mithilfe der [leistungsfähigen Containerunterstützung in Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) und [Visual Studio Code](https://code.visualstudio.com/docs/azure/docker), die Unterstützung für Docker, Docker Compose, Kubernetes, Helm und andere nützliche Technologien umfasst.
- <strong>Veröffentlichen Sie Ihre Apps als Containerimages</strong> auf dem öffentlichen DockerHub, damit sie von anderen Benutzern verwendet werden können, oder in einer privaten [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) für eigene Entwicklung und Bereitstellung durch Ihre Organisation, indem Sie Push- und Pullvorgänge direkt in Visual Studio und Visual Studio Code durchführen.
- <strong>Stellen Sie Container im richtigen Maßstab in Azure</strong> oder anderen Clouds bereit:

  - Pullen Sie Ihre App (Containerimage) aus einer Containerregistrierung, z. B. aus Azure Container Registry, und stellen Sie sie dann im richtigen Maßstab mithilfe eines Orchestrators wie [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) oder [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) bereit, und verwalten Sie sie.
  - Azure Kubernetes Service stellt Container auf virtuellen Azure-Computern bereit und verwaltet sie im richtigen Maßstab unabhängig davon, ob es sich um Dutzende, Hunderte oder sogar Tausende von Containern handelt. Die virtuellen Azure-Computer führen entweder ein benutzerdefiniertes Windows Server-Image (wenn Sie eine Windows-basierte App bereitstellen) oder ein benutzerdefiniertes Ubuntu Linux-Image (wenn Sie eine Linux-basierte App bereitstellen) aus.
- <strong>Stellen Sie Container lokal</strong> mithilfe von [Azure Stack mit der AKS-Engine](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (als Vorschau mit Linux-Containern verfügbar) oder [Azure Stack mit OpenShift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack) bereit. Sie können Kubernetes auch selbst unter Windows Server einrichten (siehe [Kubernetes unter Windows](../kubernetes/getting-started-kubernetes-windows.md)). Außerdem arbeiten wir an Unterstützung für die Ausführung von [Windows-Containern auf der RedHat OpenShift-Containerplattform](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821).

## <a name="how-containers-work"></a>Funktionsweise von Containern

Ein Container ist ein isoliertes, schlankes Silo zum Ausführen einer Anwendung auf dem Hostbetriebssystem. Container bauen auf dem Kernel des Hostbetriebssystems auf (der als die verborgene Struktur des Betriebssystems betrachtet werden kann), wie in dieser Abbildung gezeigt.

![Architekturabbildung, die zeigt, wie Container über dem Kernel ausgeführt werden.](media/container-diagram.svg)

Auch wenn ein Container den Kernel des Hostbetriebssystems ebenfalls verwendet, erhält der Container keinen uneingeschränkten Zugriff darauf. Stattdessen erhält der Container eine isolierte (und in einigen Fällen virtualisierte) Ansicht des Systems. Ein Container kann z.B. auf eine virtualisierte Version des Dateisystems und der Registrierung zugreifen, aber alle Änderungen wirken sich nur auf den Container aus und werden verworfen, wenn er beendet wird. Zum Speichern von Daten kann der Container permanenten Speicher einbinden, z.B. einen [Azure-Datenträger](https://azure.microsoft.com/services/storage/disks/) oder eine Dateifreigabe (einschließlich [Azure Files](https://azure.microsoft.com/services/storage/files/)).

Ein Container basiert auf dem Kernel, aber der Kernel stellt nicht alle APIs und Dienste bereit, die eine App ausführen muss – die meisten dieser Komponenten werden von Systemdateien (Bibliotheken) bereitgestellt, die über dem Kernel im Benutzermodus ausgeführt werden. Da ein Container von der Benutzermodusumgebung des Hosts isoliert ist, benötigt der Container seine eigene Kopie dieser Benutzermodus-Systemdateien, die in ein als Basisimage bezeichnetes Paket gepackt werden. Das Basisimage dient als grundlegende Ebene, auf der Ihr Container erstellt wird, und stellt Betriebssystemdienste für ihn bereit, die nicht vom Kernel bereitgestellt werden. Wir werden Containerimages später ausführlicher erläutern.

## <a name="containers-vs-virtual-machines"></a>Container im Vergleich zu virtuellen Computern

Im Gegensatz zu einem Container führt ein virtueller Computer (VM) ein vollständiges Betriebssystem (einschließlich seines eigenen Kernels) aus, wie diese Abbildung zeigt.

![Architekturabbildung, die zeigt, wie VMs ein vollständiges Betriebssystem neben dem Hostbetriebssystem ausführen.](media/virtual-machine-diagram.svg)

Container und virtuelle Computer haben jeweils einen eigenen Verwendungszweck: In der Tat verwenden viele Implementierungen von Containern virtuelle Computer als Hostbetriebssystem, anstatt direkt auf der Hardware ausgeführt zu werden, insbesondere wenn Container in der Cloud ausgeführt werden.

Weitere Informationen zu den Ähnlichkeiten und Unterschieden dieser sich ergänzenden Technologien finden Sie unter [Container im Vergleich zu virtuellen Computern](containers-vs-vm.md).

## <a name="container-images"></a>Containerimages

Alle Container werden aus Containerimages erstellt. Containerimages sind eine Sammlung von Dateien, die in einem Stapel von Schichten organisiert sind. Sie befinden sich auf Ihrem lokalen Computer oder in einer Remotecontainerregistrierung. Das Containerimage besteht aus den Benutzermodus-Betriebssystemdateien, die zur Unterstützung Ihrer App, aller Laufzeiten und Abhängigkeiten Ihrer App erforderlich sind, sowie aus anderen Konfigurationsdateien, die Ihre App für die ordnungsgemäße Ausführung benötigt.

Microsoft bietet mehrere Images (als Basisimages bezeichnet), die Sie als Ausgangspunkt für das Erstellen eines eigenen Containerimages verwenden können:

* <strong>Windows</strong>: Enthält den vollständigen Satz der Windows-APIs und Systemdienste (ausgenommen Serverrollen).
* <strong>Windows Server Core</strong>: Ein kleineres Image, das eine Teilmenge der Windows Server-APIs enthält (nämlich die vollständige Version von .NET Framework). Es umfasst auch die meisten Serverrollen, aber leider nicht den Faxserver.
* <strong>Nano Server</strong>: Das kleinste Windows Server-Image mit Unterstützung für die .NET Core-APIs und einige Serverrollen.
* <strong>Windows 10 IoT Core</strong>: Eine Version von Windows, die von Hardwareherstellern für kleine IoT-Geräte verwendet wird, auf denen ARM- oder x86/x64-Prozessoren ausgeführt werden.

Wie bereits erwähnt, bestehen Containerimages aus einer Reihe von Schichten. Jede Schicht enthält einen Satz von Dateien, die das Containerimage darstellen, wenn Sie überlagert werden. Aufgrund der überlagerten Natur von Containern müssen Sie nicht immer ein Basisimage zum Erstellen eines Windows-Containers als Ziel verwenden. Stattdessen können Sie ein anderes Image als Ziel verwenden, das das gewünschte Framework bereits enthält. Das .NET-Team veröffentlicht z.B. ein [.NET Core-Image](https://hub.docker.com/_/microsoft-dotnet-core), das die .NET Core-Laufzeit enthält. Die Benutzer müssen die Installation von .NET Core daher nicht duplizieren – stattdessen können Sie die Schichten dieses Containerimages wiederverwenden. Das .NET Core-Image selbst basiert auf Nano Server.

Weitere Informationen finden Sie unter [Containerbasisimages](../manage-containers/container-base-images.md).

## <a name="container-users"></a>Containerbenutzer

### <a name="containers-for-developers"></a>Container für Entwickler

Container helfen Entwicklern, Apps mit höherer Qualität schneller zu entwickeln und auszuliefern. Mit Containern können Entwickler ein Containerimage erstellen, das innerhalb von Sekunden identisch in Umgebungen bereitgestellt werden kann. Container fungieren als einfache Methode, um Code für mehrere Teams freizugeben und eine Entwicklungsumgebung zu starten, ohne dass sich dies auf das Hostdateisystem auswirkt.

Container sind portabel und vielseitig einsetzbar, können in jeder Sprache geschriebene Apps ausführen und sind mit jedem Computer kompatibel, auf dem Windows 10 (Version 1607 oder höher) oder Windows Server 2016 oder höher ausgeführt wird. Entwickler können einen Container lokal auf ihrem Laptop oder Desktop erstellen und testen und dann das gleiche Containerimage in der privaten Cloud ihres Unternehmens, einer öffentlichen Cloud oder über einen Dienstanbieter bereitstellen. Die natürliche Agilität von Containern unterstützt moderne Muster für die Entwicklung von Apps in großen, virtualisierten Cloudumgebungen.

### <a name="containers-for-it-professionals"></a>Container für IT-Experten

Mithilfe von Containern können Administratoren eine Infrastruktur erstellen, die einfacher zu aktualisieren und zu verwalten ist und die Hardwareressourcen besser nutzt. IT-Experten können Container verwenden, um standardisierte Umgebungen für ihre Entwicklungs-, Qualitätssicherungs- und Produktionsteams bereitzustellen. Mithilfe von Containern können Systemadministratoren Unterschiede bei Betriebssysteminstallationen und der zugrunde liegenden Infrastruktur abstrahieren.

## <a name="container-orchestration"></a>Containerorchestrierung

Orchestratoren sind beim Einrichten einer containerbasierten Umgebung ein wichtiger Teil der Infrastruktur. Obwohl Sie einige Container manuell mithilfe von Docker und Windows verwalten können, nutzen Apps häufig fünf, zehn oder sogar Hunderte von Containern, für die Orchestratoren eingesetzt werden.

Containerorchestratoren wurden entwickelt, um die Verwaltung von Containern im richtigen Maßstab und in der Produktion zu erleichtern. Orchestratoren stellen Funktionen für Folgendes bereit:

- Bereitstellung im richtigen Maßstab
- Planung der Workload
- Systemüberwachung
- Failover bei einem Knotenausfall
- Zentrales Hoch- oder Herunterskalieren
- Netzwerk
- Dienstermittlung
- Koordinieren von App-Upgrades
- Clusterknotenaffinität

Es gibt viele verschiedene Orchestratoren, die Sie mit Windows-Containern verwenden können. Im Folgenden finden Sie die von Microsoft bereitgestellten Optionen:
- [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes): Verwenden Sie einen verwalteten Kubernetes-Dienst.
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/): Verwenden Sie einen verwalteten Dienst.
- [Azure Stack mit der AKS-Engine](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview): Verwenden Sie Azure Kubernetes Service lokal.
- [Kubernetes unter Windows](../kubernetes/getting-started-kubernetes-windows.md): Richten Sie Kubernetes selbst unter Windows ein.

## <a name="try-containers-on-windows"></a>Testen von Containern unter Windows

Informationen zu den ersten Schritten mit Containern unter Windows Server oder Windows 10 finden Sie in den folgenden Ressourcen:
> [!div class="nextstepaction"]
> [Erste Schritte: Konfigurieren der Umgebung für Container](../quick-start/set-up-environment.md)

Hilfe bei der Entscheidung, welche Azure-Dienste für Ihr Szenario geeignet sind, finden Sie unter [Azure Container Services](https://azure.microsoft.com/product-categories/containers/) und [Auswählen, welche Azure-Dienste zum Hosten Ihrer Anwendung verwendet werden sollten](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree).

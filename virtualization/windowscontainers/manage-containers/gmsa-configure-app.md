---
title: Konfigurieren Ihrer App für die Verwendung eines gruppenverwalteten Dienstkontos
description: Erfahren Sie, wie Sie Apps für die Verwendung von gruppenverwalteten Dienstkonten (gMSAs) für Windows-Container konfigurieren.
keywords: Docker, Container, aktives Verzeichnis, gMSA, Apps, Anwendungen, gruppenverwaltetes Dienstkonto, gruppenverwaltete Dienstkonten, Konfiguration
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909790"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Konfigurieren Ihrer App für die Verwendung eines gMSAs

In der typischen Konfiguration erhält ein Container nur ein gruppenverwaltetes Dienstkonto (gMSA), das immer dann verwendet wird, wenn das Containercomputerkonto versucht, sich bei Netzwerkressourcen zu authentifizieren. Das bedeutet, dass Ihre App als **lokales System** oder **Netzwerkdienst** ausgeführt werden muss, wenn sie die gMSA-Identität verwenden muss.

## <a name="run-an-iis-app-pool-as-network-service"></a>Ausführen eines IIS-App-Pools als Netzwerkdienst

Wenn Sie eine IIS-Website in Ihrem Container hosten, müssen Sie zur Verwendung des gMSAs lediglich Ihre App-Pool-Identität auf **Netzwerkdienst** festlegen. Sie können dies in Ihrer Dockerfile vornehmen, indem Sie den folgenden Befehl hinzufügen:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Wenn Sie zuvor statische Benutzeranmeldeinformationen für Ihren IIS-App-Pool verwendet haben, betrachten Sie das gMSA als Ersatz für diese Anmeldeinformationen. Sie können das gMSA zwischen Entwicklungs-, Test- und Produktionsumgebungen wechseln, und der IIS übernimmt automatisch die aktuelle Identität, ohne dass das Containerimage geändert werden muss.

## <a name="run-a-windows-service-as-network-service"></a>Ausführen eines Windows-Diensts als Netzwerkdienst

Wenn Ihre Container-App als Windows-Dienst ausgeführt wird, können Sie den Dienst so einstellen, dass er als **Netzwerkdienst** in Ihrer Dockerfile ausgeführt wird:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Ausführen beliebiger Konsolenanwendungen als Netzwerkdienst

Bei generischen Konsolenanwendungen, die nicht in IIS oder Service Manager gehostet werden, ist es oft am einfachsten, den Container als **Netzwerkdienst** auszuführen, sodass die Anwendung automatisch den gMSA-Kontext erbt. Dieses Feature ist ab Windows Server Version 1709 verfügbar.

Fügen Sie die folgende Zeile zu Ihrer Dockerfile hinzu, damit sie standardmäßig als Netzwerkdienst ausgeführt wird:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Sie können auch einmalig mit `docker exec` eine Verbindung mit einem Container als Netzwerkdienst herstellen. Dies ist besonders nützlich, wenn Sie Probleme mit der Konnektivität in einem aktiven Container beheben, wenn der Container normalerweise nicht als Netzwerkdienst ausgeführt wird.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Nächste Schritte

Neben der Konfiguration von Apps können Sie gMSAs auch für Folgendes verwenden:

- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrieren von Containern](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, überprüfen Sie unseren [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) auf mögliche Lösungen.

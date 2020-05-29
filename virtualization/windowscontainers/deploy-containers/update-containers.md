---
title: Aktualisieren von Windows Server-Containern
description: Hier erfahren Sie, wie Windows Container versionsübergreifend erstellen und ausführen kann.
keywords: Metadaten, Container, Version
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 84413f27bfce66e7d259c05795a280ed34b582ab
ms.sourcegitcommit: 6f216408434a437da87a72d582500a4ca6c2679c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/21/2020
ms.locfileid: "80112693"
---
# <a name="update-windows-server-containers"></a>Aktualisieren von Windows Server-Containern

Im Rahmen der monatlichen Wartung von Windows Server veröffentlichen wir regelmäßig aktualisierte Containerimages des Windows Server-Basisbetriebssystems. Mit diesen Updates können Sie die Erstellung aktualisierter Containerimages automatisieren oder sie manuell aktualisieren, indem Sie die neueste Version mithilfe von Pull übertragen. Windows Server-Container besitzen keinen Bereitstellungsstapel wie Windows Server. Sie können keine Updates innerhalb eines Containers erhalten, wie es bei Windows Server der Fall ist. Daher erstellen wir jeden Monat die Containerimages des Windows Server-Basisbetriebssystems mit den Updates neu und veröffentlichen die aktualisierten Containerimages.

Andere Containerimages, z. B. .NET oder IIS, werden auf der Grundlage der aktualisierten Containerimages des Basisbetriebssystems neu erstellt und monatlich veröffentlicht.

## <a name="how-to-get-windows-server-container-updates"></a>Abrufen von Windows Server-Containerupdates

Wir aktualisieren Containerimages des Windows Server-Basisbetriebssystems in Übereinstimmung mit dem Windows-Rhythmus. Aktualisierte Containerimages werden am zweiten Dienstag jedes Monats veröffentlicht, was gelegentlich als „B“-Veröffentlichung bezeichnet wird, mit einer Präfixnummer, die auf dem Monat der Veröffentlichung basiert. Wir nennen z. B. unser Update im Februar „2B“ und unser Update im März „3B“. Dieses monatliche Updateereignis ist das einzige reguläre Release, das neue Sicherheitspatches enthält.

Der Server, auf dem diese Container gehostet werden, der Containerhost oder auch nur „Host“ genannt wird, kann während zusätzlicher Updateereignisse gewartet werden, bei denen es sich nicht um „B“-Versionen handelt. Weitere Informationen zum Windows Update-Rhythmus finden Sie im Blogbeitrag [Windows Update-Rhythmus](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376).

Neue Containerimages des Windows Server-Basisbetriebssystems werden kurz nach 10:00 Uhr PST am zweiten Dienstag eines jeden Monats in der Microsoft Container Registry (MCR) live geschaltet, und die unterstützten Tags sind auf die neueste „B“-Version ausgerichtet. Beispiele:

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker pull mcr.microsoft.com/windows/servercore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker pull mcr.microsoft.com/windows/servercore:1909

Wenn Sie mit Docker Hub besser vertraut sind als mit MCR, finden Sie in [diesem Blogbeitrag](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/) eine ausführlichere Erklärung.  

Für jede Version wird auch das jeweilige Containerimage mit zwei zusätzlichen Tags für die Revisionsnummer und die KB-Artikelnummer veröffentlicht, um bestimmte Revisionen von Containerimages gezielt anzusprechen. Beispiel:

- docker pull mcr.microsoft.com/windows/servercore:10.0.17763.1040
- docker pull mcr.microsoft.com/windows/servercore:1809-KB4546852

Diese Beispiele übertragen beide das Windows Server 2019 Server Core-Containerimage mit dem Sicherheitsreleaseupdate vom 18. Februar mit Pull.  

Eine vollständige Liste der Containerimages des Windows Server-Basisbetriebssystems, der Versionen und der entsprechenden Tags finden Sie in diesen [Windows-Basisbetriebssystem-Containerimages](https://hub.docker.com/_/microsoft-windows-base-os-images) auf Docker Hub.

Die monatlich bereitgestellten Windows Server-Images, die von Microsoft in Azure Marketplace veröffentlicht werden, enthalten auch vorinstallierte Containerimages des Basisbetriebssystems. Weitere Informationen finden Sie in unserer [Preisübersicht: Windows Server in Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice). Normalerweise aktualisieren wir diese Images etwa fünf Arbeitstage nach der „B“-Version.

Eine vollständige Liste der Windows Server-Images und -Versionen finden Sie unter [Windows Server-Version in Azure Marketplace: Updateverlauf](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history).

## <a name="host-and-container-version-compatibility"></a>Kompatibilität von Host- und Containerversionen

Es gibt zwei Arten von Isolationsmodi für Windows-Container: Prozessisolation und Hyper-V-Isolation. Die Hyper-V-Isolation ist hinsichtlich Kompatibilität von Host- und Containerversion flexibler. Weitere Informationen finden Sie unter [Versionskompatibilität](version-compatibility.md) und [Isolationsmodi](../manage-containers/hyperv-container.md). Dieser Abschnitt konzentriert sich auf prozessisolierte Container, sofern nicht anders angegeben.

Wenn Sie Ihren Containerhost oder Ihr Containerimage mit den monatlichen Updates aktualisieren, müssen die Revisionen von Host- und Containerimage nicht übereinstimmen, damit der Container normal gestartet und ausgeführt werden kann, solange sowohl Host- als auch Containerimage unterstützt werden (Windows Server Version 1809 oder höher).

Bei der Verwendung von Windows Server-Containern mit der Sicherheitsupdateversion vom 11. Februar 2020 (auch „2B“ genannt) oder späteren monatlichen Sicherheitsupdateversionen können jedoch Probleme auftreten. Weitere Informationen finden Sie im folgenden [Microsoft-Support-Artikel](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t). Diese Probleme ergaben sich aus einer Sicherheitsänderung, die eine Änderung der Schnittstelle zwischen Benutzermodus und Kernelmodus erforderte, um die Sicherheit Ihrer Anwendungen sicherzustellen. Diese Probleme treten nur bei prozessisolierten Containern auf, da prozessisolierte Container den Kernelmodus mit dem Containerhost gemeinsam nutzen. Dies bedeutet, dass Containerimages ohne die aktualisierte Benutzermoduskomponente ungesichert und mit der neuen gesicherten Kernelschnittstelle inkompatibel waren.

Wir haben eine Korrektur zum 18. Februar 2020 veröffentlicht. In dieser neuen Version wurde eine „neue Baseline“ eingeführt. Diese neue Baseline folgt diesen Regeln:

- Jede Kombination von Hosts und Containern, die beide ältere Versionen als „2B“ aufweisen, wird funktionieren.
- Jede Kombination von Hosts und Containern, die beide neuere Versionen als „2B“ aufweisen, wird funktionieren.
- Jede Kombination von Hosts und Containern auf verschiedenen Seiten der neuen Baseline wird nicht funktionieren. Ein 3B-Host und ein 1B-Container funktionieren z. B. nicht.

Lassen Sie uns die monatliche Version des Sicherheitsupdates für März 2020 als Beispiel nehmen, um Ihnen zu zeigen, wie diese neuen Kompatibilitätsregeln funktionieren. In der folgenden Tabelle wird das Sicherheitsupdate vom März 2020 als „3B“, das Update vom Februar 2020 als „2B“ und das Update vom Januar 2020 als „1B“ bezeichnet.

| Host | Container | Kompatibilität |
|---|---|---|
| 3B | 3B | Ja |
| 3B | 2B | Ja |
| 3B | 1B oder früher | Nein |
| 2B | 3B | Ja |
| 2B | 2B | Ja |
| 2B | 1B oder früher | Nein |
| 1B oder früher | 3B | Nein |
| 1B oder früher | 2B | Nein |
| 1B oder früher | 1B oder früher | Ja |

Als Referenz sind in der folgenden Tabelle die Versionsnummern für Containerimages des Basisbetriebssystems mit monatlichen 1B-, 2B- und 3B-Sicherheitsupdateversionen für verschiedene größere Betriebssystemversionen von Windows Server 2016 bis zur neuesten Version von Windows Server, Version 1909, aufgeführt.

| Windows Server-Version (unverankertes Tag) | Updateversion vom 14.01.2020 (1B)| Updateversion vom 18.02.2020 (2B) | Updateversion vom 10.03.2020 (3B) |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (KB4534271) | 10.0.14393.3506 (KB4546850) | 10.0.14393.3568 (KB4551573) |
| Windows Server, Version 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | Diese Version hat das Ende der Unterstützung erreicht. Weitere Informationen finden Sie unter [Wartungslebenszyklen von Basisimages](base-image-lifecycle.md).|
| Windows Server, Version 1809 (1809)| 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server, Version 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server, Version 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>Problembehandlung bei Konflikten zwischen Host- und Containerimage

Bevor Sie beginnen, machen Sie sich unbedingt mit den Informationen in [Versionskompatibilität](version-compatibility.md) vertraut. Diese Informationen helfen Ihnen herauszufinden, ob Ihr Problem durch nicht übereinstimmende Patches verursacht wurde. Wenn Sie nicht übereinstimmende Patches als Ursache festgestellt haben, können Sie den Anweisungen in diesem Abschnitt folgen, um das Problem zu beheben.

### <a name="query-the-version-of-your-container-host"></a>Abfragen der Version des Containerhosts

Wenn Sie auf Ihren Containerhost zugreifen können, können Sie den Befehl `ver` ausführen, um seine Betriebssystemversion abzurufen. Wenn Sie z. B. `ver` auf einem System ausführen, auf dem Windows Server 2019 mit der neuesten Version des Sicherheitsupdates vom Februar 2020 ausgeführt wird, werden Sie Folgendes sehen:

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>Die Revisionsnummer in diesem Beispiel wird als 1039 und nicht als 1040 angezeigt, da die Sicherheitsupdateversion vom Februar 2020 keine Out-of-Band-2B-Version für Windows Server hatte. Es gab nur eine Out-of-Band-2B-Version für Container, die die Revisionsnummer 1040 aufwiesen.

Wenn Sie keinen direkten Zugriff auf den Containerhost haben, wenden Sie sich an Ihren IT-Administrator. Wenn Sie in der Cloud arbeiten, überprüfen Sie die Website des Cloudanbieters, um herauszufinden, welche Version des Containerhostbetriebssystems er ausführt. Wenn Sie z. B. Azure Kubernetes Service (AKS) verwenden, finden Sie die Version des Hostbetriebssystems in den [AKS-Versionshinweisen](https://github.com/Azure/AKS/releases).

### <a name="query-the-version-of-your-container-image"></a>Abfragen der Version des Containerimages

Folgen Sie diesen Anweisungen, um herauszufinden, welche Version Ihr Container ausführt:

1. Führen Sie das folgende Cmdlet in PowerShell aus:

    ```powershell
    docker images
    ```

    Die Ausgabe sollte etwa wie folgt aussehen:

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    In diesem Beispiel wird die Betriebssystemversion des Containers als `10.0.17763.1039` angezeigt.

    Wenn Sie bereits einen Container ausführen, können Sie auch den Befehl `ver` innerhalb des Containers selbst ausführen, um die Version abzurufen. Wenn Sie z. B. `ver` in einem Server Core-Containerimage von Windows Server 2019 mit der neuesten Version des Sicherheitsupdates vom Februar 2020 ausführen, wird Folgendes angezeigt:

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```

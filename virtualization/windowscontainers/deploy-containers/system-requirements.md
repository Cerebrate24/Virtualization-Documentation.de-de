---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910500"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

In diesem Leitfaden sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## <a name="operating-system-requirements"></a>Anforderungen an das Betriebssystem

- Das Windows-Containerfeature ist unter Windows Server (Halbjährlicher Kanal), Windows Server 2019, Windows Server 2016 und Windows 10 Professional und Enterprise-Editionen (Version 1607 und höher) verfügbar.
- Die Hyper-V-Rolle muss vor der Ausführung der Hyper-V-Isolation installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Container mit Hyper-V-Isolation bereitgestellt werden.

## <a name="virtualized-container-hosts"></a>Virtualisierte Containerhosts

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Isolation hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server (Halbjährlicher Kanal), Windows Server 2019, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (vollständige Installation oder Server Core) auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens zwei virtuelle Prozessoren.

### <a name="memory-requirements"></a>Arbeitsspeicheranforderungen

Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Wieviel Arbeitsspeicher zum Starten eines Containers und zum Ausführen grundlegender Befehle (ipconfig, dir, usw.) mindestens vorhanden sein muss, ist unten aufgeführt.

>[!NOTE]
>Diese Werte berücksichtigen weder eine gemeinsame Nutzung von Ressourcen durch Container noch die Anforderungen der Anwendung, die im Container ausgeführt wird.  Beispielsweise kann ein Host mit 512 MB freiem Speicher mehrere Server Core-Container unter Hyper-V-Isolierung ausführen, da diese Container Ressourcen gemeinsam nutzen.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB Auslagerungsdatei |
| Server Core | 50 MB                     | 325 MB + 1 GB Auslagerungsdatei |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (Halbjährlicher Kanal)

| Base image  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB Auslagerungsdatei |
| Server Core | 45 MB                     | 360 MB + 1 GB Auslagerungsdatei |

## <a name="see-also"></a>Siehe auch

[Unterstützungsrichtlinie für Windows-Container und Docker in lokalen Szenarien](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)
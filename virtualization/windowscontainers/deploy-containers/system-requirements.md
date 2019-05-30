---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d3df0631a8a61db16ad207f49163a7304c5db717
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681050"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

In diesem Leitfaden sind die Anforderungen für einen Windows-Container Host aufgelistet.

## <a name="os-requirements"></a>Betriebssystemanforderungen

- Das Windows-Container Feature steht nur unter Windows Server 2016 (Core und mit Desktop Experience), Windows 10 Professional und Enterprise (Anniversary Edition) und höher zur Verfügung.
<<<<<<< KOPF
- Die Hyper-v-Rolle muss installiert sein, bevor die Hyper-v-Isolierung ausgeführt werden kann.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V-isolierte Container bereitgestellt werden.
=======
- Die Hyper-v-Rolle muss vor dem Ausführen von Containern mit Hyper-v-Isolierung installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V Container bereitgestellt werden.
>>>>>>> Herkunft/Master

## <a name="virtualized-container-hosts"></a>Virtualisierte Container Hosts

<<<<<<< Kopf wenn ein Windows-Container Host von einem virtuellen Hyper-v-Computer ausgeführt wird und auch die Hyper-v-Isolierung hosten soll, muss die geschachtelte Virtualisierung aktiviert sein. Geschachtelte Virtualisierung hat die folgenden Voraussetzungen: = = = = Wenn ein Windows-Container Host von einem virtuellen Hyper-v-Computer ausgeführt wird und auch Container mit Hyper-v-Isolierung gehostet wird, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:
>>>>>>> Herkunft/Master

- Mindestens 4GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2019, Windows Server, Version 1803, Windows Server, Version 1709, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (Full, Core) auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
<<<<<<< KOPF
- Der Container-Host-VM benötigt auch mindestens zwei virtuelle Prozessoren.

## <a name="supported-base-images"></a>Unterstützte Basisbilder

<a name="windows-containers-are-offered-with-four-container-base-images-windows-server-core-nano-server-windows-and-iot-core-not-all-configurations-support-both-os-images-this-table-details-the-supported-configurations"></a>Windows-Container werden mit vier Container-Basisbildern angeboten: Windows Server Core, Nano Server, Windows und vieles mehr. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.
=======
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.

## <a name="supported-base-images"></a>Unterstützte Basisimages

Windows-Container werden mit vier Container-Basisbildern angeboten: Windows Server Core, Nano Server, Windows und vieles mehr. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Hostbetriebssystem</center></th>
<th><center>Windows Server-Container</center></th>
<th><center>Hyper-V-Isolierung</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016/2019 (Standard oder Rechenzentrum)</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>Windows<a href="#warn-2">**</a></center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>Nicht verfügbar</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">* Ab Windows Server steht Version 1709 Nano Server nicht mehr als Container-Host zur Verfügung.</span>

> <span id="warn-2">* * Erfordert das Windows 10 October 2018-Update, und Sie fordern die Prozessisolierung direkt an `--isolation=process` , indem Sie die Kennzeichnung `docker run`verwenden, wenn Sie Ihre Container über ausführen.</span>
>>>>>>> Herkunft/Master

|Host-Betriebssystem|Windows-Container|Hyper-V-Isolierung|
|---------------------|-----------------|-----------------|
|Windows Server 2016 oder Windows Server 2019 (Standard oder Datacenter)|Server Core, Nano Server, Windows|Server Core, Nano Server, Windows|
|Nano Server|Nano Server|Server Core, Nano Server, Windows|
|Windows 10 pro oder Windows 10 Enterprise|Nicht verfügbar|Server Core, Nano Server, Windows|
|IoT Core|IoT Core|Nicht verfügbar|

> [!WARNING]  
> Ab Windows Server, Version 1709, steht Nano Server nicht mehr als Container Host zur Verfügung.

### <a name="memory-requirements"></a>Arbeitsspeicheranforderungen

Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Nachfolgend sind die Mindestanforderungen für den Arbeitsspeicher zum Starten eines Containers und zum Ausführen von grundlegenden Befehlen (ipconfig, dir usw.) aufgeführt.

>[!NOTE]
>Bei diesen Werten wird die Ressourcenfreigabe zwischen Containern oder Anforderungen der im Container ausgeführten Anwendung nicht berücksichtigt.  Beispielsweise kann ein Host mit 512 MB freiem Speicher mehrere Server Core-Container unter Hyper-V-Isolierung ausführen, da diese Container Ressourcen gemeinsam nutzen.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Basis Bild  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB Auslagerungsdatei |
| Server Core | 50 MB                     | 325 MB + 1 GB Auslagerungsdatei |

#### <a name="windows-server-version-1709"></a>Windows Server, Version 1709

| Basis Bild  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB Auslagerungsdatei |
| Server Core | 45 MB                     | 360 MB + 1 GB Auslagerungsdatei |

### <a name="base-image-differences"></a>Grundlegende Bild Unterschiede

Wie wählt man das richtige Basis Bild aus, auf dem aufgebaut werden soll? Zwar können Sie mit dem, was Sie möchten, zusammenbauen, doch dies sind die allgemeinen Richtlinien für jedes Bild:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Wenn Ihre Anwendung das vollständige .NET Framework benötigt, ist dies das beste zu verwendende Bild.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): für Anwendungen, die nur .net Core erfordern, bietet Nano Server ein wesentlich schlankeres Bild.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): Sie können feststellen, dass Ihre Anwendung von einer Komponente oder DLL abhängt, die in Server Core-oder Nano-Server Abbildern fehlt, beispielsweise in GDI-Bibliotheken. Dieses Bild enthält den vollständigen abhängigkeitssatz von Windows.
- Viel [Core](https://hub.docker.com/_/microsoft-windows-iotcore): dieses Bild ist speziell für viele [Anwendungen](https://developer.microsoft.com/windows/iot)konzipiert. Sie sollten dieses Container Bild verwenden, wenn Sie auf einen viel Core-Host abzielen.

Für die meisten Benutzer ist Windows Server Core oder Nano Server das am besten geeignete Bild, das Sie verwenden können. Im folgenden finden Sie einige Punkte, die Sie berücksichtigen sollten, wenn Sie über den NanoServer auf dem neuesten Stand sind:

- Der Bereitstellungsstapel wurde entfernt.
- .NET Core ist nicht enthalten (Sie können jedoch das [.NET Core Nano Server-Image](https://hub.docker.com/r/microsoft/dotnet/) verwenden).
- PowerShell wurde entfernt.
- WMI wurde entfernt.
- Ab Windows Server der Version 1709 werden Anwendungen unter einem Benutzerkontext ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Sie können das Container-Administratorkonto über das--User-Flag (wie docker-Lauf-Benutzer-ContainerAdministrator) angeben, doch in Zukunft beabsichtigen wir, Administratorkonten vollständig aus dem Server zu entfernen.

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
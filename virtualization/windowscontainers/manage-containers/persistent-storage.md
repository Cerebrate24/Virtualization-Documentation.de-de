---
title: Permanente Speicherung in Containern
description: Permanente Speicherung in Windows-Containern
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: cwilhit
ms.openlocfilehash: 8bdf45a46f2e88a2206894f7d412cb93d4491cac
ms.sourcegitcommit: 57b1c0931a464ad040a7af81b749c7d66c0bc899
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/04/2020
ms.locfileid: "84421006"
---
# <a name="persistent-storage-in-containers"></a>Permanente Speicherung in Containern

<!-- Great diagram would be great! -->

Möglicherweise gibt es Fälle, in denen es wichtig ist, dass eine Anwendung in der Lage ist, Daten permanent in einem Container zu speichern, oder Sie möchten Dateien in einem Container anzeigen, die bei der Erstellung des Containers nicht enthalten waren. Permanenter Speicher kann Containern auf verschiedene Weisen zugewiesen werden:

- Binden von Bereitstellungen
- Benannte Volumes

Docker hat eine gute Übersicht zum [Verwenden von Volumes](https://docs.docker.com/engine/admin/volumes/volumes/), daher sollten Sie die Informationen zuerst lesen. Der Rest der Seite konzentriert sich auf die Unterschiede zwischen Linux und Windows und gibt Beispiele für Windows.

## <a name="bind-mounts"></a>Binden von Bereitstellungen

Durch das [Binden von Bereitstellungen](https://docs.docker.com/engine/admin/volumes/bind-mounts/) kann ein Container ein Verzeichnis mit dem Host gemeinsam nutzen. Dies ist hilfreich, wenn Sie einen Speicherort für Dateien auf dem lokalen Computer benötigen, die verfügbar sind, wenn Sie einen Container neu starten, oder diese auf mehreren Containern freigeben möchten. Wenn der Container auf mehreren Computern mit Zugriff auf dieselben Dateien ausgeführt werden soll, sollte stattdessen ein benanntes Volume oder eine SMB-Mount verwendet werden.

### <a name="permissions"></a>Berechtigungen

Das Berechtigungsmodell für das Binden von Bereitstellungen variiert je nach Isolationsstufe für den Container.

Container mit **Hyper-V-Isolation** verwenden ein einfaches schreibgeschütztes oder Lese-/ Schreibberechtigungsmodell. Der Zugriff auf Dateien erfolgt auf dem Host mithilfe des `LocalSystem`-Kontos. Wenn der Zugriff verweigert auf den Container verweigert wird, stellen Sie sicher, dass `LocalSystem` Zugriff auf das Verzeichnis auf dem Host hat. Wenn das Schreibschutzkennzeichen verwendet wird, sind Änderungen am Volume innerhalb des Containers für das Verzeichnis auf dem Host nicht sichtbar oder nicht beständig.

Windows-Container mit **Prozessisolation** sind etwas anders, da sie die Prozess-ID innerhalb des Containers verwenden, um auf Daten zuzugreifen, d. h. die Dateizugriffssteuerungslisten werden berücksichtigt. Die Identität des im Container ausgeführten Prozesses (standardmäßig "ContainerAdministrator" auf Windows Server Core und "ContainerUser" auf Nano Server-Containern) wird verwendet, um auf die Dateien und Verzeichnisse in den bereitgestellten Volumes anstelle des `LocalSystem` zuzugreifen. Es muss Ihnen der Zugriff auf die Daten gewährt werden.

Da diese Identitäten nur im Kontext des Containers vorhanden sind (und nicht auf dem Host, auf dem die Dateien gespeichert sind), sollten Sie eine allgemein bekannte Sicherheitsgruppe wie `Authenticated Users` verwenden, wenn Sie ACLs konfigurieren, um den Zugriff auf die Container zu gewährleisten.

> [!WARNING]
> Binden Sie keine Bereitstellungen für sensible Verzeichnisse wie z. B. `C:\` in einem nicht vertrauenswürdigen Container. Dadurch könnten diese Dateien auf dem Host geändert werden, auf die normalerweise kein Zugriff gewährleistet ist und es könnte daraus eine Sicherheitsverletzung resultieren.

Beispielsyntax:

- `docker run -v c:\ContainerData:c:\data:RO` für den schreibgeschützten Zugriff
- `docker run -v c:\ContainerData:c:\data:RW` für den Schreibzugriff
- `docker run -v c:\ContainerData:c:\data` für den Schreibzugriff (Standard)

### <a name="symlinks"></a>Symbolische Verknüpfungen

Symbolische Verknüpfungen werden im Container aufgelöst. Beim Binden von Bereitstellungen in einem Host-Pfad zu einem Container, der eine symbolische Verknüpfung ist oder symbolische Verknüpfungen enthält, kann dieser Container nicht darauf zugreifen.

### <a name="smb-mounts"></a>SMB-Bereitstellungen

Unter Windows Server Version 1709 und höher ermöglicht ein neues Feature namens „globale Zuordnung in SMB“ eine SMB-Freigabe auf den Host und übergibt diese Verzeichnisse anschließend der Freigabe in einen Container. Der Container muss nicht mit einem bestimmten Server, einer bestimmten Freigabe, einem Nutzernamen oder Kennwort konfiguriert werden – diese Aufgaben werden alle auf dem Host behandelt. Der Container funktioniert so, als ob es einen lokalen Speicher hätte.

#### <a name="configuration-steps"></a>Konfigurationsschritte

1. Ordnen Sie auf dem Containerhost die SMB-Remotefreigabe global zu:
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Dieser Befehl verwendet die Anmeldeinformationen zur Authentifizierung beim SMB-Remoteserver. Ordnen Sie anschließend den Pfad für die Remotefreigabe auf G: Laufwerkbuchstabe zu (dies kann jeder verfügbare Laufwerkbuchstabe sein). Die auf diesem Containerhost erstellten Container können jetzt ihre Datenvolumes auf einen Pfad auf dem Laufwerk G: zuordnen.

    > [!NOTE]
    > Wenn die globale Zuordnung in SMB für Container verwendet wird, können alle Benutzer auf dem Containerhost auf die Remotefreigabe zugreifen. Jede auf dem Containerhost ausgeführte Anwendung hat außerdem Zugriff auf die zugeordnete Remotefreigabe.

2. Erstellen Sie Container mit Datenvolumes, die global bereitgestellten SMB-Freigaben zugeordnet sind. Führen Sie den Docker aus. Nennen Sie die Demo -v g:\ContainerData:c:\AppData1 mcr.microsoft.com/windows/servercore:ltsc2019 cmd.exe

    Innerhalb des Containers wird „c:\AppData1“ dem Verzeichnis der Remotefreigabe „ContainerData“ zugeordnet. Alle Daten, die auf global zugeordneter Remotefreigabe gespeichert sind, sind für Anwendungen innerhalb des Containers verfügbar. Mehrere Container können mit dem gleichen Befehl Lese-/Schreibzugriff auf diese gemeinsam genutzten Daten erhalten.

Diese Unterstützung für die globale Zuordnung von SMB ist eine Feature für SMB-Clients, mit auf allen kompatiblen SMB-Servern funktioniert, einschließlich:

- Skalierten Dateiservern auf direkten Speicherplätzen (S2D) oder einem herkömmlichen SAN
- Azure Files (SMB-Freigabe)
- Herkömmliche Dateiserver
- Drittanbieter-Implementierung eines SMB-Protokolls (z. B.: NAS-Geräte)

> [!NOTE]
> Die globale Zuordnung in SMB unterstützt keine DFS-, DFSN-, DFSR-Freigabe unter Windows Server Version 1709.

## <a name="named-volumes"></a>Benannte Volumes

Mit benannten Volumes können Sie anhand des Namens ein Volume erstellen, dieses einem Container zuweisen und es später mit dem gleichen Namen wiederverwenden. Sie müssen nicht den tatsächlichen Pfad verfolgen, in dem es erstellt wurde, nur den Namen. Das Docker-Modul unter Windows verfügt über ein integriertes benannte Volume-Plug-In, das Volumes auf dem lokalen Computer erstellen kann. Ein zusätzliches Plug-In ist erforderlich, wenn Sie benannte Volumes auf mehreren Computern verwenden möchten.

Beispielschritte:

1. `docker volume create unwound` – Erstellen Sie ein Volume mit dem Namen „unwound“.
2. `docker run -v unwound:c:\data microsoft/windowsservercore` – Starten Sie einen Container, dessen Volume c:\data zugeordnet ist.
3. Schreiben Sie einige Dateien auf c:\data im Container, und beenden Sie anschließend den Container
4. `docker run -v unwound:c:\data microsoft/windowsservercore` – Starten Sie einen neuen Container.
5. Führen Sie `dir c:\data` im neuen Container aus – die Dateien sind auch weiterhin vorhanden

> [!NOTE]
> Windows Server konvertiert Zielpfadnamen (den Pfad innerhalb des Containers) in Kleinbuchstaben (d. h. `-v unwound:c:\MyData`, oder `-v unwound:/app/MyData` in Linux-Containern). Dies führt dazu, dass ein Verzeichnis innerhalb des Containers von `c:\mydata` (oder `/app/mydata` in Linux-Containern) abgebildet wird (und auch erstellt, falls nicht vorhanden).
